---
Title: Enterprise Identity and Security
Domain: Azure Platform Engineering
Difficulty: Senior
Estimated Duration: 150 minutes
Prerequisites:
  - Module 01 — Enterprise Azure Platform
  - Basic understanding of Active Directory concepts
  - Basic understanding of authentication vs authorisation
Learning Objectives:
  - Explain Microsoft Entra ID architecture and its role in Azure
  - Describe Azure RBAC and design a least-privilege model
  - Explain Managed Identity and when to use it over Service Principals
  - Describe Privileged Identity Management and why it exists
  - Explain Conditional Access and how it enforces identity controls
  - Describe Microsoft Secure Score critically — not as a target, as a signal
  - Explain Defender for Cloud and its role in platform security posture
  - Describe Key Vault integration patterns for application secrets
  - Design identity governance standards for an enterprise platform
Interview Focus:
  - Senior Platform Engineer
  - Principal Platform Engineer
---

# Module 04 — Enterprise Identity and Security

---

## 1. Learning Goal

After completing this module you will be able to design enterprise identity and security architecture on a whiteboard and justify every decision.

You will understand that identity is the security boundary in Azure — not networking.

You will be able to recommend Managed Identity, design RBAC strategy, implement least privilege, and discuss Secure Score critically without treating it as the objective.

You will be prepared to defend identity decisions under questioning by a Principal Engineer.

---

## 2. Why This Exists

### The Shift from Network-Based to Identity-Based Security

On-premises security was perimeter-based.

The firewall was the boundary.

If you were inside the network, you were trusted.

Cloud changed this fundamentally.

In Azure, resources are accessed via APIs.

APIs are protected by identity — tokens, credentials, role assignments.

An attacker who compromises a service account with excessive permissions can move laterally across an entire Azure subscription without ever touching the network.

Network controls remain important.

But identity is the primary security boundary.

The majority of cloud security breaches exploit misconfigured identity — overprivileged accounts, hardcoded credentials, absent multi-factor authentication.

### The Problem in Practice

Early enterprise Azure environments exhibited consistent patterns:

- Teams requested Owner rights to deploy faster and never had them reduced
- Applications stored connection strings and storage keys in configuration files
- Service accounts had Contributor rights at subscription scope rather than specific resource scope
- No one reviewed who had access to what
- Automation pipelines ran with human identities attached to them

These patterns created significant exposure.

An unrotated storage key in a config file checked into a repository could be discovered by an attacker within minutes of the repository being made public.

A service account with Contributor rights on a production subscription could be used to deploy malicious workloads or exfiltrate data.

### What the Platform Team Must Design

The platform team's responsibility is not to review individual permissions.

It is to build the identity architecture, RBAC standards, and automation patterns that prevent these situations from arising.

---

## 3. Core Concepts

### Microsoft Entra ID

Microsoft Entra ID (formerly Azure Active Directory) is the identity provider for Azure.

It is not a directory service in the traditional sense — it is a cloud-native identity platform.

It provides:

- Authentication — who are you?
- Authorisation foundation — what groups are you in?
- Token issuance — here is a token proving your identity
- Conditional Access — should this authentication be allowed?
- Identity governance — who should still have access?

Every action in Azure goes through Entra ID.

When a user opens the Azure Portal, Entra ID authenticates them and issues a token.

When a Terraform pipeline deploys a resource, it authenticates to Entra ID and receives a token.

When an application reads from Key Vault, it authenticates to Entra ID and receives a token.

Understanding Entra ID means understanding that identity is everywhere in Azure — not just at the login screen.

### Authentication vs Authorisation

These are frequently confused and frequently tested.

**Authentication:** Proving who you are. Entra ID handles this. The result is a token.

**Authorisation:** Determining what you are allowed to do. Azure RBAC handles this. The token is presented. RBAC checks what roles are assigned to the identity represented by that token.

A user can be authenticated — successfully logged in — but unauthorised to perform a specific action. These are independent concerns handled by different systems.

### Azure RBAC

Azure Role-Based Access Control is the authorisation system for Azure resources.

RBAC assigns roles to identities at a scope.

The three elements:

- **Identity** — a user, group, Service Principal, or Managed Identity
- **Role** — a collection of permissions (e.g. `Reader`, `Contributor`, `Storage Blob Data Reader`)
- **Scope** — Management Group, Subscription, Resource Group, or individual resource

An identity is granted a role at a scope. They inherit permissions for everything at or below that scope.

