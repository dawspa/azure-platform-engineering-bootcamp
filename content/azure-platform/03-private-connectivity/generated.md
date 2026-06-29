---
Title: Private Connectivity
Domain: Azure Platform Engineering
Difficulty: Senior
Estimated Duration: 120 minutes
Prerequisites:
  - Module 01 — Enterprise Azure Platform
  - Module 02 — Enterprise Azure Networking
  - Basic understanding of DNS resolution
  - Basic understanding of IP addressing
Learning Objectives:
  - Explain what a Private Endpoint is and how it works
  - Explain the role of DNS in Private Endpoint connectivity
  - Describe the difference between Private Endpoint and Service Endpoint
  - Explain Private Link and how it relates to Private Endpoints
  - Describe Private DNS Zone design for enterprise environments
  - Explain hybrid DNS resolution across on-premises and Azure
  - Troubleshoot Private Endpoint connectivity failures
  - Justify Private Endpoint over Service Endpoint for regulated workloads
Interview Focus:
  - Senior Platform Engineer
  - Principal Platform Engineer
---

# Module 03 — Private Connectivity

---

## 1. Learning Goal

After completing this module you will be able to explain how private connectivity works in Azure from first principles.

You will understand why DNS is the foundation of Private Endpoint — not the network path.

You will be able to design Private DNS resolution for hybrid environments, troubleshoot connectivity failures, and compare Private Endpoint against Service Endpoint with clear justification.

You will be prepared to explain this architecture on a whiteboard during a Senior Platform Engineer interview.

---

## 2. Why This Exists

### The Problem

Azure PaaS services — Storage Accounts, Key Vault, SQL Database, Service Bus — are accessed over the public internet by default.

Their hostnames resolve to public IP addresses.

Traffic flows through the public internet even when the source is inside a VNet.

In a regulated enterprise environment this creates several problems:

- Security teams cannot guarantee that data never traverses the public internet
- Audit findings require that sensitive data — encryption keys, database credentials, financial records — is not accessible over public endpoints
- Compliance frameworks such as PCI-DSS and ISO 27001 require network-level access controls on sensitive services
- Public endpoints can be targeted by denial-of-service attacks or credential stuffing

The earliest mitigation was to restrict access to specific IP ranges using storage firewall rules.

This was insufficient.

IP-based restrictions cannot guarantee traffic stays on private network paths.

An application with the correct IP allowlisted could still transmit data over a route that traverses public infrastructure.

### What Microsoft Introduced

**Service Endpoints** were introduced first.

A Service Endpoint extends a VNet's private address space to the Azure service over the Microsoft backbone.

Traffic stays on the Microsoft network rather than the public internet.

The Azure service can restrict access to specific VNet subnets.

This was an improvement but had significant limitations.

**Private Endpoints** were introduced to resolve those limitations.

A Private Endpoint places a private IP address from a VNet subnet directly in front of an Azure PaaS service.

The service is reachable exclusively through that private IP.

The public endpoint can be disabled entirely.

Traffic from inside the VNet — and from on-premises via ExpressRoute or VPN — reaches the service through the private IP without ever touching the public internet.

### Why Service Endpoints Were Insufficient

Service Endpoints do not provide a private IP for the service.

The service's public IP remains accessible from the internet.

Access controls are applied at the service layer — a resource can be allowed from specific VNets — but the network path is not truly private.

From on-premises, traffic to a Service Endpoint still routes to the service's public endpoint unless expensive custom routing is configured.

Service Endpoints do not work with Private DNS Zones.

Private Endpoints provide a truly private, IP-level address for the service inside the VNet.

The public endpoint can be disabled.

The architecture is consistent across hybrid environments.

---

## 3. Core Concepts

### Private Endpoint

A Private Endpoint is a network interface with a private IP address from a VNet subnet, connected to a specific Azure PaaS resource via Private Link.

When a Private Endpoint is created for a Storage Account, Azure assigns it a private IP — for example `10.0.1.5`.

From inside the VNet, the Storage Account is reachable at that IP.

The private IP is only reachable from:

- Resources inside the same VNet
- Resources in peered VNets with appropriate routing
- On-premises resources connected via ExpressRoute or VPN

The Storage Account's public endpoint can be completely disabled, making it unreachable from the internet.

