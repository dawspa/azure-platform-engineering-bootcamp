# Mission 05 — Enterprise Terraform, Governance & Cloud Migration

Version: 1.0

---

# Mission

You have joined a Cloud Center of Excellence responsible for building reusable Azure platform capabilities for multiple engineering teams.

The organization is migrating workloads from on-premises to Azure using a Lift-and-Shift strategy.

Infrastructure is provisioned exclusively with Terraform.

The platform team owns reusable modules, governance standards, migration enablement and audit remediation.

Your responsibility is not to deploy infrastructure.

Your responsibility is to design a platform that enables hundreds of future deployments safely and consistently.

---

# Business Context

The organization is modernizing its infrastructure incrementally.

Current migration targets include:

- Azure Virtual Machines
- Azure App Service

Future modernization may include containers, but Kubernetes is not currently in scope.

Terraform is the standard Infrastructure as Code tool.

Engineering teams consume reusable platform modules maintained by the Cloud Center of Excellence.

Recent security audits identified governance inconsistencies, excessive permissions and missing policy enforcement.

---

# Why This Matters

Platform Engineers do not build infrastructure one environment at a time.

They build reusable foundations that scale across teams.

Terraform, governance and migration strategy are tightly connected.

Successful migrations depend more on platform design than on migration tooling.

---

# Learning Objectives

After completing this mission you should be able to explain:

- Terraform module design
- Module versioning
- Remote State
- State isolation
- Repository organization
- Reusable platform modules
- Azure Policy
- Governance strategy
- Naming standards
- Tagging strategy
- Audit remediation
- Lift-and-Shift migration
- Rehost vs Replatform
- Migration planning
- Platform ownership

---

# Required Knowledge

Terraform

Modules

Variables

Outputs

Remote State

AzureRM Provider

Repository Structure

Azure Policy

Initiatives

Management Groups

Resource Organization

Tagging

Naming Standards

Lift-and-Shift

Rehost

Replatform

App Service

Virtual Machines

---

# Required Architectural Thinking

Understand platform ownership.

Platform Team

↓

Reusable Modules

↓

Governance

↓

Engineering Teams

↓

Applications

Infrastructure should become easier to consume over time.

---

# Enterprise Scenarios

## Scenario 1

Three application teams need identical App Service environments.

Would you duplicate Terraform code or build reusable modules?

How would you structure them?

---

## Scenario 2

An audit finds inconsistent tagging and naming conventions across subscriptions.

How would you prevent the issue from recurring?

---

## Scenario 3

A legacy IIS application is being migrated to Azure.

Would you choose Virtual Machines or App Service?

What factors influence your decision?

---

## Scenario 4

The organization plans to migrate another 400 applications within two years.

Which platform decisions should be standardized today?

---

## Scenario 5

Application teams request exceptions to Azure Policy because deployments are failing.

How would you balance governance with developer productivity?

---

# Expected Interview Questions

Describe your Terraform module strategy.

How would you version reusable modules?

How would you organize repositories?

How would you structure remote state?

How should Azure Policy be used?

How would you improve governance after an audit?

Explain Lift-and-Shift.

When would you choose App Service over Virtual Machines?

What technical debt should be avoided during migration?

How would you onboard a new engineering team?

---

# Common Mistakes

Designing Terraform around a single project.

Overengineering modules.

Ignoring module consumers.

Using Azure Policy only for compliance.

Treating migration as a one-time project.

Migrating technical debt unchanged.

Ignoring operational ownership.

Thinking only about deployment instead of long-term maintainability.

---

# AI Generation Package

Teacher Persona

Daniel Kovacs

Learning Mode

Teacher

Difficulty

Senior Platform Engineer

Teaching Style

Architecture-first.

Pragmatic.

Platform-oriented.

Focused on maintainability and trade-offs.

---

# Expected Deliverables

Explain reusable Terraform architecture.

Design module boundaries.

Describe governance strategy.

Explain Azure Policy in enterprise environments.

Compare App Service and Virtual Machines for migration.

Explain Lift-and-Shift trade-offs.

Describe how a Cloud Center of Excellence enables engineering teams.

Defend architectural decisions.

---

# Exit Criteria

You can explain enterprise Terraform architecture.

You understand governance beyond compliance.

You can discuss migration strategy confidently.

You understand the responsibilities of a Platform Engineering team.

You can justify architectural decisions using trade-offs.

---

# Mission Complete

Complete the agreed workflow before considering the mission finished.