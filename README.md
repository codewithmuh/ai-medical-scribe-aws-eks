# Karute (カルテ): AI Medical Scribe on AWS EKS

[![ci](https://github.com/codewithmuh/ai-medical-scribe-aws-eks/actions/workflows/ci.yml/badge.svg)](https://github.com/codewithmuh/ai-medical-scribe-aws-eks/actions/workflows/ci.yml)
[![release](https://github.com/codewithmuh/ai-medical-scribe-aws-eks/actions/workflows/release.yml/badge.svg)](https://github.com/codewithmuh/ai-medical-scribe-aws-eks/actions/workflows/release.yml)
[![License: learning only, no commercial use](https://img.shields.io/badge/License-learning%20only%2C%20no%20commercial%20use-orange.svg)](LICENSE)

**Talk to the patient; the chart writes itself.** Karute is an AI medical scribe: it listens to a doctor's
visit, shows a live transcript, then drafts the note, the prescriptions and a summary for the patient in their
own language. It runs on Amazon EKS with the visit data locked inside your AWS network.

> **A teaching project, not a medical device.** Synthetic, role-played visits only; no real patient data. The
> AI writes drafts; a clinician must review and sign every note.

**Status: phase 0, building in public.** The repository, CI and the API's plumbing are done; the live transcript is next.
See the [roadmap](docs/roadmap.md).

## Part 2 of AI on EKS

Five production AI apps on Amazon EKS, one industry each. Every part adds one new layer of Kubernetes on top of
the parts before it.

| Part | Project | Industry | New on Kubernetes |
|---|---|---|---|
| 1 | [Heya](https://github.com/codewithmuh/ai-virtual-staging-aws-eks): AI virtual staging | Real estate | EKS basics: ALB Ingress with HTTPS, HPA, Pod Identity, then Argo CD |
| **2** | **Karute: AI medical scribe (this repo)** | **Healthcare** | **Security: NetworkPolicies, VPC endpoints, External Secrets, WebSockets** |
| 3 | [Kisekae](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks): AI virtual try-on | Fashion | GPUs: Karpenter, KEDA on SQS, Spot, scale to zero |
| 4 | [Fukikae](https://github.com/codewithmuh/ai-video-dubbing-aws-eks): AI dubbing in your own voice | Media | GPU pipelines: Argo Workflows, GPU sharing, RBAC |
| 5 | [Mimamori](https://github.com/codewithmuh/ai-cctv-video-search-aws-eks): AI CCTV video search | Security & retail | Operators (CRD), batched model serving, GPU metrics, SLOs |

## The demo

1. Press record and role-play a doctor's visit.
2. The transcript appears as you talk, split by speaker.
3. When the visit ends: a structured note (SOAP), a draft prescription list and a summary for the patient in their
   own language.
4. The clinician edits and signs. Nothing leaves draft without a signature.

Then the security demo: from inside the API pod, `curl` to the internet fails, while calls to Bedrock and
Transcribe go through private VPC endpoints. The audit log shows every read of the note.

## What it proves

| Question an interviewer asks | How Karute answers it |
|---|---|
| "How do you keep sensitive data inside your network?" | App pods have no internet egress (NetworkPolicies); Bedrock, Transcribe, S3 and Secrets Manager are reached through VPC endpoints |
| "Where do secrets live?" | AWS Secrets Manager, synced into the cluster by External Secrets; nothing in git or in images |
| "How do WebSockets survive a deploy?" | Draining on shutdown, and a client that reconnects to any pod and resumes the visit |
| "Who read this record?" | An append-only audit log of every read and edit; transcript text never appears in application logs |
| "Why not run Postgres in the cluster?" | An ADR: Amazon RDS with KMS encryption, automated backups and a timed restore |

The full design, rules and the numbers it will publish are in [docs/scope.md](docs/scope.md).

## The stack

| Layer | Choice |
|---|---|
| AI | Amazon Transcribe streaming (speech to text with speaker labels); Claude on Amazon Bedrock (the note, as validated structured output) |
| App | Next.js web, FastAPI API with WebSockets, PostgreSQL on Amazon RDS |
| Kubernetes | Helm chart, Argo CD, External Secrets, NetworkPolicies (Amazon VPC CNI), Pod Security Standards (`restricted`) |
| AWS | VPC endpoints (Bedrock, Transcribe, S3, Secrets Manager, ECR, STS, CloudWatch Logs), RDS, KMS, Secrets Manager, all in Terraform |

## Before you start

- An AWS account with Amazon Bedrock access to a Claude model, and Amazon Transcribe, in the same region.
- A domain in Route 53 (for HTTPS).
- Phase 0 runs locally but calls Transcribe and Bedrock with your own AWS credentials.
- Every phase publishes its measured cost per hour and ends with a teardown that leaves nothing running.

## Quick start

Requirements: [uv](https://docs.astral.sh/uv/), [pre-commit](https://pre-commit.com/) and Docker, or just open
the **Dev Container**.

```bash
make setup    # install dependencies and git hooks
make check    # lint, type-check and test (same as CI)
make up       # the API on :8000

curl localhost:8000/healthz          # {"status":"ok"}
curl localhost:8000/readyz           # {"status":"ready"}

make down
```

Run `make` with no arguments to list every command.

## Repository layout

```
services/api/      FastAPI service: src/api (config, logs, probes, metrics), tests/, Dockerfile. The
                   visit and note endpoints go here as they are built.
docs/              scope, roadmap, architecture, threat model, ADRs, runbooks
compose.yaml       local run with production-like hardening
Makefile           every command you need
```

The web app, Terraform and Kubernetes manifests arrive with their phases in the [roadmap](docs/roadmap.md).

## How changes flow

1. Open a pull request with a [Conventional Commit](https://www.conventionalcommits.org/) title.
2. CI must pass: lint, types, tests, secret and vulnerability scans, image build and smoke test.
3. release-please keeps a Release PR open. Merging it tags the version and publishes a signed image to GHCR with an
   SBOM.

## Production-grade checklist

Every item must pass before v1.0.

- [ ] Code quality: linters, formatters, type checks and pre-commit pass
- [ ] Tests: unit + integration, ≥ 80% coverage, a smoke test and a load test with published numbers
- [ ] CI: every PR runs lint, tests, build and scans; `main` is protected
- [ ] Security: no secrets in git, least privilege, scanned and signed images, SBOM, threat model
- [ ] Infrastructure as code: remote state, `make up` / `make down`, cost estimate, clean teardown
- [ ] Releases: semantic versions, changelog, images in a registry
- [ ] Observability: logs, metrics, traces, dashboards as code, alerts linked to runbooks
- [ ] Reliability: probes, graceful shutdown, timeouts/retries/idempotency, tested backups, written SLOs
- [ ] Docs: README with quick start, architecture, ADRs, runbooks, contributing, license
- [ ] Developer experience: one-command setup, Dev Container or Compose, seed data
- [ ] Demo: screenshots, a short demo video, a write-up

## License

**Learning only. Commercial use is not allowed.** You may clone, run, modify and deploy this project to learn, use
it to teach at a school or in free workshops, and show it in your portfolio with credit. You may not sell it, run it
as a service, use it in company or client work, or use it in paid or monetised courses and videos without written
permission. The full terms are in [LICENSE](LICENSE).

For a commercial license, or if you are unsure whether your use is allowed: **contact@codewithmuh.com**

© 2026 Muhammad Rashid ([CodeWithMuh](https://youtube.com/@codewithmuh)), Senior Platform Engineer ·
[codewithmuh.com](https://codewithmuh.com) · [GitHub](https://github.com/codewithmuh) ·
[LinkedIn](https://www.linkedin.com/in/muhammad-rashid-daha/)