### Private Link

Private Link is the underlying service that enables Private Endpoints.

Private Link exposes Azure PaaS services — and custom services — as private endpoints inside VNets.

You can also publish your own services via Private Link Service, allowing other organisations to create Private Endpoints to your service without VNet peering.

### DNS — The Critical Component

Creating a Private Endpoint is necessary but not sufficient.

Applications do not connect to IP addresses directly.

They connect to hostnames.

When an application connects to `mystorageaccount.blob.core.windows.net`, DNS resolves that name to an IP address.

Without DNS configuration, the hostname resolves to the Storage Account's public IP — even when a Private Endpoint exists.

The application would connect to the public endpoint, bypassing the Private Endpoint entirely.

**Private DNS Zones** solve this.

An Azure Private DNS Zone `privatelink.blob.core.windows.net` is created.

An A record is added: `mystorageaccount` → `10.0.1.5`.

The Private DNS Zone is linked to the VNet.

Now when a VM in the VNet queries `mystorageaccount.blob.core.windows.net`, Azure DNS resolves through the CNAME chain to `mystorageaccount.privatelink.blob.core.windows.net` and then to `10.0.1.5`.

The application connects to the private IP.

The public endpoint is never involved.

### Private DNS Zones

Each Azure service uses a specific Private DNS Zone hostname suffix.

| Service | Private DNS Zone |
|---|---|
| Blob Storage | privatelink.blob.core.windows.net |
| File Storage | privatelink.file.core.windows.net |
| Key Vault | privatelink.vaultcore.azure.net |
| SQL Database | privatelink.database.windows.net |
| App Service | privatelink.azurewebsites.net |
| Service Bus | privatelink.servicebus.windows.net |
| Container Registry | privatelink.azurecr.io |

Each zone is created once.

Each Private Endpoint adds one A record to the zone.

The zone is linked to the Hub VNet.

All Spokes inherit resolution through Hub peering when Azure DNS is configured correctly.

### Service Endpoint

A Service Endpoint optimises the network path from a VNet subnet to an Azure service.

It routes traffic over the Microsoft backbone rather than the public internet.

It does not create a private IP for the service.

It does not prevent public internet access to the service.

It extends network rules on the service — you can allow access from specific subnets.

Service Endpoints are simpler and cheaper than Private Endpoints.

They are appropriate for non-regulated workloads where the goal is network path optimisation, not true private access.

### Hybrid DNS Resolution

In a hybrid environment, DNS must work for both Azure-hosted resources and on-premises resources.

On-premises DNS servers typically cannot resolve Azure Private DNS Zone records because they cannot query Azure's internal DNS resolver directly.

Two approaches exist:

**DNS Forwarders in Azure:** Deploy DNS forwarders — often Azure Private DNS Resolver — in the Hub VNet. Configure on-premises DNS servers to forward Azure-specific queries to these forwarders. The forwarders query Azure DNS, which resolves Private DNS Zone records.

**Conditional Forwarding:** On-premises DNS forwards queries for `privatelink.*` and `azure.com` domains to Azure DNS Resolver. Azure DNS Resolver returns the private IP. On-premises applications reach Azure services over ExpressRoute or VPN using the private IP.

Without this configuration, on-premises applications cannot resolve Private Endpoint hostnames to private IPs. They resolve to public IPs and either fail — if the public endpoint is disabled — or bypass the private path.

### Split-Brain DNS

Split-brain DNS refers to a DNS configuration where the same hostname resolves differently depending on the source of the query.

From inside Azure: `mystorageaccount.blob.core.windows.net` → `10.0.1.5` (private IP via Private DNS Zone)

From the internet: `mystorageaccount.blob.core.windows.net` → `52.x.x.x` (public IP)

This is intentional and correct.

Internal traffic reaches the private path.

External traffic — if the public endpoint is not disabled — reaches the public path.

If the public endpoint is disabled, external resolution succeeds but connection fails — which is the desired security outcome.

---

## 4. Architecture

### DNS Resolution Chain for Private Endpoint

