# Mission 04 — Enterprise Identity & Security

Version: 1.0

---

# Mission

You have joined a Cloud Center of Excellence responsible for building a secure Azure platform used by multiple engineering teams.

The organization has recently completed a cloud security audit.

Several findings relate to identity, excessive permissions and inconsistent governance.

Your responsibility is not to fix individual resources.

Your responsibility is to design identity and security standards that every engineering team will follow.

---

# Business Context

The organization is migrating workloads from on-premises to Azure.

Engineering teams deploy infrastructure using Terraform.

The platform team owns:

- Microsoft Entra ID architecture
- RBAC standards
- Managed Identity strategy
- Azure Policy
- Secure Score improvements
- Key Vault standards
- Governance

Application teams consume the platform.

They should not make identity decisions independently.

---

# Why This Matters

Identity is the security boundary in Azure.

Most successful attacks exploit identity rather than networking.

Senior Platform Engineers must understand how identity, governance and automation work together.

---

# Learning Objectives

After completing this mission you should be able to explain:

- Microsoft Entra ID architecture
- Azure RBAC
- Least Privilege
- Managed Identity
- Service Principal
- Privileged Identity Management (PIM)
- Conditional Access
- Microsoft Secure Score
- Defender for Cloud
- Key Vault integration
- Identity governance

---

# Required Knowledge

Microsoft Entra ID

Azure RBAC

Managed Identity

Service Principal

PIM

Conditional Access

Secure Score

Defender for Cloud

Key Vault

Identity Governance

---

# Required Architectural Thinking

Understand ownership.

Identity

↓

Authorization

↓

Governance

↓

Automation

↓

Operations

Identity decisions affect the entire platform.

---

# Enterprise Scenarios

Scenario 1

Developers request Owner permissions to deploy faster.

How would you respond?

---

Scenario 2

Your audit recommends eliminating secrets from application code.

How would you redesign the platform?

---

Scenario 3

Management wants to increase Microsoft Secure Score quickly.

Would you enable every recommendation?

Why or why not?

---

Scenario 4

Multiple applications access Azure Storage and SQL Database.

Would you use Service Principals or Managed Identities?

Explain your decision.

---

# Expected Interview Questions

Explain Microsoft Entra ID.

What is the difference between authentication and authorization?

Why use Managed Identity?

When is a Service Principal still appropriate?

How would you implement least privilege?

What is PIM?

Would you optimize for Secure Score?

How do Azure Policy and RBAC complement each other?

---

# Common Mistakes

Treating RBAC as identity.

Using Owner permissions by default.

Hardcoding secrets.

Ignoring operational impact.

Treating Secure Score as the objective.

Focusing on compliance instead of risk.

---

# AI Generation Package

Teacher Persona

Sarah Mitchell

Learning Mode

Teacher

Difficulty

Senior Platform Engineer

Teaching Style

Architecture-first.

Risk-driven.

Enterprise-focused.

Discussion-based.

---

# Expected Deliverables

Explain identity architecture.

Design RBAC strategy.

Recommend Managed Identity.

Explain Secure Score critically.

Describe enterprise governance.

Defend architectural decisions.

---

# Exit Criteria

You can explain enterprise identity architecture without documentation.

You can justify least-privilege decisions.

You understand Managed Identity deeply.

You can discuss Secure Score critically.

You think in terms of risk rather than features.

---

# Mission Complete

Complete the agreed workflow before moving to the next mission.