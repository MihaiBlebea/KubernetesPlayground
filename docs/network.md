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


## Play with network namespaces in Linux
See more details in this interesting article https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/
- Create a namespace `ip netns add blue` - blue is the name of the namespace
- See all the namespaces `ip netns list` or just `ip netns`
- Now let's add some virtual interfaces to the namespace
- See all the available interfaces with `ip link list`
- Create a pair of virtual interfaces. They must be 2 corresponding interfaces that connect like a tunnel:
    - `sudo ip link add veth0 type veth peer name veth1` - For example `veth0` and `veth1` here
    - You can remove them with `sudo ip link delete veth0`
- Now both interfaces `veth0` and `veth1` belong to the global namespace. To connect the global namespace to the blue namespace you neet to attach one part of the interface (veth1) to the **blue** namespace:
    - `ip link set veth1 netns blue`
- If you list the interfaces again, **veth1** is no longer in the global namespace. Check if it is in the **blue** namespace:
    - `ip netns exec blue ip link list`
- Set an ip to the blue **veth1** interface:
    - `sudo ip netns exec blue ip addr add 10.1.1.1/24 dev veth1` - Assign IP
    - `sudo ip netns exec blue ip link set dev veth1 up` - Bring the interface UP
    - `sudo ip netns exec blue ifconfig` - Check the interfaces and the ips assigned
- If you have 2 namespaces **red** and **blue**, attach one side of the interface to each of them and assign ips, then you can ping from red to blue and vice versa:
    - `sudo ip netns exec blue ping 10.1.1.2` & `sudo ip netns exec red ping 10.1.1.1`

## Create a virtual network on host using a virtual switch
- Use Linux Bridge
- Create a bridge on the host:
    - `ip link add v-net-0 type bridge` - **v-net-0** is the name of the bridge
    - `ip addr add 192.168.1.1 dev v-net-0` - **v-net-0** is the name of the bridge and **192.168.1.1** the ip we want to assign
    - On host add an iptable rule so host sends packages outside coming from guest in it's host name:
        - `iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE`

## Interesting subjects:
- [ ] Linux Policy Routing 
    - https://blog.scottlowe.org/2013/05/29/a-quick-introduction-to-linux-policy-routing/