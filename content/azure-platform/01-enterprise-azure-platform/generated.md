---
Title: Enterprise Azure Platform
Domain: Azure Platform Engineering
Difficulty: Foundational
Estimated Duration: 120 minutes
Prerequisites:
  - Basic Azure familiarity
  - Conceptual understanding of cloud computing
Learning Objectives:
  - Explain what a Landing Zone is and why it exists
  - Explain why Management Groups are required at enterprise scale
  - Explain why enterprises separate subscriptions
  - Describe how governance works in Azure
  - Explain why platform teams exist and what they own
  - Articulate the difference between platform engineering and infrastructure engineering
  - Explain why application teams should not own Azure architecture
  - Describe how reusable infrastructure improves engineering productivity
Interview Focus:
  - Senior Platform Engineer
---

# Module 01 — Enterprise Azure Platform

---

## 1. Learning Goal

After completing this module you will be able to explain how enterprises structure their Azure environments at scale.

You will be able to describe the purpose of Management Groups, Landing Zones, and subscription separation without referencing Azure documentation.

You will be able to reason about why platform teams exist and what problems they solve for application teams.

You will be prepared to answer architecture-level interview questions about enterprise Azure governance.

---

## 2. Why This Exists

### The Problem Microsoft Was Solving

When organisations first adopted Azure, individual teams created subscriptions independently.

Each team configured their own networking, identity, policies, and security controls.

The result was an environment with hundreds of subscriptions, inconsistent security postures, duplicated networking configurations, and no centralised governance.

Auditing was impossible.

Compliance was unachievable.

Cost visibility was absent.

Onboarding new teams took weeks because every team invented their own baseline.

### What Microsoft Introduced

Microsoft introduced the Enterprise Scale Architecture (now called Azure Landing Zones) as a prescriptive reference architecture for large organisations.

The architecture defines:

- How to structure Management Groups
- How to separate subscriptions by purpose
- How to enforce governance through Azure Policy
- How to centralise networking
- How to onboard application teams consistently

### Why Previous Approaches Were Insufficient

Flat subscription models did not scale.

A single subscription with hundreds of teams sharing resources created risk, contention, and governance gaps.

Role-Based Access Control alone was insufficient for compliance at scale.

Manual configuration could not guarantee consistency across hundreds of teams.

---

## 3. Core Concepts

### Management Groups

Management Groups are containers that exist above subscriptions in the Azure hierarchy.

They allow you to apply governance — policies and role assignments — to multiple subscriptions simultaneously.

A policy applied to a Management Group is inherited by every subscription beneath it.

Enterprises use Management Groups to enforce security baselines, naming conventions, and regional restrictions without configuring each subscription individually.

### Subscriptions

A subscription is an Azure billing and access boundary.

Enterprises use separate subscriptions to:

- Isolate blast radius
- Control costs per application or business unit
- Apply different governance requirements to different workloads
- Satisfy regulatory separation requirements

A single subscription for all workloads creates a shared fate problem.

If one team exhausts a quota, all teams are affected.

If one misconfiguration causes a policy violation, all workloads are implicated.

### Resource Groups

Resource groups are logical containers within a subscription.

They group resources that share a lifecycle.

They are not a security boundary.

They are not a governance boundary.

They are a management boundary.

### Landing Zones

A Landing Zone is a pre-configured Azure environment that is ready for application teams to consume.

It contains:

- A subscription
- Networking baseline
- Security controls
- Policy assignments
- Role assignments
- Monitoring configuration

A Landing Zone is not a product you install.

It is a pattern you implement.

The Cloud Center of Excellence builds Landing Zones.

Application teams receive them.

### Governance

Governance in Azure is primarily implemented through Azure Policy.

Azure Policy allows the platform team to:

- Deny resource creation that violates standards
- Automatically remediate non-compliant resources
- Audit compliance across all subscriptions
- Enforce tagging, naming, and regional restrictions

Governance is not optional at enterprise scale.

Without it, 500 application teams will make 500 different decisions.

---

## 4. Architecture

### Azure Management Hierarchy

```
Tenant Root Group
        |
        ├── Platform Management Group
        │       ├── Management Subscription
        │       ├── Connectivity Subscription
        │       └── Identity Subscription
        │
        ├── Landing Zones Management Group
        │       ├── Corp Management Group
        │       │       ├── App Team A Subscription
        │       │       ├── App Team B Subscription
        │       │       └── App Team C Subscription
        │       │
        │       └── Online Management Group
        │               ├── App Team D Subscription
        │               └── App Team E Subscription
        │
        ├── Sandbox Management Group
        │       └── Dev / Experimentation Subscriptions
        │
        └── Decommissioned Management Group
```

