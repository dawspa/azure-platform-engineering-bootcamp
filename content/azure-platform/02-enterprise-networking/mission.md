# Mission 02 — Enterprise Azure Networking

Version: 1.0

---

# Mission

You are joining a Platform Engineering team responsible for designing Azure networking standards used by hundreds of applications.

Your goal is not to configure networking.

Your goal is to design networking that remains maintainable for years.

Networking should enable application teams.

Not slow them down.

---

# Business Context

The organization is migrating applications from on-premises.

Some applications remain on Virtual Machines.

Others move to Azure App Service.

The company expects continuous growth.

Networking decisions made today must support future migrations, hybrid connectivity and security requirements.

---

# Why This Matters

Networking is usually the first deep technical topic during Azure Platform interviews.

Interviewers rarely ask about subnet syntax.

They ask why the architecture looks the way it does.

---

# Learning Objectives

After completing this mission you should be able to explain:

- Virtual Network strategy
- Hub-and-Spoke architecture
- Virtual Network Peering
- Network Security Groups
- User Defined Routes
- Azure Firewall
- NAT Gateway
- Azure Bastion
- Enterprise routing principles

---

# Required Knowledge

Virtual Network

Subnet Design

NSG

UDR

VNet Peering

Hub-and-Spoke

Azure Firewall

NAT Gateway

Azure Bastion

Routing

DNS basics

---

# Required Architectural Thinking

Understand responsibilities.

Hub

↓

Shared Services

↓

Connectivity

↓

Security

↓

Application VNets

Think in terms of ownership.

Who owns networking?

Who owns applications?

Where should security controls live?

---

# Enterprise Scenarios

Scenario 1

A new application team requests a dedicated VNet.

Would you create one?

Why?

---

Scenario 2

Application teams want to peer VNets directly.

Would you allow mesh networking?

Why?

---

Scenario 3

The networking team proposes Hub-and-Spoke.

The application team argues it is unnecessary complexity.

Defend your recommendation.

---

Scenario 4

Your company expects acquisitions and new business units.

How should networking evolve?

---

# Expected Interview Questions

Explain Hub-and-Spoke.

Why not Mesh?

When would you use Azure Firewall?

Why Bastion?

What belongs inside the Hub?

What belongs inside the Spoke?

How would you design networking for 300 applications?

What operational concerns exist?

---

# Common Mistakes

Thinking networking starts with subnets.

Ignoring ownership.

Ignoring operations.

Ignoring routing.

Thinking NSG replaces Firewall.

Designing networking around today's applications.

---

# AI Generation Package

Teacher Persona

Michael Andersen

Learning Mode

Teacher

Difficulty

Senior Platform Engineer

Teaching Style

Architecture discussion.

Whiteboard thinking.

Trade-offs.

---

# Expected Deliverables

Draw Hub-and-Spoke.

Explain routing.

Explain Peering.

Explain Firewall placement.

Explain Bastion.

Explain ownership boundaries.

Defend architectural decisions.

---

# Exit Criteria

You can design enterprise networking on a whiteboard.

You understand architectural trade-offs.

You can justify Hub-and-Spoke.

You can explain networking ownership.

You are comfortable discussing enterprise networking with a Principal Engineer.

---

# Mission Complete

Complete the full workflow before continuing.