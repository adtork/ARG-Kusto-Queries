# Changelog

## [1.1.0] - 2026-05-30

- Added **10 new ARG inventory queries** for networking products: Azure Firewalls, ExpressRoute circuits, VPN/ER gateways, vWAN hubs, NSGs (with rules), route tables, Private Endpoints, Application Gateways, Load Balancers, Public IPs.
- Added **12 Log Analytics troubleshooting queries** under `queries/logs/` covering Azure Firewall (network/app/DNS/IDPS), ExpressRoute Traffic Collector, VPN Gateway tunnel + BGP diagnostics, NSG Flow Logs (Traffic Analytics), Application Gateway WAF + backend errors, and Private Link Service metrics.
- README restructured into ARG inventory + Log Analytics troubleshooting sections.

## [1.0.0] - 2026-05-30

- Split queries into versioned `.kql` files.
- Added catalog README.
- Preserved original prose.