```
Application VM (inside VNet)
        |
        ↓  Query: mystorageaccount.blob.core.windows.net
Azure DNS (168.63.129.16)
        |
        ↓  CNAME: mystorageaccount.blob.core.windows.net
        |       → mystorageaccount.privatelink.blob.core.windows.net
Private DNS Zone: privatelink.blob.core.windows.net
        |
        ↓  A Record: mystorageaccount → 10.0.1.5
Application VM
        |
        ↓  TCP connection to 10.0.1.5 (Private Endpoint NIC)
Private Endpoint NIC (inside VNet subnet)
        |
        ↓  Private Link connection
Azure Storage Account
```

If the Private DNS Zone is not linked to the VNet or the A record is missing, the CNAME resolves to the public IP instead of `10.0.1.5`.

### DNS Resolution Chain — Hybrid (On-Premises)

```
On-Premises Application Server
        |
        ↓  Query: mystorageaccount.blob.core.windows.net
On-Premises DNS Server
        |
        ↓  Conditional Forward: *.blob.core.windows.net → Azure DNS Resolver IP
Azure Private DNS Resolver (Hub VNet — Inbound Endpoint)
        |
        ↓  Queries Azure DNS on behalf of on-premises
Azure DNS (168.63.129.16)
        |
        ↓  Resolves via Private DNS Zone: 10.0.1.5
On-Premises Application Server
        |
        ↓  TCP connection to 10.0.1.5 via ExpressRoute / VPN
Private Endpoint NIC
        |
        ↓  Private Link connection
Azure Storage Account
```

### Hub-Centric Private DNS Design

```
Hub VNet
  ├── AzurePrivateDNSResolver Subnet
  │     ├── Inbound Endpoint (on-premises queries)
  │     └── Outbound Endpoint (Azure → on-premises queries)
  │
  └── Private DNS Zone Links
        ├── privatelink.blob.core.windows.net → linked to Hub VNet
        ├── privatelink.vaultcore.azure.net → linked to Hub VNet
        ├── privatelink.database.windows.net → linked to Hub VNet
        └── (one zone per service)

Spoke VNets
  └── No Private DNS Zone links required
      (Resolution inherited via Hub peering when
       "Use Remote DNS" is enabled and Hub VNet is
       the DNS server for Spokes)
```

### Private Endpoint vs Service Endpoint

```
Service Endpoint                    Private Endpoint

VNet                                VNet
  |                                   |
  | Route optimised to                | Private IP (e.g. 10.0.1.5)
  | Microsoft backbone                | assigned in subnet
  |                                   |
  ↓                                   ↓
Azure Service                       Private Endpoint NIC
(still accessed via public IP)          |
                                        ↓ Private Link
                                    Azure Service
                                    (public endpoint can be disabled)
```

---

## 5. Design Decisions

### Private Endpoint vs Service Endpoint — When to Use Each

| Factor | Private Endpoint | Service Endpoint |
|---|---|---|
| True private IP | Yes | No |
| Public endpoint can be disabled | Yes | No |
| Works from on-premises | Yes (via ExpressRoute/VPN + DNS) | Limited |
| DNS configuration required | Yes | No |
| Cost | Additional charge per endpoint | Free |
| Complexity | Higher | Lower |
| Required for regulated workloads | Yes | Insufficient |

**Decision rule for enterprises:**

Use Private Endpoints for all services containing sensitive data — Key Vault, Storage with credentials or financial data, SQL Database, Service Bus in production.

Service Endpoints are acceptable for lower-sensitivity services in non-regulated environments where the goal is performance rather than true isolation.

In practice, most enterprise platform teams standardise on Private Endpoints for all PaaS services in production. The operational complexity is manageable at scale through Terraform automation.

### Centralised vs Distributed Private DNS Zones

**Centralised (recommended for enterprise):**

One Private DNS Zone per service type.

Hosted in a central subscription owned by the platform team.

Linked to the Hub VNet.

All Spokes resolve through the Hub.

All Private Endpoint A records are in the central zones.

Pro: Single source of truth. Consistent resolution. Easy to audit.

Con: Platform team owns all DNS record lifecycle. Requires automation for scale.

**Distributed:**

Each application team creates their own Private DNS Zone in their Landing Zone subscription.

Pro: Team autonomy. No dependency on platform team for DNS record creation.

Con: VNet links must be created to every Spoke for every zone. DNS resolution becomes inconsistent. Auditing is impossible at scale.

**Recommendation:** Always centralised. DNS in enterprise is infrastructure, not application configuration. The platform team owns it.

