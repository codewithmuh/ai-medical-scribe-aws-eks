# Architecture

> Keep this page current: update it in the same pull request that changes the architecture.
> The target design is in [scope.md](scope.md). This page describes what is **built** today.

## Context

Karute listens to a role-played doctor's visit, shows a live transcript and drafts the clinical note for the
clinician to review and sign. It is part 2 of AI on EKS and is about security: health data that stays inside
your AWS network, secrets that never touch git, and an audit trail of every read.

## Built so far

```mermaid
flowchart LR
    client([Client]) --> api[api<br/>FastAPI :8000]
    api -. /metrics .-> prom[(Prometheus)]
```

| Component | Responsibility | Tech | Status |
|---|---|---|---|
| `api` | HTTP API, runtime contract | Python 3.13, FastAPI, uvicorn | Plumbing done; the live transcript next |

Planned components are in the [roadmap](roadmap.md).

## Runtime contract (every service)

| Concern | How |
|---|---|
| Liveness | `GET /healthz`: process is up; never checks dependencies |
| Readiness | `GET /readyz`: 503 until started and during shutdown |
| Metrics | `GET /metrics`: Prometheus `http_requests_total`, `http_request_duration_seconds` |
| Logs | One JSON line per event on stdout, with `request_id` |
| Shutdown | On SIGTERM: fail readiness, drain in-flight requests (≤ 20 s), exit 0 |
| Config | Environment variables only; secrets from the platform's secret manager |
| Security | Non-root user, read-only filesystem, no capabilities, minimal base image |

## Data

Planned: PostgreSQL on Amazon RDS (visits, transcripts, notes, the audit log), encrypted with a KMS key;
visit audio in S3 for at most 24 hours. Backups are tested with a timed restore. See [scope.md](scope.md).

## Deployment

Planned: Terraform for AWS; the app as a Helm chart synced by Argo CD from `main` from the first deploy.
Roll back by reverting the commit.

## Reliability

SLOs are written in phase 2 of the [roadmap](roadmap.md), each with an alert and the runbook it links to.

## Security

Trust boundaries and controls: see [threat-model.md](threat-model.md).

## Decisions

See [adr/](adr/).