Built-in roles range from broad (`Owner`, `Contributor`) to narrowly scoped (`Storage Blob Data Reader`, `Key Vault Secrets User`).

The principle of least privilege requires assigning the narrowest role at the lowest scope that satisfies the requirement.

### Least Privilege

Least privilege means granting only the permissions required to perform a specific task.

Nothing more.

In practice:

- Do not assign `Contributor` when `Storage Blob Data Contributor` is sufficient
- Do not assign at Subscription scope when Resource Group scope is sufficient
- Do not assign permanently when just-in-time access via PIM is appropriate
- Do not assign to individuals when groups provide better lifecycle management

Least privilege reduces blast radius.

If an identity is compromised, an attacker can only perform actions within the permissions assigned.

Broad permissions — especially `Owner` — mean a compromised identity is catastrophic.

### Managed Identity

A Managed Identity is an identity in Entra ID that is automatically managed by Azure.

It is assigned to an Azure resource — a Virtual Machine, an App Service, a Function App, an Azure DevOps agent.

Azure creates and manages the underlying credential.

The application code never sees a password, a client secret, or a certificate.

When the application needs to access another Azure resource — Key Vault, Storage, SQL — it requests a token from the Azure Instance Metadata Service using the Managed Identity. Azure returns a short-lived token. The application uses the token to authenticate.

There are two types:

**System-assigned:** Created with the resource. Deleted when the resource is deleted. One-to-one relationship.

**User-assigned:** Created independently. Can be assigned to multiple resources. Lifecycle is managed separately.

Managed Identities eliminate the most common credential management failure: hardcoded or long-lived secrets.

### Service Principal

A Service Principal is a non-human identity registered as an application in Entra ID.

It has credentials — a client secret or a certificate.

Those credentials must be managed — rotated before expiry, stored securely, distributed to the workloads that need them.

Service Principals are appropriate when:

- The workload runs outside Azure — GitHub Actions, on-premises pipelines, external tools
- Managed Identity is not supported by the service
- Cross-tenant authentication is required

For workloads running inside Azure, Managed Identity is almost always preferable.

Service Principals introduce credential management risk that Managed Identities eliminate.

### Privileged Identity Management (PIM)

PIM is an Entra ID feature that provides just-in-time privileged access.

Instead of permanent role assignments, PIM uses eligible assignments.

An identity is eligible for a role but does not hold it continuously.

When privileged access is needed — deploying a production change, investigating an incident — the engineer activates the role.

The activation requires approval, multi-factor authentication, and a business justification.

The role is granted for a time-limited period — typically 1–8 hours.

After the period expires, the role is automatically removed.

PIM creates an audit trail.

Every activation is logged — who activated, which role, which scope, what justification, at what time.

Without PIM, a `Contributor` assignment on a production subscription is permanently held by every engineer who ever needed it.

With PIM, that assignment does not exist until it is explicitly activated.

### Conditional Access

Conditional Access is a policy engine in Entra ID that evaluates authentication attempts against a set of conditions.

Conditions include:

- User or group identity
- Device compliance state
- Location — IP range, country
- Application being accessed
- Sign-in risk score — calculated by Entra ID based on patterns

If conditions are met, the policy can:

- Grant access
- Require multi-factor authentication
- Block access
- Require a compliant device

Conditional Access is how enterprises enforce that access to production Azure environments only comes from managed devices with MFA.

It operates independently of RBAC.

A user can have RBAC permissions but Conditional Access can block the authentication before those permissions are ever evaluated.

### Microsoft Secure Score

Microsoft Secure Score is a measurement of an organisation's security posture.

It is calculated from a set of recommendations across Entra ID, Microsoft Defender, and Azure.

Completing a recommendation increases the score.

**Critical point:** Secure Score is a signal, not an objective.

Blindly enabling every recommendation without evaluating business impact is dangerous.

Example: A recommendation may suggest enabling MFA for all users. In isolation, correct. But enabling this during an incident response window without testing could lock out engineers who need emergency access.

Example: A recommendation may suggest disabling legacy authentication protocols. Correct in general. But if a legacy application still uses those protocols, disabling them will break the application.

**The correct approach:** Evaluate each recommendation against business context. Prioritise high-impact, low-disruption improvements. Deprioritise or accept recommendations that carry operational risk. Use the score to identify gaps, not to drive decisions.

### Defender for Cloud

Defender for Cloud is a unified security posture management and threat protection service.