### Disabling Public Endpoints

Where Private Endpoints are in use, disable the public endpoint on the Azure service.

The public endpoint being enabled creates residual risk — misconfigured DNS or a network path change could cause traffic to fall back to the public endpoint.

Disabling it enforces private-only access at the service level, independent of DNS configuration.

Some services allow this via `publicNetworkAccess: Disabled`.

Enforce this through Azure Policy — deny creation of Storage Accounts or Key Vaults with public network access enabled in production subscriptions.

---

## 6. Real World Scenario

A healthcare company is migrating patient data management applications to Azure.

The applications store medical records in Azure Blob Storage and retrieve encryption keys from Azure Key Vault.

The Data Protection Officer has mandated:

- Patient data must not traverse the public internet at any point
- Storage Accounts must not be accessible from the internet
- Key Vault must not be accessible from the internet
- On-premises clinical systems must be able to reach both services
- All access attempts must be logged

**Platform team design:**

Private Endpoints are created for each Storage Account and Key Vault.

Private Endpoints are deployed into a dedicated subnet in each application's Spoke VNet.

A centralised `privatelink.blob.core.windows.net` and `privatelink.vaultcore.azure.net` Private DNS Zone is hosted in the platform team's connectivity subscription.

A records are registered for each Private Endpoint.

Both zones are linked to the Hub VNet.

An Azure Private DNS Resolver is deployed in the Hub with an Inbound Endpoint.

On-premises DNS is configured to conditionally forward:

- `*.blob.core.windows.net` → Azure Private DNS Resolver inbound IP
- `*.vaultcore.azure.net` → Azure Private DNS Resolver inbound IP

Public network access is disabled on all Storage Accounts and Key Vaults.

An Azure Policy is assigned at the Management Group level denying creation of any Storage Account or Key Vault with public network access enabled.

Diagnostic logging is enabled on both services — all data plane operations are sent to a central Log Analytics Workspace.

**What the application team experiences:**

They deploy their application.

DNS resolution for `medicalrecords.blob.core.windows.net` returns `10.0.2.4` — the Private Endpoint IP.

Their application connects to Storage over a private path.

Key Vault is reached at `patientvault.privatelink.vaultcore.azure.net` — `10.0.2.5`.

The public endpoint cannot be reached from anywhere — the Azure Portal confirms `Public network access: Disabled`.

On-premises clinical systems connect over ExpressRoute. DNS forwards to Azure Private DNS Resolver. Private IPs are returned. Connections succeed.

**Audit outcome:**

The DPO runs a compliance report. Azure Policy compliance shows 100% — no Storage Account or Key Vault with public access enabled exists. Diagnostic logs confirm all access originates from private IPs.

---

## 7. Common Mistakes

### Mistake 1: Creating a Private Endpoint Without Configuring DNS

An engineer creates a Private Endpoint for a Storage Account.

They test connectivity by pinging `mystorageaccount.blob.core.windows.net`.

The ping fails.

They conclude the Private Endpoint is not working.

**Why it is wrong:** The Private Endpoint is working correctly. DNS is not configured. The hostname still resolves to the public IP. Until a Private DNS Zone is linked to the VNet with the correct A record, applications will attempt to reach the public endpoint.

**Rule:** Always verify DNS resolution after creating a Private Endpoint. Run `nslookup` from a VM inside the VNet. Confirm the returned IP is the Private Endpoint's private IP.

### Mistake 2: Confusing Service Endpoints with Private Endpoints

An engineer is asked to make a Storage Account private.

They enable the Service Endpoint for `Microsoft.Storage` on the VNet subnet.

They configure the Storage Account firewall to allow the VNet.

They report the task is complete.

**Why it is wrong:** The Storage Account public endpoint is still accessible from the internet. Anyone with the storage account URL can still attempt access. Service Endpoints do not provide a private IP. They do not disable the public endpoint. For data that must not be reachable from the internet, Private Endpoints are required.

### Mistake 3: Linking Private DNS Zones to Every Spoke Individually

An engineer creates Private DNS Zones in the connectivity subscription.

For each new Spoke, they manually create a VNet link.

At 100 Spokes, they have 100 VNet links per zone.

DNS queries from Spokes resolve correctly.

