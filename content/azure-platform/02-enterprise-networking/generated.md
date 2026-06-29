---
Title: Enterprise Azure Networking
Domain: Azure Platform Engineering
Difficulty: Senior
Estimated Duration: 150 minutes
Prerequisites:
  - Module 01 — Enterprise Azure Platform
  - Basic understanding of IP addressing and subnets
  - Conceptual familiarity with firewalls and routing
Learning Objectives:
  - Explain Hub-and-Spoke architecture and justify it against alternatives
  - Describe what belongs in the Hub and what belongs in the Spoke
  - Explain Virtual Network Peering and its limitations
  - Describe how routing works in Azure and why User Defined Routes exist
  - Explain the role of Azure Firewall in enterprise networking
  - Explain why NSGs alone are insufficient for enterprise security
  - Explain NAT Gateway and outbound connectivity patterns
  - Explain Azure Bastion and why direct VM access is a risk
  - Define clear ownership boundaries between platform and application teams
Interview Focus:
  - Senior Platform Engineer
  - Principal Platform Engineer
---

# Module 02 — Enterprise Azure Networking

---

## 1. Learning Goal

After completing this module you will be able to design enterprise Azure networking on a whiteboard and justify every decision.

You will be able to explain why Hub-and-Spoke exists, what problem it solves, and why mesh networking fails at scale.

You will be able to reason about routing, security control placement, and operational ownership without referencing Azure documentation.

You will be comfortable discussing enterprise networking architecture with a Principal Engineer.

---

## 2. Why This Exists

### The Problem with Flat Networking

In early Azure adoption, teams created Virtual Networks independently.

Each team peered their VNet to whatever other VNets they needed connectivity to.

The result was a mesh.

Every VNet knew about every other VNet.

Security teams had no centralised point to inspect traffic.

Routing was inconsistent.

Shared services like DNS, VPN gateways, and monitoring agents were duplicated in every VNet.

When a new team joined, they needed to peer with every other team individually.

At 300 application teams this model collapses.

### What Was Needed

A centralised architecture where:

- Shared services exist in one place
- All traffic flows through a single inspection point
- Application teams connect without designing their own networking
- Routing is controlled centrally
- New teams can join without topology changes to existing VNets

### What Microsoft Introduced

Hub-and-Spoke is the reference architecture that addresses all of these requirements.

It is not a feature.

It is a pattern.

The Hub is a centralised Virtual Network owned by the platform team.

Spokes are application VNets that connect to the Hub through peering.

All traffic between Spokes — and all traffic to the internet or on-premises — passes through the Hub.

This gives the platform team full visibility and control over traffic without application teams needing to understand networking.

---

## 3. Core Concepts

### Virtual Network (VNet)

A Virtual Network is a logically isolated network in Azure.

Resources inside a VNet can communicate with each other by default.

Resources in different VNets cannot communicate without explicit connectivity.

A VNet is scoped to a single Azure region.

### Subnet

A subnet is a range of IP addresses within a VNet.

Subnets are used to organise resources and apply Network Security Groups.

Some Azure services require dedicated subnets — Azure Firewall, Azure Bastion, VPN Gateway.

### VNet Peering

VNet Peering creates a low-latency, high-bandwidth connection between two VNets.

Traffic across a peering stays on the Microsoft backbone — it does not traverse the public internet.

Peering is non-transitive by default.

If VNet A is peered with Hub and Hub is peered with VNet B, VNet A cannot reach VNet B through the Hub unless routing is explicitly configured.

This is a critical property. It is why Hub-and-Spoke requires User Defined Routes — to force transitivity.

### Network Security Group (NSG)

An NSG is a stateful packet filter.

It operates at Layer 4.

It allows or denies traffic based on source IP, destination IP, port, and protocol.

NSGs can be associated with subnets or individual network interfaces.

NSGs do not perform deep packet inspection.

NSGs do not understand application-layer protocols.

NSGs cannot perform URL filtering, TLS inspection, or threat detection.

### User Defined Route (UDR)

Azure routes traffic according to system routes by default.

A User Defined Route overrides system routing.

The most common use case in enterprise networking is a UDR that routes all internet-bound and inter-VNet traffic through Azure Firewall.

Without UDRs, peered Spokes would send traffic directly — bypassing the Firewall.

### Azure Firewall

