# Statement of Work — Orders Platform: Multi-Region Microservices on EKS

**Author:** Platform Engineering · **Status:** For planning review · **Region scope:** eu-west-2 (primary), eu-west-1 (secondary)

## 1. Background & Objective

The Orders Platform is the transactional core of our e-commerce estate: it accepts
orders, reserves inventory, takes payment, and emits fulfilment events. The current
single-cluster deployment in `eu-west-2` is a single point of failure and cannot
meet the business continuity targets agreed with the risk committee. This SOW
describes the work to re-platform Orders as a set of microservices running on
**Amazon EKS across two AWS regions**, with automated **backup** and a tested
**disaster-recovery** capability.

The objective is a platform that survives the loss of an Availability Zone with no
customer impact, and the loss of an entire region within a defined recovery window,
while keeping the engineering team's day-to-day deployment workflow simple.

## 2. Goals and Non-Goals

**Goals.** Active-passive multi-region topology; automated cross-region data
replication; one-command, regularly-tested regional failover; per-service
autoscaling and resource governance; least-privilege networking; and full
observability (metrics, logs, traces) across regions.

**Non-Goals.** Active-active multi-master writes (we accept a brief RPO on
failover); on-premise or non-AWS targets; re-architecting the upstream catalogue
service (consumed as-is over its existing API).

## 3. Architecture

Each region runs a dedicated **EKS cluster** (three AZs, managed node groups) that
hosts the Orders microservices: **order-api** (synchronous REST + gRPC),
**inventory** (reservation + stock), **payments** (PSP integration), and a
**fulfilment-worker** that consumes the order event stream. Services communicate
over the cluster network; east-west traffic is mediated by the service mesh and
all ingress arrives through an ALB managed by the AWS Load Balancer Controller.

State lives in managed data services, not in the cluster. The system of record is
**Amazon RDS for PostgreSQL** (Multi-AZ in the primary region) with a
**cross-region read replica** in the secondary region that can be promoted on
failover. Hot state and the order event stream use **Amazon ElastiCache for Redis**
and **Amazon MSK (Kafka)**; the secondary region runs warm-standby replicas.
Static assets and event archives live in **S3 with Cross-Region Replication**.

DNS is fronted by **Route 53** with health-checked failover records, so promoting
the secondary region is a controlled, observable cut-over rather than a manual
scramble.

## 4. Backup & Disaster Recovery

Backups are continuous and automated: RDS automated backups plus periodic snapshots
copied cross-region, Redis snapshots to S3, and S3 CRR for objects. We target an
**RPO of 5 minutes** and an **RTO of 30 minutes** for a full regional failover.

Failover is **codified and rehearsed**, not improvised. A documented runbook (and
the automation behind it) promotes the RDS replica, scales up the secondary
region's node groups and workloads, repoints Route 53, and verifies health. The DR
procedure is exercised on a scheduled game-day at least quarterly, and the runbook
is updated from each exercise.

## 5. Security & Compliance

All ingress is least-privilege: no security group is open to `0.0.0.0/0` except the
public ALB, and even that is restricted to 80/443. Service-to-service traffic is
governed by NetworkPolicies and mesh mTLS. Secrets (database URLs, PSP keys) are
delivered from AWS Secrets Manager via the CSI driver — never baked into images or
manifests. Workloads use IRSA (IAM Roles for Service Accounts) for least-privilege
AWS access, and the payments service is isolated in its own namespace with a
tightened policy set for PCI scope.

## 6. Requirements

- order-api, inventory, payments, and fulfilment-worker each deploy as an
  independent EKS Deployment with liveness/readiness probes and CPU/memory limits
- Each service has a Horizontal Pod Autoscaler and a default-deny NetworkPolicy
  with explicit allowances
- The primary region runs RDS PostgreSQL Multi-AZ; a cross-region read replica
  exists in the secondary region and can be promoted by the failover automation
- Redis (ElastiCache) and Kafka (MSK) run in the primary region with warm standby
  in the secondary
- S3 buckets for assets and event archive use Cross-Region Replication and
  block all public access
- Route 53 health-checked failover records route traffic to the healthy region
- A documented, automated regional failover meets RPO ≤ 5 min and RTO ≤ 30 min and
  is rehearsed quarterly
- No security group permits `0.0.0.0/0` except the public ALB on ports 80/443
- All secrets are injected from AWS Secrets Manager via the CSI driver; none appear
  in images or manifests
- Workloads authenticate to AWS using IRSA; the payments service is namespace- and
  policy-isolated for PCI scope
- A CI/CD pipeline builds and scans images, runs tests, and deploys to both regions
  via Helm
- Metrics, logs, and traces are collected from both regions into a single
  observability backend with alerting on the failover health checks

## 7. Out of Scope

Active-active multi-master writes; a third region; migration of the upstream
catalogue service; and any change to the customer-facing web front-end beyond the
new regional DNS behaviour.
