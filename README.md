# NACL2

NACL2 is 2nd approach to the New Netbox Automation Abstration Caching & Compute Logic Layer for SaltStack (and others).

NACL2 is based on the experiences made with [NACL](https://github.com/BarbarosaTM/nacl/) which was a prototype hacked together with a strong focus on [Freifunk Hochstift SaltStack](https://github.com/FreifunkHochstift/ffho-salt-public).

## Architecture

NACL2 is designed as a modular framework to act as a abstraction, caching and compute layer between Netbox and your automation.
As Freifunk Hochstift - which is one of the main users - uses SaltStack for automation, ext_pillar modules for SaltStack will be available. Others might follow.

NACL2 will consist of **Sources** which will be queried for inventory information or anyting which might be relevant to feed your automation and providing this informaton in well defined **Data Types** to NACL2.
The primary source obviously is `netbox` for now.

Once all configured sources are queried **Logic modules** will be run in a configured order and gather and/or mangle information needed from the inventory, annotate devices, VMs, interfaces, etc. and generate configuration items based on the those. The aim is to provide generic modules where possible and have a framework which provides freedom to add own magic dust as well.

Sources, Data types, and Logic Modules will be designed as Python modules following an internal (to be defined) API.

### Sources

Source modules are responsible for retrieving inventory and/or configuration information as well as any other *intent* from any source useful and store the information in pre- or self-defined data types and structures. 

### Data Types

NACL2 will provide data types for devices, VMs, interfaces, prefixes, IPs, etc. which provide easy access to relevant attributes and references between objects (device -> interfaces, interface -> IPs, IP -> prefix, etc.). It will be possible to define own data types as well.

### Logic Modules

Logic modules are responsible for annotating, mangling, and/or creating configuration data as required for your automation.

Example modules could for example
 * apply defaults for any configuration item according to certain properties
 * set MTU of (underlying) network devices according to a pre-defined rules
 * create (overlay) network interfaces based on tags on interfaces or other intent
 * compute OSPF configuration for interfaces based on interface properties (IPs, prefixes, interface names, tags, ...)
 * compute iBGP peerings between nodes based on the roles and remote roles (sessions to RRs, between RRs, etc.)
 * ...


## Use Cases

### Freifunk Hochstift Infrastructure Management

The aim is to manage all nodes - for example of the Freifunk Hochstift network - within [NetBox](https://github.com/digitalocean/netbox).
This includes physical devices as well as virtual machines (including their ressources and placement on VM hosts).
Physical devices could be general Linux boxes which run a Salt minion, switches (which might be managed by NAPALM in the future) or wireless backbone devices.

Those devices will be documentated including
 * their OS (platform)
 * network interfaces and connections
 * IP addresses and their VRFs
 * roles of the device
 * SSH keys and SSL host cert/keys (stored in config contexts)

The aim is to remove all those information from Salt pillar and use NetBox + NACL as the only source of truth for all devices.
