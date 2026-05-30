# Query Index

## ARG inventory (`queries/*.kql`)

| File | Description | Resource Types |
| --- | --- | --- |
| [`list-vms-with-ips.kql`](./list-vms-with-ips.kql) | VMs with private + public IPs across subscriptions | `microsoft.compute/virtualmachines`, `microsoft.network/networkinterfaces`, `microsoft.network/publicipaddresses` |
| [`list-vnets-with-address-space.kql`](./list-vnets-with-address-space.kql) | VNets with their address-space prefixes | `microsoft.network/virtualnetworks` |
| [`list-azure-firewalls.kql`](./list-azure-firewalls.kql) | Azure Firewalls with SKU/tier/policy/IPs | `microsoft.network/azurefirewalls` |
| [`list-expressroute-circuits.kql`](./list-expressroute-circuits.kql) | ExpressRoute circuits with provider/bandwidth/state | `microsoft.network/expressroutecircuits` |
| [`list-vpn-and-er-gateways.kql`](./list-vpn-and-er-gateways.kql) | VPN + ER virtual network gateways with BGP info | `microsoft.network/virtualnetworkgateways` |
| [`list-vwan-hubs-and-gateways.kql`](./list-vwan-hubs-and-gateways.kql) | Virtual WAN hubs with attached gateway IDs | `microsoft.network/virtualhubs` |
| [`list-nsgs-with-rules.kql`](./list-nsgs-with-rules.kql) | NSGs flattened to custom security rules | `microsoft.network/networksecuritygroups` |
| [`list-route-tables.kql`](./list-route-tables.kql) | UDR route tables with routes + next hop | `microsoft.network/routetables` |
| [`list-private-endpoints.kql`](./list-private-endpoints.kql) | Private Endpoints with target / private IP / state | `microsoft.network/privateendpoints` |
| [`list-application-gateways.kql`](./list-application-gateways.kql) | App Gateways with WAF posture + capacity | `microsoft.network/applicationgateways` |
| [`list-load-balancers.kql`](./list-load-balancers.kql) | LBs with frontends + pool/rule counts | `microsoft.network/loadbalancers` |
| [`list-public-ips.kql`](./list-public-ips.kql) | PIPs with allocation, SKU, attached resource | `microsoft.network/publicipaddresses` |

## Log Analytics troubleshooting (`queries/logs/*.kql`)

| File | Description | Table |
| --- | --- | --- |
| [`logs/azure-firewall-top-denies.kql`](./logs/azure-firewall-top-denies.kql) | Top denied network-rule flows | `AZFWNetworkRule` |
| [`logs/azure-firewall-top-app-rule-hits.kql`](./logs/azure-firewall-top-app-rule-hits.kql) | Top FQDN matches on application rules | `AZFWApplicationRule` |
| [`logs/azure-firewall-dns-proxy.kql`](./logs/azure-firewall-dns-proxy.kql) | DNS proxy queries + NXDOMAIN rate | `AZFWDnsQuery` |
| [`logs/azure-firewall-idps-hits.kql`](./logs/azure-firewall-idps-hits.kql) | IDPS signature hits (Premium tier) | `AZFWIdpsSignature` |
| [`logs/expressroute-traffic-collector-top-flows.kql`](./logs/expressroute-traffic-collector-top-flows.kql) | Top talker 5-tuples over ExR | `ExpressRouteFlowLogs` |
| [`logs/expressroute-top-destinations-by-bytes.kql`](./logs/expressroute-top-destinations-by-bytes.kql) | Top Azure destinations by bytes in | `ExpressRouteFlowLogs` |
| [`logs/vpn-gateway-tunnel-events.kql`](./logs/vpn-gateway-tunnel-events.kql) | IPsec tunnel state changes | `AzureDiagnostics` (TunnelDiagnosticLog) |
| [`logs/vpn-gateway-bgp-route-changes.kql`](./logs/vpn-gateway-bgp-route-changes.kql) | BGP route add/withdraw events | `AzureDiagnostics` (RouteDiagnosticLog) |
| [`logs/nsg-flow-top-denies.kql`](./logs/nsg-flow-top-denies.kql) | Top denied NSG flows | `AzureNetworkAnalytics_CL` |
| [`logs/app-gateway-waf-blocks.kql`](./logs/app-gateway-waf-blocks.kql) | WAF blocked / matched requests | `AGWFirewallLogs` |
| [`logs/app-gateway-backend-5xx.kql`](./logs/app-gateway-backend-5xx.kql) | Backend 5xx by pool member | `AGWAccessLogs` |
| [`logs/private-link-service-metrics.kql`](./logs/private-link-service-metrics.kql) | PLS bytes / flow counts | `AzureMetrics` |