Azure Firewall is a managed, stateful network security service.

It operates at Layer 4 and Layer 7.

It supports:

- Network rules — IP, port, and protocol
- Application rules — FQDN-based filtering with TLS inspection
- Threat intelligence — known malicious IPs and domains
- IDPS — intrusion detection and prevention

Azure Firewall is the central inspection point in a Hub-and-Spoke architecture.

All traffic flows through it.

Application teams do not configure it.

The platform team owns Firewall policy.

### NAT Gateway

NAT Gateway provides deterministic, scalable outbound internet connectivity for resources that do not have public IP addresses.

Without NAT Gateway, Azure uses SNAT from a shared pool of IP addresses.

SNAT port exhaustion is a real failure mode for high-volume workloads.

NAT Gateway provides a dedicated set of outbound IPs with 64,512 SNAT ports per IP address.

It also allows the security team to allowlist a known set of outbound IPs in downstream systems.

### Azure Bastion

Azure Bastion provides browser-based SSH and RDP access to Virtual Machines without exposing them to the public internet.

Without Bastion, engineers open port 22 or 3389 directly on a VM's public IP address.

This is a significant attack surface.

Bastion sits in a dedicated subnet in the Hub.

All VM access routes through Bastion.

The VM has no public IP.

The NSG on the VM blocks all inbound access from the internet.

### Hub-and-Spoke

Hub-and-Spoke is a topology in which:

- One centralised VNet (the Hub) contains shared services
- Many application VNets (Spokes) connect to the Hub via peering
- Traffic between Spokes is routed through the Hub
- All internet-bound traffic exits through the Hub
- All on-premises traffic enters through the Hub

The Hub is owned by the platform team.

Spokes are provisioned by the platform team and consumed by application teams.

---

## 4. Architecture

### Hub-and-Spoke Topology

```
                    On-Premises
                        |
                   VPN / ExpressRoute
                        |
            ┌───────────────────────────┐
            │          HUB VNet          │
            │                           │
            │  ┌─────────────────────┐  │
            │  │   GatewaySubnet     │  │
            │  │  (VPN / ER Gateway) │  │
            │  └─────────────────────┘  │
            │                           │
            │  ┌─────────────────────┐  │
            │  │   AzureFirewallSubnet│  │
            │  │   (Azure Firewall)  │  │
            │  └─────────────────────┘  │
            │                           │
            │  ┌─────────────────────┐  │
            │  │  AzureBastionSubnet │  │
            │  │   (Azure Bastion)   │  │
            │  └─────────────────────┘  │
            │                           │
            │  ┌─────────────────────┐  │
            │  │   DNS / Monitoring  │  │
            │  └─────────────────────┘  │
            └───────────────────────────┘
                  |             |
           VNet Peering   VNet Peering
                  |             |
    ┌─────────────────┐   ┌─────────────────┐
    │  Spoke A VNet   │   │  Spoke B VNet   │
    │  (App Team A)   │   │  (App Team B)   │
    │                 │   │                 │
    │  ┌───────────┐  │   │  ┌───────────┐  │
    │  │  App Subnet│  │   │  │  App Subnet│  │
    │  └───────────┘  │   │  └───────────┘  │
    └─────────────────┘   └─────────────────┘
```

### Traffic Flow — Spoke to Internet

```
VM in Spoke A
     |
     ↓  UDR: 0.0.0.0/0 → Azure Firewall
Azure Firewall (Hub)
     |
     ↓  Application Rule: allow/deny FQDN
NAT Gateway
     |
     ↓  Known outbound IP
Internet
```

### Traffic Flow — Spoke A to Spoke B

```
VM in Spoke A
     |
     ↓  UDR: Spoke B CIDR → Azure Firewall
Azure Firewall (Hub)
     |
     ↓  Network Rule: allow/deny
VM in Spoke B
```

Without the UDR, traffic would attempt direct peering — which does not exist between Spokes.

Without the Firewall, traffic would bypass inspection.

### Traffic Flow — On-Premises to Spoke

```
On-Premises Server
     |
     ↓  VPN / ExpressRoute
VPN Gateway (Hub)
     |
     ↓  Route: Spoke CIDRs advertised
Azure Firewall (Hub)
     |
     ↓  Network Rule
VM in Spoke
```

### Ownership Boundaries