It provides:

- Security posture assessment — evaluates resources against security benchmarks
- Regulatory compliance dashboard — maps controls to standards like PCI-DSS, ISO 27001, CIS
- Threat protection — detects active attacks, suspicious activity, and compromised identities
- Recommendations — specific, actionable steps to remediate security gaps

Defender for Cloud should be enabled at the Management Group level so all subscriptions are covered.

The platform team reviews recommendations centrally and prioritises remediation across all Landing Zones.

Application teams are not responsible for Defender for Cloud — the platform team is.

### Key Vault

Azure Key Vault is the secure store for secrets, certificates, and encryption keys.

Applications should never have credentials in code, configuration files, or environment variables.

The pattern:

1. Secret is stored in Key Vault
2. Application's Managed Identity is assigned `Key Vault Secrets User` role on the Key Vault
3. Application calls the Key Vault SDK with its Managed Identity token
4. Key Vault returns the secret
5. No human ever handles the secret in transit

Key Vault provides:

- Access control via RBAC or Access Policies
- Audit logging — every secret access is logged
- Soft delete and purge protection — accidental deletion is recoverable
- Versioning — secrets can be rotated without application changes

The platform team defines Key Vault standards.

Application teams create Key Vaults within those standards.

---

## 4. Architecture

### Identity and Authorisation Flow

```
User / Application / Pipeline
        |
        ↓  Authenticate
Microsoft Entra ID
        |
        ↓  Conditional Access Policy evaluated
        |       (MFA required? Device compliant? Risk acceptable?)
        |
        ↓  Token issued (if allowed)
Azure Resource Manager / Azure Service
        |
        ↓  RBAC evaluated
        |       (Does this token's identity have a role assignment
        |        at this scope that permits this action?)
        |
        ↓  Action permitted or denied
```

### Managed Identity — Application to Key Vault

```
Application (App Service / VM / Function)
  │
  │  (1) Request token from Azure Instance Metadata Service
  ↓
Azure IMDS (169.254.169.254)
  │
  │  (2) Token issued for Managed Identity
  ↓
Application
  │
  │  (3) Present token to Key Vault
  ↓
Key Vault
  │
  │  (4) RBAC check: does this Managed Identity have
  │      Key Vault Secrets User at this scope?
  ↓
Key Vault returns secret value
  │
  ↓
Application uses secret — no credential ever stored in code
```

### RBAC Scope Hierarchy

```
Management Group
    │  ← Assign: Security Reader for audit team
    │
    Subscription
        │  ← Assign: Contributor for platform pipeline (scoped tightly)
        │
        Resource Group
            │  ← Assign: Storage Blob Data Contributor for application MI
            │
            Individual Resource
                │  ← Assign: Key Vault Secrets User for specific vault
```

Assign at the lowest scope that satisfies the requirement.

### PIM Activation Flow

```
Engineer needs production access
        |
        ↓  Submits PIM activation request
        |       Role: Contributor
        |       Scope: Production Subscription
        |       Justification: "Deploy hotfix for incident INC-4821"
        |       Duration: 2 hours
Approver reviews request
        |
        ↓  Approves (or denies)
MFA challenge issued to engineer
        |
        ↓  Engineer completes MFA
Role assigned for 2 hours
        |
        ↓  Engineer performs deployment
Role automatically removed after 2 hours
        |
        ↓  Audit log: activation, approval, actions taken, expiry
```

### Enterprise Identity Ownership

```
Platform Team owns:
┌──────────────────────────────────────────────────────────┐
│  Entra ID tenant configuration                           │
│  Conditional Access policies                             │
│  PIM configuration and role eligibility assignments      │
│  RBAC standards and built-in role selection              │
│  Managed Identity strategy                               │
│  Key Vault standards and policy                          │
│  Defender for Cloud configuration                        │
│  Secure Score oversight                                  │
└──────────────────────────────────────────────────────────┘

Application Team owns:
┌──────────────────────────────────────────────────────────┐
│  Their application's Managed Identity                    │
│  RBAC assignments within their Landing Zone              │
│  Key Vault for their application secrets                 │
└──────────────────────────────────────────────────────────┘

Application Team does NOT own:
┌──────────────────────────────────────────────────────────┐
│  Subscription-level RBAC assignments                     │
│  Conditional Access policies                             │
│  Service Principal creation (without platform approval)  │
│  Cross-subscription role assignments                     │
└──────────────────────────────────────────────────────────┘
```

