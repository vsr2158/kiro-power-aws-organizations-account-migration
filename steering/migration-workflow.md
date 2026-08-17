# Migration Workflow and Go/No-Go Reporting

## Purpose

Turn read-only preflight results into an account-specific, auditable migration decision. This file governs planning and reporting; it never authorizes an AWS mutation by itself.

Load `reporting.md` and update the generated `migration-readiness-report.md` before presenting any go/no-go decision. The report template is the durable system of record; the summary below defines the minimum required contents.

## Allowed lifecycle states

`DRAFT → DISCOVERING → BLOCKED | READY_FOR_APPROVAL → INVITED → ACCEPTED_IN_TARGET_ROOT → ROOT_VALIDATED → PLACED_IN_TARGET_OU → CLOSED`

- Advance only one account at a time and only when the required evidence and approval are present.
- `BLOCKED` is a valid terminal state for the window. Do not bypass it with a standalone removal path, broad exception, or unapproved retry.
- For cohorts, pause dependent accounts when a shared condition, target landing-zone behavior, or central service validation fails.

## Readiness decision rules

### Required hard blockers

The report must explicitly evaluate these conditions:

1. Direct transfer is the approved path; no `LeaveOrganization` or `RemoveAccountFromOrganization` action is planned.
2. Account is `ACTIVE`, age eligible, and suitable for the approved window.
3. Target organization age, account/invitation quotas, feature-set/partition compatibility, and source/target profiles are verified.
4. Target root and target OU are identified; effective target policy and Control Tower enrollment effects are reviewed.
5. Target management recovery, break-glass access, target administrative access, and owner/contact coverage are tested.
6. Source/target policy, role-trust, resource/KMS, and organization-condition dependencies are resolved or accepted by the accountable security/workload owner.
7. Delegated administration, trusted access, StackSets, RAM, networking/DNS, logging/Config, security integrations, and automation callers have a transition plan.
8. Identity Center and independent access paths are ready before source assignments/access are retired.
9. Billing/CUR, RI/SP, commercial/support, cost tags, budgets, and reporting continuity decisions are complete.
10. Pilot success, manifest approval, change window, communication/on-call plan, evidence location, and named business/security/finance approvals are current.

A hard blocker cannot be `UNKNOWN`. An exception must identify the risk, owner, approver, expiration, compensating control, and revalidation condition.

## Required go/no-go report

Use this structure in every account report:

```text
Account: <ACCOUNT_ID> / <workload>
Migration state: <STATE>
Change ticket/window: <ID and timezone>
Manifest reference: <secure location and revision>

Source: <org ID, account/profile role, source OU>
Target: <org ID, account/profile role, target root and final OU>
Accepting identity: <account/profile role>

Owners: business=<...>; technical=<...>; security=<...>; finance=<...>; on-call=<...>

Hard blockers:
- <PASS | FAIL | UNKNOWN | N/A> <check>: <evidence reference>; owner=<...>; decision=<...>

Findings and exceptions:
- <ID>: impact=<...>; owner=<...>; remediation/exception=<...>; expiry=<...>; revalidation=<...>

Approved next action: <none | invite | accept | root validation | move | close>
Required approver and evidence: <...>
No-go conditions: <...>
```

Never include credentials, tokens, root recovery details, sensitive support-case content, or unredacted policy/export data in the report.

## Approval gates

### Gate A — pre-invitation

Require the signed manifest, all applicable hard blockers `PASS` or formally accepted, target/root/OU mapping, named owners, pilot evidence, active window, and explicit approval to invite exactly the approved account.

### Gate B — pre-acceptance

Require the expected handshake ID and source/target/account identity verification. The accepting identity must be in the migrating account, not the source or target management account. Require explicit approval to accept the verified handshake.

### Gate C — pre-OU move

Require target membership/root evidence, root-placement validation, critical workload smoke-test results, target logging/security/Config coverage, break-glass verification, and a fresh target-OU policy/Control Tower impact review. Require explicit approval to move to the exact final OU.

### Gate D — closure and cleanup

Require post-transfer validation and business/security/finance closure approval. Every source-side role, policy, share, StackSet, delegated-administrator, or billing cleanup action has its own scope, impact, evidence, and explicit approval.

## Communicating a blocker

When blocked, respond with a concise operational statement:

```text
Migration state: BLOCKED
Account: <ACCOUNT_ID>
Blocked transition: <invite | accept | move | close>
Reason: <specific evidence-based condition>
Impact: <what cannot safely proceed>
Owner: <person/team>
Resolution/revalidation: <exact action and check>
Next action: no AWS mutation; update evidence and reassess
```

Do not describe a failed readiness check as low risk simply because the account can technically be invited.