### Governance Inheritance

```
Tenant Root Group Policy
        |
        ↓  Inherited
Platform Management Group Policy
        |
        ↓  Inherited
Connectivity Subscription Policy
        |
        ↓  Inherited
Resource Group Policy
        |
        ↓  Inherited
Individual Resources
```

Policies assigned at a higher scope apply to everything below.

This is the foundation of enterprise governance.

### Landing Zone Anatomy

```
Landing Zone Subscription
        |
        ├── Hub-Spoke Peering
        │       └── Connected to Connectivity Hub
        │
        ├── Resource Groups
        │       ├── Networking RG
        │       ├── Application RG
        │       └── Monitoring RG
        │
        ├── Azure Policy (inherited + local)
        │
        └── RBAC Role Assignments
                ├── Platform Team (Owner)
                └── App Team (Contributor scoped)
```

---

## 5. Design Decisions

### When to Use Separate Subscriptions

Use separate subscriptions when:

- Workloads have different compliance requirements
- Teams need independent quota pools
- Cost allocation must be isolated
- Blast radius must be contained

Avoid using a single subscription for multiple unrelated workloads at scale.

### When NOT to Use a Management Group Deeply

Do not create unnecessarily deep Management Group hierarchies.

Six levels is a hard limit in Azure.

Deep hierarchies increase complexity of policy assignment without proportional benefit.

Keep hierarchies shallow unless regulatory requirements force deeper separation.

### Alternatives Considered

Some organisations use Resource Groups as isolation boundaries instead of subscriptions.

This works at small scale.

It fails at enterprise scale because:

- Resource Groups do not isolate quota
- Resource Groups do not provide billing isolation
- RBAC at Resource Group scope becomes difficult to manage across hundreds of teams

### Trade-offs

| Approach | Benefit | Cost |
|---|---|---|
| One subscription per team | Full isolation | Higher management overhead |
| Shared subscriptions | Simpler billing | Shared blast radius, quota contention |
| Centralised platform team | Consistent governance | Slower onboarding if not automated |
| Distributed ownership | Team autonomy | Inconsistent security posture |

---

## 6. Real World Scenario

A European financial services company is migrating 500 applications to Azure over three years.

Applications are owned by 40 different engineering teams across 12 business units.

Regulatory requirements mandate that production workloads are isolated from development workloads.

The compliance team requires evidence that no production system can be modified without approval.

The security team requires that all resources are deployed only in approved regions.

The finance team requires cost reports per business unit.

**How the platform team responds:**

The Cloud Center of Excellence builds the following structure:

- One Management Group hierarchy representing business units
- Separate subscriptions for production, non-production, and sandbox per team
- Azure Policy assignments at Management Group level enforcing:
  - Approved regions only
  - Required resource tags for cost allocation
  - Diagnostic settings enforced on all resources
  - Deny public IP creation in production subscriptions
- Landing Zone templates that provision a compliant subscription in 15 minutes
- Terraform modules that application teams use to deploy within the Landing Zone

Application teams do not interact with the platform.

They submit a request.

The platform team provisions a Landing Zone.

The application team receives credentials and begins deploying.

**Outcome:**

Every subscription is compliant from day one.

No team can deviate from security baselines.

Cost reporting is automatic because tagging is enforced.

Regional restrictions are guaranteed.

The compliance team can audit all 500 applications from a single Management Group view.

---

## 7. Common Mistakes

### Mistake 1: Treating Subscriptions as a Resource Container

Engineers familiar with on-premises environments treat Azure subscriptions like servers.

They create one subscription and put everything in it.

This immediately breaks cost allocation, quota management, and governance at scale.

**Why it is wrong:**

A subscription is a boundary, not a container.

It exists to isolate, separate, and govern — not to hold resources.

### Mistake 2: Applying Policies at Resource Group Level

New platform engineers apply Azure Policies individually to each Resource Group.

This scales to approximately 10 teams before it becomes unmanageable.

**Why it is wrong:**

Policy inheritance through Management Groups means you assign once and every subscription beneath inherits it.

Applying at Resource Group level bypasses the inheritance model and creates configuration drift.

### Mistake 3: Giving Application Teams Subscription Owner Rights

Some organisations grant application teams Owner rights on their subscriptions.

This allows teams to remove policies, change RBAC assignments, and bypass governance controls.

**Why it is wrong:**

The platform team's role is to enforce standards.

Owner rights on a subscription allow teams to undo platform decisions.

Application teams should receive Contributor rights scoped to specific Resource Groups.

### Mistake 4: Building Landing Zones Manually

Some platform engineers provision Landing Zones manually for each team.

At 500 applications this takes months and introduces human error.

