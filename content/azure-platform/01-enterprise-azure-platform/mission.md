# Mission 01 — Enterprise Azure Platform

Version: 1.0

---

# Mission

You have joined a Cloud Center of Excellence responsible for migrating enterprise workloads from on-premises to Microsoft Azure.

Over the next three years, approximately 500 applications will be migrated.

Your team is not responsible for developing applications.

Your responsibility is to build the Azure platform they will run on.

This platform must be secure, scalable, governable and easy for application teams to consume.

Your goal is to understand how Platform Engineers think before discussing Azure services.

---

# Business Context

The customer currently operates an on-premises environment.

Applications are being migrated using a Lift-and-Shift strategy.

Some applications will be deployed to Azure App Service.

Others will remain on Virtual Machines.

Different engineering teams will consume the platform.

The Cloud Center of Excellence owns:

- Azure governance
- Landing Zones
- Networking standards
- Security baselines
- Terraform reusable modules
- Azure Policies
- Identity standards

Application teams consume the platform.

They do not build it.

---

# Why This Matters

Most engineers think in terms of Azure resources.

Platform Engineers think in terms of platforms.

Interviewers are interested in your reasoning.

Not your ability to remember Azure documentation.

---

# Learning Objectives

After completing this mission you should be able to explain:

What is a Landing Zone?

Why Management Groups exist.

Why enterprises separate subscriptions.

How governance works.

Why platform teams exist.

Why application teams should not own Azure architecture.

How reusable infrastructure improves engineering productivity.

How platform engineering differs from infrastructure engineering.

---

# Required Knowledge

Enterprise Scale Architecture

Management Groups

Subscriptions

Resource Groups

Landing Zones

Governance

Azure Policy (high level)

Role separation

Cloud Center of Excellence

Platform Engineering

Shared Responsibility

---

# Required Architectural Thinking

Understand the difference between

Platform

↓

Landing Zone

↓

Subscription

↓

Resource Group

↓

Application

Instead of memorizing services, understand responsibilities.

Who owns what?

Why?

---

# Enterprise Scenarios

Scenario 1

The company creates a new Azure subscription for every application.

Would you recommend this?

Why?

---

Scenario 2

The security team wants complete control over every subscription.

Application teams want full autonomy.

How would you balance governance and developer productivity?

---

Scenario 3

A development team asks for Owner permissions because deploying resources is easier.

How would you respond?

---

Scenario 4

Your organization expects to migrate another 300 applications next year.

What decisions would you make today to avoid future platform redesign?

---

# Expected Interview Questions

Explain the purpose of Azure Landing Zones.

Why not place everything inside one subscription?

What problems do Management Groups solve?

What belongs inside a Landing Zone?

Who owns governance?

What responsibilities belong to a Platform Team?

How would you organize Azure for hundreds of applications?

How would you onboard a new engineering team?

---

# Common Mistakes

Thinking in Azure resources instead of platform architecture.

Treating Resource Groups as security boundaries.

Using subscriptions without governance.

Ignoring organizational structure.

Confusing Landing Zones with networking.

Thinking Platform Engineering is Infrastructure Engineering.

---

# AI Generation Package

Teacher Persona

Michael Andersen

Learning Mode

Teacher

Difficulty

Senior Platform Engineer

Target Duration

90 minutes

Teaching Style

Interactive.

Architecture-first.

Discussion-driven.

No long lectures.

---

# Expected Deliverables

After finishing the mission you should be able to:

Draw a basic enterprise Azure hierarchy.

Explain Platform Engineering.

Explain Landing Zones.

Explain Management Groups.

Explain subscription strategy.

Explain governance responsibilities.

Defend architectural decisions.

---

# Exit Criteria

You can explain Enterprise Azure architecture on a whiteboard.

You can defend your subscription strategy.

You can explain Landing Zones without documentation.

You understand Platform Team responsibilities.

You understand why governance exists.

You can answer Senior-level interview questions confidently.

---

# Mission Complete

Do not continue to the next mission until:

Teacher completed.

Coach completed.

Interview completed.

Review completed.

Gap Closure completed (if required).

Validation Interview passed.