---

## 5. Design Decisions

### Managed Identity vs Service Principal

| Factor | Managed Identity | Service Principal |
|---|---|---|
| Credential management | None — Azure manages it | Required — must rotate secrets/certs |
| Works inside Azure | Yes — most services supported | Yes |
| Works outside Azure | No | Yes |
| Audit trail | Yes | Yes |
| Risk of credential exposure | None | High if not managed |
| Cross-tenant scenarios | Limited | Yes |

**Decision rule:**

Use Managed Identity for everything running inside Azure.

Use Service Principal for pipelines running outside Azure (e.g. GitHub Actions, on-premises agents) and cross-tenant scenarios.

Use Workload Identity Federation with Service Principals when possible — this eliminates secrets on Service Principals for pipeline scenarios.

### Permanent vs Just-In-Time Access

**Never assign permanent privileged roles** — `Owner`, `Contributor`, `User Access Administrator` — directly to individuals on production subscriptions.

All privileged access must flow through PIM with:

- Approval required for production scope
- MFA enforced on activation
- Time-limited duration (maximum 8 hours)
- Business justification logged

Standing Contributor on a production subscription means any account compromise grants an attacker full control for as long as the account exists.

PIM activation means a compromised account cannot activate PIM without MFA and approval — two additional barriers.

### RBAC at Group vs Individual Level

Assign roles to Entra ID groups, not individual users.

Benefits:

- Lifecycle management: when an engineer leaves, removing them from the group removes all their Azure permissions
- Consistency: all members of the group have identical permissions
- Auditability: group membership is audited through Entra ID access reviews

At scale, individual role assignments become unmanageable.

The platform team defines role groups — `PlatformTeam-Contributor`, `AppTeamA-Reader`, `AppTeamA-Contributor` — and manages membership centrally.

### Secure Score — Critical Evaluation

Secure Score recommendations fall into categories:

**Enable immediately — low risk, high value:**
- Enable MFA for all users
- Disable legacy authentication
- Enable Defender for Cloud plans
- Require HTTPS on App Services

**Evaluate carefully — potential operational impact:**
- Restrict admin portal access (may break workflows)
- Enable just-in-time VM access (requires process change)
- Remove deprecated accounts (verify nothing depends on them)

**Accept with justification — legitimate business reasons to not implement:**
- Enabling a specific Defender plan that generates cost exceeding the risk reduction
- A recommendation that conflicts with a regulatory control the organisation has already addressed differently

The platform team should review Secure Score quarterly.

Remediate high-impact recommendations.

Document accepted risks with business justification.

Never treat the number itself as the goal.

---

## 6. Real World Scenario

An e-commerce company completed a cloud security audit.

The findings included:

- 14 engineers have permanent `Owner` access on the production subscription
- 6 applications store database connection strings in `appsettings.json` in source control
- 3 CI/CD pipelines authenticate using Service Principal client secrets that have never been rotated
- Multi-factor authentication is not enforced for Azure Portal access
- Microsoft Secure Score is 34% — the auditor flagged this without further context

**How the platform team responds:**

**Finding 1 — Permanent Owner access:**

PIM is enabled on the production subscription.

All 14 engineers have their permanent `Owner` assignments converted to PIM-eligible assignments.

They no longer hold the role.

When production access is needed, they activate via PIM, with approval from their team lead and MFA.

Daily deployments are done via automation pipelines using Managed Identity or narrowly scoped Service Principals — not human accounts.

**Finding 2 — Secrets in source control:**

Each application is assigned a system-assigned Managed Identity.

Key Vaults are created per application.

Database passwords are moved to Key Vault.

Application code is updated to use the Azure SDK's `DefaultAzureCredential` — which automatically uses the Managed Identity when running in Azure.

Connection strings are removed from all configuration files.

An Azure Policy is assigned at the Landing Zones Management Group that audits App Service configurations for connection strings containing keywords like `Password=`.

**Finding 3 — Unrotated Service Principal secrets:**

The three pipelines are migrated from client secret authentication to Workload Identity Federation — GitHub Actions and Azure DevOps both support this.

Federation eliminates secrets entirely from these pipelines.

The old Service Principal client secrets are revoked.

**Finding 4 — No MFA on Azure Portal:**

A Conditional Access policy is created:

- Users: All users
- Cloud apps: Microsoft Azure Management
- Conditions: Any location
- Grant: Require multi-factor authentication, require compliant device

