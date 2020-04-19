# Network

## How to connect 2 or more computers in a private network?
- You can do this using a switch.
- See interfaces on the host:
    - `ip link` or `ifconfig` or `ipconfig`
- Assign both computer ips on the same network. For example, if the switch is on ip `192.168.12.1`, then all the computers should have ips in the region of `192.168.12.*`
    - Assign an ip address to an interface `sudo ifconfig eth0 192.168.12.2 netmask 255.255.255.0`

## How to use a box to act as a switch
- Given that we have:
    - node-1 with ip 192.168.50.1
    - node-2 with ip 192.168.50.2 on interface eth1
    - node-2 with ip 192.168.51.2 on interface eth2
    - node-3 with ip 192.168.51.1
- And we install nginx servers on all 3 nodes on port 80
- How do we reach node-3 from node-1?
    - In node-1 we add a route rule `sudo ip route add 192.168.51.1 via 192.168.50.2`
    - In node-3 we add a backwards rule `sudo ip route add 192.168.50.1 via 192.168.51.2`
    - All there is left to do is to allow port forwarding on node-2 with `sudo nano /proc/sys/net/ipv4/ip_forward` and switch from 0 to 1

## Usefull commands
- `netstat` - Tool. Ex: `netstat -ln`
- `ip` - IP management tool
- `arp` - Show arp (Address Resolution Protocol) table
- `route` - Get all the routes that the linux box can reach
- `ip route add 192.168.10.1/24 via 192.168.1.1` - Add a new rule. Reach 192.168.10.1 from the switch/router with ip 192.168.1.1
- `ip route add default via 192.168.1.1` - Add a new rule. Reach any other unknown ip from the switch/router with ip 192.168.1.1. You can also substitute default with 0.0.0.0