```
Platform Team owns:
┌──────────────────────────────────────────────────────────┐
│  Hub VNet                                                │
│  VPN / ExpressRoute Gateways                             │
│  Azure Firewall + Firewall Policy                        │
│  Azure Bastion                                           │
│  NAT Gateway                                             │
│  DNS Resolvers                                           │
│  VNet Peering (Hub side)                                 │
│  UDRs applied to Spoke subnets                           │
└──────────────────────────────────────────────────────────┘

Application Team owns:
┌──────────────────────────────────────────────────────────┐
│  Resources deployed inside their Spoke                   │
│  NSGs on their subnets (within platform-defined limits)  │
│  Application-level connectivity decisions                │
└──────────────────────────────────────────────────────────┘

Application Team does NOT own:
┌──────────────────────────────────────────────────────────┐
│  The Spoke VNet itself (provisioned by platform)         │
│  Peering configuration                                   │
│  Routes                                                  │
│  Firewall rules                                          │
│  Public IP addresses (in production)                     │
└──────────────────────────────────────────────────────────┘
```

---

## 5. Design Decisions

### Why Hub-and-Spoke and Not Mesh

Mesh networking creates a direct peering between every VNet that needs connectivity.

At 10 VNets: 45 peering connections.

At 100 VNets: 4,950 peering connections.

There is no centralised inspection point.

Adding a new firewall rule requires changes in every VNet.

A single rogue team can peer with any other team and bypass security controls.

Operations becomes unmanageable.

Hub-and-Spoke centralises control, reduces connection count to N peerings for N Spokes, and provides a single security inspection point.

**Rule:** Always Hub-and-Spoke at enterprise scale. Never mesh.

### What Belongs in the Hub

| Component | Reason |
|---|---|
| VPN / ExpressRoute Gateway | Single entry point for on-premises connectivity |
| Azure Firewall | Centralised security inspection for all traffic |
| Azure Bastion | Centralised VM access — no public IPs on VMs |
| NAT Gateway | Deterministic outbound IP for all Spoke traffic |
| Private DNS Resolvers | Centralised DNS for Private Endpoints |
| Monitoring agents | Single point for network flow logs |

Nothing that belongs to a specific application team goes in the Hub.

### What Belongs in the Spoke

Application VMs, App Service VNet Integration subnets, Private Endpoints for application-specific services.

Each Spoke is scoped to one application team or one Landing Zone.

### NSG vs Azure Firewall

| Capability | NSG | Azure Firewall |
|---|---|---|
| Layer 4 filtering | Yes | Yes |
| Layer 7 filtering | No | Yes |
| FQDN-based rules | No | Yes |
| TLS inspection | No | Yes |
| Threat intelligence | No | Yes |
| Centralised management | No | Yes |
| Cost | Free | Significant |

NSGs are a perimeter control for individual subnets.

Azure Firewall is the centralised inspection point for the network.

Both are required.

They are complementary, not alternatives.

**Common mistake:** Teams replace Azure Firewall with NSGs to reduce cost. This eliminates Layer 7 visibility and centralised inspection. Never do this in a regulated environment.

### When to Use NAT Gateway

Use NAT Gateway when:

- Applications make high volumes of outbound connections
- The security team needs to allowlist outbound IPs at partner firewalls
- You need to avoid SNAT port exhaustion

Do not rely on default Azure SNAT for production workloads.

Default SNAT uses shared IPs that can change and provides limited port capacity.

### Alternatives to Hub-and-Spoke

**Azure Virtual WAN** — Microsoft-managed hub-and-spoke with automated routing. Good for organisations with many regions or complex branch connectivity. Removes operational overhead of managing routing tables manually. More expensive. Less customisable.

**Mesh with NVA** — Third-party Network Virtual Appliances in every VNet. Works but creates management complexity and requires consistent NVA configuration across hundreds of VNets.

**Hub-and-Spoke remains the default** for organisations where the platform team wants full control over routing and firewall policy.

---

## 6. Real World Scenario

A manufacturing company is migrating 300 applications from on-premises data centres to Azure.

Applications run on Virtual Machines.

The company has factories connected via MPLS to their data centres.

Factories must remain connected after migration.

The security team requires that all internet-bound traffic from Azure is inspected.

The compliance team requires that no VM has a public IP address.

The infrastructure team is concerned that networking will slow down application onboarding.

