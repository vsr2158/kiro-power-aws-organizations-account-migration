# Approved Direct-Transfer Execution

## Purpose

Execute a single approved account migration using the supported direct-transfer path. This procedure applies only after the preflight report is `READY_FOR_APPROVAL` and the relevant named approvers have explicitly approved the exact account and transition.

Open `migration-execution-journal.md` from the assessment report directory before any state-changing call. Add a `PLANNED` row after approval and before the call; add the API outcome, required verification, and next permitted action immediately after it. If any result is unexpected, write a `BLOCKED` journal entry before stopping.

## Non-negotiable rules

- Do not call `LeaveOrganization` or `RemoveAccountFromOrganization`.
- Do not execute an action using an identity whose caller account/role has not been verified in the current window.
- Do not accept a handshake without checking its account, source, target, status, and ID against the approved manifest.
- Do not move the account out of the target root until root-placement validation passes.
- Do not use an execution failure to justify broad policy changes, disabled safety controls, unmanaged cleanup, or a second unapproved account.

## State machine

`READY_FOR_APPROVAL → INVITED → ACCEPTED_IN_TARGET_ROOT → ROOT_VALIDATED → PLACED_IN_TARGET_OU → CLOSED`

On any unexpected API response, failed validation, conflicting account/organization identity, or expired window, set the state to `BLOCKED`, preserve evidence, alert the accountable owner, and stop.

## Transition 1: Invite from the target management account

### Before the call

Present and obtain explicit approval for:

- Exact account ID, target organization ID, verified target caller ARN/account, and manifest revision.
- Invitation quota/handshake capacity and the intended target root/OU placement plan.
- Expected effect: an open invitation only; the account remains in the source organization until acceptance.
- Recovery limitation: cancelling or expiring an invitation is not a substitute for remediation; do not proceed to acceptance until validation is complete.

### Execute and evidence

From the verified target management account, send an invitation only for the approved account. Record the handshake ID, target/source identities, timestamp, request ID, and response reference.

### Required verification before advancing

Use read-only handshake/account checks to confirm an open handshake matches the approved account, the target organization, and the expected source. If it does not match exactly, do not accept it.

## Transition 2: Accept from the migrating account

### Before the call

Present the verified handshake details and obtain explicit approval to accept. Confirm the caller identity is the migrating account's approved accepting role/session. Confirm the account is eligible and all acceptance prerequisites remain valid.

### Execute and evidence

Accept only the verified handshake. Record actor ARN/account, handshake ID, timestamp, request ID, and response reference.

### Required verification before advancing

Verify, from the migrating account and target side, that:

- `describe-organization` reports the target organization from the migrated account.
- The target parent relationship places the account in the target root.
- The account has not been placed under a final OU yet.

If membership or parent evidence is delayed or inconsistent, pause instead of retrying an unrelated transition.

## Transition 3: Validate target-root placement

Root validation must show that the account can safely continue before target policies are expanded by final-OU placement. Collect and review:

- Target break-glass and required user/role access, including target Identity Center assignments where applicable.
- Critical workload smoke tests, scheduled jobs, event delivery, queues, secrets, encryption, and external integration paths.
- Target CloudTrail, Config recorder/aggregator, central logging, monitoring/alarms, backup, and security-service membership/findings.
- RAM shares, network/DNS/routes/endpoints, KMS grants, StackSet/resource state, EventBridge permissions, and data/observability delivery.
- Target billing access, budgets/alarms/tags, RI/SP coverage review, target CUR delivery, and approved reporting continuity.

Report each result. A failure does not require returning the account to the source organization; it requires an owner-led remediation decision and a paused state.

## Transition 4: Move to the approved target OU

### Before the call

Re-verify:

- Exact destination OU ID and target caller identity.
- Effective SCP/RCP/declarative/tag/backup/S3/service-management policies.
- Control Tower OU registration, auto-enrollment, baseline/guardrail rollout, and service impacts.
- Root-validation evidence, named owner availability, active window, and explicit approval to move to this OU.

### Execute and evidence

Move the approved account from the target root to the approved target OU. Record request/response metadata, previous parent, new parent, and the policy/Control Tower review reference.

### Required verification before closure

Verify target parent, effective policies, intended Control Tower behavior, workload API paths, and all critical logging/security/access/integration checks under final-OU controls.

## Source management-account exception

Do not use this procedure for a source management account as a normal cohort member. First migrate or dispose of member accounts, resolve suspended/pending-closure account states, establish that the old organization is eligible for deletion, and execute a dedicated reviewed plan before inviting the former management account.
