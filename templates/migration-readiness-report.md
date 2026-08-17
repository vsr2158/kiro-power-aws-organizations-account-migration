# Migration Readiness Report

> Report status: `DRAFT`<br>
> Safe-content rule: record summaries and secure evidence references only. Do not include credentials, tokens, root recovery details, raw policy documents, unredacted exports, or customer data.

## Assessment metadata

| Field | Value |
|---|---|
| Assessment ID | `<YYYYMMDDTHHMMSSZ>` |
| Created / last updated (UTC) | `<timestamp>` / `<timestamp>` |
| Account ID / workload | `<account-id>` / `<workload>` |
| Change ticket / window | `<reference>` / `<window and timezone>` |
| Migration state | `DISCOVERING` |
| Manifest reference / revision | `<secure location>` |
| Organization discovery report | `<relative path or secure reference>` |
| Report directory | `reports/<account-id>/<assessment-id>/` |

## Verified identities and placement

| Scope | Expected | Verified identity / result | Evidence reference | Status |
|---|---|---|---|---|
| Source organization / profile | `<org-id / role>` | `<result>` | `<secure reference>` | `PASS / FAIL / UNKNOWN` |
| Target organization / profile | `<org-id / role>` | `<result>` | `<secure reference>` | `PASS / FAIL / UNKNOWN` |
| Migrating account / accepting role | `<account-id / role>` | `<result>` | `<secure reference>` | `PASS / FAIL / UNKNOWN` |
| Source parent | `<OU/root>` | `<result>` | `<secure reference>` | `PASS / FAIL / UNKNOWN` |
| Target root / final OU | `<root / OU>` | `<result>` | `<secure reference>` | `PASS / FAIL / UNKNOWN` |

## Owners and approvals

| Role | Name/team | Approval / contact reference | Status |
|---|---|---|---|
| Business owner | `<owner>` | `<reference>` | `PENDING / APPROVED` |
| Technical owner | `<owner>` | `<reference>` | `PENDING / APPROVED` |
| Security approver | `<owner>` | `<reference>` | `PENDING / APPROVED` |
| Finance approver | `<owner>` | `<reference>` | `PENDING / APPROVED` |
| On-call contact | `<owner>` | `<reference>` | `CONFIRMED / UNKNOWN` |

## Hard-blocker assessment

| Check | Status | Summary / impact | Owner | Evidence or exception reference | Revalidate before |
|---|---|---|---|---|---|
| Direct-transfer path; no standalone leave/removal | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Account active and age eligible | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Target age, quota, and handshake capacity | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Source/target feature set, partition, and Region compatibility | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Target root/OU policy and Control Tower impact | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Recovery, contacts, independent administration, and Identity Center | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Policy, trust, KMS, and organization-condition dependencies | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Delegated administration, trusted access, and governance services | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| RAM, StackSets, networking, observability, and integrations | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Billing, CUR/reporting, RI/SP, commercial, and support readiness | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |
| Pilot, manifest, window, communications, and approvals | `UNKNOWN` | `<summary>` | `<owner>` | `<reference>` | `<time>` |

## Findings and exceptions

| ID | Severity / status | Finding and affected dependency | Owner | Remediation or compensating control | Approval / expiry | Evidence reference |
|---|---|---|---|---|---|---|
| `<finding-id>` | `<status>` | `<summary>` | `<owner>` | `<action>` | `<reference/date>` | `<secure reference>` |

## Decision

- Go/no-go result: `PENDING / GO / NO-GO / BLOCKED`
- Permitted next action: `<none | obtain approval to invite | remediate | reassess>`
- No-go conditions: `<conditions>`
- Required approver and evidence: `<name / reference>`
- Execution journal: [`migration-execution-journal.md`](migration-execution-journal.md)
