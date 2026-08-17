# AWS Organizations Account-Migration Readiness Checklist

A production-oriented checklist for moving AWS member accounts between AWS Organizations. It synthesizes the supplied [migration-readiness guide](https://aws-samples.github.io/cloud-operations-best-practices/docs/guides/AWS%20Organizations%20and%20AWS%20Accounts/account-migration-readiness/), the current [AWS Organizations migration documentation](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_account_migration.html), and these repositories:

- [aws-account-migration-automation](https://github.com/awslabs/aws-account-migration-automation) — batch SAM/Step Functions implementation.
- [aws-account-migration-example](https://github.com/aws-samples/aws-account-migration-example) — direct-transfer CLI example.
- [Account Assessment for AWS Organizations](https://github.com/aws-solutions/account-assessment-for-aws-organizations) — policy, delegated-administrator, and trusted-access assessment solution.

> [!WARNING]
> Account transfer is a production change. Complete the discovery, remediation, pilot, and approvals below before sending an invitation or accepting one. This document is a broad operational checklist, not a guarantee that every workload-specific dependency has been found.

## How to use this checklist

1. Create one evidence folder and one migration record per account. Record account ID, source/target organization IDs, source and target OU, owner, change window, rollback/contact path, and every command's output.
2. Treat `Automated` items as collection or validation steps, not approval to change production. An engineer must review all findings and exceptions.
3. Complete the **hard blockers** before scheduling a pilot. Migrate a representative non-production account first, then migrate independent accounts in dependency-aware cohorts.
4. Use direct account transfer where applicable: the target management account invites the migrating account, then the migrating account accepts. It avoids a standalone period.

Status legend: `☐ not started` · `◐ in progress` · `☑ complete` · `N/A documented` · `BLOCKED`

## Hard blockers and required decisions

| Status | Check / decision | How to assess | Evidence / exit criterion | Mode |
|---|---|---|---|---|
| ☐ | Define an accountable migration owner, workload owner, security approver, finance approver, and on-call contact. | Change record and migration plan. | Named owners, approved window, communications plan, and escalation path. | Manual |
| ☐ | Confirm the intended path is **direct transfer**, not remove-then-invite. | Review runbook and chosen automation. | Target sends invitation; migrating account accepts it without `LeaveOrganization`. | Manual |
| ☐ | Confirm the account is open and active. Closed or suspended accounts cannot migrate. | `aws organizations describe-account --account-id <ACCOUNT_ID>` from source; consume the `Account.State` response field. | Account state is `ACTIVE`; do not rely on the deprecated `Status` field after 2026-09-09. | Automated |
| ☐ | Check account age. An account created in an organization must be at least four days old; invited accounts are exempt. | Compare `JoinedTimestamp`/account-creation evidence with change window. | Eligibility documented. | Automated + review |
| ☐ | Check target organization age. It must be at least seven days old. | Target organization creation record. | Organization age meets requirement. | Manual |
| ☐ | Check target account/invitation quotas and service limits. | From the target management account, query Service Quotas (including the maximum-account quota/utilization exposed through `GetServiceQuota`) and `list-handshakes-for-organization`. | Capacity exists for the batch, including retries; request an increase before the change window if required. | Automated + review |
| ☐ | Preserve account-departure protections. | Inspect source/target SCPs for `organizations:LeaveOrganization` and account-close controls; newly console-created organizations can have default departure SCPs. | Direct transfer is validated without disabling these controls or using a legacy standalone/leave path. | Automated + review |
| ☐ | Confirm source/target Organization feature sets, AWS partitions, and enabled Regions are compatible with the migration design. | `aws organizations describe-organization`; regional inventory. | Exceptions and service-impact plan approved. | Automated + review |
| ☐ | Choose a target landing OU and transition policy set. | Review target OU, SCP/RCP/other policy attachments and IAM permissions before an account is moved beneath it. | Destination is approved; account will not be stranded or disrupted on placement. | Manual |
| ☐ | Run a non-production pilot covering the same services and governance controls as the production cohort. | Execute full runbook and validation for pilot. | Pilot findings remediated or formally accepted. | Manual |

## 1. Build the migration inventory and evidence package

| Status | Check | Automated collection | Human review / required action |
|---|---|---|---|
| ☐ | Account inventory | Enumerate source accounts, parent OUs, account tags, contacts, Regions, resource inventory, and ownership. | Categorize accounts by production criticality, data classification, business owner, and dependency cohort. |
| ☐ | Organization inventory | Export source/target organization details, roots/OUs, policy types enabled, policy attachments, delegated administrators, trusted access, and StackSets. | Identify organization-wide controls or central services that will cease to apply. |
| ☐ | Workload dependency map | Query resource inventories, IaC repositories, CMDB, Network/Transit Gateway, DNS, observability, CI/CD, partner integrations, and CloudTrail Organizations API activity. | Confirm cross-account callers, data flows, encryption keys, allow lists, endpoints, domains, ownership dependencies, and automation that assumes the source management organization. |
| ☐ | Baseline snapshots | Export CloudTrail/Config posture, IAM roles, policies, account contacts, budgets, alarms, tags, RAM shares, StackSet state, and assessment CSVs. | Store immutable evidence and define expected post-transfer state. |
| ☐ | Cutover plan | Generate a per-account execution checklist and validation commands. | Establish sequencing, freeze criteria, communications, rollback alternatives, and an explicit no-go list. |

Use distinct source and target profiles with least privilege. Never place credentials, root credentials, MFA seed data, support-case data, or assessment exports containing sensitive policies in this repository.

## 2. Deploy and run automated discovery

### 2.1 Account Assessment for AWS Organizations

Deploy the Hub, Org-Management, and Spoke stacks as described in the [implementation guide](https://docs.aws.amazon.com/solutions/latest/account-assessment-for-aws-organizations/). For production, use a separate shared-services/security account for the Hub where feasible; the Org-Management stack remains in the management account. Deploy the Spoke stack to every assessed account, including the management account separately because service-managed StackSets do not deploy to it.

| Status | Automated assessment | Required review |
|---|---|---|
| ☐ | Run **Delegated Admin** scan and download CSV. | Every delegated service is assigned a replacement or has a service-specific deregistration plan. |
| ☐ | Run **Trusted Access** scan and download CSV. | Determine whether each service integration needs reconfiguration, re-registration, or a target-side delegated admin. |
| ☐ | Run/trigger **Policy Explorer** then check Job History. | Job status must be `SUCCEEDED`; investigate `SUCCEEDED_WITH_FAILED_TASKS`, `FAILED`, access-denied errors, and unscanned account/region/service/resource tasks. |
| ☐ | Search Policy Explorer by `Add OrgId` and by `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, source Org ID, target Org ID, account IDs, and relevant OU paths. | Review every finding in full context; do not remove a policy `Condition` just to make a finding disappear. Remediate and validate intended access with IAM Access Analyzer and workload tests. |
| ☐ | Search identity-based, resource-based, and SCP policy types by principal, action, resource, condition, effect, Region, and `GLOBAL`. | Confirm all material policies are included. Policy Explorer is a string-contains search and its nightly scan is not real time. |

**Tool boundary:** Account Assessment finds policy, delegated-admin, and trusted-access dependencies. It does not establish policy correctness, and it does not cover every migration dependency. In particular, AWS RAM, StackSets, service quotas, billing, Identity Center assignments, RCPs, declarative policies, and most management policies require additional checks below.

### 2.2 CFAT and target landing-zone assessment

The readiness guide uses the Cloud Foundation Assessment Tool (CFAT) as a target-foundation assessment. Run it against the target organization and preserve the report.

| Status | Check | Exit criterion |
|---|---|---|
| ☐ | Control Tower / landing-zone readiness | If source governance includes Control Tower, plan the supported unmanage/AFT removal procedure and the source `AWSControlTowerExecution` trust disposition. Before target OU placement, determine whether a Control Tower landing zone at version 3.1+ has auto-enrollment enabled; moving the account can then apply baseline resources and controls automatically. If Control Tower governance is intended, confirm whether the target OU must be registered or auto-enrolled and when its baseline/guardrail rollout will occur. | Target OU enrollment and its resource/control effects are deliberately staged, tested, and approved; they are not an unexamined consequence of moving the account. |
| ☐ | Target management-account recovery and foundational identities | Review the target management account root MFA, break-glass procedure, and monitored root email. If Control Tower is used, identify separately owned, monitored email/distribution-list addresses for the Log Archive and Audit accounts. | Recovery and ownership are tested; no individual-only or reused email address is a single point of failure for management, logging, or audit operations. |
| ☐ | CloudTrail, Config, and central logging | Source organization trails and Config aggregators will stop collecting from the moved account. Identify any account-level trail needed for uninterrupted audit coverage, target organization-trail inclusion, target Config aggregator, and configuration-recorder state. | Target CloudTrail/log archive/security tooling includes the account. Enable the Config recorder before target organizational rules/conformance packs apply; their deployment retry window can be limited. |
| ☐ | Identity Center and access model | Target Identity Center, break-glass roles, groups, permission sets, and account assignment design are ready. |
| ☐ | Governance foundation | Required policy types are enabled and target guardrails have been tested against the account's actual resources. |

## 3. Source-account and target-organization readiness checks

### 3.1 Access, recovery, and identity

| Status | Check | Suggested evidence / read-only collection | Required remediation / validation |
|---|---|---|---|
| ☐ | Root email is valid and accessible | `aws account get-primary-email --account-id <ACCOUNT_ID> --region us-east-1` | Confirm the mailbox can receive a password reset; for Organizations-created accounts, perform the root-password recovery setup before migration if necessary. |
| ☐ | Current contact and alternate contacts | `aws account get-contact-information` and `get-alternate-contact` for `SECURITY`, `BILLING`, and `OPERATIONS`. | Confirm phone, address, and monitored contacts are current. |
| ☐ | Root MFA and break-glass recovery | Secure recovery procedure / break-glass test record. | Confirm root MFA owner/device process and at least one independent, tested administrative recovery route. Never record secrets in evidence. |
| ☐ | Independent account administrator | IAM inventory and role trust-policy review. | Ensure an admin path does not rely only on source Organization permissions or source Identity Center assignments. |
| ☐ | IAM Identity Center access | From source Identity Center: `list-instances`, `list-permission-sets-provisioned-to-account`, and `list-account-assignments`. | Export all assignments. Recreate equivalent target permission sets/groups/assignments after transfer and test user access. Source assignments are removed when the account leaves. |
| ☐ | Source management-account access role | Inspect `OrganizationAccountAccessRole`, custom management roles, permission boundaries, and trust policies. | After the target access path is verified, remove source-management access roles only if the old organization must no longer retain access. This is an intentional post-cutover security change. |
| ☐ | Identity and resource policy conditions | Policy Explorer plus IAM role, inline/managed policy, resource policy, KMS key policy, and trust-policy exports. | Update `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, `aws:ResourceOrgID`, and `aws:ResourceOrgPaths` source values to target values or appropriate account-specific principals; validate least privilege. |

### 3.2 Organization policies and governance

All source organization policies stop applying when the account transfers. Inventory both policies directly attached to the account and those inherited from its source OU/root. Reproduce applicable controls in the target before cutover and evaluate whether target controls will deny existing workloads.

| Status | Policy class | Automated collection / CLI | Required review |
|---|---|---|---|
| ☐ | Enabled organization policy types | `aws organizations list-roots --query 'Roots[0].PolicyTypes'` | Limit subsequent checks to enabled types, but record disabled types too. |
| ☐ | SCPs | `list-policies-for-target --target-id <ACCOUNT_ID> --filter SERVICE_CONTROL_POLICY`; `describe-policy` | Confirm source restrictions to retain, target policies to apply, and that IAM/SCPs do not block invitation, acceptance, account movement, or remediation. Review the full IAM language now available to SCPs, including `Condition`, individual resource ARNs, `NotAction`, and `NotResource`. |
| ☐ | Resource control policies (RCPs) | `list-policies-for-target --filter RESOURCE_CONTROL_POLICY` | Account Assessment Policy Explorer does not cover RCPs. Validate resource access under target RCPs. |
| ☐ | Declarative policies for EC2 | `list-policies-for-target --filter DECLARATIVE_POLICY_EC2` | Validate every affected EC2 resource against target declarative requirements, including allowed AMIs and public-snapshot controls. |
| ☐ | Tag and Backup policies | `list-policies-for-target --filter TAG_POLICY` / `BACKUP_POLICY`; `describe-effective-policy` | Recreate target policy intent and ensure resource/account tags and backup plans remain compliant. For accounts using Redshift Serverless namespaces or Aurora DSQL clusters, validate their explicit backup-policy selection. |
| ☐ | S3 policies | `list-policies-for-target --filter S3_POLICY`; `describe-policy` | Validate target organization-level S3 Block Public Access enforcement against every bucket and workload path before placement. |
| ☐ | AI/security/service management policies | For enabled types, collect `AISERVICES_OPT_OUT_POLICY`, `BEDROCK_POLICY`, `CHATBOT_POLICY`, `INSPECTOR_POLICY`, `NETWORK_SECURITY_DIRECTOR_POLICY`, `SECURITYHUB_POLICY`, and `UPGRADE_ROLLOUT_POLICY`. | Reproduce applicable target policies and verify service behavior. For `INSPECTOR_POLICY`, confirm the target scan types and Regions; attachment/inheritance can enable Inspector automatically when the account joins or moves to an OU. Include any new enabled policy type discovered by `list-roots`. |
| ☐ | Organization/account/OU tags | `list-tags-for-resource` for accounts and OUs. | Recreate required organizational tags in target and confirm tag policy enforcement. |

For each attachment, traverse `list-parents --child-id <ACCOUNT_ID>` and the parent hierarchy, save `describe-policy` content, record the effective policy, and map the source control to a target control or approved exception.

### 3.3 Delegated administration and trusted access

| Status | Check | Automated collection | Manual/service-specific action |
|---|---|---|---|
| ☐ | Delegated administrator registration | `aws organizations list-delegated-administrators` and `list-delegated-services-for-account --account-id <ACCOUNT_ID>`. | Register a replacement before deregistration where the service supports it, export service configuration, and deregister the migrating account before transfer. Confirm service data-retention implications. |
| ☐ | Trusted access | `aws organizations list-aws-service-access-for-organization`; Account Assessment CSV. | Identify source organization integrations that cease to govern the account and configure target equivalents. |
| ☐ | Firewall Manager transition | From source administrator, collect Firewall Manager policies, applications, protocol lists, scope/resource-cleanup settings, compliance state, managed-resource inventory, and regional service quotas. | Recreate target policies and confirm target compliance before removing source protections. Intentionally retain or clean up source-managed resources, account for concurrent source/target protections, and validate DNS/Network Firewall RAM rule-group replacements. |
| ☐ | Config and CloudTrail organization integrations | Source Config aggregator/rules/conformance packs and CloudTrail organization trails; target Config recorder, aggregator, rules/conformance packs, and organization trail. | Source aggregation and organization trails cease for the moved account. Preserve audit coverage with an account trail if necessary, enable the target Config recorder before organization deployment, and verify target log/Config data arrives. |
| ☐ | Security analytics and security membership | Per Region: Detective graph/membership, IAM Access Analyzer organization analyzers/findings, GuardDuty, Inspector, Macie, Security Hub, and related auto-enable settings. | Preserve/export required Detective and Access Analyzer data before delegated-admin changes; re-associate the account with target security administrators or validate target auto-enrollment and finding aggregation in every required Region. |
| ☐ | Systems Manager organization operations | Explorer resource data sync, Change Manager requests, OpsCenter cross-account roles/resource policies, Patch Manager, and Systems Manager StackSets. | Plan for permanent source sync-data loss and failed scheduled/pending Change Manager actions. Recreate target syncs/permissions and update or replace `OpsItemGroup` policies and cross-account execution roles. |
| ☐ | IPAM and Network Manager | IPAM delegated administration, pool CIDR allocations, pool RAM shares/principals, compliance/import settings, Network Manager global networks, and registered transit gateways. | Release or transfer source IPAM allocations before loss of management where required; configure target IPAM/RAM principals and re-register target global-network transit gateways. Validate CIDR monitoring and routes. |
| ☐ | Service Catalog, License Manager, and Marketplace | Service Catalog portfolios/products/AppRegistry shares and provisioned products; License Manager configurations, grants, resource discovery, and AWS Marketplace entitlements. | Recreate or re-associate target portfolios/products and required permissions without disrupting provisioned products. Recreate/accept license shares and validate Marketplace grants/entitlements after the organization/OU share changes. |
| ☐ | Artifact and Audit Manager | AWS Artifact organization agreements and Audit Manager assessment scopes/evidence collection. | Obtain legal/compliance decision on target agreements and enable/accept them where required. Add the account to target Audit Manager assessment scopes; it is not added automatically. |
| ☐ | Organization-wide operational views | AWS Health organizational view, S3 Storage Lens, Trusted Advisor, Compute Optimizer, DevOps Guru, Well-Architected workload/custom-lens shares, and other enabled trusted services. | Re-establish required target aggregation, dashboard scope, recommendations, and organization/OU shares. Preserve required historical data and document accepted observability gaps. |

### 3.4 Cross-account resources, policy, and integration checks

| Status | Dependency | Collection / automation | Required review and remediation |
|---|---|---|---|
| ☐ | AWS RAM shares owned by the account | In each used Region: `aws ram get-resource-shares --resource-owner SELF`, `list-resources --resource-owner SELF`, and `get-resource-share-associations --association-type PRINCIPAL`. | Record direct-account, OU, and organization principals. For organization-scoped shares, update consumers or enable `retain-sharing-on-account-leave-organization` from the share owner where supported and approved. Retention converts organization principals to explicitly invited external principals, so record/accept each invitation and test actual resources and principals. For resource types that cannot be shared externally, co-migrate the owner/resource or redesign; when retention is unavailable and owner/consumer both move, sequence consumer before owner. |
| ☐ | AWS RAM shares consumed by the account | `get-resource-shares --resource-owner OTHER-ACCOUNTS` and `list-resources --resource-owner OTHER-ACCOUNTS`. | Recreate/accept target share invitations and test access to every shared resource (for example, subnets, TGW, Route 53 Resolver rules, licenses). |
| ☐ | CloudFormation StackSets | From source management/delegated StackSets account: `list-stack-sets`, `list-stack-instances --stack-instance-account <ACCOUNT_ID>`, and `describe-stack-set`. | Service-managed StackSets can delete resources when the account leaves if `RetainStacksOnAccountRemoval=false`. Set retention or explicitly plan replacement/redeployment; validate drift and ownership. |
| ☐ | CloudFormation stack policies | In every used Region, inventory stacks and run `aws cloudformation get-stack-policy --stack-name <STACK_NAME>`. | Search policy JSON for `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, source/target Org IDs, account IDs, and OU paths. Update intended policies without weakening resource-protection controls. |
| ☐ | EventBridge cross-account event buses | `aws events describe-event-bus --name default --region <REGION>` for every used Region; review custom buses/rules/targets. | Replace/add organization conditions and source Org ID references; test delivery, retries, DLQs, and permissions. |
| ☐ | Resource-based policies | Account Assessment results plus S3, KMS, SQS, SNS, Lambda, ECR, Secrets Manager, API Gateway, EventBridge, and service-specific policy exports. | Remediate organization ID, organization path, and source account principals without broadening public or cross-account access. |
| ☐ | KMS encryption dependencies | KMS key policies, grants, aliases, and cross-account consumers. | Preserve required target-side principals and test decrypt/encrypt for workloads, logs, backups, queues, and secrets. |
| ☐ | Networking and DNS | VPC/TGW/Cloud WAN/VPN/Direct Connect, Route 53 zones/resolver rules, PrivateLink, VPC endpoints, security groups, IP allow lists, and Network Firewall. | Validate cross-account shares and routes continue; update policies, RAM shares, DNS associations, and source/target account/Org allow lists. |
| ☐ | CI/CD and automation | IaC, CodePipeline/CodeBuild, GitHub/GitLab integrations, EventBridge schedules, Lambda, Step Functions, and scripts using Organizations APIs. | Update account/Org IDs, role trusts, permission boundaries, and assumptions. Lambda/automation that calls Organizations APIs may fail after the account changes management organization. |
| ☐ | Organizations API caller inventory | Query organization CloudTrail logs with Athena (or equivalent) for `eventSource = 'organizations.amazonaws.com'` over at least the prior 90 days; capture caller identity, account, API action, time, and result. | Assign every active caller to a target management/delegated-administrator path or retire it. Test target-side automation and alerting; do not infer this inventory only from source repositories. |
| ☐ | Data/observability integrations | Cross-account CloudWatch, OpenSearch, Athena/Glue Lake Formation, S3 log buckets, CloudTrail, Config aggregators, Security Lake, dashboards, alarms, SNS/chatops. | Restore aggregation, delivery permissions, subscriptions, queries, and alerts; verify data arrives after cutover. |
| ☐ | External/SaaS dependencies | Partner integrations, support tools, certificate/identity providers, allow lists, tax/legal ownership, and billing contacts. | Update account identifiers, principals, contacts, or contractual artifacts as required. |

### 3.5 Billing, commercial, and reporting

| Status | Check | Automated collection | Required action |
|---|---|---|---|
| ☐ | Billing history and reports | Export Cost Explorer, CUR, budgets, Cost Categories, RI/SP coverage/utilization, invoices, tax evidence, and relevant management-account reports. `aws cur describe-report-definitions --region us-east-1` identifies CUR configuration. | Preserve reports: organizational-level reporting/history remains with the source management account after transfer. Configure and validate the target CUR delivery, destination access, and downstream reporting before cutover. Where reporting must span the payer transition, use an approved cross-account reporting/replication design and validate both mixed-payer and single-target-payer reporting periods. |
| ☐ | Cost allocation tags | `aws ce list-cost-allocation-tags --status Active`. | Record active tags and reactivate equivalent tags in the target organization; allow propagation time and validate allocation. |
| ☐ | Savings Plans and Reserved Instances | From the migrating account, `aws savingsplans list-savings-plans --states active` and `aws ec2 describe-reserved-instances --filters Name=state,Values=active --region <REGION>`. | RIs/SPs owned by the migrating account move with it; source or target organization-wide sharing benefits do not span organizations. Assess lost coverage/cost impact and contact AWS Support if ownership must change. |
| ☐ | Discount-sharing mode | Source and target Billing Console preference for RI/SP sharing (organization-wide, prioritized group, or restricted group). | Model impact on both organizations and schedule migration to minimize uncovered usage. |
| ☐ | Billing and support setup | Billing console/account settings, Support plan, payment/tax configuration, tax exemptions, budgets, billing alarms, cost categories, and contacts. | Direct transfer preserves payment method, contact information, and Support plan; do not treat their transfer-time reconfiguration as a prerequisite. Still validate payer compatibility, tax settings/exemptions, budgets, alarms, and current contact data after transfer. The sample CLI does not perform these checks. |
| ☐ | Target payer commercial enrollment | Target payer agreement/discount program, invoicing, and Enterprise Support eligibility as applicable. | Confirm the new payer is enrolled in the intended commercial and support arrangements before migrating production spend; obtain the AWS account-team/contractual decision where terms or billing ownership must change. |
| ☐ | Billing access | Test a read-only Cost Explorer query with intended post-transfer roles. | Fix billing permission/access configuration before cutover; do not rely on the batch tool's historical five-day `GetCostAndUsage` check as complete billing validation. |

### 3.6 Conditional: legacy consolidated-billing organizations and management-account workloads

Apply this section only when the source organization has the `CONSOLIDATED_BILLING` feature set, when the migration is intended to move to an all-features organization, or when the source management account runs workload resources. It is **not** a general prerequisite for an all-features-to-all-features direct transfer. Do not revive the blog's legacy payment-method or member-account `LeaveOrganization` steps; use the current direct-transfer process where it is supported.

| Status | Conditional check | Suggested collection | Required decision / action |
|---|---|---|---|
| ☐ | Feature-set and topology decision | `aws organizations describe-organization`; inventory management-account resources and identities. | If the existing organization has no management-account workloads, evaluate enabling all features in place. If workloads cannot safely move, use a dedicated, approved plan to establish a replacement governance/management account and migrate the former management account only after its members and old organization are handled. |
| ☐ | Management-account workload disposition | Resource inventory, CloudTrail, Config, IaC, billing, networking, KMS, and identity inventory for the source management account. | Migrate workloads to member accounts where feasible. Otherwise document workload continuity, target ownership, and the dedicated sequencing required to convert the former management account to a target member account. |
| ☐ | Source organization retirement eligibility | Inventory every remaining member account and its state, including suspended or pending-closure accounts, and review the organization-deletion prerequisites. | Before the former management account can become standalone, all member-account relationships must have an approved disposition and the old organization must be eligible for deletion. Escalate unresolved account closure/removal states rather than discovering them during the cutover window. |
| ☐ | Organization hierarchy reconstruction | Export root ID and, for every OU/account, name, ID, email, parent ID/name, and parent type using Organizations list/describe APIs. | Recreate the target OU hierarchy or approve a documented redesign before member-account placement. Map every account to its target OU and expected policy set. |
| ☐ | Target management-account commercial foundation | If a new target management account is created, collect CUR definitions, active cost-allocation tags, tax-inheritance settings, budgets, alarms, Cost Categories, and billing contacts. | Configure target CUR, activate required cost tags, validate tax/payer configuration, and preserve historical CUR files outside the retiring management account. Direct transfer avoids reconfiguring a transferred member account's payment/contact/Support plan; this row applies to a newly created management account. |
| ☐ | Transitional placement for the former management account | Target OU policy and access review. | If the former management account will run workloads as a member, place it initially in a purpose-built transitional OU with only the approved minimum policies, then validate and move it to its final OU. Do not use a policy-free OU without explicit security approval. |

## 4. Automation strategy and guardrails

### Use each supplied tool for its appropriate scope

| Tool | Appropriate use | Important limitations / required controls |
|---|---|---|
| **Account Assessment for AWS Organizations** | Baseline discovery of identity/resource/SCP policies, delegated administrators, and trusted access. Export CSV evidence. | Policy Explorer is nightly rather than real-time and is not a policy-correctness validator. Resolve scan errors and cover unscanned resources manually. |
| **CFAT** | Target organization foundation/landing-zone readiness. | It complements, rather than replaces, per-account dependency, policy, billing, and workload validation. |
| **aws-account-migration-example** | Small, controlled direct-transfer execution. It invites from the target, assumes a predeployed `AwsAccountMigrationAcceptInvitationRole` in the source account to accept, then moves accepted accounts to a target OU. | Use an explicit account ID (`-a`) for pilot/cohorts; keep the interactive confirmation; do not use `-q` in production automation. The repository warns that running without an account can include the source management account and ultimately delete the source organization. It does not manage payment method, Support plan, phone verification, or migration readiness. |
| **aws-account-migration-automation** | Reference architecture for batch orchestration: XLS input, DynamoDB state, Step Functions, wait/notification gates, Access Analyzer policy scan, billing-access test, support-case workflow, parallel processing, reports, invitations, and OU placement. | **Do not run unchanged for direct transfer.** Its `leave_organization.py` removes a linked member account before `join_organization.py` invites it, creating a standalone period contrary to the current direct-transfer approach. It is Python 3.8/SAM-era code, uses broad permissions (including `support:*`), creates migration roles, and defaults to parallel processing. Review/update code, runtime, permissions, rate limits, idempotency, approval gates, and audit controls before any use. |

### Minimum safe automation design

| Status | Control | Requirement |
|---|---|---|
| ☐ | Read-only discovery first | Separate discovery roles/workflows from execution roles. Discovery must not invite, accept, move, leave, deregister, or mutate policies. |
| ☐ | Explicit allow list | Require a signed per-account manifest; never infer a batch from all accounts in an organization. |
| ☐ | Preflight gates | Fail closed on eligibility, incomplete assessment jobs, unresolved high-risk findings, missing recovery proof, missing owner approval, or policy incompatibility. |
| ☐ | Human approval | Require named approvers before invitation, before acceptance, and before any post-transfer security cleanup. |
| ☐ | Direct-transfer state machine | `preflight → invite from target → verify open handshake → accept from migrating account → verify target membership/root → validate → move to target OU → validate again`. Do not call `LeaveOrganization` in this path. |
| ☐ | Concurrency and retries | Rate-limit Organizations operations, handle `ConcurrentModificationException`, use idempotent request/state checks, and stop the cohort on unexpected state. |
| ☐ | Least privilege | Scope execution roles to specific accounts/actions and narrowly trusted principals. Do not use broad `support:*`, wildcard cross-account trust, or persistent admin roles without explicit approval. |
| ☐ | Audit and evidence | Log actor, source/target organization, account, handshake ID, timestamps, approvals, command/API response, result, and exception per transition. |
| ☐ | Safe failure behavior | A failed check pauses an account and alerts the owner; it must not silently skip a dependency or proceed to the next irreversible transition. |

## 5. Pre-transfer go/no-go checklist

Complete every applicable line for each account immediately before the change window.

| Status | Go/no-go check |
|---|---|
| ☐ | Account eligibility, source/target IAM permissions, invitation quota, organization age, and account age have been revalidated. |
| ☐ | Source and target organization IDs, root IDs, target OU ID, source/target account IDs, Region set, and execution identities are independently verified. |
| ☐ | Account Assessment results are current; all jobs succeeded with no unreviewed failed tasks; Policy Explorer findings are remediated or formally accepted. |
| ☐ | Every direct/inherited source policy and expected target policy is mapped; the target OU is safe for initial placement. |
| ☐ | Delegated admins are replaced/deregistered and service-specific export/rebuild plans are approved. |
| ☐ | RAM, StackSet, EventBridge, KMS, network, DNS, IAM trust, logging, and workload integration plans are complete. |
| ☐ | Identity Center assignments and independent recovery/access paths are documented and tested. |
| ☐ | Root email, contact phone, alternate contacts, and root MFA/recovery procedure are verified. |
| ☐ | Billing reports are exported; RI/SP impacts, cost tags, Support plan, tax, budgets, and alarms have approved remediation plans. |
| ☐ | Target logging, monitoring, security, backup, Config, and incident response integrations are ready. |
| ☐ | Pilot completed successfully and production-specific differences are reviewed. |
| ☐ | Freeze window, business-owner approval, on-call coverage, communication plan, and escalation/AWS Support plan are active. |
| ☐ | The execution manifest is limited to approved accounts and the automation dry-run/preflight output matches it exactly. |

**No-go:** stop if ownership, recovery, access, policy remediation, active workload dependencies, target controls, billing impact, or evidence is unknown. A direct transfer is not a substitute for remediation.

## 6. Controlled direct-transfer procedure

Only perform this section under an approved production change. The following is intentionally procedural rather than a copy/paste batch command.

1. From the **target organization management account**, send an invitation for the approved account and record the handshake ID.
2. From the **migrating account** through the preapproved accepting role/session, verify the invitation target/source/handshake details and accept it.
3. Verify the account joined the expected target organization and initially resides in its root. Do not immediately apply a restrictive OU until access and critical workload smoke tests pass.
4. Move the account to the approved target OU only after validating its policies and permissions are appropriate for that OU.
5. Execute the post-transfer checklist below, remediate failures, and obtain application/security/finance sign-off.

The source management account requires special sequencing: migrate/remove its member accounts, delete the old organization, then treat the former management account as a standalone account that can accept the target invitation. Do not attempt this in a general batch without a dedicated, reviewed plan.

## 7. Post-transfer validation and closure

| Status | Validation | Expected result |
|---|---|---|
| ☐ | Organization membership | From the migrated account, `describe-organization` reports the target organization. From target, account inventory/parent relationship is correct. |
| ☐ | OU and policies | Account is in the approved target OU; expected SCPs/RCPs/declarative/management policies are attached and effective. Validate workload API calls under real target controls. |
| ☐ | Administrator and user access | Target break-glass access works; required Identity Center permission sets/assignments are provisioned and end users can access required roles. Source management access is removed only after confirmation. |
| ☐ | Policy remediation | Re-run Account Assessment/Policy Explorer and IAM Access Analyzer validations. Confirm no stale source Org ID/OU path references remain where access is intended. During cutover troubleshooting, capture the policy ARN in same-account/same-organization access-denied errors to identify the exact SCP/RCP/IAM/session/boundary policy responsible. |
| ☐ | RAM and networking | Required resource shares are retained/recreated/accepted; DNS, routes, endpoint policies, KMS access, and cross-account connectivity are tested. |
| ☐ | StackSets and IaC | Retained resources exist; target StackSets/IaC deployment ownership is clear; drift and resource health are acceptable. |
| ☐ | Logging and security | CloudTrail, Config, central logs, Security Hub/GuardDuty/Inspector/Macie findings, backup, monitoring, alarms, and incident routing include the account. |
| ☐ | Workload health | Application owner completes smoke tests for critical paths, scheduled jobs, event delivery, queues, secrets, encryption, data stores, and external integrations. |
| ☐ | Billing and commercial | Target budgets/alarms/cost categories/tags are active, Support plan and tax settings are correct, and RI/SP coverage/cost impact is reviewed. |
| ☐ | Source cleanup | Source organization policy, delegated-admin, RAM, StackSet, billing, and access artifacts are retired only when their removal is approved and no longer required. |
| ☐ | Closure | Update CMDB/IaC/inventory, attach evidence, document exceptions, capture lessons learned, and obtain owner/security/finance closure approvals. |

## Evidence manifest template

Maintain this record outside source control for every account:

```text
Account ID:
Account name / workload:
Source organization ID / OU:
Target organization ID / target OU:
Migration cohort and change ticket:
Business owner / technical owner / security owner / finance owner:
Direct-transfer handshake ID:
Assessment job IDs and CSV locations:
Policy findings and remediation/exception references:
Delegated-admin and trusted-access disposition:
RAM / StackSet / EventBridge / KMS / network disposition:
Identity Center and break-glass validation result:
Billing, RI/SP, support, tax, and cost-tag disposition:
Preflight approver / timestamp:
Invitation sent / accepted / account moved timestamps:
Post-transfer validation results:
Outstanding risks, exceptions, and expiration dates:
Closure approvers / timestamp:
```

## AWS announcement review (September 2025–August 2026)

This review is scoped to announcements that change, automate, constrain, or improve the AWS Organizations account-migration workflow. It is not a catalogue of unrelated AWS service releases.

| Date | Announcement | Workflow impact | Documentation disposition |
|---|---|---|---|
| Sep 2025 | [Account `State` field](https://aws.amazon.com/about-aws/whats-new/2025/09/aws-organizations-provides-account-state-information-member-accounts/) | `Status` is deprecated after 2026-09-09; `State` distinguishes `SUSPENDED`, `PENDING_CLOSURE`, and `CLOSED`. | Added `Account.State` eligibility and automation requirement. |
| Sep 2025 | [Full IAM language for SCPs](https://aws.amazon.com/about-aws/whats-new/2025/09/aws-organizations-iam-language-service-control-policies/) | Target policies can use conditions, individual resource ARNs, `NotAction`, and `NotResource`, increasing both precision and migration risk. | Expanded SCP review requirements. |
| Nov 2025 | [Control Tower automatic enrollment](https://aws.amazon.com/about-aws/whats-new/2025/11/aws-control-tower-automatic-enrollment/) | Moving an account to an OU can apply Control Tower baselines and controls automatically when the landing zone has opted in. | Added an explicit pre-placement auto-enrollment assessment. |
| Nov 2025 | [Direct account transfers](https://aws.amazon.com/about-aws/whats-new/2025/11/aws-organizations-direct-account-transfers/) | The target invitation plus account acceptance flow avoids a standalone period and preserves payment method, contact information, Support plan, governance, and consolidated billing continuity. | Retained direct transfer as the required path; removed transfer-time payment/contact/Support-plan reconfiguration as a prerequisite. |
| Nov 2025 | [Inspector organization policy](https://aws.amazon.com/about-aws/whats-new/2025/11/amazon-inspector-organization-wide-management-aws-organizations-policies/) | Target policy attachment/inheritance can automatically enable Inspector scan types and Regions for a joining or moved account. | Added scan-type/Region validation. |
| Nov 2025 | [Organization-level S3 Block Public Access](https://aws.amazon.com/about-aws/whats-new/2025/11/amazon-s3-block-public-access-organization-level-enforcement) | Target S3 policy can enforce Block Public Access across inherited accounts. | Added an S3-policy and workload-path validation. |
| Jan 2026 | [Policy ARNs in access-denied errors](https://aws.amazon.com/about-aws/whats-new/2026/01/additional-policy-details-access-denied-error/) | Same-account/same-organization denies can identify the responsible SCP, RCP, IAM, session, or boundary policy. | Added a cutover troubleshooting/evidence step. |
| Feb 2026 | [RAM share retention](https://aws.amazon.com/about-aws/whats-new/2026/02/aws-resource-access-manager/) | Retained organization shares become explicit external-account shares that require invitation acceptance. | Added invitation acceptance and resource/principal testing. |
| Apr 2026 | [Backup policies for Redshift Serverless and Aurora DSQL](https://aws.amazon.com/about-aws/whats-new/2026/04/backup-policies-aurora-dsql-redshift-serverless/) | Target backup policies can directly select these resource types. | Added a conditional policy-selection check. |
| Jul 2026 | [Default account-departure controls](https://aws.amazon.com/about-aws/whats-new/2026/07/aws-organizations-security-controls-new-orgs-console/) | New organizations created through the console can have SCPs denying member-account departure and closure. | Preserve these controls; direct transfer must not use a legacy `LeaveOrganization` path. |
| Jul 2026 | [RCP quota increase](https://aws.amazon.com/about-aws/whats-new/2026/07/aws-organizations-resource-control-policy-limit-increase-2000/) | The RCP-per-organization quota increased to 2,000. | No workflow change beyond the existing quota/capacity validation. |
| Aug 2026 | [Maximum-account quota visibility](https://aws.amazon.com/about-aws/whats-new/2026/08/aws-organizations/) | Target management accounts can inspect maximum-account quota and utilization with Service Quotas. | Added `GetServiceQuota` as the preferred preflight capacity check. |

## Source coverage summary

| Source | Incorporated checks |
|---|---|
| [AWS migration-readiness guide](https://aws-samples.github.io/cloud-operations-best-practices/docs/guides/AWS%20Organizations%20and%20AWS%20Accounts/account-migration-readiness/) | Account Assessment and CFAT deployment/use, StackSets, Identity Center, RAM, organization policy types, delegated admins, EventBridge, billing, recovery, direct transfer, and post-transfer validation. |
| [AWS Organizations migration guide](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_account_migration.html) | Eligibility constraints, direct-transfer permissions/flow, management-account sequencing, reporting backup, old management access, tags, billing/Savings Plans/RIs, and post-migration review. |
| [AWS Organizations member-account migration blog series](https://aws.amazon.com/blogs/mt/aws-organizations-moving-an-organization-member-account-to-another-organization-part-1/) | Organization policy inheritance, RAM association/sequencing, all organization condition keys, delegated-administrator transition, trusted-access service effects, Control Tower, Config/CloudTrail, Firewall Manager, security analytics, commercial services, and organization-wide operational views. |
| [Account Assessment solution](https://github.com/aws-solutions/account-assessment-for-aws-organizations) | Policy Explorer search scope/freshness, delegated-admin and trusted-access scans, job-history validation, and scanner limitations. |
| [aws-account-migration-example](https://github.com/aws-samples/aws-account-migration-example) | Named source/target profiles, accepting role, explicit account selection, interactive confirmation, target invitation/accept/move workflow, and commercial-preparation gaps. |
| [aws-account-migration-automation](https://github.com/awslabs/aws-account-migration-automation) | Batch orchestration patterns, notifications, reporting, Access Analyzer scan, billing-access check, support-case gate, parallelism/concurrency considerations, and the critical remove-then-invite incompatibility with direct transfer. |

## References

### AWS documentation

- [Migrate an account to another organization with AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_account_migration.html)
- [AWS Organizations quotas](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_reference_limits.html)
- [Managing service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [Managing organization-level S3 Block Public Access](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_s3.html)
- [AWS RAM resource-share configuration](https://docs.aws.amazon.com/ram/latest/userguide/working-with-sharing-create.html)
- [AWS Control Tower account auto-enrollment](https://docs.aws.amazon.com/controltower/latest/userguide/account-auto-enrollment.html)
- [Troubleshooting IAM access denied errors](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_access-denied.html)

### Readiness and assessment tooling

- [AWS Organizations account migration readiness guide](https://aws-samples.github.io/cloud-operations-best-practices/docs/guides/AWS%20Organizations%20and%20AWS%20Accounts/account-migration-readiness/)
- [Account Assessment for AWS Organizations implementation guide](https://docs.aws.amazon.com/solutions/latest/account-assessment-for-aws-organizations/)
- [Account Assessment for AWS Organizations — use the solution](https://docs.aws.amazon.com/solutions/latest/account-assessment-for-aws-organizations/use-the-solution.html)
- [Cloud Foundation Assessment Tool (CFAT)](https://github.com/cloud-foundations-on-aws/cloud-foundations-templates/tree/main/cfat)

### Reference implementations

- [aws-account-migration-example](https://github.com/aws-samples/aws-account-migration-example)
- [aws-account-migration-automation](https://github.com/awslabs/aws-account-migration-automation)
- [Account Assessment for AWS Organizations source](https://github.com/aws-solutions/account-assessment-for-aws-organizations)

### Background and announcement history

- [Moving an organization member account: Part 1 — policies, RAM, and condition keys](https://aws.amazon.com/blogs/mt/aws-organizations-moving-an-organization-member-account-to-another-organization-part-1/)
- [Moving an organization member account: Part 2 — delegated administrators](https://aws.amazon.com/blogs/mt/aws-organizations-moving-an-organization-member-account-to-another-organization-part-2/)
- [Moving an organization member account: Part 3 — trusted access](https://aws.amazon.com/blogs/mt/aws-organizations-moving-an-organization-member-account-to-another-organization-part-3/)
- [Migrating accounts between AWS Organizations with consolidated billing to all features](https://aws.amazon.com/blogs/mt/migrating-accounts-between-aws-organizations-with-consolidated-billing-to-all-features/)
- [AWS Organizations direct account transfers](https://aws.amazon.com/about-aws/whats-new/2025/11/aws-organizations-direct-account-transfers/)
- [AWS RAM share retention when accounts change organizations](https://aws.amazon.com/about-aws/whats-new/2026/02/aws-resource-access-manager/)

For other workflow-impacting announcements, see the [AWS announcement review](#aws-announcement-review-september-2025august-2026) above.
