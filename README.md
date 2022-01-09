# NACL2

NACL2 is 2nd approach to the Netbox Abstration & Caching Layer for SaltStack (and others).

NACL2 is based on the experiences made with [NACL](https://github.com/BarbarosaTM/nacl/) which was a prototype hacked together with a strong focus on [Freifunk Hochstift SaltStack](https://github.com/FreifunkHochstift/ffho-salt-public).

## Architecture Overview

NACL2 is designed as a sole abstraction and caching layer between Netbox (and later Nautobot) and your automation.

The basic idea behind this approach is to have a daemon / micro service (nacl2) which is responsible for querying all relevant data from Netbox, represent them as objects with pointers in between for reference.  This allows all information stored within Netbox to be queried very fast (as of gRPC), in strongly typed objects, and following a stable API.

The interval in which nacl2 will check if data within Netbox has been changed will be configureable and a background job will re-fetch all data and re-compute the entire information, which will be updated internally as it has been computed.

### Data Types

NACL2 will provide data types for devices, VMs, interfaces, prefixes, IPs, etc. which provide easy access to relevant attributes and references between objects (device -> interfaces, interface -> IPs, IP -> prefix, etc.).

### API

NACL2 will provide a gRPC based API and maybe a REST-API as well to retrieve the computed information in part or in full and trigger a reload of the configuration and data.

The API might in the future allow to change information in the sources, for example add devices, IPs, etc.

## Use Cases

### Freifunk Hochstift Infrastructure Management

The aim is to manage all nodes - for example of the Freifunk Hochstift network - within [NetBox](https://github.com/digitalocean/netbox).
This includes physical devices as well as virtual machines (including their ressources and placement on VM hosts).
Physical devices could be general Linux boxes which run a Salt minion, switches or wireless backbone devices.

Those devices will be documentated including
 * their OS (platform)
 * network interfaces and connections
 * IP addresses and their VRFs
 * roles of the device
 * SSH keys and SSL host cert/keys (stored in config contexts)

The aim is to remove all those information from Salt pillar and use NetBox + NACL as the only source of truth for all devices.