**Why it is wrong operationally:** This approach works technically but creates significant management overhead. The correct design is DNS forwarding from Spokes to Azure DNS through the Hub. When Spoke VNets use the Hub's Azure DNS Resolver as their DNS server, resolution for Private DNS Zones linked to the Hub VNet works without individual zone links to each Spoke.

### Mistake 4: Forgetting On-Premises DNS Configuration

A platform team deploys Private Endpoints and confirms everything works from Azure VMs.

On-premises applications still fail to reach the Storage Account.

The team concludes the ExpressRoute circuit has an issue.

**Why it is wrong:** The network path is fine. DNS is the problem. On-premises DNS servers forward `blob.core.windows.net` queries to public DNS resolvers which return public IPs. The connection either fails because the public endpoint is disabled, or — worse — succeeds by routing through the internet rather than ExpressRoute. Configure conditional forwarding on on-premises DNS servers to forward private link domains to Azure Private DNS Resolver.

### Mistake 5: Using a Single Private Endpoint for Multiple Consumers

An engineer creates one Private Endpoint for a shared Storage Account in the Hub.

All application Spokes resolve to the same private IP.

**Why it is wrong in certain architectures:** This works technically when all Spokes are in the same DNS resolution domain. However, if Spokes are in different Azure regions or DNS zones, a single Private Endpoint's IP may not be reachable from all Spokes without correct routing and DNS. Evaluate Private Endpoint placement per consumer location.

### Mistake 6: Not Disabling the Public Endpoint After Creating Private Endpoint

A team creates a Private Endpoint and considers the work done.

The public endpoint remains enabled.

Six months later, a misconfigured UDR causes internet routing. The application falls back to the public endpoint and data exfiltration occurs.

**Why it is wrong:** Defence in depth requires disabling the public endpoint. Private Endpoint + disabled public endpoint provides two independent security controls. Private Endpoint alone provides one. Always disable the public endpoint.

---

## 8. Troubleshooting

### Scenario 1: Application Cannot Connect to Storage Account via Private Endpoint

**Symptoms:**

Application receives `Connection refused` or a TLS certificate error when connecting to the Storage Account.

The Private Endpoint is shown as `Succeeded` in the Azure Portal.

**Possible Causes:**

- DNS not resolving to the private IP
- Private DNS Zone not linked to the VNet
- A record missing or incorrect in the Private DNS Zone
- NSG blocking traffic to the Private Endpoint subnet
- Private Endpoint connection is in `Pending` state (requires manual approval)

**Investigation:**

From a VM inside the same VNet, run:

```
nslookup mystorageaccount.blob.core.windows.net
```

If the result is a public IP (e.g. `52.x.x.x`): DNS is not configured correctly. The Private DNS Zone is not linked or the A record is missing.

If the result is a private IP (e.g. `10.0.x.x`): DNS is correct. Check NSG rules on the Private Endpoint subnet — port 443 inbound must be allowed from application subnets.

Check the Private Endpoint connection state in the Portal. If `Pending`, the resource owner must approve the connection.

**Resolution:**

DNS issue: Link the Private DNS Zone to the VNet. Verify the A record exists and points to the Private Endpoint's NIC IP.

NSG issue: Add an inbound allow rule on port 443 from the application subnet CIDR to the Private Endpoint subnet.

Pending approval: Navigate to the target resource's Private Endpoint connections tab and approve the connection.

---

### Scenario 2: On-Premises Application Fails to Resolve Private Endpoint Hostname

**Symptoms:**

On-premises application connects to the Storage Account.

Connection succeeds inconsistently — sometimes using the private path, sometimes the public path.

Occasionally connects to the public endpoint — which the security team has flagged as a violation.

**Possible Causes:**

- On-premises DNS not configured to forward to Azure Private DNS Resolver
- Conditional forwarding is configured but only for some zones, not all required suffixes
- Azure Private DNS Resolver Inbound Endpoint is not reachable from on-premises
- NSG on the Azure Private DNS Resolver subnet blocks DNS queries from on-premises IP ranges

**Investigation:**

From on-premises, run:

```
nslookup mystorageaccount.blob.core.windows.net <dns-server-IP>
```

Compare results from:
- On-premises DNS server
- Azure Private DNS Resolver IP directly