**How the platform team designs the network:**

One Hub VNet is created in the primary region — West Europe.

The Hub contains:

- An ExpressRoute Gateway connecting to the MPLS network
- Azure Firewall with a centralised policy
- Azure Bastion for all VM access
- NAT Gateway for outbound internet traffic
- Azure Private DNS Resolver forwarding to on-premises DNS

For each application team, the platform team provisions a Spoke VNet.

The Spoke is peered to the Hub.

A UDR is applied to all Spoke subnets routing `0.0.0.0/0` to the Azure Firewall.

Application teams receive their Spoke with networking pre-configured.

They deploy their application resources inside the Spoke.

They do not configure routing.

They do not configure peering.

They do not open firewall rules — they submit a request to the platform team who evaluates and adds the rule to the central policy.

**Factory connectivity:**

Factories connect through the existing MPLS to the ExpressRoute circuit.

The ExpressRoute Gateway in the Hub advertises all Spoke CIDRs to on-premises.

Factories can reach applications directly.

Traffic from factories to Spokes passes through the Firewall for inspection.

**Outcome:**

300 application teams are onboarded without building their own networking.

All internet traffic is inspected.

No VM has a public IP address.

Bastion provides controlled access for all VM management.

The factory MPLS connectivity is preserved with no changes to on-premises routing.

---

## 7. Common Mistakes

### Mistake 1: Starting Networking Design with Subnets

Engineers think: "What subnets do I need?"

Platform engineers think: "What is the routing strategy? What is the security inspection model? Who owns what?"

Subnets are an output of the design, not the starting point.

**Why it is wrong:** Designing subnets first produces a network that works technically but fails operationally. You end up with no centralised inspection, no consistent routing, and no clear ownership.

### Mistake 2: Using NSGs as a Firewall Replacement

Teams suggest replacing Azure Firewall with NSGs to save cost.

NSGs cannot filter by FQDN, cannot perform TLS inspection, cannot detect threat intelligence indicators.

**Why it is wrong:** An NSG can block port 443 to an IP address. It cannot block `example-malware.com` while allowing `legitimate-service.com`. In a regulated environment this is a compliance failure, not a cost optimisation.

### Mistake 3: Allowing Application Teams to Create Peerings

If application teams can create VNet peerings, they can connect their VNet to any other VNet — including VNets outside their Landing Zone.

**Why it is wrong:** Peering bypasses the Hub and creates direct paths that circumvent Firewall inspection. All peering must be owned by the platform team.

### Mistake 4: Designing Networking for Today's Applications Only

Engineers design a `/24` VNet for 50 VMs because that is the current requirement.

Six months later the company acquires a business unit and needs to merge address spaces.

**Why it is wrong:** IP address space in Azure is not easily changed after deployment. Peering requires non-overlapping address spaces. Design for 3-5x growth from day one. Coordinate IP allocation centrally to avoid overlap during acquisitions.

### Mistake 5: Thinking Hub-and-Spoke is Unnecessary Complexity

Application teams argue that Hub-and-Spoke adds latency and slows them down.

**Why it is wrong:** The latency added by routing through Azure Firewall is typically under 1ms for intra-region traffic. The operational benefit — centralised security inspection, consistent routing, single ownership model — dramatically outweighs this. The complexity is in the platform team's domain. Application teams experience it as simplicity.

### Mistake 6: Placing Azure Bastion in a Spoke

Some engineers deploy Bastion in each Spoke for convenience.

**Why it is wrong:** Bastion is a shared service. It should be centralised in the Hub. One Bastion instance can access VMs across all peered Spokes when peering is configured with `Allow Gateway Transit`. Deploying per-Spoke multiplies cost and management overhead with no security benefit.

---

## 8. Troubleshooting

### Scenario 1: VM in Spoke Cannot Reach the Internet

**Symptoms:**

Application team reports that their VM cannot reach `apt.ubuntu.com` for package updates.

Curl to `8.8.8.8` times out.

**Possible Causes:**

- UDR is not applied to the VM's subnet
- Azure Firewall is blocking the traffic
- NAT Gateway is not associated with the subnet
- Azure Firewall application rule does not allow the FQDN

**Investigation:**

Check effective routes on the VM's network interface. Confirm `0.0.0.0/0` routes to the Azure Firewall private IP.

