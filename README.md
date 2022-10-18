# ARG-Kusto-Queries
In this document, I am going to cover some common Kusto queries that can be quickly used in ARG (Azure Resource Graph) to quickly get information about resouces in an Azure subscription. This is an alternative way to get resources besides PowerShell, CLI and RestAPI. In the portal, simply search for "ARG" or Resouce Graph Explorer and you can quickly see the KQL queries that can be generated to view information about Azure resources. If you're not familar with Kusto, more info here via the public docs: https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/

In a nutshell, Kusto is very simmilar to SQL. Its just a structured query language used to pull resouces inside Azure. Here is the blade summary of resouces that can be queried:

![image](https://user-images.githubusercontent.com/55964102/193878903-129562cf-2fe0-49b6-9fbd-e59f1411a901.png)

```bash
#Dumping VMs across all attached subscriptions 
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
    
#Dumping all Vnets and their address blocks
resources   
 | where type == "microsoft.network/virtualnetworks"
 | extend AddressSpace_AddressPrefixes = tostring(properties.['addressSpace'].['addressPrefixes'])
 | project name, resourceGroup, subscriptionId, AddressSpace_AddressPrefixes
 ```

We can see this dumps all the given VMs including hostname, IP info etc along with URI. This data can be viewed in JSON and also exported to .CSV. For VNETs, we can project away what we want and dump the address space along with other relevant info. 

![image](https://user-images.githubusercontent.com/55964102/193879712-61529fe1-a67a-432d-b986-0c9bdde86a01.png)

![image](https://user-images.githubusercontent.com/55964102/196306528-76dca023-0ac4-4f4a-82cf-5f770af220cd.png)