If on-premises DNS returns a public IP but Azure resolver returns a private IP: Conditional forwarding is missing or incorrect on the on-premises DNS server.

If Azure resolver also returns a public IP: The Private DNS Zone link to the Hub VNet is missing or the A record is absent.

Check NSG on the Azure Private DNS Resolver subnet — port 53 UDP/TCP inbound must be allowed from on-premises IP ranges.

**Resolution:**

Configure conditional forwarders on on-premises DNS for each required `privatelink.*` suffix and any other Azure-internal domains, pointing to the Azure Private DNS Resolver Inbound Endpoint IP.

Verify ExpressRoute or VPN connectivity to the Resolver subnet.

Add NSG rule allowing port 53 from on-premises IP ranges to the Resolver subnet.

---

### Scenario 3: Private Endpoint Created but Resource Still Accessible from Internet

**Symptoms:**

Security scan reports that a Storage Account is accessible from the internet.

A Private Endpoint exists and is working.

**Possible Causes:**

- Public network access is still `Enabled` on the Storage Account
- The Storage Account firewall allows specific public IPs or VNet Service Endpoints in addition to Private Endpoint
- Policy enforcing `publicNetworkAccess: Disabled` is in Audit mode, not Deny

**Investigation:**

In the Azure Portal, navigate to the Storage Account → Networking.

Confirm whether `Public network access` is set to `Disabled` or `Enabled from selected virtual networks and IP addresses`.

Review whether Storage Account firewall rules allow any public IP ranges.

Check Azure Policy compliance — verify the policy effect is `Deny` not `Audit`.

**Resolution:**

Set `Public network access: Disabled` on the Storage Account.

Remove any Storage Account firewall rules that allow public IP ranges.

Change the Azure Policy effect from `Audit` to `Deny` to prevent future resources from being created with public access enabled.

---

### Scenario 4: Intermittent DNS Resolution Failure for Private Endpoint Hostname

**Symptoms:**

DNS resolution succeeds most of the time.

Periodically the application returns `Name not resolved` errors.

The errors last 30–120 seconds and then resolve.

**Possible Causes:**

- DNS resolver is overloaded or unavailable momentarily
- Azure Private DNS Resolver scaling event causing brief interruption
- TTL on cached records expired and re-query fails during a transient DNS issue
- Multiple Private DNS Zones with conflicting records

**Investigation:**

Enable diagnostic logging on Azure Private DNS Resolver.

Review DNS query logs for failed resolution events during the outage window.

Check Azure Private DNS Resolver availability metrics in Azure Monitor.

Review all Private DNS Zones — confirm no duplicate A records exist across zones for the same hostname.

**Resolution:**

Ensure Azure Private DNS Resolver is deployed with both Inbound and Outbound Endpoints across availability zones.

Review DNS record TTLs — very short TTLs increase DNS query volume and amplify transient failures. Use 300–600 seconds for Private Endpoint A records.

Remove duplicate zone configurations if present.

---

## 9. Interview Questions

### Junior

**Q: What is a Private Endpoint?**

A Private Endpoint is a network interface with a private IP address from a VNet subnet, created for a specific Azure PaaS service. It allows resources inside the VNet — and on-premises resources connected via ExpressRoute or VPN — to reach the Azure service over a private IP address without using the public internet. The service's public endpoint can be disabled entirely.

**Q: What is the difference between a Private Endpoint and a Service Endpoint?**

A Service Endpoint optimises the routing path from a VNet subnet to an Azure service over the Microsoft backbone but does not assign a private IP to the service. The service's public endpoint remains accessible from the internet. A Private Endpoint assigns a private IP from the VNet to the service. The public endpoint can be disabled. Private Endpoints provide true private access. Service Endpoints provide optimised routing but not isolation.

**Q: Why is DNS important for Private Endpoints?**

Applications connect to services using hostnames, not IP addresses. Without DNS configuration, the hostname resolves to the service's public IP even when a Private Endpoint exists. The Private DNS Zone creates a record mapping the service hostname to the Private Endpoint's private IP. Without this, the Private Endpoint is unused. DNS is what causes applications to use the private path.

---

### Mid

**Q: Walk me through the DNS resolution chain when an application connects to a Storage Account via Private Endpoint.**

