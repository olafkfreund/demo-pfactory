# Customer Feedback Service

Build a service that collects, stores, and summarises customer feedback so the
product team can see sentiment trends without trawling support tickets.

## Acceptance Criteria
- Users can submit feedback via a REST API endpoint with a rating and free text
- Feedback is persisted to a database with the submitter, timestamp, and product area
- A daily job aggregates sentiment and surfaces the top themes
- The summary is exposed via an authenticated read-only API
- All endpoints require a valid JWT and rate-limit abusive clients
