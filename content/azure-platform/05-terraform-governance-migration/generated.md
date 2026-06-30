---
Title: Enterprise Terraform, Governance and Cloud Migration
Domain: Azure Platform Engineering
Difficulty: Senior
Estimated Duration: 180 minutes
Prerequisites:
  - Module 01 — Enterprise Azure Platform
  - Module 04 — Enterprise Identity and Security
  - Basic Terraform knowledge (variables, outputs, modules)
  - Basic Azure knowledge (VMs, App Service)
Learning Objectives:
  - Design reusable Terraform modules for platform teams
  - Explain module versioning and repository organisation strategies
  - Describe remote state management and state isolation patterns
  - Explain Azure Policy as a governance instrument, not just compliance
  - Design naming and tagging standards that scale to hundreds of applications
  - Compare Lift-and-Shift versus Replatform migration strategies
  - Explain technical debt decisions during migration
  - Design a Cloud Center of Excellence capable of onboarding teams rapidly and safely
Interview Focus:
  - Senior Platform Engineer
  - Principal Platform Engineer
---

# Module 05 — Enterprise Terraform, Governance and Cloud Migration

---

## 1. Learning Goal

After completing this module you will be able to design enterprise Terraform architecture that scales to hundreds of applications without duplicated code.

You will understand how governance through Azure Policy, naming standards, and tagging enables platform teams to enforce consistency without becoming a bottleneck.

You will be able to evaluate migration scenarios and justify Lift-and-Shift versus Replatform decisions based on business context and technical considerations.

You will be prepared to describe Cloud Center of Excellence operations and responsibilities during a Principal Engineer interview.

---

## 2. Why This Exists

### The Problem with Ad-Hoc Terraform

In early cloud adoption, each team writes their own Terraform code.

Team A deploys an App Service with 50 lines of Terraform.

Team B deploys an App Service with 75 lines of Terraform.

Team C deploys an App Service with 45 lines of Terraform.

All three do similar things but structured differently.

Naming conventions are inconsistent — some use `prod-appname-app`, others use `appname-production`.

Tagging is optional — some resources have cost centre tags, others do not.

RBAC assignments are different — some teams request Contributor scope, others request Reader plus specific role assignments.

When an audit finds that 40% of resources have missing tags, fixing this requires 300 individual Terraform configuration changes across 40 teams.

When security policy changes — for example, Key Vault now requires private endpoints — updating 300 configurations to align with the new policy takes months.

When a team wants to provision infrastructure, they do not know what naming convention to use. They guess.

### What Platform Teams Must Build

The platform team builds reusable Terraform modules.

A module is a directory containing Terraform code that deploys a logical resource unit — an App Service with all required configuration, a Virtual Machine with all baseline settings, a Virtual Network with standard subnets.

Modules define inputs — variables that allow a consuming team to customise behaviour without modifying code.

Modules define outputs — values like the resource ID or the public endpoint that other modules need.

Governance standards — policies, naming, tagging, RBAC — are baked into modules.

When a platform team updates a module, consuming teams upgrade their module version to get the governance update automatically.

### What Changes in Practice

With reusable modules:

Team A provisions an App Service by calling the platform team's `app_service` module. The App Service is deployed with standard tagging, correct naming, required RBAC assignments, and monitoring configuration. The team provides a 5-variable configuration.

Team B provisions an App Service identically.

Team C provisions an App Service identically.

An audit finds that 100% of resources have correct tags — because tagging is in the module, not the team's responsibility.

A security policy changes — private endpoints required for Key Vault. The platform team updates the Key Vault module once. All teams that upgrade get the new policy automatically.

---

## 3. Core Concepts

### Terraform Module Structure

A module is a directory containing Terraform code with a specific structure:

```
app-service-module/
  ├── main.tf           # Resource definitions
  ├── variables.tf      # Input variables
  ├── outputs.tf        # Output values
  ├── locals.tf         # Local computed values
  ├── terraform.tf      # Provider requirements
  └── README.md         # Documentation
```

**Key principle:** A module encapsulates complexity.

The consuming team should provide 5–10 variables and get a fully configured resource with governance standards applied.

If a module requires 40 variables, it is overengineered.

If a module accepts a single variable, it is too rigid.

### Module Versioning

Modules are versioned.

When a platform team updates a module — fixing a bug, adding a feature, implementing a new policy — the version number is incremented.

Consuming teams explicitly request a version:

```hcl
module "app_service" {
  source = "git@github.com:platform-team/terraform-azure-app-service.git?ref=v2.3.0"
  
  app_name  = var.application_name
  location  = var.location
  ...
}
```