If the route is correct, check Azure Firewall logs — Network Rules log and Application Rules log — for deny entries matching the VM's private IP.

**Resolution:**

If UDR is missing: Apply the UDR route table to the subnet.

If Firewall is blocking: Add an application rule allowing the required FQDN or IP range. Do not create overly broad allow rules.

If NAT Gateway is missing: Associate a NAT Gateway to the subnet for outbound connectivity.

---

### Scenario 2: Spoke A Cannot Communicate with Spoke B

**Symptoms:**

Application team A reports they cannot reach a service in Application team B's Spoke.

Ping times out. TCP connection refused.

**Possible Causes:**

- No UDR routing Spoke B's CIDR through the Firewall
- Firewall Network Rule blocking the traffic
- NSG on Spoke B's subnet blocking inbound traffic from Spoke A
- VNet peering between the Spoke and Hub is not in Connected state

**Investigation:**

Check effective routes on the source VM. Confirm Spoke B's CIDR routes through the Firewall.

Check Azure Firewall logs for the connection attempt.

If the Firewall is allowing the traffic, check the NSG on Spoke B's subnet for inbound deny rules.

Check peering status — both peerings (Spoke A to Hub and Hub to Spoke B) must be in `Connected` state and both directions of each peering must be enabled.

**Resolution:**

If UDR is missing: Add a route for Spoke B's CIDR to the Firewall in the Spoke A route table.

If Firewall is blocking: Add a Network Rule permitting the traffic between the two application teams.

If NSG is blocking: Review whether the deny is intentional or misconfigured. Application teams may not have anticipated traffic from another Spoke.

---

### Scenario 3: On-Premises Servers Cannot Reach Azure VMs After Migration

**Symptoms:**

Factory servers report connection timeouts to VMs migrated to Azure.

VMs are confirmed running.

**Possible Causes:**

- ExpressRoute or VPN circuit is down
- Route advertisement is missing — Spoke CIDRs not propagated to on-premises
- UDR in the Spoke is overriding gateway routes
- Azure Firewall is blocking on-premises traffic

**Investigation:**

Check ExpressRoute/VPN Gateway connection status in the Azure portal.

Check the effective routes on the Spoke VM's NIC — confirm the on-premises CIDR is routable.

Check effective routes on an on-premises router — confirm Azure Spoke CIDRs are advertised.

Check Azure Firewall logs for drops from the factory IP range.

**Resolution:**

If gateway is down: Follow ExpressRoute/VPN troubleshooting procedures with the connectivity provider.

If routes are missing: Check that `Use Remote Gateway` is enabled on the Spoke peering and `Gateway Transit` is enabled on the Hub peering.

If UDR is overriding gateway routes: Review the UDR — adding a specific route for on-premises CIDRs via the Gateway, alongside the default route to the Firewall, resolves asymmetric routing.

If Firewall is blocking: Add Network Rules allowing the factory IP ranges to reach the application VM ports.

---

### Scenario 4: VM Access Via Bastion Fails

**Symptoms:**

Engineer cannot connect to a VM via Azure Bastion.

Browser returns a connection error.

**Possible Causes:**

- Bastion is not deployed or is in a failed state
- NSG on `AzureBastionSubnet` is blocking required ports
- NSG on the VM's subnet is blocking inbound from the Bastion subnet CIDR
- VM is deallocated

**Investigation:**

Check Bastion resource status in the portal.

Verify `AzureBastionSubnet` NSG — inbound rules must allow port 443 from the internet (GatewayManager service tag) and port 8080/5701 for internal Bastion health probes. Outbound rules must allow ports 22 and 3389 to the VNet.

Check the VM's NIC NSG — inbound from the Bastion subnet CIDR must be allowed on port 22 or 3389.

**Resolution:**

If NSG is misconfigured: Apply the required NSG rules per Microsoft's Bastion NSG documentation.

If VM NIC NSG is too restrictive: Add an inbound allow rule from the `AzureBastionSubnet` CIDR on port 22/3389.

---

## 9. Interview Questions

### Junior

**Q: What is a Virtual Network?**

A Virtual Network is a logically isolated network in Azure. Resources inside the VNet can communicate with each other by default. Resources in different VNets need explicit connectivity such as VNet Peering or a VPN Gateway.

**Q: What is VNet Peering?**

