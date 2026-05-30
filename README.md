# ARG-Kusto-Queries

![Azure](https://img.shields.io/badge/Azure-Resource%20Graph-0078D4)
![KQL](https://img.shields.io/badge/Language-KQL-blue)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

A growing catalog of Azure Resource Graph (KQL) queries for inventorying Azure resources at scale.

## Table of Contents

- [🎯 What is Azure Resource Graph?](#-what-is-azure-resource-graph)
- [🔍 How to Run These Queries](#-how-to-run-these-queries)
- [📊 Query Catalog](#-query-catalog)
- [📜 Query Source](#-query-source)
- [💡 Tips](#-tips)
- [📚 References](#-references)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

## 🎯 What is Azure Resource Graph?

In this document, I am going to cover some common Kusto queries that can be quickly used in ARG (Azure Resource Graph) to quickly get information about resouces in an Azure subscription. This is an alternative way to get resources besides PowerShell, CLI and RestAPI. In the portal, simply search for "ARG" or Resouce Graph Explorer and you can quickly see the KQL queries that can be generated to view information about Azure resources. If you're not familar with Kusto, more info here via the public docs: https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/

In a nutshell, Kusto is very simmilar to SQL. Its just a structured query language used to pull resouces inside Azure. Here is the blade summary of resouces that can be queried:

![image](https://user-images.githubusercontent.com/55964102/193878903-129562cf-2fe0-49b6-9fbd-e59f1411a901.png)

## 🔍 How to Run These Queries

### Portal

Search "Resource Graph Explorer", paste the query, and select **Run**.

### Azure CLI

```bash
az graph query -q "$(cat queries/list-vms-with-ips.kql)"
```

### PowerShell

```powershell
Search-AzGraph -Query (Get-Content queries/list-vms-with-ips.kql -Raw)
```

## 📊 Query Catalog

| Query | File | Description | Returns |
| --- | --- | --- | --- |
| List VMs with IPs | [`queries/list-vms-with-ips.kql`](./queries/list-vms-with-ips.kql) | All VMs with private + public IPs across subscriptions | vmName, privateIps, publicIps |
| List VNets | [`queries/list-vnets-with-address-space.kql`](./queries/list-vnets-with-address-space.kql) | All VNets with their address space prefixes | name, resourceGroup, subscriptionId, addressPrefixes |

## 📜 Query Source

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