Versioning provides:

- **Stability:** Teams control when they upgrade. They do not get breaking changes automatically.
- **Auditability:** Every deployed infrastructure references a specific module version. Compliance audits can trace what standard was used.
- **Rollback:** If a module version introduces a bug, teams revert to the prior version immediately.

Semantic versioning is standard:

- `v1.0.0` — initial release
- `v1.1.0` — minor feature addition (backwards compatible)
- `v2.0.0` — breaking change requiring code updates

### Remote State

Terraform state is stored in a remote backend — typically Azure Blob Storage with proper access controls and locking.

Local state is used only for development.

Production state is always remote and protected.

**Remote state provides:**

- **Team access:** Multiple team members can work on the same Terraform configuration. The state is shared, not local to one person's laptop.
- **CI/CD integration:** Automation pipelines can read and modify state. No manual `terraform apply` commands needed.
- **Locking:** Azure Blob Storage with locking prevents concurrent modifications. If two pipelines attempt to apply simultaneously, one is locked out and fails cleanly.
- **Backup:** Blob Storage handles replication and disaster recovery. Lost local state means lost track of deployed resources. Lost blob-backed state is recoverable.

### State Isolation

A critical principle: each Landing Zone subscription gets its own state file.

One state file per subscription. Never shared.

This provides:

- **Blast radius isolation:** If a Terraform error occurs in subscription A, it cannot corrupt state in subscription B.
- **RBAC alignment:** Access to state maps to RBAC scope. The platform team manages subscription A's state. Application Team A manages only their subscription A state.
- **Performance:** Large state files slow down operations. Separating state by subscription keeps files manageable.

State isolation requires discipline.

If a Terraform configuration spans multiple subscriptions — deploying to Subscription A and Subscription B in a single configuration — the state must be split into separate Terraform configurations with separate state files, one per subscription.

### Repository Organisation

**Monorepo vs Separate Repos:**

**Separate repos** — each module is a separate Git repository.

```
platform-team/terraform-azure-app-service
platform-team/terraform-azure-virtual-machine
platform-team/terraform-azure-vnet
```

Pro: Clear boundaries. Teams can version modules independently.

Con: Managing 40 repositories is overhead. Shared utility functions require coordination.

**Monorepo** — all modules in one repository in separate directories.

```
terraform-azure-modules/
  ├── modules/
  │   ├── app-service/
  │   ├── virtual-machine/
  │   └── vnet/
  ├── governance/
  │   ├── policies/
  │   └── standards/
```

Pro: Easier governance. One PR process. Easier to refactor across modules.

Con: Must version the entire monorepo. If the app-service module needs a breaking change, the version affects all modules.

**Recommendation for enterprise:** Monorepo for the platform team's modules. Clear subdirectories, one CI/CD pipeline, semantic versioning of releases.

### Naming Standards

Naming standards must be:

- **Deterministic:** Given a set of inputs, the name is always the same
- **Parseable:** Engineers can identify resource purpose from the name
- **Concise:** Azure has resource name length limits (usually 24–63 characters)
- **Enforceable:** Azure Policy can validate names match the pattern

Standard pattern:

```
{environment}-{application}-{resource-type}

Examples:
prod-billing-app
prod-billing-db
prod-billing-kv
staging-billing-app
```

Naming standards are enforced through:

1. Terraform modules include a local that constructs the name
2. Consuming teams cannot override the name
3. Azure Policy validates resource names match the approved pattern

### Tagging Standards

Every resource must have a minimum set of tags:

| Tag | Purpose | Values |
|---|---|---|
| Environment | Deployment target | prod, staging, dev |
| Application | Owning application | billing, hr, inventory |
| CostCentre | Cost allocation | accounting centre code |
| Owner | Team contact | team email |
| Lifecycle | Resource purpose | permanent, temporary, experimental |

Tags are applied by:

1. Terraform modules set default tags via `local_tags`
2. Consuming teams can add application-specific tags
3. Azure Policy requires all standard tags to be present

A policy rule might be:

```
Deny creation of resources without Environment and CostCentre tags
```

This is enforced at the API level, not through audit reviews.

### Azure Policy for Governance

Azure Policy is not primarily for compliance.

It is the mechanism through which platform teams enforce standards.

**Compliance** is a byproduct.

Examples of platform team policies:

- Require private endpoints for all Key Vaults and Storage Accounts
- Deny creation of VMs without Managed Identity
- Deny resources in unapproved regions
- Require specific tags on all resources
- Deny public IP assignment to VMs in production subscriptions

These are not "compliance checks."

These are architecture standards enforced at the API level.

