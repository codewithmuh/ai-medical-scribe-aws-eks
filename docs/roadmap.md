# Roadmap

Built in public, phase by phase. Every phase ends with measured results, not claims. See [scope.md](scope.md).

## Phase 0: the app, locally (Transcribe and Bedrock called with your own credentials)

- [x] Repository from the production template: CI, scans, signed releases, FastAPI plumbing (probes, metrics,
      JSON logs, graceful shutdown)
- [ ] ADRs: Transcribe streaming for speech, Claude on Bedrock for notes, RDS as the system of record, no internet
      egress for app pods
- [ ] Visit over WebSocket: browser audio → Transcribe streaming → live transcript with speaker labels
- [ ] Draft: SOAP note, prescriptions and patient summary as validated structured output; edit and sign (rule R1)
- [ ] Audit log, retention job, and a test that keeps visit text out of logs (rules R4 to R6)
- [ ] Web: visit screen, live transcript, note editor, sign-off
- [ ] Synthetic visit scripts for demos and tests

## Phase 1: on EKS, locked down

- [ ] Terraform: VPC with endpoints (Bedrock, Transcribe, S3, Secrets Manager, ECR, STS, CloudWatch Logs), EKS,
      RDS PostgreSQL with KMS, Secrets Manager
- [ ] Argo CD from day one; the app as a Helm chart
- [ ] External Secrets: database credentials from Secrets Manager
- [ ] NetworkPolicies: default deny; the API reaches only RDS and the VPC endpoints; a test proves internet access
      fails
- [ ] Pod Security Standards `restricted` on the app namespace
- [ ] WebSockets through the ALB: idle timeout, draining on shutdown, client reconnect that resumes the visit

## Phase 2: measured

- [ ] Speech-to-screen latency and time to a full draft, p50 and p95
- [ ] Drill: redeploy the API mid-visit; no transcript lost
- [ ] Drill: timed RDS restore from backup
- [ ] Cost per hour and per visit; clean teardown

## Phase 3: v1.0

- [ ] All 11 areas of the production-grade checklist
- [ ] Results table in the README, demo video and write-up
