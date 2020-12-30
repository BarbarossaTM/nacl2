# NACL 2

NACL 2 is 2nd approach to the Netbox Abstration / Automation & Caching Layer for Salt Stack.

NACL 2 is based on the experiences made with [NACL](https://github.com/BarbarosaTM/nacl/) which was a prototype hacked together and with focus on [Freifunk Hochstift Salt Stack](https://github.com/FreifunkHochstift/ffho-salt-public).

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

NACL 2 follows a modular approach for `source` and `compute` modules which request data from sources like netbox and provide compute modules which will generate config parts from information gathered from source modules.