This enforces MFA for all Azure Portal and Azure CLI access regardless of location.

**Finding 5 — Secure Score 34%:**

The platform team evaluates the recommendations.

They immediately implement 8 high-value, low-risk recommendations — raising the score to 61%.

They evaluate and schedule 4 medium-risk recommendations.

They document 3 accepted risks — specific recommendations that conflict with existing operational processes — with business justifications.

They report that the score of 61% represents a significantly improved posture and that the remaining 39% includes accepted risks and long-term improvement items.

**Outcome:**

The annual audit produces zero critical identity findings.

No engineer holds permanent privileged access to production.

No application credentials exist in source control or environment variables.

MFA is enforced for all Azure management operations.

---

## 7. Common Mistakes

### Mistake 1: Treating RBAC as an Identity System

Engineers describe RBAC as "how identity works in Azure."

RBAC is the authorisation system.

Entra ID is the identity system.

**Why it matters:** Conflating the two leads to incorrect architectural decisions. A developer who "needs access to Azure" needs both an Entra ID identity (authentication) and an appropriate RBAC role (authorisation). Fixing only RBAC without considering Conditional Access leaves authentication controls incomplete.

### Mistake 2: Assigning Owner by Default

Teams request Owner permissions to "avoid permission issues."

This is the most common privileged access mistake.

**Why it is wrong:** Owner can assign roles, modify policies, and delete resources. Giving Owner to every engineer on a subscription means any account compromise — phishing, credential stuffing, session hijack — grants an attacker full control. `Contributor` is sufficient for most engineering tasks. Specific data plane operations require even narrower roles.

### Mistake 3: Using Service Principals When Managed Identity Is Available

Terraform pipelines running on Azure DevOps agents create Service Principals with client secrets.

The secrets are stored in pipeline variables.

The secrets are never rotated.

**Why it is wrong:** Azure DevOps agents running in Azure can use Managed Identity if the agent pool is hosted on VMs or Container Apps with Managed Identity assigned. Managed Identity eliminates the secret entirely. Service Principal client secrets that are never rotated are a persistent credential exposure risk.

### Mistake 4: Hardcoding Secrets in Application Configuration

Application teams place database passwords, storage keys, and API keys in `appsettings.json`, environment variables, or Kubernetes secrets backed by etcd.

**Why it is wrong:** Configuration files end up in source control. Environment variables are visible in logs, crash dumps, and deployment pipelines. The secure pattern is Managed Identity + Key Vault. No human ever handles the secret in transit. No credential appears in configuration. Rotation happens in Key Vault without application changes.

### Mistake 5: Optimising Secure Score as the Primary Goal

A security team spends three weeks enabling every Secure Score recommendation.

They enable just-in-time VM access but forget to communicate the process change.

Engineers can no longer connect to VMs during an incident.

**Why it is wrong:** Secure Score is a signal. Implementing recommendations without evaluating operational impact creates new problems while fixing others. Every recommendation requires a risk assessment: what is the impact of enabling this, what is the risk of not enabling it, and what process changes are required?

### Mistake 6: Ignoring PIM Activation Audit Logs

PIM is configured and engineers activate roles as required.

Nobody reviews the activation logs.

An engineer activates a production Owner role 40 times in a single month with the justification "routine work" for each activation.

**Why it is wrong:** The audit trail is valuable only if it is reviewed. Unusual activation patterns — frequency, duration, scope, justification quality — are signals of either policy non-compliance or potential misuse. PIM audit logs should be integrated with SIEM and reviewed at minimum monthly.

---

## 8. Troubleshooting

### Scenario 1: Application Cannot Access Key Vault — AuthorizationFailed

**Symptoms:**

Application returns `AuthorizationFailed` when attempting to retrieve a secret from Key Vault.

The Managed Identity is enabled on the App Service.

**Possible Causes:**

- RBAC role not assigned to the Managed Identity on the Key Vault
- Key Vault is using Access Policies instead of RBAC — and no Access Policy exists for the Managed Identity
- The wrong Managed Identity is assigned (user-assigned MI, but code uses system-assigned)
- Key Vault private endpoint DNS is not resolving correctly (if Key Vault is private)

**Investigation:**

Check the Key Vault's Access Control (IAM) blade. Search for the App Service's Managed Identity. Confirm `Key Vault Secrets User` is assigned.

If using Access Policies: Check whether the Managed Identity's object ID is listed with `Get` secret permission.

