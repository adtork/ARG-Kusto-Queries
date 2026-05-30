# ARG-Kusto-Queries

![Azure](https://img.shields.io/badge/Azure-Resource%20Graph-0078D4)
![KQL](https://img.shields.io/badge/Language-KQL-blue)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

A growing catalog of **KQL queries** for Azure — split into two flavors:

1. **Azure Resource Graph (ARG)** — inventory queries that run against the ARM metadata plane (Resources table). Great for asking "what do I have?" across all subscriptions.
2. **Azure Monitor / Log Analytics** — troubleshooting queries against diagnostic logs and metrics emitted by individual networking products (Azure Firewall, ExpressRoute Traffic Collector, VPN Gateway, NSG Flow Logs, Application Gateway WAF, Private Link, etc.).

## Table of Contents

- [🎯 What is Azure Resource Graph?](#-what-is-azure-resource-graph)
- [🔍 How to Run These Queries](#-how-to-run-these-queries)
- [📊 ARG Inventory Catalog](#-arg-inventory-catalog)
- [🛠️ Log Analytics Troubleshooting Catalog](#️-log-analytics-troubleshooting-catalog)
- [📜 Sample Query Sources](#-sample-query-sources)
- [💡 Tips](#-tips)
- [📚 References](#-references)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

## 🎯 What is Azure Resource Graph?

In this document, I am going to cover some common Kusto queries that can be quickly used in ARG (Azure Resource Graph) to quickly get information about resouces in an Azure subscription. This is an alternative way to get resources besides PowerShell, CLI and RestAPI. In the portal, simply search for "ARG" or Resouce Graph Explorer and you can quickly see the KQL queries that can be generated to view information about Azure resources. If you're not familar with Kusto, more info here via the public docs: https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/

In a nutshell, Kusto is very simmilar to SQL. Its just a structured query language used to pull resouces inside Azure. Here is the blade summary of resouces that can be queried:

![image](https://user-images.githubusercontent.com/55964102/193878903-129562cf-2fe0-49b6-9fbd-e59f1411a901.png)

## 🔍 How to Run These Queries

### ARG inventory queries (files under `queries/`)

**Portal** — search "Resource Graph Explorer", paste the query, **Run**.

**Azure CLI**
```bash
az graph query -q "$(cat queries/list-vms-with-ips.kql)"
```

**PowerShell**
```powershell
Search-AzGraph -Query (Get-Content queries/list-vms-with-ips.kql -Raw)
```

### Log Analytics troubleshooting queries (files under `queries/logs/`)

These queries target **diagnostic logs / metrics** in a Log Analytics workspace — *not* the ARG plane. Open the workspace (or the resource's "Logs" blade) in the portal and paste the query. The required diagnostic settings are noted in the header of each file.

**Azure CLI**
```bash
az monitor log-analytics query \
  -w <workspace-guid> \
  --analytics-query "$(cat queries/logs/azure-firewall-top-denies.kql)"
```

## 📊 ARG Inventory Catalog

| Query | File | Description |
| --- | --- | --- |
| List VMs with IPs | [`queries/list-vms-with-ips.kql`](./queries/list-vms-with-ips.kql) | All VMs with private + public IPs across subscriptions |
| List VNets | [`queries/list-vnets-with-address-space.kql`](./queries/list-vnets-with-address-space.kql) | All VNets with their address-space prefixes |
| List Azure Firewalls | [`queries/list-azure-firewalls.kql`](./queries/list-azure-firewalls.kql) | Firewalls with SKU, tier, policy id, threat-intel mode, IPs |
| List ExpressRoute circuits | [`queries/list-expressroute-circuits.kql`](./queries/list-expressroute-circuits.kql) | Circuits with provider, peering location, bandwidth, state |
| List VPN + ER gateways (classic VNet) | [`queries/list-vpn-and-er-gateways.kql`](./queries/list-vpn-and-er-gateways.kql) | VNet GWs with type, SKU, BGP ASN, active-active flag |
| List vWAN hubs + gateways | [`queries/list-vwan-hubs-and-gateways.kql`](./queries/list-vwan-hubs-and-gateways.kql) | Virtual hubs with attached VPN / ER / P2S GW IDs |
| List NSGs with rules | [`queries/list-nsgs-with-rules.kql`](./queries/list-nsgs-with-rules.kql) | All custom NSG rules flattened with priority + direction |
| List route tables | [`queries/list-route-tables.kql`](./queries/list-route-tables.kql) | UDRs with prefix, next-hop type, next-hop IP |
| List Private Endpoints | [`queries/list-private-endpoints.kql`](./queries/list-private-endpoints.kql) | PEs with target resource, group id, private IP, connection state |
| List Application Gateways | [`queries/list-application-gateways.kql`](./queries/list-application-gateways.kql) | AGW SKU/tier/capacity, WAF mode + ruleset, listener/backend counts |
| List Load Balancers | [`queries/list-load-balancers.kql`](./queries/list-load-balancers.kql) | LB SKU, frontends, backend pool / rule / NAT counts |
| List Public IPs | [`queries/list-public-ips.kql`](./queries/list-public-ips.kql) | PIPs with address, allocation, SKU, attached resource, FQDN |

## 🛠️ Log Analytics Troubleshooting Catalog

| Product | Query | File | Table(s) |
| --- | --- | --- | --- |
| Azure Firewall | Top denied network-rule flows | [`queries/logs/azure-firewall-top-denies.kql`](./queries/logs/azure-firewall-top-denies.kql) | `AZFWNetworkRule` |
| Azure Firewall | Top application-rule hits (FQDN) | [`queries/logs/azure-firewall-top-app-rule-hits.kql`](./queries/logs/azure-firewall-top-app-rule-hits.kql) | `AZFWApplicationRule` |
| Azure Firewall | DNS proxy volume + NXDOMAIN | [`queries/logs/azure-firewall-dns-proxy.kql`](./queries/logs/azure-firewall-dns-proxy.kql) | `AZFWDnsQuery` |
| Azure Firewall (Premium) | IDPS signature hits | [`queries/logs/azure-firewall-idps-hits.kql`](./queries/logs/azure-firewall-idps-hits.kql) | `AZFWIdpsSignature` |
| ExpressRoute Traffic Collector | Top talker flows | [`queries/logs/expressroute-traffic-collector-top-flows.kql`](./queries/logs/expressroute-traffic-collector-top-flows.kql) | `ExpressRouteFlowLogs` |
| ExpressRoute Traffic Collector | Top Azure destinations by bytes in | [`queries/logs/expressroute-top-destinations-by-bytes.kql`](./queries/logs/expressroute-top-destinations-by-bytes.kql) | `ExpressRouteFlowLogs` |
| VPN Gateway | IPsec tunnel state changes | [`queries/logs/vpn-gateway-tunnel-events.kql`](./queries/logs/vpn-gateway-tunnel-events.kql) | `AzureDiagnostics` / `TunnelDiagnosticLog` |
| VPN Gateway | BGP route add/withdraw events | [`queries/logs/vpn-gateway-bgp-route-changes.kql`](./queries/logs/vpn-gateway-bgp-route-changes.kql) | `AzureDiagnostics` / `RouteDiagnosticLog` |
| NSG Flow Logs (Traffic Analytics) | Top denied flows | [`queries/logs/nsg-flow-top-denies.kql`](./queries/logs/nsg-flow-top-denies.kql) | `AzureNetworkAnalytics_CL` |
| Application Gateway WAF | Blocked / matched requests | [`queries/logs/app-gateway-waf-blocks.kql`](./queries/logs/app-gateway-waf-blocks.kql) | `AGWFirewallLogs` |
| Application Gateway | Backend 5xx by pool member | [`queries/logs/app-gateway-backend-5xx.kql`](./queries/logs/app-gateway-backend-5xx.kql) | `AGWAccessLogs` |
| Private Link Service | Bytes processed + flow counts | [`queries/logs/private-link-service-metrics.kql`](./queries/logs/private-link-service-metrics.kql) | `AzureMetrics` |

> Resource-specific tables (`AZFW*`, `AGW*`, `ExpressRouteFlowLogs`) require **Resource-specific** mode on the diagnostic setting — if you only see `AzureDiagnostics`, switch the diagnostic-setting destination type and re-enable.

## 📜 Sample Query Sources

### List VMs with IPs

```kusto
// ============================================================
// List VMs across all attached subscriptions
// ============================================================
// Returns: vmName, privateIps[], publicIps[]
// Resources joined: virtualmachines, networkinterfaces, publicipaddresses
// Usage: Paste into Azure Resource Graph Explorer in the portal
//        or run via `az graph query -q "$(cat queries/list-vms-with-ips.kql)"`
// ============================================================
Resources
    | where type =~ 'microsoft.compute/virtualmachines'
    | project vmId = tolower(tostring(id)), vmName = name
    | join (Resources
        | where type =~ 'microsoft.network/networkinterfaces'
        | mv-expand ipconfig=properties.ipConfigurations
        | project vmId = tolower(tostring(properties.virtualMachine.id)), privateIp = ipconfig.properties.privateIPAddress, publicIpId = tostring(ipconfig.properties.publicIPAddress.id)
        | join kind=leftouter (Resources
            | where type =~ 'microsoft.network/publicipaddresses'
            | project publicIpId = id, publicIp = properties.ipAddress
        ) on publicIpId
        | project-away publicIpId, publicIpId1
        | summarize privateIps = make_list(privateIp), publicIps = make_list(publicIp) by vmId
    ) on vmId
    | project-away vmId1
    | sort by vmName asc
```

We can see this dumps all the given VMs including hostname, IP info etc along with URI. This data can be viewed in JSON and also exported to .CSV. For VNETs, we can project away what we want and dump the address space along with other relevant info.

![image](https://user-images.githubusercontent.com/55964102/193879712-61529fe1-a67a-432d-b986-0c9bdde86a01.png)

### List VNets

```kusto
// ============================================================
// List VNets across all attached subscriptions
// ============================================================
// Returns: name, resourceGroup, subscriptionId, addressPrefixes
// Resources queried: virtualnetworks
// Usage: Paste into Azure Resource Graph Explorer in the portal
//        or run via `az graph query -q "$(cat queries/list-vnets-with-address-space.kql)"`
// ============================================================
resources   
 | where type == "microsoft.network/virtualnetworks"
 | extend AddressSpace_AddressPrefixes = tostring(properties.['addressSpace'].['addressPrefixes'])
 | project name, resourceGroup, subscriptionId, AddressSpace_AddressPrefixes
```

![image](https://user-images.githubusercontent.com/55964102/196306528-76dca023-0ac4-4f4a-82cf-5f770af220cd.png)

## 💡 Tips

- ARG queries are read-only and free.
- Cross-subscription ARG requires `--subscriptions` flag or you'll only see your default sub.
- ARG results paginated at 1000 rows by default — use `| top N` for larger pulls.
- For Log Analytics queries, narrow the time window first (`| where TimeGenerated > ago(1h)`) — diagnostic tables can be huge.
- Resource-specific log tables (e.g., `AZFWNetworkRule`, `AGWAccessLogs`) require enabling **Resource specific** mode in the diagnostic setting.
- Use `mv-expand` to flatten nested arrays, and `make_set` / `make_list` to re-aggregate after a join.

## 📚 References

- [Kusto Query Language documentation](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Azure Resource Graph documentation](https://learn.microsoft.com/en-us/azure/governance/resource-graph/)
- [Azure Resource Graph sample queries gallery](https://learn.microsoft.com/en-us/azure/governance/resource-graph/samples/starter)
- [KQL quick reference](https://learn.microsoft.com/en-us/azure/data-explorer/kql-quick-reference)
- [Azure Firewall structured logs](https://learn.microsoft.com/azure/firewall/firewall-structured-logs)
- [ExpressRoute Traffic Collector overview](https://learn.microsoft.com/azure/expressroute/traffic-collector-overview)
- [VPN Gateway diagnostics](https://learn.microsoft.com/azure/vpn-gateway/troubleshoot-vpn-with-azure-diagnostics)
- [Traffic Analytics schema](https://learn.microsoft.com/azure/network-watcher/traffic-analytics-schema)
- [Application Gateway WAF logs](https://learn.microsoft.com/azure/web-application-firewall/ag/web-application-firewall-logs)

## 🤝 Contributing

PRs are welcome. Add new queries as `.kql` files under `queries/` (ARG) or `queries/logs/` (Log Analytics) and follow the comment-block format used by the existing catalog entries.

## 📄 License

This project is licensed under the MIT License. See [LICENSE](./LICENSE) for details.

### List VMs with IPs

```kusto
// ============================================================
// List VMs across all attached subscriptions
// ============================================================
// Returns: vmName, privateIps[], publicIps[]
// Resources joined: virtualmachines, networkinterfaces, publicipaddresses
// Usage: Paste into Azure Resource Graph Explorer in the portal
//        or run via `az graph query -q "$(cat queries/list-vms-with-ips.kql)"`
// ============================================================
Resources
    | where type =~ 'microsoft.compute/virtualmachines'
    | project vmId = tolower(tostring(id)), vmName = name
    | join (Resources
        | where type =~ 'microsoft.network/networkinterfaces'
        | mv-expand ipconfig=properties.ipConfigurations
        | project vmId = tolower(tostring(properties.virtualMachine.id)), privateIp = ipconfig.properties.privateIPAddress, publicIpId = tostring(ipconfig.properties.publicIPAddress.id)
        | join kind=leftouter (Resources
            | where type =~ 'microsoft.network/publicipaddresses'
            | project publicIpId = id, publicIp = properties.ipAddress
        ) on publicIpId
        | project-away publicIpId, publicIpId1
        | summarize privateIps = make_list(privateIp), publicIps = make_list(publicIp) by vmId
    ) on vmId
    | project-away vmId1
    | sort by vmName asc
```

We can see this dumps all the given VMs including hostname, IP info etc along with URI. This data can be viewed in JSON and also exported to .CSV. For VNETs, we can project away what we want and dump the address space along with other relevant info.

![image](https://user-images.githubusercontent.com/55964102/193879712-61529fe1-a67a-432d-b986-0c9bdde86a01.png)

### List VNets

```kusto
// ============================================================
// List VNets across all attached subscriptions
// ============================================================
// Returns: name, resourceGroup, subscriptionId, addressPrefixes
// Resources queried: virtualnetworks
// Usage: Paste into Azure Resource Graph Explorer in the portal
//        or run via `az graph query -q "$(cat queries/list-vnets-with-address-space.kql)"`
// ============================================================
resources   
 | where type == "microsoft.network/virtualnetworks"
 | extend AddressSpace_AddressPrefixes = tostring(properties.['addressSpace'].['addressPrefixes'])
 | project name, resourceGroup, subscriptionId, AddressSpace_AddressPrefixes
```

![image](https://user-images.githubusercontent.com/55964102/196306528-76dca023-0ac4-4f4a-82cf-5f770af220cd.png)

## 💡 Tips

- ARG queries are read-only and free.
- Cross-subscription requires `--subscriptions` flag or you'll only see your default sub.
- Results paginated at 1000 rows by default — use `| top N` for larger pulls.
- Use `mv-expand` to flatten nested arrays like ipConfigurations.
- Use `join kind=leftouter` to preserve unmatched rows.

## 📚 References

- [Kusto Query Language documentation](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Azure Resource Graph documentation](https://learn.microsoft.com/en-us/azure/governance/resource-graph/)
- [Azure Resource Graph sample queries gallery](https://learn.microsoft.com/en-us/azure/governance/resource-graph/samples/starter)
- [KQL quick reference](https://learn.microsoft.com/en-us/azure/data-explorer/kql-quick-reference)

## 🤝 Contributing

PRs are welcome. Add new queries as `.kql` files under `queries/` and follow the comment-block format used by the existing catalog entries.

## 📄 License

This project is licensed under the MIT License. See [LICENSE](./LICENSE) for details.