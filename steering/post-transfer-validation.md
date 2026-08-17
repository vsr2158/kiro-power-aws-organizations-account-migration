# Post-Transfer Validation and Closure

## Purpose

Prove that the migrated account is safely operating under the target organization and final OU before closing the migration record or removing any source-side access/integration.

Update `migration-closure-report.md` in the assessment report directory as validation areas are completed. The closure report remains `PENDING` until all final-OU validation evidence and named closure approvals are recorded; it must be `REOPENED` if a material post-transfer issue is found.

## Required validation evidence

### Organization, placement, and governance

- The migrating account's `describe-organization` identifies the target organization.
- The target account inventory and `list-parents` show the approved final OU.
- Expected effective SCPs, RCPs, declarative/tag/backup/S3/service policies are attached and tested against real workload operations.
- Control Tower registration or auto-enrollment outcome matches the approved design; baseline/guardrail operations are observed through completion or an approved exception.

### Access and identity

- Target break-glass access works through the approved recovery path.
- Required target Identity Center permission sets and assignments are provisioned and tested.
- Application and operational cross-account role paths work under corrected trust/resource/KMS conditions.
- Source management-account access is retained until target access is verified, then removed only through a separate approved cleanup decision.

### Workload, networking, and integrations

- Application owners complete critical-path, scheduled-job, event, queue, secret, encryption, and data-store smoke tests.
- RAM shares are retained/recreated/accepted; network routes, DNS, Resolver rules, endpoints, PrivateLink/TGW connectivity, KMS usage, and external allow lists are validated.
- StackSet/resource retention and ownership are clear; drift and resource health are acceptable.
- CI/CD, IaC, Organizations API callers, observability pipelines, and external/SaaS integrations run with target account/organization references.

### Logging, security, and operations

- Target CloudTrail, Config, central log delivery, Security Hub/GuardDuty/Inspector/Macie/Detective/Access Analyzer coverage, backup, monitoring, alarms, and incident routing include the account.
- Target Config recorder is active before target organization rules/conformance packs are relied on.
- Findings, aggregation, and alert routing are checked in every required Region.

### Billing, commercial, and reporting

- Target billing roles can perform the intended read-only Cost Explorer/reporting access.
- Budgets, alarms, cost categories, active cost-allocation tags, support/tax/payer setup, and RI/SP coverage impact are reviewed.
- Target CUR delivery and downstream reporting work. If reporting crosses the payer transition, mixed-payer and single-target-payer periods have been validated against the approved design.

## Closure record

Record at least:

```text
Account ID / workload:
Source organization and source OU:
Target organization and final OU:
Invitation and handshake evidence:
Acceptance and OU-move timestamps/responses:
Target-root and final-OU validation results:
Policy, IAM, RAM, StackSet, network, logging, security, billing, and integration evidence references:
Remaining exceptions, owner, expiry, and revalidation date:
Business, security, finance, and technical closure approvals:
CMDB/IaC/inventory updates and lessons learned:
```

Keep the record outside source control when it includes account-sensitive evidence.

## Source-side cleanup

Source cleanup is a separate production change, not an automatic final step. For every proposed cleanup action, present:

- Exact resource/policy/role/service configuration and owner.
- Proof that target replacement is working.
- Security, workload, billing, and recovery impact.
- Reversibility and rollback/fallback procedure.
- Explicit approval for that exact item.

Examples requiring separate approval include removing source-management roles, detaching/deleting source policies, deregistering delegated administrators, deleting or retaining StackSets/resources, terminating/altering RAM shares, disabling legacy trails or Config rules, and retiring billing/reporting artifacts.

Never delete historical CUR data, logs, evidence, or source security coverage simply because account transfer succeeded.

## Reopen conditions

Reopen the migration record and notify the accountable owners if post-transfer workload failures, denied operations attributable to target policy, missing logs/security findings, broken cross-account access, CUR/reporting gaps, or unresolved cleanup dependencies occur. Attach evidence and follow the same change/approval process for remediation.
