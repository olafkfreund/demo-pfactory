# Tic-Tac-Toe — Cloud Service on EKS (Scoring + Leaderboard)

Re-platform the Tic-Tac-Toe game as a production cloud service running on
**Amazon EKS**, split into microservices, with **Redis** for fast game state and
a **PostgreSQL** database for durable scores. Add player scoring and a global
**Top-10 scoreboard**. This is the multiplayer, persistent evolution of the MVP.

## Architecture
- A **game-api** microservice (REST + WebSocket) handling match lifecycle and moves
- A **scoreboard** microservice that aggregates results and serves the leaderboard
- **Redis** for live game state and a real-time leaderboard sorted set
- **PostgreSQL** (RDS) for durable match results and player profiles
- A **web** front-end (the MVP UI extended for online play and the scoreboard)
- Deployed to **Amazon EKS** with Helm, behind an ingress/ALB, in `eu-west-2`

## Acceptance Criteria
- Players can create or join an online match identified by a shareable game id
- Moves are validated server-side; illegal or out-of-turn moves are rejected
- Live game state is stored in Redis and synced to both players in real time over WebSocket
- A completed match (win/draw) is written to PostgreSQL with players, result, and timestamp
- Each win awards points; a player's running score is updated atomically
- A `GET /leaderboard` endpoint returns the **Top 10** players by score, served from a Redis sorted set
- The game-api and scoreboard run as separate Deployments on EKS with health checks and resource limits
- Redis and PostgreSQL connection secrets are injected from Kubernetes Secrets (no plaintext)
- Each service has horizontal pod autoscaling and a NetworkPolicy restricting traffic
- A CI/CD pipeline builds container images, runs tests, and deploys to EKS via Helm
- The endpoints require authentication and are rate-limited

## Out of Scope
- Native mobile apps; matchmaking/ranking beyond a simple Top-10 by score