Check Key Vault diagnostic logs for the specific `SecretGet` operation — the log entry will show the calling identity's object ID and the deny reason.

**Resolution:**

Assign `Key Vault Secrets User` role to the App Service's Managed Identity at the Key Vault scope.

If migrating from Access Policies to RBAC: Enable the RBAC authorisation model on the Key Vault and create the role assignment.

---

### Scenario 2: PIM Activation Succeeds but Access is Denied

**Symptoms:**

Engineer activates a PIM-eligible `Contributor` role on a production subscription.

Activation succeeds — PIM shows the assignment as active.

Attempting to deploy resources returns `AuthorizationFailed`.

**Possible Causes:**

- Role propagation delay — Entra ID RBAC changes can take 1–5 minutes to propagate
- The resource operation requires a role the engineer does not hold even with Contributor (e.g. data plane access)
- A Deny assignment exists at the resource or Management Group level overriding the Contributor role
- The engineer is operating in a different subscription than the one they activated PIM for

**Investigation:**

Wait 5 minutes and retry — propagation delay is the most common cause.

Check the Activity Log for the specific failed operation — note the exact permission denied.

Check for Deny assignments at the subscription and Management Group level using Azure CLI: `az role assignment list --include-deny`.

Verify the engineer is operating in the correct subscription context.

**Resolution:**

For propagation: Wait and retry.

For Deny assignment: Review whether the deny is legitimate. If it is a platform-controlled deny that should allow exceptions, create a scope-specific exemption — do not remove the deny.

For data plane access: Add the specific data plane role (e.g. `Storage Blob Data Contributor`) via PIM — Contributor does not grant data plane permissions on storage and Key Vault.

---

### Scenario 3: Service Principal Client Secret Expired — Pipeline Fails

**Symptoms:**

CI/CD pipeline fails with `AADSTS7000222: The provided client secret keys for app ... are expired`.

Deployments to all environments stop.

**Possible Causes:**

- Service Principal client secret passed its expiry date
- Secret was manually rotated in Entra ID but the new value was not updated in the pipeline

**Investigation:**

Check the Service Principal registration in Entra ID — review the Certificates & Secrets tab for expiry dates.

Confirm which pipeline variable or secret manager holds the credential.

**Resolution:**

Generate a new client secret in Entra ID.

Update the pipeline variable or Key Vault secret with the new value.

Trigger the pipeline to verify resolution.

**Long-term:** Migrate the pipeline to Workload Identity Federation, eliminating client secrets entirely. Set monitoring on all Service Principal secret expiry dates — alert 60 days before expiry.

---

### Scenario 4: Conditional Access Blocks Legitimate Azure Access

**Symptoms:**

An engineer cannot sign in to the Azure Portal.

Conditional Access shows the sign-in was blocked.

The block policy requires a compliant device.

The engineer is using a new corporate laptop not yet enrolled in Intune.

**Possible Causes:**

- Device compliance policy requires Intune enrollment
- New device has not completed Intune registration
- Sign-in attempt from a personal device

**Investigation:**

Check the Entra ID Sign-in Logs for the specific sign-in attempt. The Conditional Access result section shows which policy blocked access and which condition was not satisfied.

Confirm whether the device has completed Intune enrollment.

**Resolution:**

For unenrolled corporate device: Complete Intune enrollment and allow the compliance check to complete — typically 15–30 minutes.

For emergency access during an incident: Use a break-glass account that is excluded from the Conditional Access policy. Break-glass accounts exist specifically for scenarios where standard access paths are unavailable.

**Note:** Never modify the Conditional Access policy to resolve individual access issues. Break-glass accounts handle emergency scenarios. Policy exceptions undermine the security boundary.

---

## 9. Interview Questions

### Junior

**Q: What is the difference between authentication and authorisation in Azure?**

Authentication is the process of proving who you are. In Azure, Microsoft Entra ID handles authentication — it verifies your identity and issues a token. Authorisation is the process of determining what you are allowed to do. Azure RBAC handles authorisation — it checks your token against role assignments to determine whether the requested action is permitted. A user can successfully authenticate but be unauthorised to perform a specific operation. They are independent systems.

**Q: What is a Managed Identity?**

A Managed Identity is an identity in Microsoft Entra ID that is automatically created and managed by Azure. It is assigned to an Azure resource such as a Virtual Machine or App Service. The application running on that resource can obtain a token for the Managed Identity from the Azure Instance Metadata Service without any credentials in code. The application uses that token to access other Azure services — Key Vault, Storage, SQL — without ever handling a password or secret.