A resource that violates the policy cannot be created.

Teams do not provision first and remediate later.

---

## 4. Architecture

### Enterprise Terraform Organization

```
Cloud Center of Excellence
│
├── Reusable Modules (monorepo)
│   ├── app-service/
│   ├── virtual-machine/
│   ├── vnet/
│   ├── key-vault/
│   └── storage/
│
├── Governance
│   ├── Azure Policy Definitions
│   ├── Initiative Assignments
│   └── Naming & Tagging Standards
│
└── Migration Enablement
    ├── Migration Assessment Tools
    ├── Migration Playbooks
    └── Runbooks
```

### State Management Architecture

```
Platform Team — Connectivity Subscription
  └── State Storage Account
        ├── terraform-hub.tfstate
        ├── terraform-connectivity.tfstate
        └── Locked (no application teams can modify)

Application Team A — Landing Zone Subscription
  └── State Storage Account
        ├── terraform-app-resources.tfstate
        └── Access: Application Team A only

Application Team B — Landing Zone Subscription
  └── State Storage Account
        ├── terraform-app-resources.tfstate
        └── Access: Application Team B only
```

### Reusable Module Flow

```
Platform Team publishes module v2.3.0
        ↓
Module stored in Git repository with tag v2.3.0
        ↓
Application Team A references v2.3.0 in their configuration
        ↓
Application Team A runs 'terraform init'
  Git fetches the specific version from the repository
        ↓
Module code is evaluated with Team A's variables
        ↓
Resources are deployed with governance standards
  (naming, tagging, RBAC applied by module)
```

### Platform Ownership Model

```
Platform Team owns:
┌──────────────────────────────────────────────────────────┐
│  Reusable module development and maintenance             │
│  Module versioning and releases                          │
│  Azure Policy definitions and assignments                │
│  Naming and tagging standards                            │
│  Remote state infrastructure                             │
│  Landing Zone provisioning (subscription setup)          │
│  Baseline security policies on all subscriptions         │
└──────────────────────────────────────────────────────────┘

Application Team owns:
┌──────────────────────────────────────────────────────────┐
│  Their application infrastructure code                   │
│  Variables specific to their application                 │
│  Testing of their infrastructure configuration           │
│  Terraform apply for their subscription                  │
│  Application-specific resource configuration             │
└──────────────────────────────────────────────────────────┘

Application Team does NOT own:
┌──────────────────────────────────────────────────────────┐
│  Module development                                      │
│  Naming standards                                        │
│  Tagging standards                                       │
│  Azure Policy modifications                              │
│  Cross-subscription connectivity                         │
│  Subscription-level RBAC                                 │
└──────────────────────────────────────────────────────────┘
```

---

## 5. Design Decisions

### When to Build Reusable Modules

**Build a module when:**

- The same resource pattern is deployed by multiple teams
- Governance standards must be applied consistently
- The deployment is complex enough that documenting the "correct way" has value
- Teams consume the module for at least 6 months before significant changes

**Do not build a module when:**

- Only one team will use it (use a team-specific configuration)
- The resource is trivial — a single storage account might not need a module
- The module would require 30+ variables to be flexible enough

### Monorepo vs Separate Repositories — Final Decision

For a Cloud Center of Excellence serving 40–50 teams:

**Use a monorepo.**

Each module is a subdirectory.

Release the entire monorepo as `v2.3.0`.

Benefits:

- One PR process enforces consistency across all modules
- Shared utilities (locals for naming, tagging helpers) are easy to maintain
- Governance standards are defined once
- Testing is centralised
- One CI/CD pipeline

The trade-off:

If one module needs a breaking change, the entire monorepo gets a major version bump.

In practice, this is acceptable because all modules are updated together.

Teams upgrading from v2.3.0 to v3.0.0 review all module changes at once, not individually.

### State Isolation Strategy

State is isolated per subscription.

Each subscription gets a Terraform configuration with its own state file.

For the connectivity subscription (Hub):

```
terraform/
  ├── hub/
  │   ├── main.tf
  │   ├── backend.tf (state stored in hub-state-storage-account)
  │   └── terraform.tfvars
```

For each application subscription:

```
terraform/
  ├── app-team-a/
  │   ├── main.tf
  │   ├── backend.tf (state stored in app-team-a-state-storage-account)
  │   └── terraform.tfvars
```

Never try to manage both in a single Terraform configuration.

### Naming and Tagging Trade-Offs

Strict naming standards ensure consistency but reduce flexibility.

A team might want to name a resource `prod-billing-internal-app` but the standard is `prod-billing-app`.