The application queries `mystorageaccount.blob.core.windows.net`. Azure DNS receives the query. The public DNS for that name has a CNAME to `mystorageaccount.privatelink.blob.core.windows.net`. Azure DNS checks whether a Private DNS Zone `privatelink.blob.core.windows.net` is linked to the querying VNet. It is. The zone has an A record: `mystorageaccount` → `10.0.1.5`. Azure DNS returns `10.0.1.5`. The application opens a TCP connection to `10.0.1.5`, which is the Private Endpoint NIC in the VNet. The connection flows through Private Link to the Storage Account. The public internet is never involved.

**Q: A developer reports that their application cannot connect to a Key Vault Private Endpoint. How do you start troubleshooting?**

DNS first. From a VM in the same VNet, I run `nslookup myvault.vault.azure.net`. If the result is a public IP, DNS is the problem — the Private DNS Zone is missing, not linked to the VNet, or the A record is absent. I check the zone in the portal, verify the VNet link exists, and verify the A record points to the correct Private Endpoint NIC IP.

If DNS returns the correct private IP, the network path is the problem. I check NSG rules on the Private Endpoint subnet — port 443 must be allowed from the application subnet. I check whether the Private Endpoint connection is in the `Approved` state. If it is `Pending`, the Key Vault owner must approve it.

**Q: How would you design Private DNS for an enterprise with 100 application Spokes?**

Centralised. One Private DNS Zone per service type, hosted in the platform team's connectivity subscription. All zones linked to the Hub VNet. Azure Private DNS Resolver deployed in the Hub with an Inbound Endpoint for on-premises queries.

Spoke VNets do not have individual zone links. Instead, all Spoke VMs use the Hub-resident Azure DNS (168.63.129.16) or the Private DNS Resolver as their DNS server. When they query a private endpoint hostname, the resolution flows through the Hub's DNS configuration and returns the private IP.

This scales to any number of Spokes without zone-link management overhead. Adding a new Private Endpoint adds one A record to one zone. The new endpoint is immediately resolvable from all Spokes.

---

### Senior

**Q: Design the private connectivity architecture for a regulated financial services platform migrating 200 applications to Azure. Key Vault and SQL Database must be private. On-premises branch offices must reach both.**

The architecture:

All Key Vaults and SQL Databases have Private Endpoints deployed into dedicated Private Endpoint subnets within each application's Spoke.

Centralised Private DNS Zones are created in the connectivity subscription:
- `privatelink.vaultcore.azure.net`
- `privatelink.database.windows.net`

Both zones are linked to the Hub VNet only.

Azure Private DNS Resolver is deployed in the Hub with an Inbound Endpoint. On-premises DNS servers have conditional forwarders for these domains pointing to the Resolver's Inbound Endpoint IP.

The ExpressRoute connection from on-premises terminates at the Hub VPN Gateway. Routes for all Spoke CIDRs — including Private Endpoint subnets — are advertised to on-premises.

Public network access is disabled on all Key Vaults and SQL Databases. An Azure Policy in Deny mode enforces this at the Landing Zones Management Group.

A record registration in the Private DNS Zones is automated — Terraform modules that provision Private Endpoints include a resource block creating the DNS A record in the centralised zone.

**Q: An application team asks whether they can use Service Endpoints for their SQL Database instead of Private Endpoints because they are simpler. How do you respond?**

I acknowledge that Service Endpoints are simpler to configure.

Then I explain why they are insufficient for production regulated workloads.

Service Endpoints do not create a private IP. The SQL Database public endpoint remains reachable from the internet. An attacker with valid credentials can still reach the database from outside Azure — the only protection is the database-level firewall restricting access to the VNet, which is a weaker control than disabling the public endpoint entirely.

Compliance frameworks — PCI-DSS, ISO 27001 — require that databases containing sensitive data are not accessible from public networks. Service Endpoints do not satisfy this requirement. Private Endpoints do.

For production regulated environments, the additional configuration overhead of Private Endpoints is non-negotiable. We automate it through Terraform so the application team experiences the same simplicity. They get a Landing Zone with Private Endpoints pre-configured. They do not configure it themselves.

---

### Principal

**Q: How would you design a scalable Private DNS architecture for an enterprise with 500 application teams, 15 Azure services using Private Endpoints, and 50 on-premises sites?**