**Why it is wrong:**

Landing Zones must be automated.

Terraform modules or Azure Bicep templates should provision a compliant environment in minutes.

Manual provisioning is not scalable and is not repeatable.

### Mistake 5: Confusing Platform Engineering with Infrastructure Engineering

Infrastructure engineers deploy Azure resources.

Platform engineers build the environment that other teams deploy into.

**Why the distinction matters:**

A platform engineer who thinks like an infrastructure engineer builds servers.

A platform engineer who thinks like a platform engineer builds the system that others use to build servers consistently and safely.

---

## 8. Troubleshooting

### Scenario 1: Application Team Cannot Deploy Resources to Their Subscription

**Symptoms:**

Application team reports that resource deployments fail with `AuthorizationFailed`.

**Possible Causes:**

- RBAC role assignment was not created for the application team
- Policy `Deny` effect is blocking resource creation
- Application team is deploying to the wrong subscription
- Managed Identity used by the pipeline lacks permissions

**Investigation:**

Check the Activity Log in the subscription for denied operations.

Use the Azure Policy Compliance blade to identify which policy is blocking the deployment.

Review RBAC role assignments at the subscription and Resource Group scope.

**Resolution:**

If RBAC: Assign the appropriate role to the team's identity at the correct scope.

If Policy: Investigate whether the deployment violates a legitimate policy or whether the policy needs an exemption.

If exemption is warranted, create a Policy Exemption at the resource or Resource Group scope — do not disable the policy.

---

### Scenario 2: Resources Are Deployed Outside Approved Regions

**Symptoms:**

Compliance report shows resources in `eastus` when the standard requires `westeurope`.

**Possible Causes:**

- Allowed locations policy was not assigned to the relevant Management Group
- Policy was assigned but in Audit mode instead of Deny mode
- A policy exemption exists that should not

**Investigation:**

Review the Policy assignment on the Management Group.

Check the policy effect — `Audit` logs but does not prevent. `Deny` prevents.

Check for Policy Exemptions at subscription or Resource Group level.

**Resolution:**

Change policy effect from `Audit` to `Deny` for enforcement.

Remove unauthorised exemptions.

Remediate existing non-compliant resources using Policy Remediation Tasks.

---

### Scenario 3: Cost Reports Cannot Be Separated by Team

**Symptoms:**

Finance requests a cost breakdown per application team but resources have no consistent tags.

**Possible Causes:**

- Tagging policy was not assigned
- Tagging policy is in Audit mode
- Teams applied tags inconsistently

**Investigation:**

Review Policy compliance for tagging policies across subscriptions.

Check whether a `Require Tag` policy exists and is in `Deny` mode.

**Resolution:**

Assign an `Inherit tag from subscription` policy to propagate subscription-level tags to resources automatically.

Assign a `Require tag` policy in `Deny` mode to enforce consistent tagging on new resources.

Use Remediation Tasks to backfill tags on existing resources where possible.

---

## 9. Interview Questions

### Junior

**Q: What is an Azure Landing Zone?**

A Landing Zone is a pre-configured Azure subscription environment that is ready for an application team to consume. It includes networking, security policies, RBAC assignments, and monitoring configuration. The platform team builds it. The application team uses it.

**Q: What is a Management Group?**

A Management Group is a container above subscriptions in the Azure hierarchy. It allows the platform team to apply policies and role assignments to multiple subscriptions simultaneously. Assignments at a Management Group level are inherited by all subscriptions beneath it.

**Q: Why do enterprises use multiple subscriptions?**

To isolate blast radius, separate costs per team or business unit, apply different governance requirements to different workloads, and avoid quota contention between teams.

---

### Mid

**Q: How does Azure Policy enforcement work across subscriptions?**

Azure Policy is assigned at a scope — Management Group, subscription, or Resource Group. When assigned at Management Group level, it is inherited by every subscription below. The `Deny` effect prevents non-compliant resources from being created. The `Audit` effect logs non-compliance without preventing it. Remediation Tasks can retroactively fix existing non-compliant resources using `DeployIfNotExists` or `Modify` effects.

**Q: An application team says their deployment fails because of a policy. How do you investigate?**

Check the Activity Log for the specific deployment failure and note the operation name. Review the Policy Compliance blade to identify which policy is triggering the denial. Determine whether the denial is correct — does the deployment violate a legitimate standard? If a business justification exists, create a Policy Exemption at the appropriate scope. Do not modify or disable the policy.

**Q: What is the difference between a Resource Group and a subscription as an isolation boundary?**

A subscription provides billing isolation, quota isolation, and a governance boundary. Resource Groups provide lifecycle management grouping but share the subscription's quota and billing. Resource Groups cannot isolate regulatory compliance between teams at scale.

