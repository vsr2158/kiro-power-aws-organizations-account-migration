# Read-Only Migration Preflight Discovery

## Purpose

Collect sufficient source, target, and migrating-account evidence to decide whether one explicitly approved member account is ready for a direct AWS Organizations transfer. This steering file is discovery only: it does not invite, accept, move, leave, deregister, alter policy, or modify resources.

Begin this file only after `organization-discovery.md` has presented the source/target organization report and the operator has selected a valid source member account. Load `reporting.md`, confirm the three per-account report files from `../templates/` exist in `reports/<account-id>/<assessment-id>/` (or the approved secure evidence location), and link `migration-readiness-report.md` to the organization-discovery report. Update the readiness report as each discovery section completes; leave execution and closure reports as `NOT_STARTED`/`PENDING` until their phases begin.

## Required context before collection

Confirm and report the following without guessing:

- Approved account ID, workload name, production criticality, and cohort.
- Source organization ID/profile, target organization ID/profile, and accepting-account profile or role.
- Target root ID, target OU ID, expected target policies, and Control Tower/landing-zone posture.
- Change record/window, business/technical/security/finance/on-call owners, and evidence location outside source control.

Run `sts get-caller-identity` under every supplied profile. Stop if the resulting account ID or role does not match the stated use of the profile.

## Discovery order

### 1. Eligibility and direct-transfer capacity

Use source and target read-only identities to collect:

- `organizations describe-account` for the approved account. Consume `Account.State`, not the deprecated `Status`; require `ACTIVE`.
- Account creation/join age evidence and target organization age evidence.
- `organizations describe-organization` for both organizations: ID, feature set, partition compatibility, and management account.
- Source and target roots, existing parents, enabled policy types, Regions used by the workload, and open handshakes.
- Target invitation and maximum-account quota capacity, including retry capacity.
- SCP/RCP and account-departure controls that could deny direct-transfer operations. Preserve these controls; do not propose disabling them.

Mark the account blocked if it is not active, identity/organization facts conflict, quota is insufficient, the target is too new, or a direct-transfer prerequisite cannot be verified.

### 2. Target root and OU placement impact

Before any invitation, determine the safe initial and final placement:

- Inventory target-root and target-OU policies of every enabled type, including SCPs, RCPs, declarative policies, tag/backup/S3 policies, and service-management policies.
- Capture direct and inherited source policies applied to the account and map each control to a target equivalent or approved exception.
- Confirm target IAM permissions permit invitations, acceptance, root placement, and the later OU move.
- If Control Tower is used, collect landing-zone version, auto-enrollment setting, target-OU registration status, baseline/guardrail rollout behavior, and `AWSControlTowerExecution` implications.
- Confirm target management-account root recovery, break-glass operation, monitored root email, and separately owned/monitored Log Archive and Audit email addresses where applicable.

The root is the initial placement after direct transfer. Do not recommend immediate final-OU placement before root validation succeeds.

### 3. Access, identity, and policy dependencies

Collect evidence rather than secrets:

- Root email reachability, account contacts, alternate contacts, root-MFA recovery procedure, and an independent tested administrator path.
- Source Identity Center instances, permission sets provisioned to the account, assignments, and required target equivalents.
- `OrganizationAccountAccessRole`, custom management roles, role trust policies, permission boundaries, and cross-account role consumers.
- Identity, resource, KMS, and stack policy references to source/target Organization IDs, account IDs, `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, `aws:ResourceOrgID`, and `aws:ResourceOrgPaths`.
- CloudTrail Organizations API activity for the last 90 days (or the approved retention period) to identify actual callers and automation dependencies.

Do not weaken policy conditions to eliminate findings. Assign a service/workload owner and validate intended least-privilege access after remediation.

### 4. Governance, security, logging, and service integrations

Collect source configuration, target equivalents, and a documented transition owner for:

- Delegated administrators and trusted access.
- CloudTrail organization trails, account trails, Config recorders/aggregators/rules/conformance packs, and central log delivery.
- GuardDuty, Security Hub, Inspector, Macie, Detective, IAM Access Analyzer, Firewall Manager, Systems Manager organization features, and target auto-enable settings in every used Region.
- StackSets, including `RetainStacksOnAccountRemoval`, instance state, and replacement or retention plan.
- AWS RAM shares owned/consumed by the account, external-share restrictions, retained-share invitations, and consumer/owner sequencing.
- IPAM, Network Manager, Transit Gateway/Cloud WAN, Route 53, Resolver rules, PrivateLink, endpoints, network firewall, and cross-account security/allow-list dependencies.
- EventBridge buses, resource policies, KMS grants, CI/CD roles, IaC source, data/observability pipelines, and external/SaaS integrations.

A source organization trail or Config aggregator ceases to provide organizational coverage after transfer. Require verified target coverage before final-OU placement.

### 5. Billing, commercial, and reporting continuity

Collect and review:

- Cost Explorer, CUR/report definitions, budgets, Cost Categories, active cost-allocation tags, invoices/tax evidence, and target reporting access.
- RI/Savings Plan ownership and coverage. Assets owned by the migrating linked account move with it, but organization-wide sharing does not cross organizations.
- Source/target RI/SP sharing preference, target payer commercial/discount/support eligibility as applicable, and target tax/payer compatibility.
- Target CUR delivery, destination permissions, downstream reporting, and an approved cross-account replication/reporting design if reporting spans the payer transition. Test both mixed-payer and single-target-payer periods.

Do not copy billing exports or sensitive reports into the repository.

## Required evidence output

For each collection item, record:

| Field | Requirement |
|---|---|
| Scope | Account, source/target organization, source parent, target root/OU, Region, and profile role used. |
| Result | `PASS`, `FAIL`, `UNKNOWN`, or `N/A` with the raw-response reference. |
| Finding | Clear dependency, policy, capacity, or operational impact. |
| Owner | Accountable person/team and remediation plan. |
| Decision | Remediate, time-bound exception, or no-go. |
| Revalidation | Exact check and time it must be repeated before the window. |

## Preflight stop conditions

Stop and mark the account `BLOCKED` if any of these are missing or unresolved: active/eligible state, target capacity, target recovery/access, policy safety, target landing-zone effect, named owner, source/target logging/security coverage, critical dependency plan, billing/reporting decision, evidence, or named approval.