The rule: the name is generated by the module based on a small set of inputs.

Teams do not name resources directly.

This eliminates inconsistency and removes the need for human review.

Tagging is less strictly controlled.

Standard tags are enforced.

Teams can add custom tags for their specific needs.

Example:

```hcl
local_tags = {
  Environment = var.environment    # enforced
  Application = var.application    # enforced
  CostCentre  = var.cost_centre    # enforced
  Owner       = var.team_email     # enforced
  Project     = var.project_code   # optional, team-specific
}
```

### Azure Policy — Deny vs Audit vs Modify

**Deny effect:**

Prevents resource creation if the policy condition is not met.

Applied to: critical standards (required tags, approved regions, private endpoints).

Tribes: non-negotiable. Resources that violate cannot be created.

**Audit effect:**

Logs non-compliance but does not prevent creation.

Applied to: emerging standards during transition periods.

Usage: teams are warned but not blocked. After 6 months of audit data, convert to Deny effect.

**Modify effect:**

Automatically modifies resources to comply.

Applied to: tagging standards — if a resource is created without the required tags, the policy adds them automatically.

Usage: reduces operational burden. Teams do not need to remember to tag; the policy tags automatically.

---

## 6. Real World Scenario

A retail company is migrating 150 applications from on-premises to Azure over 18 months.

Applications are a mix of Windows Server IIS applications, Java services, and .NET web applications.

The CTO has mandated:

- Infrastructure as Code only — no manual portal deployments
- Consistent governance — naming, tagging, RBAC across all subscriptions
- Self-service provisioning — teams should not depend on the platform team for every deployment
- Cost visibility — every resource tagged with cost centre for chargeback

The platform team designs and implements the following:

**Reusable Modules:**

The platform team builds six core modules:

1. `vnet` — Virtual Network with standard subnets
2. `app-service` — App Service with DNS, monitoring, RBAC configured
3. `virtual-machine` — VM with Managed Identity, monitoring, baseline security
4. `database` — Azure SQL Database with private endpoint, backup policy
5. `storage` — Storage Account with private endpoint, encryption, lifecycle policy
6. `keyvault` — Key Vault with RBAC, audit logging, private endpoint

Each module has:

- Built-in naming standards — teams provide only `environment`, `application`
- Built-in tagging — environment, application, cost centre, owner, lifecycle
- RBAC assignments — Managed Identity for VMs/App Services scoped to required resources
- Monitoring — diagnostic settings pre-configured

**Repository Structure:**

Monorepo: `terraform-azure-platform`

```
terraform-azure-platform/
  ├── modules/
  │   ├── vnet/
  │   ├── app-service/
  │   ├── virtual-machine/
  │   ├── database/
  │   ├── storage/
  │   └── keyvault/
  ├── governance/
  │   ├── naming-standards.md
  │   ├── tagging-standards.md
  │   └── azure-policies/
  └── examples/
      ├── app-service-basic/
      └── vm-with-database/
```

**Governance Standards:**

Azure Policy initiative assigned at the Landing Zones Management Group level:

- Require tags: Environment, Application, CostCentre, Owner
- Deny resources in unapproved regions (only westeurope, eastus, canadacentral)
- Deny public IP assignment to VMs in production subscriptions
- Require private endpoints for Key Vault and Storage Accounts
- Deny resources without Managed Identity when applicable
- Audit: deny unencrypted storage accounts (transitioned to Deny after 6 months)

**State Management:**

Platform team provisions a central state storage account in the connectivity subscription.

Each Landing Zone subscription is provisioned with its own state storage account.

Platform team provides a backend configuration template teams use:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stgtfstate{subscription_id}"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}
```

**Onboarding Process:**

1. Business requests a new application environment
2. Platform team provisions the Landing Zone subscription
3. Platform team provides the application team with a template Terraform configuration:

```hcl
module "app_service" {
  source = "git@github.com:platform-team/terraform-azure-platform.git//modules/app-service?ref=v1.2.0"
  
  environment   = "prod"
  application   = "myapp"
  location      = "westeurope"
  cost_centre   = "CC-1234"
  owner_email   = "team@company.com"
  