**Q: Why should applications not store secrets in configuration files?**

Configuration files frequently end up in source control, deployment logs, or environment variable dumps. A secret in source code is potentially visible to everyone with repository access and to anyone who finds the repository if it is ever made public. Secrets have a lifecycle — they expire, get compromised, need rotation. Managing them in configuration files creates coordination problems. The correct pattern is Managed Identity plus Key Vault — the application never holds a credential.

---

### Mid

**Q: When would you use a Service Principal instead of a Managed Identity?**

Service Principals are appropriate when the workload runs outside Azure — GitHub Actions runners, on-premises build agents, external tools that need Azure API access. Managed Identity only works for resources running in Azure that support it. For cross-tenant authentication where one Azure tenant's workload needs to access another tenant's resources, Service Principals with appropriate trust configurations are also used. In all other cases, Managed Identity is preferable because it eliminates credential management entirely.

**Q: How would you implement least privilege for an application team's pipeline that deploys infrastructure using Terraform?**

The pipeline runs on an Azure DevOps agent with a system-assigned Managed Identity. I identify the specific operations the Terraform configuration performs — creating Resource Groups, deploying VNets, creating App Services. I assign only the RBAC roles required for those operations, scoped to the specific subscription or Resource Group. Typically this means `Contributor` at the Resource Group scope rather than Subscription scope, plus `User Access Administrator` restricted to specific roles and scopes if the pipeline creates RBAC assignments. I review the assignments quarterly and remove any roles that are no longer required as the infrastructure matures.

**Q: What is PIM and why does it exist?**

PIM — Privileged Identity Management — provides just-in-time access to privileged roles. Instead of permanently holding a `Contributor` role on a production subscription, an engineer holds an eligible assignment. When they need to perform a privileged operation, they activate the role through PIM, providing MFA and a business justification, and receiving the role for a limited time. After the time window expires, the role is automatically removed. PIM exists because permanent standing access is a security risk — a compromised credential immediately grants an attacker whatever permanent roles that credential holds. PIM reduces the window during which a compromised identity can cause damage.

---

### Senior

**Q: Design an RBAC strategy for a platform serving 50 application teams in a regulated financial services organisation.**

The RBAC strategy is built on four principles: groups not individuals, minimum scope, PIM for privileged access, and automated lifecycle management.

All RBAC assignments use Entra ID security groups — never individual users. Group membership is managed through Entra ID access reviews, reviewed quarterly.

Groups per team follow a consistent naming convention: `AppTeam-{Name}-Reader`, `AppTeam-{Name}-Contributor`. The platform team defines the role assigned to each group type — Readers get `Reader` at the Resource Group scope, Contributors get `Contributor` scoped to their Resource Group only.

Subscription-level access is reserved for the platform team. Application teams never hold subscription-scope roles.

Production environment access for privileged operations — incident response, emergency deployments — uses PIM. Activation requires approval from the application team lead and the platform team on-call. Maximum activation duration is 4 hours for production.

The platform team holds `Owner` on subscriptions via PIM only. No permanent Owner assignments exist anywhere in the production Management Group.

Data plane roles — `Storage Blob Data Contributor`, `Key Vault Secrets User` — are assigned to Managed Identities of specific resources, not to humans. Humans use the control plane. Applications use data plane roles through Managed Identity.

**Q: Management wants to improve the company's Microsoft Secure Score from 45% to 80% within three months. How do you respond?**

I would explain that Secure Score is a measurement tool, not a business objective.

Targeting 80% without evaluating business context risks implementing recommendations that reduce operational effectiveness — for example, enabling just-in-time VM access in an environment where engineers need frequent low-latency VM access will generate more operational friction than security benefit.

My recommendation would be: audit the current recommendations and categorise them into three groups.

Group 1 — implement immediately: high-impact, low-disruption recommendations. MFA enforcement, removing unused accounts, enabling Defender plans. These likely move the score 15–20 points with minimal risk.

Group 2 — implement with planning: recommendations that require process change or testing. JIT VM access, removing legacy authentication. Schedule these with impacted teams, test in non-production first, communicate the change.

Group 3 — accept with documentation: recommendations that conflict with regulatory controls the organisation has addressed differently, or recommendations where the cost exceeds the risk reduction. Document each acceptance with justification.