VNet Peering creates a private, low-latency connection between two VNets over the Microsoft backbone. Traffic does not traverse the public internet. Peering is non-transitive — if A peers with B and B peers with C, A cannot reach C through B without explicit routing.

**Q: What does an NSG do?**

An NSG is a stateful Layer 4 packet filter. It allows or denies inbound and outbound traffic based on IP, port, and protocol. It can be applied to subnets or individual network interfaces.

---

### Mid

**Q: What is a User Defined Route and why is it needed in Hub-and-Spoke?**

A User Defined Route overrides Azure's default system routing. In Hub-and-Spoke, a UDR is applied to Spoke subnets directing all traffic — `0.0.0.0/0` and peer Spoke CIDRs — to the Azure Firewall's private IP. Without this, traffic between Spokes would fail because peering is non-transitive. Traffic to the internet would bypass the Firewall. UDRs are what make the Hub-and-Spoke topology enforce centralised inspection.

**Q: Why is Azure Firewall preferable to NSGs for centralised security?**

NSGs operate at Layer 4. They can allow or deny based on IP and port but cannot inspect content. Azure Firewall operates at Layers 4 and 7, supports FQDN-based filtering, TLS inspection, and threat intelligence feeds. In an enterprise, you need to control what domains applications can reach, not just which ports. A malicious actor can use port 443 for exfiltration. NSGs cannot detect this. Azure Firewall can.

**Q: An application team wants to open port 443 from their VM to an external SaaS service. How do you handle this?**

The application team submits a request specifying the FQDN of the SaaS service. The platform team reviews the request and adds an application rule to the central Firewall policy allowing that FQDN over HTTPS from the application team's Spoke CIDR. The application team does not modify the Firewall. The rule is logged and version-controlled.

---

### Senior

**Q: Design enterprise networking for 300 applications across 20 teams. Walk me through your decisions.**

I would implement Hub-and-Spoke in each region with an active presence.

The Hub contains: an ExpressRoute Gateway for on-premises connectivity, Azure Firewall with a centralised policy, Azure Bastion, NAT Gateway, and Private DNS Resolvers.

Each application team receives a Spoke VNet provisioned as part of their Landing Zone. The Spoke is pre-peered to the Hub. A UDR routes all traffic through the Firewall. No VM has a public IP.

Firewall policy is structured in rule collection groups — a base policy owned by the platform team, and application-specific rule collections added through a request process. Firewall policy inheritance ensures the base policy always applies.

IP addressing is planned centrally. Each region has a `/16` supernet. Each Spoke receives a `/24`. I reserve space for 5x growth.

DNS uses Azure Private DNS Resolvers in the Hub forwarding to on-premises DNS for private DNS resolution across the hybrid boundary.

**Q: An application team argues that Hub-and-Spoke adds latency and slows their deployments. How do you respond?**

The latency concern is valid but the numbers are not significant. Azure Firewall adds approximately 1ms of latency per hop for intra-region traffic. For most enterprise applications — internal systems, ERP, databases — this is immaterial.

The deployment concern is more legitimate. Hub-and-Spoke introduces a dependency on the platform team to open firewall rules. The solution is process, not architecture. A well-designed request process with automated rule deployment through Terraform should resolve firewall rule requests in minutes, not days.

The alternative — removing the Firewall — eliminates the centralised inspection point that the security team requires. For a regulated organisation this is not negotiable. The architecture is correct. The operations process is what needs to improve.

**Q: Your company is acquiring another company. Their Azure tenants use overlapping IP ranges. How do you handle it?**

This is one of the most difficult networking challenges in enterprise Azure.

Overlapping CIDRs prevent direct VNet peering.

Options:

**Azure NAT at the boundary** — Deploy Network Virtual Appliances that translate source IPs to non-overlapping ranges. This works but adds latency and operational complexity.

**Azure Private Link** — Expose specific services from the acquired company as Private Endpoints in your VNets. No full VNet connectivity, but specific services become accessible without IP conflicts. This is usually the cleanest option for a discrete set of integration points.

**Re-addressing one side** — If the acquisition is early-stage, re-IP the smaller estate. This is disruptive but produces a clean architecture long-term.

In practice, the answer depends on scale. For a handful of services, Private Link is fastest. For full network integration, re-addressing the acquired estate is the long-term correct answer.

