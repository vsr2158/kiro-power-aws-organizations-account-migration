# Automatic Organization Discovery and Account Selection

## Purpose

When the Power starts, automatically discover the source and target Organizations with read-only APIs, create an organization-discovery report, present a concise topology summary, and then ask the operator which source member account should receive a per-account readiness assessment.

This startup discovery is not a migration manifest, execution approval, or account selection by inference. It must not invite, accept, move, leave, deregister, alter policies, or change any AWS resource.

## Startup prerequisites

The only information required before startup discovery is:

1. A source read-only profile or role and a target read-only profile or role.
2. An output location if the default Git-ignored `reports/` directory is not acceptable.

If either source or target identity is missing, ask only for the missing profile/role. Do **not** ask which account to migrate before showing the organization report.

Verify every profile with `sts get-caller-identity`. Display the account ID and non-secret role ARN, then stop if the profile cannot be identified or is not the stated source/target management context.

## Mandatory read-only discovery

### 1. Organization facts

For both source and target profiles, collect:

- `organizations describe-organization`: organization ID, feature set, management account ID, and policy information available from the response.
- `organizations list-roots`: root IDs and enabled policy types.
- `organizations list-aws-service-access-for-organization` and `organizations list-delegated-administrators` only if permissions allow; mark `PARTIAL` rather than retrying or escalating privilege when unavailable.

Create `reports/organization-discovery/<source-org-id>-to-<target-org-id>/<discovery-id>/organization-discovery-report.md` from `../templates/organization-discovery-report.md` as soon as the organization IDs are known.

### 2. Recursive hierarchy and account inventory

For source and target roots, recursively enumerate all OUs with `list-organizational-units-for-parent`. For every root and OU, use `list-accounts-for-parent` to map direct accounts to their parent. Use `list-accounts` to ensure the source account inventory is complete and pagination is handled.

The report and presentation must include:

- A source OU tree with full OU paths, IDs, parent IDs/names, and direct-account counts.
- Every source account's account ID, name, `State`, and current root/OU path. Do not include email addresses or contacts.
- A target OU tree with IDs, paths, account counts, and only safe high-level Control Tower/policy notes known from read-only discovery.
- Identification of the source management account as a dedicated-plan exception rather than a normal member migration candidate.

Do not list accounts from `list-accounts` without resolving their parent relationship. Do not assume all listed accounts are eligible; inactive/suspended/closed/pending states are displayed but are not candidates for normal direct transfer.

### 3. Report and presentation

Update the organization-discovery report with `PASS`, `PARTIAL`, or `FAIL` for each discovery scope. Then present a concise summary with:

```text
Organization discovery report: <path>
Source: <org-id>; management account: <account-id>; accounts discovered: <count>; OUs discovered: <count>
Target: <org-id>; management account: <account-id>; OUs discovered: <count>

Source accounts:
| Account ID | Name | State | Current OU | Candidate note |
| ... |

Target OUs:
| OU path | OU ID | Policy / Control Tower note |
| ... |
```

If the organization is large, display every account/OU in the saved report and paginate or summarize the chat table while clearly stating the report path and total count. Do not omit accounts from the report.

## Required account-selection question

After presenting the report, ask exactly:

> Which **source member account ID** from the discovery report should be assessed for migration? Also provide the workload name and intended target OU ID. This selection starts read-only per-account readiness work only; an explicit signed manifest and named approvals remain required before any invitation, acceptance, or move.

Once the user selects an account, validate that it exists in the source report, is not the source management account, and has a normal candidate state before creating the per-account assessment report set. If the user selects an ineligible or management account, explain the reason and request another member account or a dedicated management-account plan.

## Handoff to per-account assessment

After a valid selection:

1. Request any still-missing per-account context: accepting role/profile, source/target root/OU confirmation, owners, change ticket/window, and explicit direct-transfer decision.
2. Create `reports/<account-id>/<assessment-id>/` using the three assessment templates.
3. Link the per-account readiness report to the organization-discovery report.
4. Continue with `preflight-discovery.md`; do not treat the startup report as complete per-account readiness.
