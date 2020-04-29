# Firewall-as-a-Service (FWaaS) v2 scenario

## 1. Enable FWaaS v2

### 1.1 Enable FWaaS plug-in trong file `/etc/neutron/neutron.conf`:
```sh

[service_providers]
# ...
service_provider = FIREWALL_V2:fwaas_db:neutron_fwaas.services.firewall.service_drivers.agents.agents.FirewallAgentDriver:default

[fwaas]
agent_version = v2
driver = neutron_fwaas.services.firewall.service_drivers.agents.drivers.linux.iptables_fwaas_v2.IptablesFwaasDriver
enabled = True
```
