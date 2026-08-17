# Report Generation and Safe Evidence Handling

## Purpose

Create safe, durable organization and account-specific audit trails without putting sensitive AWS exports or credentials into source control. Reports are mandatory deliverables of this Power, not optional narrative output.

## Report directories and identifiers

### Startup organization discovery

At Power startup, create one report after source and target organization IDs are discovered:

```text
reports/organization-discovery/<source-org-id>-to-<target-org-id>/<discovery-id>/
└── organization-discovery-report.md
```

- `<discovery-id>` uses UTC `YYYYMMDDTHHMMSSZ`.
- The report contains source/target summary, source account IDs/names/states/current OU paths, target OU placement choices, and safe discovery findings.
- Account selection is requested only after this report is created and presented.

### Per-account migration assessment

After the operator selects a valid source member account, create a fresh assessment directory:

```text
reports/<account-id>/<assessment-id>/
├── migration-readiness-report.md
├── migration-execution-journal.md
└── migration-closure-report.md
```

- `<account-id>` is the selected 12-digit source member account ID.
- `<assessment-id>` uses UTC `YYYYMMDDTHHMMSSZ`. Do not overwrite a previous assessment; create a new directory for a new assessment or window.
- The generated `reports/` directory is intentionally Git-ignored. Templates in `templates/` are version-controlled.
- If the user supplies an approved secure evidence directory, create the same organization or account-specific structure there instead. State the chosen output location before writing files.

## Report creation rules

1. After source and target identities are verified and their organization IDs are known, create `organization-discovery-report.md` from its template; complete and present it before requesting an account ID.
2. After a valid source member account is selected, create all three per-account Markdown reports from the matching templates.
3. Populate known metadata immediately; use `UNKNOWN`, `PENDING`, `PARTIAL`, or `N/A` rather than inventing results.
4. Update the readiness report after each per-account discovery section and before presenting a go/no-go decision.
5. Add a `PLANNED` execution-journal row before every approved mutation, then add the actual result and required verification immediately after it.
6. Complete the closure report only after final-OU validation. Record separately approved source cleanup as `NOT_STARTED`, `COMPLETE`, or `DEFERRED`; do not assume it is complete because the transfer succeeded.
7. If a check fails or becomes unknown, update the relevant report state to `BLOCKED` and record owner, impact, remediation, revalidation, and secure evidence reference before ending the operation.

## Safe report content

Reports may contain:

- Organization IDs, root/OU IDs and paths, account IDs, account names, account state, non-secret role ARNs, migration state, timestamps, request IDs, handshake IDs, concise findings, owners, approvals, and evidence locations.
- Summaries of policy, service, security, networking, logging, billing, and workload validation outcomes.

Reports must never contain:

- Access keys, session tokens, passwords, MFA seed data, root recovery details, private keys, account email addresses, contact details, or alternate contacts.
- Raw policy JSON, CloudTrail events, support-case text, CUR data, assessment CSV contents, customer data, or unredacted resource exports.
- Secrets from environment files or credentials stores.

When evidence is sensitive, store it in the approved secure evidence system and put only an access-controlled reference in the report.

## Required user-facing summary

After startup discovery, state:

```text
Organization discovery report: <path> — <PASS / PARTIAL / FAIL>
Source accounts / OUs discovered: <count> / <count>
Target OUs discovered: <count>
Next permitted action: select one source member account ID for per-account assessment; no AWS mutation
```

After creating or updating per-account reports, state:

```text
Reports updated:
- Readiness: <path> — <GO / NO-GO / BLOCKED / PENDING>
- Execution journal: <path> — <current lifecycle state>
- Closure: <path> — <PENDING / CLOSED / REOPENED>
Next permitted action: <action>
```

Do not claim that a report is final or that execution may proceed until all applicable hard blockers and required approvals are documented.