The architecture has four layers.

**Layer 1 — DNS Zones:** One Private DNS Zone per Azure service suffix, hosted in the platform team's DNS subscription. 15 zones, one for each service type. Never distributed across application subscriptions. Managed by the platform team via Terraform. Zone records are created automatically when Terraform modules provision Private Endpoints.

**Layer 2 — Hub Resolution:** All zones are linked to the Hub VNet. Azure Private DNS Resolver is deployed in the Hub in zone-redundant configuration. Two Inbound Endpoints — one per availability zone — to survive zone failures. One Outbound Endpoint for forwarding to on-premises DNS when Azure VMs need to resolve on-premises hostnames.

**Layer 3 — Spoke Resolution:** Spoke VMs use `168.63.129.16` as their DNS server. Azure DNS resolves private zone records because the Hub's zone links cover all required zones. No per-Spoke zone links are needed.

**Layer 4 — On-Premises:** All 50 on-premises DNS servers are configured with conditional forwarders for all 15 `privatelink.*` suffixes and `azure.com`, pointing to the Azure Private DNS Resolver Inbound Endpoint IPs. On-premises queries are forwarded to Azure, which resolves and returns private IPs. On-premises applications connect over ExpressRoute using private IPs.

Automation is critical at this scale. DNS A records must be created and deleted automatically when Private Endpoints are provisioned and decommissioned. Manual record management fails at 500 application teams. We enforce this through a Terraform module that platform teams must use to create Private Endpoints — the module includes the DNS record as part of its contract.

The most common failure mode at scale is stale DNS records — a Private Endpoint is deleted but the A record remains. This causes connections to the Private Endpoint IP, which no longer exists, to fail. We detect this through a daily automated reconciliation job that compares Private Endpoint inventory against DNS zone records and alerts on mismatches.

**Q: Describe the relationship between Private Endpoints, Private Link Service, and the platform team's ability to offer shared services to application teams.**

Standard Private Endpoints expose Microsoft-managed Azure PaaS services privately.

Private Link Service extends this model to allow the platform team to publish their own internal services as Private Link endpoints that application teams — or even external partners — can consume.

The use case: the platform team builds a shared logging service, a shared certificate issuance service, or a shared API gateway. Rather than requiring all application teams to peer their VNets to a shared services VNet — which creates routing complexity and blast radius concerns — the platform team exposes the service via Private Link Service.

Application teams create Private Endpoints to the platform's Private Link Service. They get a private IP in their Spoke. DNS records are created. They connect to the shared service using a private path without any VNet peering.

This is architecturally elegant because it maintains strict network isolation between Spokes while allowing controlled consumption of shared platform services. The platform team can enforce who can create Private Endpoints to their service — connections require approval, which the platform team controls.

At scale, this is how platform engineering teams should expose internal services. It is the same pattern Microsoft uses to expose Azure services privately. The platform team is building a platform on top of Azure using the same patterns Azure itself uses.

---

## 10. Interactive Exercise

### Architectural Challenge

A pharmaceutical company is deploying a new drug research platform on Azure.

The platform stores compound data in Azure Blob Storage and research results in Azure SQL Database.

Research workloads run on Azure Virtual Machines in multiple Spokes — one per research team.

The company has three on-premises research campuses connected via ExpressRoute.

**Compliance requirements:**

1. No storage account or database may be accessible from the public internet.
2. Research data must only flow over private network paths.
3. On-premises campus systems must be able to read from the SQL Database.
4. Campus DNS servers resolve using an internal DNS service — `dns.pharma-internal.com`.
5. A new research team can be onboarded in under one hour.

**Your task:**

Design the Private Endpoint and Private DNS architecture for this platform.

Specify where Private Endpoints are deployed and in which subnets.

Specify the Private DNS Zone design — centralised or distributed, and why.

Explain how on-premises DNS is configured to resolve private endpoint hostnames.

Explain how Azure DNS Resolver fits into the design.

Explain how the one-hour onboarding requirement is met.

Identify what happens if the Azure Private DNS Resolver becomes unavailable and how you mitigate this.

---

*There is no single correct answer.*

*Evaluation focuses on: correctness of the DNS design, handling of hybrid connectivity, automation reasoning for onboarding speed, and resilience planning for DNS availability.*