---

### Senior

**Q: How would you design a Management Group hierarchy for a company with 500 applications across 40 teams in 3 business units, with different compliance requirements per unit?**

I would create a hierarchy with:
- Tenant Root Group at the top with global baseline policies
- Three business unit Management Groups beneath it
- Corp and Online sub-groups within each business unit to separate internet-facing from internal workloads
- Platform Management Group for centralised platform subscriptions — connectivity, management, identity
- Sandbox Management Group for experimentation

Policies enforcing global baselines — regions, mandatory tags, diagnostic settings — would be assigned at Tenant Root Group level.

Business-unit-specific compliance controls would be assigned at the business unit Management Group level.

Teams would receive subscriptions under the appropriate Management Group node. They would inherit all applicable policies automatically.

**Q: Why should application teams not have Owner rights on their subscriptions?**

Owner rights allow teams to modify or remove RBAC assignments and Policy Exemptions. This undermines the governance model the platform team has established. A team with Owner rights can remove a `Deny` policy from their subscription, deploy non-compliant resources, and introduce security gaps the platform team cannot detect unless they are actively monitoring. Application teams should have Contributor rights scoped to their Resource Groups. The platform team retains Owner rights at the subscription level.

**Q: Your organisation has 500 application subscriptions. How do you ensure every new subscription is compliant from day one?**

By automating Landing Zone provisioning through Infrastructure as Code. A Terraform module or Azure Verified Modules template deploys a new subscription under the correct Management Group, applies RBAC assignments, creates standard Resource Groups, configures VNet peering to the connectivity hub, and enables diagnostic settings. The application team submits a request. The pipeline executes. They receive a compliant environment. Manual provisioning is eliminated. Policy inheritance ensures governance controls apply immediately.

---

### Principal

**Q: Describe the tension between platform standardisation and application team autonomy. How do you resolve it?**

Platform standardisation exists to enforce security, compliance, and consistency. Application team autonomy exists to enable engineering velocity.

The tension becomes a problem when platform standards are too prescriptive — telling teams how to deploy rather than what constraints they must satisfy.

I resolve this by separating guardrails from recommendations.

Guardrails — things like approved regions, mandatory diagnostic settings, no public IPs in production — are enforced through `Deny` policies. These are non-negotiable.

Recommendations — preferred resource types, suggested naming conventions — are communicated through documentation and internal developer portals, not policies.

Application teams are free to make any decision that does not violate a guardrail. They retain architectural ownership within their Landing Zone.

This produces a system where the platform team guarantees compliance without becoming an engineering bottleneck.

**Q: How does a Landing Zone relate to the concept of a paved road in platform engineering?**

A Landing Zone is the physical implementation of a paved road.

A paved road is the idea that the correct path should be the easiest path. You build the road so well that engineers naturally follow it rather than build their own paths.

A Landing Zone is paved in the sense that it pre-satisfies compliance, networking, and security requirements. The application team does not need to think about tagging — the policy enforces it. They do not need to think about VNet peering — the Landing Zone includes it. They do not need to understand Management Group inheritance — the platform team has applied the correct policies.

The result is that an application team can focus entirely on deploying their application. The platform handles everything beneath that.

The risk is that if the paved road has a design flaw — incorrect policy, missing security control — that flaw is replicated across every Landing Zone that was provisioned from the same template. This is why Landing Zone templates must be version-controlled, peer-reviewed, and tested before they are used in production.

---

## 10. Interactive Exercise

### Architectural Challenge

A retail company is beginning their Azure migration.

They have 200 applications owned by 30 teams across three divisions:

- Finance (highly regulated, PCI-DSS compliance required)
- Operations (internal systems, lower compliance burden)
- E-Commerce (internet-facing, performance-sensitive)

The CISO requires that Finance workloads have no connectivity to E-Commerce workloads.

The network team requires a centralised hub for all outbound internet traffic.

The finance team requires that all Finance subscriptions deny the creation of any public IP address.

The CTO requires that all Azure resources can be inventoried and cost-attributed to a division.

**Your task:**

Design the Management Group hierarchy for this organisation.

Specify which Azure Policies you would assign at each level and why.

Specify how you would enforce the network isolation requirement between Finance and E-Commerce.

Explain what a Landing Zone looks like for a Finance application team.

Explain what a Landing Zone looks like for an E-Commerce application team and how it differs.

Identify one decision in your design that involves a significant trade-off and explain both sides.

---

*There is no single correct answer.*

*The quality of your response is measured by the clarity of your reasoning, the trade-offs you identify, and your ability to connect the technical decisions to the business requirements.*
