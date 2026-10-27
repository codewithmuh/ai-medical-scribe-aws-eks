# Threat model

Lightweight [STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) review.
Revisit it when you add an external integration, a data store, or a new trust boundary.

## Assets

- Visit audio, transcripts and notes (health data, synthetic in this project)
- The audit log
- Database credentials and the KMS key
- Service availability
- Build and release pipeline (images, signing identity)

## Threats and mitigations

| STRIDE | Threat | Mitigation | Status |
|---|---|---|---|
| Spoofing | Forged release images | Keyless cosign signatures tied to this repo's workflow | ✅ |
| Tampering | Compromised GitHub Action or dependency | Actions pinned to SHAs, Dependabot, Trivy scans, SBOM | ✅ |
| Repudiation | Unknown who changed what | Protected `main`, required reviews, signed release provenance | ✅ |
| Information disclosure | Secrets committed to git | gitleaks in pre-commit and CI; `.env` ignored | ✅ |
| Elevation of privilege | Container breakout | Non-root, read-only, no capabilities, minimal base image | ✅ |
| Denial of service | Request floods exhaust the service | Rate limits at the edge (AWS WAF on the ALB) and per user in the API | ⏳ planned |
| Denial of service | Abuse runs up the AI or GPU bill ("denial of wallet") | Per-user daily limits, a global daily budget, alerts at 80% | ⏳ planned |
| Tampering | Prompt injection through uploaded content | Model output is schema-validated and never triggers an action on its own | ⏳ planned |
| Information disclosure | A compromised pod sends visit data to the internet | Default-deny NetworkPolicies; app pods reach only RDS and the VPC endpoints | ⏳ planned |
| Information disclosure | Transcript or note text leaks into logs | Logs carry IDs and timings only; a test fails if visit text reaches log output | ⏳ planned |
| Repudiation | No record of who read or changed a note | Append-only audit table written in the same transaction as the change; reads logged too | ⏳ planned |
| Tampering | An AI draft is treated as a final note | Rule R1: export needs a clinician's signature; unsigned notes are marked DRAFT | ⏳ planned |
| Information disclosure | Database or bucket read without the app | KMS encryption at rest, private subnets, no public endpoints, IAM via Pod Identity | ⏳ planned |

## Open risks

List accepted risks with an owner and a review date.
