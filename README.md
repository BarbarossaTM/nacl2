# NACL2

NACL2 is 2nd approach to the New Netbox Automation Abstration Caching & Compute Logic Layer for SaltStack (and others).

NACL2 is based on the experiences made with [NACL](https://github.com/BarbarosaTM/nacl/) which was a prototype hacked together with a strong focus on [Freifunk Hochstift SaltStack](https://github.com/FreifunkHochstift/ffho-salt-public).

## Architecture Overview

NACL2 is designed as a modular framework to act as a abstraction, caching and compute layer between Netbox and your automation.
As Freifunk Hochstift - which is one of the main users - uses SaltStack for automation, ext_pillar modules for SaltStack will be available. Others might follow.

NACL2 will consist of **Sources** which will be queried for inventory information or anyting which might be relevant to feed your automation and providing this informaton in well defined **Data Types** within NACL2. All source data will be stored in a central data structure.
The primary source obviously is `netbox` for now.

**Logic modules** will be responsible to gather and/or mangle information needed from the inventory (read: Source data), annotate devices, VMs, interfaces, etc. and generate configuration items based on the those. The aim is to provide generic modules where possible and have a framework which provides freedom to add own magic dust as well. Data computed by Logic modules will be stored in a seperate data structure besides source data; each logic module should get a reference to the source data as well as computed data set and should update the latter if required.

**View modules** can be used to generate a specific presentation of the computed data set for different platforms (Linux, Netonix, Arista, Juniper,...) for a given device. It may be desireable to cache the generated view for per device. 

Sources, Data types, Logic, and View Modules will follow an internal (to be defined) Interface/API.

### Sources

Source modules are responsible for retrieving inventory and/or configuration information as well as any other *intent* from any source useful and store the information in pre- or self-defined data types and structures.

Source modules should implement a method to check if the data in the given source has been changed and a reload is required.

### Data Types

NACL2 will provide data types for devices, VMs, interfaces, prefixes, IPs, etc. which provide easy access to relevant attributes and references between objects (device -> interfaces, interface -> IPs, IP -> prefix, etc.). It will be possible to define own data types as well.

### Logic Modules

Logic modules are responsible for annotating, mangling, and/or creating configuration data as required for your automation.

Logic modules for example could
 * apply defaults for any configuration item according to certain properties
 * set MTU of (underlying) network devices according to a pre-defined rules
 * create (overlay) network interfaces based on tags on interfaces or other intent
 * compute OSPF configuration for interfaces based on interface properties (IPs, prefixes, interface names, tags, ...)
 * compute iBGP peerings between nodes based on the roles and remote roles (sessions to RRs, between RRs, etc.)
 * ...
 
 ### View Modules
 
View modules are responsible for mangling all relevant information in the desired output format (that may be a JSON, YAML, ... data structure or text config) for a given device. Rendering configuration data per device allows inclusion of secrets on a per device basis.
 
### API

NACL2 will provide a REST-API and maybe a gRPC API to retrieve the computed information in part or in full and trigger a reload of the configuration and data.

The API might in the future allow to change information in the sources, for example add devices, IPs, etc.

## Lifecycle

On start-up NACL2 will invoke all configured Sources and run all activated Logic Modules in the configured order.
Once finished the computed data set is cached and available to be queried via the API.
Queries will always be answered from the cached data set.
If desired queries arriving before computation has finished can be delayed until the data set is ready.

NACL2 will check if a re-computation is required on a regular configurable basis; re-compututation can also be triggered via the API.
Computation will run in the background and once finished succesfully the internal cache will be updated.


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
