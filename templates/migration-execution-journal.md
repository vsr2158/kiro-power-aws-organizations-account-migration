# Migration Execution Journal

> Journal status: `NOT_STARTED`<br>
> Safe-content rule: record summarized API outcomes, request IDs, timestamps, and secure evidence references. Do not include credentials, tokens, raw policy documents, or unredacted API responses.

## Migration metadata

| Field | Value |
|---|---|
| Assessment ID | `<assessment-id>` |
| Account ID / workload | `<account-id>` / `<workload>` |
| Source / target organization | `<source-org-id>` / `<target-org-id>` |
| Target root / final OU | `<root-id>` / `<ou-id>` |
| Change ticket / approved window | `<reference>` / `<window and timezone>` |
| Readiness report | [`migration-readiness-report.md`](migration-readiness-report.md) |
| Current lifecycle state | `READY_FOR_APPROVAL` |

## Transition log

Add a `PLANNED` entry before each mutation. Add its corresponding result immediately after the API call or validation. Never move to the next transition without the required verification.

| Time (UTC) | State | Transition | Actor identity | Account / target | Approval reference | Preconditions and plan | Request / handshake reference | Result / evidence | Next permitted action |
|---|---|---|---|---|---|---|---|---|---|
| `<timestamp>` | `READY_FOR_APPROVAL` | `PLANNED: invite` | `<target role ARN>` | `<account / target-org>` | `<approval>` | `<manifest, quota, expected effect>` | `N/A` | `PENDING` | `invite after approval` |
| `<timestamp>` | `INVITED` | `invite` | `<target role ARN>` | `<account / target-org>` | `<approval>` | `<verified>` | `<request-id / handshake-id>` | `<success/failure and secure evidence>` | `verify open handshake` |
| `<timestamp>` | `INVITED` | `verify handshake` | `<read-only role ARN>` | `<account / source/target>` | `N/A` | `<expected handshake>` | `<handshake-id>` | `<match / mismatch>` | `accept only if exact match and approved` |
| `<timestamp>` | `ACCEPTED_IN_TARGET_ROOT` | `accept handshake` | `<migrating-account role ARN>` | `<account / target-org>` | `<approval>` | `<verified handshake>` | `<request-id / handshake-id>` | `<success/failure and secure evidence>` | `verify target membership/root` |
| `<timestamp>` | `ROOT_VALIDATED` | `validate target root` | `<read-only role ARN>` | `<account / target-root>` | `N/A` | `<smoke-test plan>` | `N/A` | `<result / evidence>` | `move only if passed and approved` |
| `<timestamp>` | `PLACED_IN_TARGET_OU` | `move account` | `<target role ARN>` | `<account / target-ou>` | `<approval>` | `<effective policy and CT review>` | `<request-id>` | `<success/failure and secure evidence>` | `validate final-OU controls` |
| `<timestamp>` | `<state>` | `<event>` | `<actor>` | `<scope>` | `<approval>` | `<summary>` | `<reference>` | `<result>` | `<next action>` |

## Stop conditions and incidents

| Time (UTC) | Trigger | Impact | Owner notified | Decision / remediation | Evidence reference | Resume authority |
|---|---|---|---|---|---|---|
| `<timestamp>` | `<failure/unknown>` | `<impact>` | `<owner>` | `<action>` | `<secure reference>` | `<approver>` |

## Current execution decision

- Current lifecycle state: `<state>`
- Next permitted action: `<action>`
- Blocking conditions: `<conditions or none>`
- Required approval before next mutation: `<owner / evidence reference>`