---

### Principal

**Q: Describe the trade-offs between Azure Virtual WAN and manually managed Hub-and-Spoke.**

Virtual WAN is a Microsoft-managed connectivity service that automates routing between Hubs, branches, and Spokes. It removes the need to manage UDRs manually, handles transitive routing automatically, and simplifies multi-region networking.

The trade-offs:

**Control:** Virtual WAN abstracts the routing layer. You cannot see or modify the underlying routing tables. If you need fine-grained control — custom BGP communities, asymmetric routing for specific workloads — Virtual WAN constrains you.

**Cost:** Virtual WAN introduces per-connection and per-unit charges on top of gateway and Firewall costs. At small scale it is more expensive per connection than self-managed Hub-and-Spoke.

**Customisation:** Virtual WAN supports a curated set of NVA partners for security. If your organisation has an existing investment in a third-party firewall not in the Virtual WAN ecosystem, you may need significant workarounds.

**Operational simplicity:** Virtual WAN genuinely reduces the operational burden of managing routing tables across many regions. For organisations with 10+ regions or complex branch connectivity, this is a significant advantage.

**My recommendation:** If the organisation has complex multi-region requirements and is comfortable with reduced routing control, Virtual WAN. If the organisation is single-region or has non-standard routing requirements, manually managed Hub-and-Spoke with full control over Firewall policy and UDRs.

**Q: How does network ownership interact with the Landing Zone model at scale?**

The Landing Zone model separates the platform team's responsibility — network infrastructure — from the application team's responsibility — application resources.

In a mature platform, this boundary must be enforced technically, not just by convention.

Technically: the platform team owns the subscription at Owner scope. The VNet is provisioned by the platform and locked so application teams cannot modify peering, routes, or delete the VNet. RBAC scopes application teams to Resource Groups containing their application resources, not to the networking infrastructure.

At scale, the tension emerges around Firewall rule management. Every new application requires firewall rules. If the platform team processes these manually, they become a bottleneck.

The solution is self-service with guardrails. Application teams submit firewall rule requests through an automated pipeline — a Terraform PR to a network policy repository. The platform team reviews and approves. Automation deploys. The platform team owns the policy. The application team owns the request.

This preserves the ownership boundary while removing the bottleneck. The platform team moves from provisioner to reviewer.

At very large scale — thousands of application teams — even this model stresses. The next evolution is policy-as-code with automated compliance checks: the platform team defines allowed FQDN categories, and application teams self-approve rules that fall within pre-approved categories. Rules outside the categories require manual review.

The key insight is that ownership boundaries must evolve with scale. What works at 10 teams fails at 100 teams. The architecture should be designed so the ownership model can evolve without requiring a re-architecture.

---

## 10. Interactive Exercise

### Architectural Challenge

A logistics company is building a new Azure environment.

They have three business units:

- **Logistics Operations** — internal systems, 50 applications, connected to warehouse management systems on-premises via MPLS
- **Customer Portal** — internet-facing, 10 applications, handles customer orders
- **Data Platform** — internal analytics, 5 applications, processes data from both other units

**Requirements:**

1. All internet-bound traffic from Logistics Operations and Data Platform must be inspected by a centralised firewall.
2. Customer Portal applications must be able to reach the internet but must not be able to initiate connections to Logistics Operations or Data Platform systems.
3. Data Platform must be able to pull data from both Logistics Operations and Customer Portal, but neither should be able to push to Data Platform.
4. Warehouse systems on-premises must reach Logistics Operations applications in Azure.
5. No VM in any environment should have a public IP address.
6. The security team requires that all outbound internet traffic uses a known, fixed set of IP addresses.

**Your task:**

Design the Hub-and-Spoke topology for this company.

Specify what lives in the Hub.

Specify how many Spokes are needed and what their purpose is.

Explain how you enforce the directional traffic restrictions between business units using Firewall rules and NSGs.

Explain how on-premises connectivity is implemented.

Explain how the fixed outbound IP requirement is satisfied.

Identify the most operationally complex aspect of your design and explain how you would manage it long-term.

---

*There is no single correct answer.*

*Evaluation focuses on: clarity of the Hub vs Spoke separation, correctness of routing strategy, use of Firewall rules to enforce directionality, operational reasoning, and ability to justify each decision in terms of the business requirements.*
