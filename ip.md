# Linux networking

## Interfaces up/down and routing tables:
- Bringing down an iface removes every route that references it.
- Bringing up an iface by e.g. `ifconfig eth0 192.168.99.14 netmask 255.255.255.0 up` will add a route to it's network, but no default gateway (it doesn't necesarily exist in that network).


