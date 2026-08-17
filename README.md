# AWS Organizations Account-Migration

A [Kiro Power](https://kiro.dev/powers/) for safe, evidence-driven migration of AWS member accounts between AWS Organizations.

It follows the direct-transfer workflow:

`preflight → target invitation → account acceptance → target-root validation → target OU placement → post-transfer validation`

It does not use a standalone `LeaveOrganization` or `RemoveAccountFromOrganization` path.

## What it does

1. Automatically discovers the source and target Organizations with read-only AWS APIs, creates an organization report, and presents source accounts plus source/target OUs before asking which account to assess.
2. Collects source, target, and migrating-account evidence with read-only AWS APIs after an operator selects one source member account.
3. Assesses account eligibility, policies, delegated administration, trusted access, IAM Identity Center, RAM, StackSets, networking, security/logging, billing, CUR reporting, and target landing-zone effects.
4. Creates a Git-ignored organization-discovery report at startup, then three safe Markdown reports per selected account: a readiness report, an execution journal, and a closure report. They retain summaries and secure evidence references—not credentials, raw exports, or sensitive policy data.
5. Produces a per-account go/no-go report with accountable owners and evidence references.
6. Runs the direct-transfer transition only after explicit approval at invitation, acceptance, target-OU move, and separately approved cleanup stages.
7. Validates target access, workload health, governance, observability, integrations, and commercial/reporting continuity before closure.

## Safety model

- Discovery is read-only and never creates invitations, moves accounts, changes policies, deregisters services, or deletes resources.
- Startup inventory is read-only and is used only to present account and OU choices. A selected account still requires a signed, explicit manifest before any write operation; the Power never derives an execution cohort from the organization account list.
- Every failed or unknown hard blocker pauses the account and requires remediation or a time-bounded, named exception.
- Direct transfer never calls `LeaveOrganization` or `RemoveAccountFromOrganization`.
- A source management account is not part of a general cohort; it needs a dedicated plan after member-account disposition and source-organization retirement eligibility are verified.
- Evidence must not contain root credentials, MFA material, session tokens, or unredacted sensitive exports.
- The Power creates `reports/organization-discovery/<source-org-id>-to-<target-org-id>/<discovery-id>/organization-discovery-report.md` before account selection, then `reports/<account-id>/<assessment-id>/` with a readiness report, execution journal, and closure report. Generated reports are Git-ignored; version-controlled templates only contain safe placeholders.

## Workflow

| Phase | Outcome | Write access |
|---|---|---|
| Startup discovery | Source/target organization report with accounts and OUs, followed by account-selection prompt | None |
| Account selection and safety gates | One selected source member account, intended target OU, and per-account context | None |
| Read-only preflight | Evidence inventory and initialized per-account report set | None |
| Go/no-go report | Updated readiness report, owned remediation, and approved exceptions | None |
| Direct transfer | Execution-journal entries for invitation, acceptance, root validation, and OU move | Explicit approval at each transition |
| Closure | Closure report, validation evidence, and approved scoped source cleanup | Separate explicit approval per cleanup |

## Installation

Clone this repository into a Kiro powers directory:

```bash
git clone https://github.com/vsr2158/kiro-power-aws-organizations-account-migration.git \
  .kiro/powers/aws-organizations-account-migration
```

This repository is public. Review the Power's safety model and documentation before using it against an AWS Organization.

## Prerequisites

- Kiro with an AWS MCP server available.
- Separate least-privilege source and target read-only profiles or roles for startup discovery.
- An approved account-specific manifest, target root/OU, change window, named owners, and migrating-account accepting role only before execution.
- An independently tested target recovery and administrative access path before per-account execution approval.

## Example prompts

> Use `source-readonly` and `target-readonly` to automatically discover the source and target Organizations. Create and present the organization report, including source accounts and source/target OUs, then ask which source member account should be assessed. Do not make changes.

> Assess source member account `123456789012` for direct transfer to target OU `<ou-id>` after it has been selected from the organization report. Create the per-account reports under `reports/123456789012/`; do not make changes.

> The account is approved for invitation. Show the exact target-side API call, account and target organization identities, handshake evidence to collect, and the post-invitation stop condition. Do not run it yet.

## Research

- [`ir research.md`](ir%20research.md) — the full AWS Organizations account-migration readiness checklist, source coverage, announcement review, direct-transfer procedure, and references that inform this Power.

## Documentation

- [POWER.md](POWER.md) — onboarding, automatic organization discovery, account selection, execution boundaries, and tool guidance.
- [steering/organization-discovery.md](steering/organization-discovery.md) — startup source/target topology discovery, organization report, and account-selection workflow.
- [steering/reporting.md](steering/reporting.md) — mandatory report creation, safe evidence rules, and report lifecycle.
- [templates/](templates/) — version-controlled templates for organization discovery, readiness, execution, and closure reports.
- [steering/preflight-discovery.md](steering/preflight-discovery.md) — read-only discovery plan.
- [steering/migration-workflow.md](steering/migration-workflow.md) — go/no-go reporting and blocker management.
- [steering/direct-transfer-execution.md](steering/direct-transfer-execution.md) — approved direct-transfer state machine.
- [steering/post-transfer-validation.md](steering/post-transfer-validation.md) — validation, closure, and cleanup controls.
- [`.kiro/hooks/account-migration.json`](.kiro/hooks/account-migration.json) — hooks that block legacy leave paths and require approval for migration writes.

## License

Apache-2.0