  service_plan_sku = "P1V2"
  always_on        = true
}
```

4. Application team customises the configuration for their needs
5. Application team submits a PR to their subscription's Terraform repository
6. Pipeline validates the configuration — `terraform plan` succeeds without policy violations
7. Pipeline applies automatically or requires manual approval depending on environment
8. Infrastructure is deployed with all governance standards applied

**Migration Process:**

For legacy IIS applications:

1. Assessment: the platform team determines whether the application is suitable for App Service or requires Virtual Machines
2. If App Service is suitable: migrate to `app-service` module
3. If VM is required: deploy using `virtual-machine` module with Windows Server OS
4. Either way, the migration does not duplicate code — applications consume platform modules

**Outcome:**

150 applications are migrated over 18 months.

Every application's infrastructure:

- Follows identical naming conventions
- Has correct tags (no manual remediation needed)
- Has appropriate RBAC assignments
- Has monitoring and logging enabled
- Is compliant with all Azure Policies

An audit six months in finds 100% compliance — tags, naming, regional restrictions, network isolation.

Cost chargeback works because tagging is mandatory and consistent.

---

## 7. Common Mistakes

### Mistake 1: Designing Modules Around a Single Project

A platform engineer builds a Terraform module for a specific customer project.

The module works perfectly for that project.

When a second team tries to use it, they need to modify the module to fit their requirements.

**Why it is wrong:** Modules must be designed for reuse. The first customer is not representative. If a module only works for one use case, it is a project configuration, not a reusable module. The platform team should observe at least two customers before finalising module design.

### Mistake 2: Modules with Excessive Variables

A module requires 45 input variables to accommodate every possible configuration variation.

The module is "flexible."

Documentation is 12 pages.

Consuming teams spend days understanding which variables to set.

**Why it is wrong:** Modules should have 5–15 variables. Exceeding this indicates the module is trying to handle too many variations. Break it into multiple modules or provide opinionated defaults. Consuming teams should be able to understand the module in 15 minutes, not 2 hours.

### Mistake 3: Storing State Locally or in Multiple Places

A team stores Terraform state in Git.

Another team stores state on an engineer's laptop.

A third team stores state in one Azure storage account.

**Why it is wrong:** Inconsistent state storage causes drift, prevents CI/CD integration, and loses audit history. State must be remote, consistent, and protected. One state backend per subscription in a managed storage account with locking and RBAC controls.

### Mistake 4: Using Azure Policy for Audit Only

Azure Policy is configured in Audit mode.

Policies capture violations but do not prevent them.

The platform team reviews policy reports quarterly.

By then, 500 resources have been deployed in violation of standards.

**Why it is wrong:** Audit-only policies are signals that standards have not been defined, not mechanisms for enforcement. Once a standard is defined, enforce it with a Deny policy. Audit mode should be brief — during a transition period — not permanent.

### Mistake 5: Treating Lift-and-Shift as "Rehost Everything"

A company has 500 Windows Server applications running on-premises.

The CTO mandates "Lift-and-Shift migration."

Every application is moved to Azure Virtual Machines — unchanged.

Three years later, the Azure footprint is identical to the on-premises environment.

No modernisation. No cost reduction. Only hosting costs moved to Azure.

**Why it is wrong:** Lift-and-Shift does not mean "never improve." After moving to Azure, application teams should be encouraged to evaluate modernisation opportunities — is this IIS application suitable for App Service? Can this monolith be containerised? Lift-and-Shift is the initial strategy to reduce deployment time. It should not be the endpoint of modernisation.

### Mistake 6: Ignoring Operational Ownership After Deployment

Terraform deploys the resource.

It works.

Three months later, Azure updates the storage account SKU to a premium tier.

The change is manual — not in Terraform.

When infrastructure is re-deployed, the SKU is reset to the original standard tier.

**Why it is wrong:** Terraform state is the source of truth. Manual changes are undone on the next apply. This creates a "human vs machine" conflict. Teams must understand: if it is in Terraform, do not change it manually. If manual changes are required, update Terraform to reflect the change. Operational ownership means maintaining the Terraform configuration as the system of record.

---

## 8. Troubleshooting

### Scenario 1: Terraform State Lock Blocks Deployment

**Symptoms:**

A pipeline attempts to deploy and fails immediately.

Error: `Error acquiring the lease with ID "..."` or `Another operation on this state is in progress`.

No other team is running `terraform apply`.

**Possible Causes:**

- A previous `terraform apply` did not complete and is still holding the lock
- A developer ran `terraform apply` locally and abandoned the session
- The Azure Blob Storage lease is stale from a crashed process

**Investigation:**

Check whether any ongoing `terraform apply` commands exist — in pipelines or local sessions.

Check Azure Blob Storage activity logs for the state file — look for lease acquisition/release patterns.

If the lock has been held for hours and no `terraform apply` is running, the lease is likely stale.

**Resolution:**

If a pipeline is hung: cancel the pipeline run, which should release the lock after 30 seconds.

If a local session left a stale lock: run `terraform force-unlock <lock-id>` with the lock ID from the error message. This forcefully releases the lock.

To prevent: configure state backend with a reasonable operation timeout and ensure CI/CD pipelines have timeouts that force cleanup.

---

### Scenario 2: Module Update Breaks Consuming Team's Configuration

**Symptoms:**

The platform team releases module v2.0.0 with a breaking change.

An application team upgrades their module reference to v2.0.0.

`terraform init` succeeds.

`terraform plan` fails with validation errors.

The module now requires a new mandatory variable that the team did not provide.

**Possible Causes:**

- Insufficient communication about breaking changes
- The team's CI/CD pipeline does not validate module upgrades before applying
- The breaking change was necessary and documented but not communicated to teams using the module

**Investigation:**

Review the module's CHANGELOG documenting the breaking change.

Identify which variable is missing in the team's configuration.

Check whether the team was notified before the release.

**Resolution:**

Communicate the breaking change clearly in release notes.

The consuming team adds the required variable to their configuration.

For future releases, the platform team should deprecate functionality before removing it — provide two versions with warnings before removing in v3.0.0.

---

### Scenario 3: Policy Prevents Legitimate Resource Deployment

**Symptoms:**

Application team attempts to deploy a resource using a platform module.

Terraform plan succeeds.

`terraform apply` fails.

Azure Policy rejection: "Policy violation: resource deployed in unapproved region."

The team is sure they specified the correct region.

**Possible Causes:**

- Module generates a resource in an unintended region due to default values
- The region specified in `var.location` is overridden by a computed local value
- Azure Policy is correctly preventing deployment in a region that should not allow the resource

**Investigation:**

Review the module's code — check the computed region in `main.tf`.

Review the policy definition — which regions are approved?

Confirm the module's `var.location` input variable is actually being used or if a default is overriding it.

**Resolution:**

If the module has a bug: fix the module to respect the `location` variable and release v2.1.0.

If the region is legitimately unapproved: document why and request a policy exception from the platform team.

Do not disable the policy — exceptions should be granular and limited.

---

### Scenario 4: State Drift — Manual Changes Not in Terraform

**Symptoms:**

Terraform state shows a resource with specific configuration.

The Azure Portal shows the same resource with different configuration.

Running `terraform plan` on a deployed resource shows changes that will revert the manual modification.

**Possible Causes:**

- An engineer manually changed the resource in the Azure Portal
- A different automation system modified the resource outside of Terraform
- A policy automatically modified the resource (e.g., Modify effect adding tags)

**Investigation:**

Check the resource's Activity Log in the Azure Portal for who made the manual change and when.

Check whether any other automation systems have permissions to modify this resource.

Review Azure Policy assignments — if a Modify policy is applied, the policy may have updated the resource.

**Resolution:**

If the manual change was an error: revert in the Portal and re-run `terraform apply` to confirm state is correct.

If the manual change is intentional: update the Terraform configuration to match and commit the change. `terraform apply` again to ensure state is correct.

If a policy made the change: confirm the policy is correct. The policy is right; manual configuration is wrong. Update Terraform to match what the policy created.

**Prevention:** enforce the rule: "If it is in Terraform, do not change it manually." Implement this through access controls — teams have Read on the Portal but Contributor only through Terraform CI/CD pipelines.

---

## 9. Interview Questions

### Junior

**Q: What is a Terraform module and why would a platform team build one?**

A Terraform module is a reusable package of Terraform code that deploys a logical resource unit. A platform team builds modules to standardise how resources are deployed across multiple teams. Instead of each team writing their own App Service deployment code, the platform team builds one App Service module. All teams use that module. It ensures consistency — naming, tagging, RBAC, monitoring — without requiring each team to implement those standards independently.

**Q: What is the difference between local and remote state?**

Local state is stored on an engineer's laptop in a `terraform.tfstate` file. It is not shared. Remote state is stored in a centralised location — Azure Blob Storage — accessible to all team members and CI/CD pipelines. Remote state is required for teams to work together. Local state should only be used for development and experimentation, never for production infrastructure.

**Q: What is semantic versioning for Terraform modules?**

Semantic versioning has three numbers: `MAJOR.MINOR.PATCH` — e.g. `v1.2.3`. PATCH version increments for bug fixes. MINOR increments for backwards-compatible features. MAJOR increments for breaking changes. A team using module v1.2.3 can safely upgrade to v1.2.4 or v1.3.0 without code changes. Upgrading to v2.0.0 may require configuration changes.

---

### Mid

**Q: How would you structure Terraform code and state management for an enterprise with 40 application teams?**

Each of the 40 teams has a Landing Zone subscription. Each subscription gets its own state file stored in a dedicated Azure storage account. The state file is isolated — Application Team A cannot access Team B's state.

Reusable modules are stored in a central monorepo owned by the platform team, versioned and released together.

Each team's Terraform configuration references the module versions they consume:

```hcl
module "app_service" {
  source = "git@github.com:platform/modules.git//app-service?ref=v1.2.0"
}
```

CI/CD pipelines run `terraform plan` to validate changes before `terraform apply` deploys.

The platform team updates modules. Teams upgrade module references at their pace. State isolation ensures no team's infrastructure is affected by another team's Terraform execution.

**Q: An audit finds that 300 resources are missing required tags. How would you prevent this from recurring using Terraform and Azure Policy?**

First: implement tagging in all reusable modules. Every module includes a `local_tags` block that enforces required tags. Teams do not set tags — the module does.

Second: implement an Azure Policy with Modify effect that automatically adds required tags to resources created without them.

Third: implement a Deny policy that prevents resource creation without required tags — as a backstop.

The combination means: modules enforce tags, policy enforces tags, and new resources created outside the module are forced into compliance by policy. The audit three months later finds 100% compliance.

**Q: Compare Lift-and-Shift versus Replatform for a legacy IIS application in an enterprise migration context.**

Lift-and-Shift moves the application to Azure Virtual Machines unchanged. Fastest deployment. Lowest technical risk. Long-term cost is high because you are running the same infrastructure Azure was designed to replace.

Replatform converts the application to a new platform — IIS to App Service. Higher upfront effort. More technical risk during the conversion. Lower long-term cost because App Service is cheaper and more operationally efficient than VMs.

For a 500-application migration, the strategy is: Lift-and-Shift for applications where risks of replatforming are too high, Replatform for applications where the cost savings justify the effort.

Criteria: if the application is a standard web application with no exotic dependencies, replatform. If the application has custom components, specific OS dependencies, or is rarely modified, lift-and-shift.

---

### Senior

**Q: Design a Terraform strategy for a company planning a 400-application migration over 24 months, starting with zero cloud infrastructure.**

The strategy has three layers:

**Layer 1 — Platform Foundation (Months 1–3):**

The platform team builds the core Cloud Center of Excellence. They design and implement:

- Landing Zone subscription provisioning automation
- Six reusable modules covering the most common deployment patterns: App Service, Virtual Machine, Database, Virtual Network, Storage, Key Vault
- Azure Policy framework with policies for naming, tagging, regional restrictions, private endpoints, Managed Identity
- State management infrastructure in the connectivity subscription
- CI/CD pipeline templates teams use for deploying infrastructure

**Layer 2 — Pilot Applications (Months 4–6):**

5–10 application teams migrate using the platform modules. Feedback is collected. Modules are refined. Teams validate that the platform reduces deployment time and enforces governance automatically.

**Layer 3 — Scale (Months 7–24):**

Remaining 390 applications migrate in cohorts. Each team consumes the platform modules. New module needs are identified and added to the platform team's roadmap. Module updates roll out to all teams as new versions.

**Technical decisions:**

Monorepo for modules — all modules versioned together and released as a unit.

Semantic versioning strictly enforced — breaking changes require MAJOR version bumps and team communication.

State isolation — each Landing Zone subscription has its own state file. No shared state.

Policy-as-code — all policies stored in Git, peer-reviewed, versioned.

Modules are "opinionated" — they enforce standards, not preferences. If a team needs customisation, they request it from the platform team, not branch modules.

After 24 months, 400 applications are deployed from 40 different teams using consistent infrastructure, consistent naming, consistent tagging, 100% compliance with policies.

**Q: An application team requests an exception to an Azure Policy because it conflicts with their application requirements. How would you evaluate the request?**

I would follow a decision tree:

1. **Is the policy correct?** Verify that the policy standard is still justified. If standards have changed, update the policy. Do not grant exceptions to correct policies.

2. **Is the requirement legitimate?** Understand why the application needs the exception. "We want to bypass the policy" is not a requirement. "Our application requires IP access control at the resource level instead of private endpoints because of legacy integrations" is a requirement.

3. **Is there an alternative?** Could the requirement be satisfied differently? The team requests `AllowPublicAccess: true` on a Key Vault. The alternative: use Private Endpoints and manage firewall rules through the policy exceptions. Same outcome, maintains the security boundary.

4. **Is the risk acceptable?** If the exception is approved, what is the blast radius? An exception on one Key Vault is low risk. An exception disabling private endpoints on all storage accounts is high risk.

If all four questions are answered satisfactorily, grant a scoped exception:

- Specific scope: only this Key Vault, not all Key Vaults
- Limited duration: review the exception in 6 months
- Documented: the business requirement is logged
- Audited: the exception is logged in the policy exception records

Never disable the policy. Never grant broad exceptions. Never make exceptions permanent without review.

---

### Principal

**Q: Design the operating model of a Cloud Center of Excellence managing a Terraform module platform for a Fortune 500 company with 10,000 engineers across 100 business units.**

The operating model has three components: **governance**, **operations**, and **evolution**.

**Governance Component:**

A 10-person platform team owns the Terraform module platform. They define:

- Reusable modules (60–80 modules across common Azure resource types)
- Azure Policy framework (100+ policies defining organisational standards)
- Naming and tagging standards (consistent across 10,000 engineers)
- Change control process (all module changes go through peer review and staged rollout)

The governance model is "policy as the contract." Every module and policy is stored in Git. Every change is a PR. Every PR requires approval. No ad-hoc changes.

**Operations Component:**

Three operational tiers:

**Tier 1 — Self-service:** Teams use modules to deploy standard resources. 80% of deployments are tier 1. Modules handle 95% of the engineering work. Teams provide 5–10 variables and infrastructure is deployed with governance enforced.

**Tier 2 — Platform team support:** 15% of deployments require platform team involvement. Teams need a new module, or a custom configuration outside the platform's standard patterns. Platform team evaluates the request — is this a one-off or a pattern? If a pattern, a new module is designed. If one-off, the platform team provides a custom configuration.

**Tier 3 — Infrastructure engineering:** 5% of deployments involve low-level networking, complex cross-tenant connectivity, or on-premises hybrid scenarios. A dedicated infrastructure engineering team — separate from the platform team — handles these.

The 10-person platform team is sized such that tier 2 requests do not accumulate. Response time for "platform team, please review this request" is 1–2 business days. Teams are not blocked waiting for platform decisions.

**Evolution Component:**

The platform team dedicates 30% of capacity to long-term evolution:

- Quarterly module reviews — do modules meet team needs?
- Technology evaluation — new Azure services or Terraform patterns that should be adopted
- Policy updates — compliance requirements or security threats that require new policies
- Automation improvements — CI/CD pipeline efficiency, state management, cost tracking

Module updates are communicated 30 days in advance. Major version updates require team effort to upgrade. This is announced with impact analysis. Teams plan upgrades during their sprint planning. The platform team provides migration support.

Breaking changes happen once per year maximum. Non-breaking updates happen every quarter.

**Success Metrics:**

- 99%+ of production infrastructure deployed through the platform (vs. off-platform, non-standard deployments)
- <5% of deployments fail policy validation (initial deployments)
- <2 week average deployment time from approval to production
- 100% audit compliance on naming, tagging, security policies
- <30 minute mean time to deploy a new team's infrastructure
- 95% of deployments use module versions <6 months old (not stale)

At 10,000 engineers, this operating model scales because humans are removed from the deployment process. Governance is enforced by policy and modules, not by people reviewing every deployment. The platform team becomes an enabler, not a gatekeeper.

---

## 10. Interactive Exercise

### Architectural Challenge

A global manufacturing company is embarking on a three-year cloud migration of 1,200 applications.

**Requirements:**

1. Infrastructure is deployed exclusively through Terraform — no manual Azure Portal deployments
2. Governance must be consistent — all applications follow the same naming, tagging, RBAC standards
3. Deployment time must be <30 minutes from approval to production for standard application templates
4. Cost chargeback must be possible per business unit
5. The company has 85 application teams distributed across 12 business units
6. Teams range from infrastructure-experienced (10%) to application-focused (90%)
7. A recent security audit flagged inconsistent security configurations across applications

**Your task:**

1. **Define the reusable Terraform module strategy.** What modules would you build? How would you version them? Would you use a monorepo or separate repositories?

2. **Design the state management architecture.** How many state files? Where are they stored? How do you ensure isolation?

3. **Specify governance standards.** What naming convention would you enforce? What tagging standards? Which Azure Policies would you assign?

4. **Describe the deployment workflow.** How would an application team deploy infrastructure? What steps ensure governance compliance?

5. **Address the pilot phase.** Which application types would you target for the first 10 migrations? Why?

6. **Explain the operating model.** How many platform engineers do you need? What are their responsibilities?

---

*There is no single correct answer.*

*Evaluation focuses on: scalability of the module design, correctness of state isolation, comprehensiveness of governance standards, automation to reduce human error, realistic assessment of team size and responsibilities, and ability to defend trade-offs made.*