The outcome is a realistic improvement in security posture — probably from 45% to 65–70% — achieved without creating operational incidents. The remaining gap is documented. This is more defensible to an auditor than reaching 80% by implementing changes without impact assessment.

---

### Principal

**Q: Describe how you would design an enterprise identity governance model for an organisation running 500 Azure applications across 40 teams, with high staff turnover.**

The governance model has five components.

**Provisioning:** Landing Zone provisioning automatically creates Entra ID groups for each application team — Reader and Contributor groups. Group membership is assigned by the team's manager in Workday, which syncs to Entra ID via SCIM. Engineers get Azure access the day they join the team by being added to the group. No manual RBAC assignments.

**Access reviews:** Quarterly access reviews are triggered through Entra ID Identity Governance for all groups holding Contributor or higher roles on production subscriptions. Each review is assigned to the team manager, who must certify each member's continued need. Certifications that are not completed within 14 days automatically revoke access. This prevents the accumulation of stale access from departures and role changes.

**Privileged access lifecycle:** PIM manages all production privileged access. Eligibility is reviewed annually. Eligibility is removed when engineers change teams. No production Owner assignment exists permanently — the platform team holds Owner eligibility via PIM, required for subscription-level operations.

**Service identity lifecycle:** Service Principals are registered with an owner. Owners receive 60-day expiry alerts. Service Principals without logins in 90 days are automatically flagged for decommission. Workload Identity Federation is the standard for new pipelines — secrets are eliminated.

**Automated deprovisioning:** When an account is disabled in Workday — departure, extended leave — the Entra ID connector disables the account within 4 hours. All active PIM activations are revoked. Group memberships are not removed immediately — they are queued for the next access review to allow the account to be re-enabled if the departure was in error.

At high staff turnover, the system cannot rely on humans to revoke access. Every step must be automated and time-bounded. The humans in the process are approvers and reviewers, not provisioners.

**Q: How does Workload Identity Federation change the Service Principal security model, and where should it be prioritised in an enterprise migration?**

Workload Identity Federation allows an external identity system — GitHub Actions, Azure DevOps, Kubernetes service accounts — to exchange a token from that system for an Entra ID token, without any shared secret.

The trust relationship is established by the platform team: a Service Principal is configured to trust tokens from a specific GitHub repository and workflow, or a specific Kubernetes service account in a specific namespace.

When the pipeline runs, GitHub generates a short-lived OIDC token. The pipeline presents this to Entra ID and exchanges it for an Azure access token. No client secret exists. No credential needs rotating. No secret needs storing in a pipeline variable.

Prioritisation in an enterprise migration:

**Highest priority:** Pipelines with broad Azure permissions — Terraform deployments, infrastructure automation. These are the highest-value targets for an attacker. Eliminating secrets from them first reduces the most significant credential exposure risk.

**Second priority:** Service Principals with secrets that have never been rotated or have long expiry periods. These are the most likely to be stale and most likely to be compromised without detection.

**Lower priority:** Service Principals for internal integrations between Azure services — these should be migrated to Managed Identity rather than Workload Identity Federation.

The migration itself is low-risk. Creating a federation credential on an existing Service Principal does not affect the existing secret — both work during the transition. After validating the federated authentication, the secret is revoked. Zero downtime migration with no credential regeneration required.

---

## 10. Interactive Exercise

### Architectural Challenge

A financial services company is preparing for a regulatory audit in six months.

The auditors will assess identity, access management, and security posture.

Pre-audit findings include:

1. 22 engineers have permanent `Contributor` access to the production subscription.
2. 9 applications use connection strings stored in `appsettings.json` files committed to a private Git repository.
3. 4 CI/CD pipelines authenticate to Azure using Service Principal secrets that expire in 30 days and have not been rotated in 18 months.
4. No multi-factor authentication is enforced for Azure Portal access.
5. Microsoft Secure Score is 41%.
6. A developer was recently terminated and their access was revoked from Azure AD but their Service Principal credentials — which they had personal knowledge of — were not rotated.

**Your task:**

Prioritise the six findings in the order you would address them. Justify the order.

For each finding, describe the specific remediation action the platform team should take.

Explain which findings represent immediate risk versus long-term structural problems.

Explain how automation prevents these findings from recurring in future.

Identify any finding where the remediation might introduce operational risk and explain how you would manage that risk.

---

*There is no single correct answer.*

*Evaluation focuses on: risk prioritisation reasoning, depth of remediation design, understanding of automation to prevent recurrence, and recognition of operational impact during remediation.*
