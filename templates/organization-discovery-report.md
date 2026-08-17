# Organization Discovery Report

> Discovery status: `IN_PROGRESS`<br>
> Safe-content rule: record account IDs, names, state, OU/root placement, and non-secret role ARNs only. Do not include account email addresses, contacts, credentials, raw policies, raw API responses, or sensitive exports.

## Discovery metadata

| Field | Value |
|---|---|
| Discovery ID | `<YYYYMMDDTHHMMSSZ>` |
| Created / last updated (UTC) | `<timestamp>` / `<timestamp>` |
| Source profile / caller ARN | `<profile / non-secret ARN>` |
| Target profile / caller ARN | `<profile / non-secret ARN>` |
| Report directory | `reports/organization-discovery/<source-org-id>-to-<target-org-id>/<discovery-id>/` |

## Organization summary

| Organization | ID | Management account ID | Feature set | Root ID | Discovery status | Evidence reference |
|---|---|---|---|---|---|---|
| Source | `<org-id>` | `<account-id>` | `<feature-set>` | `<root-id>` | `PASS / PARTIAL / FAIL` | `<secure reference>` |
| Target | `<org-id>` | `<account-id>` | `<feature-set>` | `<root-id>` | `PASS / PARTIAL / FAIL` | `<secure reference>` |

## Source organization hierarchy

| OU path | OU ID | Parent | Direct account count | Notes |
|---|---|---|---|---|
| `<root / path>` | `<id>` | `<parent>` | `<count>` | `<notes>` |

## Source member accounts

List every source account available through the approved read-only profile. Mark the source management account and do not offer it as a normal migration candidate.

| Account ID | Account name | State | Current parent / OU path | Candidate note |
|---|---|---|---|---|
| `<account-id>` | `<name>` | `<ACTIVE / state>` | `<path>` | `<member candidate / source management account: dedicated plan>` |

## Target organization hierarchy and placement choices

| OU path | OU ID | Parent | Direct account count | Control Tower / policy note |
|---|---|---|---|---|
| `<root / path>` | `<id>` | `<parent>` | `<count>` | `<known status or UNKNOWN>` |

## Startup findings

| ID | Status | Finding | Owner / next action | Evidence reference |
|---|---|---|---|---|
| `<finding-id>` | `PASS / PARTIAL / BLOCKED` | `<summary>` | `<action>` | `<reference>` |

## Account-selection prompt

> Organization discovery is read-only and does not approve a transfer. Select exactly one **source member account ID** from the table for per-account readiness assessment. Then provide the workload name, intended target OU, and source/target/migrating-account profiles or roles. Account selection does not replace the signed execution manifest required before any write operation.

- Selected account: `PENDING`
- Selected target OU: `PENDING`
- Next permitted action: `request account selection; no AWS mutation`
