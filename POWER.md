---
name: "aws-organizations-account-migration"
displayName: "AWS Organizations Account-Migration"
description: "Guides and safeguards direct AWS Organizations account migrations through read-only discovery, account-specific readiness evidence, human approval gates, and post-transfer validation."
keywords: ["aws organizations", "account migration", "direct transfer", "readiness", "governance", "control tower", "iam identity center", "ram", "cloudtrail", "config", "billing", "cur", "multi-account"]
author: "AWS Organizations Account-Migration Power"
repository: "https://github.com/vsr2158/kiro-power-aws-organizations-account-migration"
---

# AWS Organizations Account-Migration

## Overview

This power plans and safeguards migration of an AWS member account between AWS Organizations. It is designed for the direct-transfer path:

`preflight → invite from target → verify handshake → accept from migrating account → verify target root → validate → move to target OU → validate → close`

It treats the migration as a production change. Discovery is read-only. No invitation, handshake acceptance, account move, policy change, role change, deregistration, or cleanup is performed without explicit user confirmation at the applicable transition.

**This power never uses `LeaveOrganization` or `RemoveAccountFromOrganization` for a direct transfer.** A source management account is excluded from general migration cohorts and requires its own reviewed sequence.

## Onboarding

### Startup inputs

At Power startup, request only what is required for read-only organization discovery:

1. Source read-only profile or role.
2. Target read-only profile or role.
3. An approved secure report location only if the default Git-ignored `reports/` directory is unsuitable.

Do **not** ask the operator which account to migrate before presenting the source/target organization report.

### Required inputs after account selection

After the operator selects an account from the organization report, obtain and display:

1. Selected account ID and workload name.
2. Source and target Organization IDs, source profile, target profile, and migrating-account accepting profile or role.
3. Target root and intended target OU ID, plus the initial and final placement.
4. Named business, technical, security, finance, and on-call owners.
5. Change ticket, approved window, pilot/cohort, rollback/contact path, and explicit direct-transfer decision.
6. The operator's execution mode: **read-only readiness assessment** or **approved execution**.

Never infer an execution cohort from `list-accounts`. Account selection authorizes only per-account read-only assessment; a signed, explicit per-account manifest remains mandatory before any write operation.

### Credential and profile safety

- Prefer separate least-privilege source, target, and migrating-account profiles. Do not use a production administrator profile for read-only collection when a read-only profile is available.
- Before using each profile, run `sts get-caller-identity` and report account ID and role ARN. Confirm it matches the intended source, target, or migrating account.
- Do not read, print, store, or request root credentials, MFA seed material, session tokens, support-case contents, or unredacted policy exports.
- Store evidence outside source control. The included `.gitignore` excludes `evidence/`, reports, and common export formats.
- On startup, load `steering/organization-discovery.md` and create an organization-discovery report from `templates/organization-discovery-report.md`. After a valid account selection, load `steering/reporting.md` and create the three per-account report files from `templates/`: `reports/<account-id>/<assessment-id>/migration-readiness-report.md`, `migration-execution-journal.md`, and `migration-closure-report.md`. Keep only safe summaries and secure evidence references in all reports.

## Workflow

### Phase 0: Automatic organization discovery

When the Power starts, load `steering/organization-discovery.md`. After source and target read-only profiles are available, automatically:

1. Verify both caller identities with `sts get-caller-identity`.
2. Discover source and target organization details, roots, recursively enumerated OUs, and account-to-parent relationships with read-only APIs.
3. Create `reports/organization-discovery/<source-org-id>-to-<target-org-id>/<discovery-id>/organization-discovery-report.md` from the template.
4. Present the organization report: source/target IDs and management accounts, source account table with ID/name/state/current OU, and target OU table with IDs and safe policy/Control Tower notes.
5. Ask which **source member account ID** should be assessed and which target OU is intended.

This phase must not infer a candidate, create a manifest, or perform any AWS mutation. The source management account is shown but marked as a dedicated-plan exception.

### Phase 1: Account selection and scope gates

Validate that the selected account appears in the source discovery report, is not the source management account, and has an eligible candidate state. Then collect the required per-account inputs and confirm the direct-transfer path. Stop and request correction if the request includes a standalone period, `LeaveOrganization`, `RemoveAccountFromOrganization`, an unbounded batch, or an unspecified target OU.

### Phase 2: Read-only per-account preflight discovery

Load `steering/reporting.md`, create the selected account's three Git-ignored Markdown reports from `templates/`, then load `steering/preflight-discovery.md`. Create the reports in `reports/<account-id>/<assessment-id>/` unless the user has supplied an approved secure evidence directory. Link the readiness report to the organization-discovery report and collect only the selected account's source and target facts:

- Account state (`Account.State` must be `ACTIVE`), account age, target organization age, feature sets, partitions, enabled Regions, invitation/account quota capacity, and current handshakes.
- Source and target organization hierarchy, policy types, effective controls, target-root/OU policy impact, Control Tower registration or auto-enrollment impact, and target foundational recovery/access readiness.
- IAM Identity Center assignments, independent administrative access, root-email/contact recovery evidence, role trusts, IAM/resource/KMS policy organization conditions, and cross-account callers.
- Delegated administration, trusted access, organization trails, Config, security services, RAM shares, StackSets, networking, EventBridge, DNS, encryption, CI/CD, and external integrations.
- Billing/CUR history, cost-allocation tags, RI/Savings Plan ownership and coverage, target payer commercial/support arrangements, and report continuity.

Read-only collection is evidence, not approval. Flag unknowns, incomplete results, failed assessment jobs, missing source/target access, policy conflicts, and unowned findings as blockers.

### Phase 3: Create and present the go/no-go report

Load `steering/migration-workflow.md`. Update `migration-readiness-report.md` after each discovery area, then produce an account-specific report with:

- Confirmed identities, account/organization/OU IDs, profile-role identities, change window, manifest status, and named approvers.
- Pass, fail, unknown, and not-applicable result for every applicable hard blocker.
- Finding owner, remediation, exception reference, expiry, evidence location, and required revalidation for every non-pass item.
- Exact direct-transfer state, next permitted action, and no-go conditions.

Do not continue to execution while any hard blocker is unknown, unowned, unresolved, or only verbally accepted.

### Phase 4: Approved direct-transfer execution

Load `steering/direct-transfer-execution.md`. Before every state-changing action, add a `PLANNED` row to `migration-execution-journal.md`; immediately add the API outcome and required verification after it. Execute one approved account at a time, with the required named approval before each state-changing action:

1. **Invitation:** target management account invites only the manifest account; record the handshake ID.
2. **Acceptance:** migrating account verifies invitation source/target/handshake, then accepts through its approved accepting role or session.
3. **Target-root validation:** confirm target membership and root placement; validate break-glass access, critical workload paths, logging, Config, security, RAM/network, and billing/CUR reporting.
4. **OU move:** move only after a fresh policy and Control Tower enrollment/registration impact review and explicit approval.
5. **Post-transfer validation:** validate identity, policies, workload health, integrations, logging, security, billing, and source cleanup prerequisites.

A failed check pauses that account and its dependent cohort. Never silently continue, skip a finding, or attempt an irreversible workaround.

### Phase 5: Closure and approved cleanup

Load `steering/post-transfer-validation.md`. Update `migration-closure-report.md` with final validation evidence and obtain business, security, and finance sign-off. Source-side roles, policies, delegated administration, StackSets, RAM shares, and billing artifacts are not removed by default; present each cleanup action and wait for separate explicit approval.

## Execution safety rules

1. **No direct-transfer leave path:** Block `LeaveOrganization` and `RemoveAccountFromOrganization`. Do not disable departure protections to bypass a failed migration.
2. **No production mutations without approval:** Present the account ID, profile identity, API action, resources affected, expected result, recovery/rollback limitations, and evidence before every write. Obtain explicit approval for invitation, acceptance, OU move, and each cleanup action.
3. **Fail closed:** Eligibility, recovery, target access, policy, service dependency, landing-zone, billing, or evidence uncertainty is a no-go.
4. **Preserve access and observability:** Do not remove source access, trails, Config, security coverage, encryption access, or sharing until target replacements are verified.
5. **Least privilege:** Discovery workflows must not mutate. Execution permissions are limited to the approved account and transition.
6. **Management-account exception:** A former source management account can only join the target after members are handled and the source organization is eligible for deletion; require a dedicated approved plan.

## When to load steering files

- Startup organization discovery and account selection → `steering/organization-discovery.md`
- Report creation and safe evidence handling → `steering/reporting.md`
- Preflight evidence collection → `steering/preflight-discovery.md`
- Producing readiness decision and managing blockers → `steering/migration-workflow.md`
- Invitation, acceptance, root validation, and OU placement → `steering/direct-transfer-execution.md`
- Post-transfer evidence and separately approved source cleanup → `steering/post-transfer-validation.md`

## AWS MCP tools

Use the AWS MCP server for read-only discovery and, only after approval, exact scoped AWS API calls. Prefer `aws___run_script` for multi-step read-only inventory and `aws___call_aws` for single exact operations. Use `aws___search_documentation` or `aws___read_documentation` when current AWS behavior or quota requirements need confirmation.

`mcp.json` intentionally does not auto-approve AWS mutation-capable tools. Tool prompts and the local hooks enforce explicit approval boundaries.

## Quick test

After installation, request:

> Use `source-readonly` and `target-readonly` to discover both organizations. Create and present the organization report, then ask me which source member account should be assessed. Do not make changes.

The expected result is a safe, Git-ignored organization-discovery report showing source accounts and source/target OUs, followed by an account-selection question. Only after selection does the Power create the three per-account reports and begin the read-only readiness assessment.
