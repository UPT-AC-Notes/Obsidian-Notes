# Networking training

Throughout this training, we'll be exploring what networking is, how different nodes can connect and communicate with each other, different important networking-related services and why / how are they useful, and how to figure out what works and what doesn't and why, aka some networking debugging tips.

For this training, we'll be working in a "virtualized environment" using Hyper-V VMs, which allows us to have a more complex environment and network topology. Note that even if it's "emulated", it's meant to emulate real case scenarios and network topologies, especially since VMs are meant to very closely resemble real life counter parts, and the same applies for any networking-device used by them. Additionally, the way networking works, it's completely irelevant if it's from a virtualized environment, or physical, the rules and how things work remain completely the same.

**TIP**: We are going to include some tips for troubleshooting network issues along the way!

**TIP 2**: A lot of networking has some parallels in real life, more specifically to how a postal office works. For example, similarly to how we deliver packages through a post office, we deliver packets through switches and routers.

If you've followed the lab environment guide, this is how the networks should look like:

```
+---------------------------------------------------+
|                                                   |
| +---------------------+  +---------------------+  |
| | gateway-vm          |  | client-vm           |  |
| |                     |  |                     |  |
| |                     |  |                     |  |
| |                     |  |                     |  |
| | +------+   +------+ |  | +------+   +------+ |  |
| | | eth0 |   | eth1 | |  | | eth0 |   | eth1 | |  |
| | +------+   +------+ |  | +------+   +------+ |  |
| |    |          |     |  |     |          |    |  |
| |    |          |     |  |     |          |    |  |
| +---------------------+  +---------------------+  |
|      |          |              |------    |       |
|      |          |-------------        |   |       |
|      |                       |        |   |       |
| +----------------+          +-----------------+   |
| | Default Switch |          | Isolated Switch |   |
| +----------------+          +-----------------+   |
|         |                               |         |
|         |                               |         |
| +---------------+       +----------------------+  |
| | Internet conn |       | vEthernet (isolated) |  |
| +---------------+       +----------------------+  |
|         |                                         | 
|         |                                         |
|         |                  Windows / Hyper-V Host |
+---------------------------------------------------+
          |
          |
   +----------+
   | Internet |
   +----------+
```

Basically, we have 2 nodes (VMs) and 2 networks: ``Default Switch`` (created by Hyper-V, sharing and routing traffic to the external network (aka Internet in most cases)), and ``Isolated``, which we have created, and which isolated, and it exists only on the Windows node itself, nothing can get into the ``Isolated`` network directly. Additionally, the Windows host also has a NIC in that ``Isolated`` network (created by Hyper-V), and has the static IP ``192.168.100.2`` on it.

Each node has 2 Network Interface Cards (NICs), connected as follows:

- ``gateway-vm`` (``lab-codengo-gateway``):
    - ``eth0`` <-> ``Default Switch`` (IP: automatically set through DHCP)
    - eth1 <-> Isolated vSwitch (static IP: ``192.168.100.1``)
- ``client-vm`` (``lab-codengo-client``):
    - ``eth0`` <-> ``Isolated vSwitch`` (IP: None)
    - ``eth1`` <-> ``Isolated vSwitch`` (static IP: ``192.168.100.10``)


## OSI model

The standard when it comes to networking. Everyone respects it, and so should you! All networks follow this model (with one exception: social networks, jk jk).

- Has 7 layers.

Good thing to note here is the model itself / its architecture. It's a "layered model", a common design pattern, which has a few benefits: a layer is only responsible with interacting with the layer right below it (not skipping layers), it adds additional features using the layer below, and is offering an interface to be used to the layer above it. This simplifies things, as the layer's only concern is the "interface" exposed by the layer below it, without having to worry about too many technical details; they are abstracted.

We've mostly been users of the Layer 7 (Application Layer). But, let's say that for some reason our application's networking doesn't work. Why? How / Where do we even start debugging the issue? Well, we should probably start at the bottom. We're going to learn how they work, and see if our networking issues stem from there and how we can address them.


### L1 Physical Layer

- NICs (network interface cards), transmission medium (cable, air). Mostly responsible for transmitting data (bits) across the wire using transmission protocols.

- L1 network: physical, "flat" network.
- Allows us to connect one node to another directly through a cable. For multiple nodes, we'll have to go up to L2. 

In our case, we have a L1 network in Hyper-V, we have vSwitches to which Hyper-V VMs are connected.
We can see the vswitches by running this in powershell:

```
Get-VMSwitch
```

We can see a VM's connection to a vswitch:

```
get-vm -name lab-codengo-gateway | Get-VMNetworkAdapter
```

We can see existing network connections on the Windows host as well by running:

```
ipconfig /all 


ipconfig

Windows IP Configuration


Ethernet adapter Ethernet:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : intranet.cs.upt.ro

Ethernet adapter vEthernet (Default Switch):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::5b55:baab:c0b4:dc0%29
   IPv4 Address. . . . . . . . . . . : 172.27.96.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . :

Ethernet adapter vEthernet (isolated):

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.100.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :

Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . :
   IPv6 Address. . . . . . . . . . . : 2a02:2f09:3013:600:7bd1:60e1:506e:b931
   Temporary IPv6 Address. . . . . . : 2a02:2f09:3013:600:c0c8:33a8:cc58:1179
   Link-local IPv6 Address . . . . . : fe80::dcd8:7a15:23ff:d3c6%18
   IPv4 Address. . . . . . . . . . . : 192.168.1.131
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::f2a7:31ff:fe6f:c848%18
                                       192.168.1.1
```

In this, we can see that the Ethernet adapter is not connected, and that only the Wi-Fi is connected. Additionally, we can see 2 other Network adapters, one for each of the Hyper-V vSwitches. That's because Hyper-V allows the host to "join" those networks as well, unless explicitly setting the vSwitches to "private".

In the ``lab-codengo-gateway`` VM, we can check the connected network interfaces by running this command:

```
# connect to the VM
ssh ubuntu@192.168.100.1

ip -c a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:01:28:42 brd ff:ff:ff:ff:ff:ff
    inet 172.27.106.69/20 brd 172.27.111.255 scope global dynamic eth0
       valid_lft 86040sec preferred_lft 86040sec
    inet6 fe80::215:5dff:fe01:2842/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:01:28:43 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe01:2843/64 scope link
       valid_lft forever preferred_lft forever
```

It's important to note here the ``lo`` device, aka the "loopback" device. That interface and IP (``127.0.0.1``) always refers to itself, and it's always true for any Host.

Also note that there are multiple Ethernet devices (``eth0``, ``eth1``), similar to how you'd have multiple Network Adapters attached to that VM.

Additionally, we can see that the interfaces are ``UP`` and connected.

Let's check how it changes if we "disconnect" one of "cables" from it from the Hyper-V Manager: right-click the VM, Settings, select the ``Default Switch`` Adapter, change it to ``Not Connected``.

Now, let's check how this affected the VM itself:

```
ip -c a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 00:15:5d:01:28:42 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::215:5dff:fe01:2842/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:01:28:43 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe01:2843/64 scope link
       valid_lft forever preferred_lft forever
```

We can now see that ``eth0`` is now ``DOWN``, meaning that it is disconnected, and that NIC is effectively unplugged.

**TIP**: Always make sure that the network cable is plugged. Otherwise, there's no network connectivity.

**TIP 2**: Kinda obvious, but a cable has 2 endpoints. Make sure that both are plugged properly. Not the case in our scenario, but it might be in other scenarios, like in a cloud, where we have we have an SDN (software defined networking) solution.


### L2 Data Link Layer

This layer is responsible for node-to-node communication (and potential corrections if needed, especially for Wi-Fi where we can have packet collision), as it actually includes a data "frame", which includes the source address, destination address, etc.

Multiple nodes can communicate through each other through switches. The multinode network connected through a switch is called a **LAN** (Local Area Network).

The nodes on this layer are identified through their hardware address hardcoded into the Network Adapters, aka MAC addresses.

We can see this MAC address on each of the NICs above.

We can see known neighbours by running:

```
ip n

172.27.96.1 dev eth0 lladdr 00:15:5d:6e:e0:48 STALE
192.168.100.2 dev eth1 lladdr 00:15:5d:01:28:3f REACHABLE
```

- we can see 192.168.100.2's MAC address, as that's the IP we're connecting from. But we don't see 192.168.100.10. That's because those caches / tables are not "discovered" / known yet. However, we can discover it:

```
ping -c 4 192.168.100.10
ip n

ping -c 4 192.168.100.10
PING 192.168.100.10 (192.168.100.10) 56(84) bytes of data.
64 bytes from 192.168.100.10: icmp_seq=1 ttl=64 time=0.515 ms
...
--- 192.168.100.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 0ms

ip n
192.168.100.10 dev eth1 lladdr 00:15:5d:01:28:45 REACHABLE
172.27.96.1 dev eth0 lladdr 00:15:5d:6e:e0:48 STALE
192.168.100.200 dev eth1 lladdr 00:15:5d:01:28:0e STALE
192.168.100.2 dev eth1 lladdr 00:15:5d:01:28:3f REACHABLE
```

Note that MAC addresses MUST be unique in a network. If it's not, the packet might be sent to another node than expected. There are networking attacks based on impersonating other MAC addresses in a network (MAC spoofing attacks). If you have duplicate MACs in a network, you have a problem.

One interesting thing to note here, is that you can basically have different network "planes" over the same physical network (effectively over the same network cable and switch), which are going to be treated as if they're separated. Those networks are called "virtual networks", or VLANs, or L2 networks, and they are separated through a VLAN ID or segmentation ID (14-bit field, range: 1-4094). Nodes that have different VLAN IDs cannot see each other, and you can have multiple hosts that have exactly the same IPs but on different VLANs.

We can see try this out in Hyper-V as well. Right now, the 2 VMs are in the same physical network and have the same VLAN ID, which means that they can see each other:

```
ping -c 1 192.168.100.10
PING 192.168.100.10 (192.168.100.10) 56(84) bytes of data.
64 bytes from 192.168.100.10: icmp_seq=1 ttl=64 time=0.443 ms

--- 192.168.100.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.443/0.443/0.443/0.000 ms
```

However, if we change lab-codengo-client's Network Adapters (NICs) VLAN ID to 100, the 2 VMs are no longer on the same network, even though they are connected to the same vSwitch:

```
ping -c 1 192.168.100.10
PING 192.168.100.10 (192.168.100.10) 56(84) bytes of data.

--- 192.168.100.10 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms
```

You've probably never had to deal with anything like this. In most scenarios, it's not something that you should worry about; you might have already been using VLANs without knowing (switches can automatically add / remove VLAN IDs). But in some professional environments (datacenters), VLANs are being used to separate networks by  purpose (e.g.: management network, admin network, storage network, public network), isolating them and routing them, and enhancing network performance (by simply limiting the "noise" generated in a network, and limiting the broadcast domain boundaries).

So, how does a L2 packet look like? See here, 802.3 Ethernet packet and frame structure, https://en.wikipedia.org/wiki/Ethernet_frame


### L3 Network Layer

We've talked so far about Local Area Networks. But how do we even connect to other networks? How do we establish inter-network communication, aka internet?

Let's consider 2 different networks: Red and Blue.

We're going to need a way of addressing sub-networks (subnets) in a wider inter-network (we'll just call it internet from now), and then assign addresses to nodes in a subnet. We will assign Internet Protocol Addresses to these nodes (IP Addresses).

So, what is an IP? It's a 4-byte number separated by dots, for example ``192.168.100.10``.

- Now, let's consider that on the same physical network there's another node, ``192.168.100.200``. The question is: can ``192.168.100.10`` communicate with ``192.168.100.200``? 

The answer is: it depends. Typically, an IP address is also followed by a netmask. Let's see it on Windows:

```
ipconfig

Windows IP Configuration

...

Ethernet adapter vEthernet (isolated):

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.100.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```

We can see here the "Subnet Mask", also known as the netmask, and we have the IP 192.168.100.2. If we apply a bit-wise AND over these 2 numbers (I know, you loved those in your first year uni), we'd get this:

```
192.168.100.2 &
255.255.255.0
-------------
192.168.100.0
```

The ``192.168.100.0`` result is called the "subnet". If 2 IP addresses have the same "subnet", then they are in the same network and can communicate with each other! If not, they cannot communicate with each other, even if they're on the same physical network (it means that they are "logically separated").

Now, to answer the question: can ``192.168.100.10`` and ``192.168.100.200`` communicate with each other, if they have the netmask ``255.255.255.0``? But what if the netmask is ``255.255.255.128`` instead? But what if we try to reach ``8.8.8.8`` from ``labs-codengo-client``?

Considering this operation, how many IPs are there in a subnet? (hint: for netmask ``255.255.255.0``, there are 255 IPs available).

These sorts of operations are performed by any type of networking equipment in order to determine if / how an IP can be reached.

Let's check the IP address and netmask on a Linux node:

```
ip -c a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:01:28:43 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe01:2843/64 scope link
       valid_lft forever preferred_lft forever
```

Weird, we don't see a netmask here. Or do we? *queue vsauce music*. We actually see the IP address ``192.168.100.1/24``, which is a **CIDR** notation (Classless Inter-Domain Routing), and that ``"24"`` means how many bits of 1 there are from left to right in the netmask. ``/24`` essentially means netmask ``255.255.255.0`` (255 is ``1111 1111``, and there are 3 255 bytes).

- To test out our theory that the networks are indeed separated logically, let's add a static IP to the ``labs-codengo-client`` VM's ``eth0`` interface. Edit its netplan:

```
sudo vim /etc/netplan/00-installer-config.yaml
```

And set in the ``eth0`` section with:

```
      #dhcp4: true
      addresses:
      - 192.168.101.10/24
```

Next, apply the changes and verify them:

```
sudo netplan apply
ip -c a
```

From the ``labs-codengo-gateway`` VM, we can see that we can't reach ``192.168.101.10``, even though it's on the same physical network:

```
ping -c 2 192.168.101.10
PING 192.168.101.10 (192.168.101.10) 56(84) bytes of data.

--- 192.168.101.10 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1015ms
```


#### Special subnets and IP addresses, and where do they come from?

Note the ``"brd 192.168.100.255"`` above, which actually means the "broadcast" address. The broadcast address is always the last IP in a subnet. Sending a message to it will essentially send a message to ALL the nodes in a subnet. Note that nodes may ignore broadcasts. To override this:

```
sudo su
echo 0 >/proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
ping -b 192.168.100.255

WARNING: pinging broadcast address
PING 192.168.100.255 (192.168.100.255) 56(84) bytes of data.
64 bytes from 192.168.100.1: icmp_seq=1 ttl=64 time=0.019 ms
```

Another special IP which should be noted is the "Gateway IP", which we'll discuss later.

Another important thing to note here: We've all been connecting to ``192.168.100.1`` and ``192.168.100.10`` during the training, but we're actually connecting to our own VMs, and not to each other's VMs. How can this be?

There are a few subnets meant for private networks:

- ``10.0.0.0/8`` (class A network)
- ``172.16.0.0/12`` (class B network)
- ``192.168.0.0/16`` (class C network)

Of course, these networks can be subdivided into subnets, we're already familiar with the ``192.168.168.100.0/24`` subnet. The network we've been using only exists on each person's environment, isolated from each other, we all have different ``192.168.100.10/24`` networks. Not to mention that we have multiple IPs, at least one per Network Interface.

There are a few other special networks worth mentioning:

- ``127.0.0.0/8`` - loopback network, referring to self, the packets sent there never leave the host.
- ``169.254.0.0/16`` - if a node doesn't have an IP assigned to it, it might have a random IP from this range. There are a few other magic IPs in this range, typically used in clouds, like ``169.254.169.254``.

**TIP**: As mentioned above, if you have some ``169.254.0.0/16`` address, you might not "really" have internet connectivity there, and you definitively don't have **any** network connectivity on that network if you don't have an IP. See below how IP assignment works.

Now, who decides those IPs and who gets them? Well, generally, the network administrator does. In our isolated environment, we are the network administrator! Basically, we decided to use the ``192.168.100.0/24`` network, but it could have been any other subnet. When choosing a subnet for your networks, it's recommended to pick a private network subnet mentioned above.

In our scenario, we've chosen ``192.168.100.1`` for the ``labs-codengo-gateway`` VM, ``192.168.100.2`` for the Windows host (we needed that in order to actually be able to SSH into the VMs, otherwise, we wouldn't be on the same (logical) network), and ``192.168.100.10`` for ``labs-codengo-client``. We can see this in their netplans:

```
cat /etc/netplan/00-installer-config.yaml
```

This is called "Static IP assignment". Of course, this works (and DHCP as well, we'll talk about it later) on any device, including those Linux nodes, Windows, your phone. In Ubuntu's case, we've set this in a netplan, in which we can describe the network configuration, and potentially more advanced setups and configurations (bonds, bridges, VLANs). Other Linux distros may use some other network manager and configuration, so always check how to configure your distro's networking, but the principles remain the same.

But still, when we connect to a new network, do we have to statically set the IP ourselves? What subnet should I even set in the first place? What if that IP is already in use? There should be a better way. And there is! It's called **DHCP** (Dynamic Host Configuration Protocol).

So, if we look at `labs-codengo-gateway`'s netplan, we see this:

```
network:
  ethernets:
    eth0:
      dhcp4: true
    eth1:
      addresses:
      - 192.168.100.1/24
      nameservers:
        addresses: []
        search: []
  version: 2
```

It seems that DHCP is enabled for ``eth0``, there are no static IP set for it. And if we look at its IPs:

```
ip -c a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:01:28:42 brd ff:ff:ff:ff:ff:ff
    inet 172.27.106.69/20 brd 172.27.111.255 scope global dynamic eth0
       valid_lft 59310sec preferred_lft 59310sec
    inet6 fe80::215:5dff:fe01:2842/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:01:28:43 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe01:2843/64 scope link
       valid_lft forever preferred_lft forever
```

We can see that ``eth0`` has an IP (your may be different). Where does that IP come from? Well, in this case, it comes from Hyper-V, from its ``Default Switch``, as it's configured to act as a DHCP server and serve IPs and other important networking configurations, including the gateway, and DNS. Hyper-V only serves DHCP for the "Default Switch" though, which is why we won't see it on the other Hyper-V internal networks that we have created. But we can have our own DHCP servers if we want.

In general, when connecting to a network, the router / gateway (typically has the address that ends with a ``.1``, ``172.27.96.1`` in our scenario) will also act as the DHCP server. Keep in mind that in most cases, there should only be **ONE DHCP server** in a physical network, otherwise you'd end up with IP conflicts or be configured with improper networking configuration.

We can manually ask for an IP from the DHCP server for a network adapter:

```
sudo dhclient -v eth0

Internet Systems Consortium DHCP Client 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/eth0/00:15:5d:01:28:42
Sending on   LPF/eth0/00:15:5d:01:28:42
Sending on   Socket/fallback
DHCPREQUEST for 172.27.106.69 on eth0 to 255.255.255.255 port 67 (xid=0x4dbdac3b)
DHCPACK of 172.27.106.69 from 172.27.96.1 (xid=0x3bacbd4d)
RTNETLINK answers: File exists
bound to 172.27.106.69 -- renewal in 34652 seconds.

# Or on Windows
ipconfig /renew
```

**TIP**: In general, when connecting to a network (through Wi-Fi or ethernet), there should be a DHCP server in the network assigning you an IP (typically, the router). If you don't have an IP, either the DHCP server didn't reply to your DHCP requests, or you may have issues reaching it. If you know the network's subnet, try assigning a random IP in that subnet and try to see if you can can reach the gateway through ping.

Question: if we have DHCP, why would we ever want to use Static IPs instead? There are scenarios in which Static IPs are preferred, typically when it comes to servers. DHCP leases IPs randomly; imagine having your server randomly changing IPs and still expect users to connect to it to the old IP. It wouldn't be great. Typically, for servers you'd have a Static IP Pool, which you'd exclude in the DHCP server, so it wouldn't allocate IPs from that Pool.


#### Inter-network communication

By default, nodes on the same subnet can communicate with each other. But what about other subnets? Especially since we've seen that we can't reach ``8.8.8.8`` from ``labs-codengo-client``.

- Well, before that, we need to talk about routes. How does a node even know how to reach other nodes? We already know that nodes in the same subnet can reach other. We can check this by running this on the ``labs-codengo-client`` node:

```
ip route

192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.10
192.168.101.0/24 dev eth0 proto kernel scope link src 192.168.101.10
```

- What we see here is that for a certain subnet, the traffic has to go through a certain Network Adapter. We don't see any other route, so, when the node will try to access `8.8.8.8`, it won't know where to go:

```
ping 8.8.8.8
ping: connect: Network is unreachable
```

It is an isolated / closed environment, so it cannot reach any external networks. Instead, if we try the same command from the ``labs-codengo-gateway`` node:

```
ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=55 time=10.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=55 time=10.6 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 10.555/10.590/10.625/0.035 ms
```

We see that we can reach that IP. That means that there's a route that leads to it. Let's see:

```
ip route

ip route
default via 172.27.96.1 dev eth0
default via 172.27.96.1 dev eth0 proto dhcp src 172.27.106.69 metric 100
172.27.96.0/20 dev eth0 proto kernel scope link src 172.27.106.69
172.27.96.1 dev eth0 proto dhcp scope link src 172.27.106.69 metric 100
192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.1
```

We don't see a route specifically for ``8.8.8.8``, but we do see something called a ``"default"``. That means that, if there's an IP or subnet we don't have a specific route for, it'll go to the default. The default in this case is a "gateway", and this route has been automatically given by the DHCP server.

- Basically, if we don't know the route for something, we go towards a "gateway" / "router", which may know where that subnet is, or at least where to send it next. It's similar to how you'd send packages across the world: I don't know how to send a package in a foreign country, I only know the address; but I know what the Postal Office will know where to send it. From the moment the package leaves my hands, it's no longer my problem, the Postal Office will send the package further, based on their "configured" routes; it may go to a biger Postal Office branch, then to an international office, then intercontinental, then to the target country, target city, target local Postal Office, and then finally to the actual destination. We can see something similar in our case if we run:

```
traceroute 8.8.8.8

traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  WarmMachine.mshome.net (172.27.96.1)  0.250 ms  0.232 ms  0.196 ms
 2  10.7.1.1 (10.7.1.1)  3.159 ms  3.142 ms  3.104 ms
 3  x.x.x.x (x.x.x.x)  3.125 ms  3.101 ms  3.064 ms
 4  10.192.48.169 (10.192.48.169)  3.369 ms  4.753 ms  3.307 ms
 5  10.190.152.65 (10.190.152.65)  4.694 ms  4.671 ms  4.652 ms
 6  10.190.152.66 (10.190.152.66)  4.656 ms  4.308 ms  4.281 ms
 7  10.220.142.94 (10.220.142.94)  3.091 ms  4.104 ms  4.076 ms
 8  10.220.138.208 (10.220.138.208)  4.370 ms  4.340 ms  4.321 ms
 9  10.220.177.110 (10.220.177.110)  10.121 ms 10.220.178.148 (10.220.178.148)  15.119 ms 10.220.153.18 (10.220.153.18)  15.103 ms
10  72.14.216.212 (72.14.216.212)  12.242 ms  11.114 ms 209.85.168.182 (209.85.168.182)  11.108 ms
11  * * *
12  dns.google (8.8.8.8)  13.382 ms  13.364 ms 216.239.49.77 (216.239.49.77)  11.600 ms
```

As we can see above, this is exactly what it happened. The Linux node didn't know how to reach ``8.8.8.8``, so it sent it to the gateway (``172.27.96.1``, Hyper-V gateway), which then sent it our local Wi-Fi's gateway (``10.7.1.1``), then company's IP / gateway (``x.x.x.x``), and so on, until it finally reached ``8.8.8.8``.

So, who determines those routes? That is a very tough and difficult question to answer, which would require a lot of explanations, which would include thougher topics like "route advertising" via BGP (Border Gateway Protocol). But let's say that on a basic level, the Network Administrators of a particular hop can decide how subnets are routed.

Considering that we basically own our own private networks that we created in Hyper-V, we can route however we please. At the moment, our Hyper-V ``Isolated`` network is currently isolated, but we could route external network traffic externally. For this, all the nodes in our "isolated" subnet will have to send "external" traffic to a particular node, to the gateway ("labs-codengo-gateway" in our case), which will then be responsible for routing the traffic further. To do this, on the ``labs-codengo-gateway`` node, we'll have to define a gateway in its netplan:

```
sudo nano /etc/netplan/00-installer-config.yaml 

network:
  ethernets:
    eth0:
      #dhcp4: true
      addresses:
      - 192.168.101.10/24
    eth1:
      addresses:
      - 192.168.100.10/24
      gateway4: 192.168.100.1
      nameservers:
        addresses: []
        search: []
  version: 2
```

We can then apply the changes, and we should be able to see those changes:

```
sudo netplan apply

ip route

default via 192.168.100.1 dev eth1 proto static
192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.10
192.168.101.0/24 dev eth0 proto kernel scope link src 192.168.101.10
```

Great, now we have a default route. Keep in mind that a DHCP server could also serve this kind of information, if we'd had one on our isolated network. Also note that we can also define multiple routes for more subnets, there's a field "routes" in netplan for that.

So, let's now see what happens if we now ping ``8.8.8.8`` from the ``labs-codengo-client`` node:

```
ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
^C
--- 8.8.8.8 ping statistics ---
22 packets transmitted, 0 received, 100% packet loss, time 21487ms
```

- So, it no longer says ``"Network is unreachable"``, but we still don't get any replies. That's because we haven't configured ``192.168.100.1`` to act as gateway / router yet. But first of all, we should at least make sure that we can reach the gateway:

```
ping 192.168.100.1
PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data.
64 bytes from 192.168.100.1: icmp_seq=1 ttl=64 time=0.555 ms
64 bytes from 192.168.100.1: icmp_seq=2 ttl=64 time=0.464 ms
^C
--- 192.168.100.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1016ms
rtt min/avg/max/mdev = 0.464/0.509/0.555/0.045 ms
```

It seems that we can. Of course, if we couldn't have reached the gateway, we couldn't have reached anything further anyways.

**TIP**: So, if you're trying to access the internet and can't reach it, at least try to see if you can at least reach the gateway. If you can't, you have issues in your local network. Try to see why the gateway is not responding (note that some gateways have firewall rules which blocks pinging it, but generally it shouldn't be a concern, and there are other ways of checking if it's reachable, not only ping). If it is reachable, try using traceroute to see if your requests is going more than 1-3 hops after your gateway (basically, still in your network's vicinity).

But we can at least see if the traffic is actually sent to it. On the ``labs-codengo-gateway``, run:

```
sudo tcpdump -v -eni eth1 | grep 192.168.100.10

tcpdump: listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
    192.168.100.10 > 8.8.8.8: ICMP echo request, id 3, seq 3, length 64
    192.168.100.10 > 8.8.8.8: ICMP echo request, id 3, seq 4, length 64
    192.168.100.10 > 8.8.8.8: ICMP echo request, id 3, seq 5, length 64
08:01:28.823558 00:15:5d:01:28:45 > 00:15:5d:01:28:43, ethertype ARP (0x0806), length 42: Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.100.1 tell 192.168.100.10, length 28
    192.168.100.10 > 8.8.8.8: ICMP echo request, id 3, seq 6, length 64
^C20 packets captured
22 packets received by filter
0 packets dropped by kernel
```

We are using a grep to filter out some of the traffic, otherwise we'd be overwhelmed by the volume of data we'd see. But tcpdump is a great tool for analysing network traffic on a network. It allows us to see what traffic is on a network, and in this case, it allows us to see that the ``"8.8.8.8 ICMP (ping) echo request"`` is on the right network. For Windows, you can use Wireshark.

Now, we're going to need route traffic received ``labs-codengo-gateway`` externally. We already have a default route there on ``eth0``, which can have access to the Internet. We're going to have to forward traffic onto ``eth0``. On this node, run:

```
# become root.
sudo su

# Enable IP forwarding.
echo 1 >> /proc/sys/net/ipv4/ip_forward

# It should already from at this point.

# Or edit /etc/sysctl.conf to make this change permanent, uncomment this line:
#net.ipv4.ip_forward=1

# Always accept loopback traffic.
iptables -A INPUT -i lo -j ACCEPT

# Accept traffic from the "Default Switch" network.
iptables -A INPUT -i eth0 -j ACCEPT

# Allow established connections.
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Masquerade.
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
# fowarding

# Forward traffic from eth1 to eth0.
iptables -A FORWARD -i eth1 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Allow outgoing connections from the LAN side.
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT

# exit from root access.
exit
```

- With these changes, "labs-codengo-gateway" forwards traffic from ``eth1`` (which is connected to our ``"Isolated"`` network) and forwards it to ``eth0``, from which it will be forwarded further and further. We should already see that ``labs-codengo-client`` has access ``8.8.8.8``, that it can get replies!

Cool, we have internet connectivity! Let's try to reach out to google.com:

```
ping google.com
ping: google.com: Temporary failure in name resolution
```

Oh, somehow our node does not know who google.com is. Well, we'll have to learn how accessing something like "google.com" actually works. Technically, all connections are made from an IP to an IP, but "google.com" is not an IP, it is a "domain name", and this name was given because it's easier to remember a name rather than a bunch of numbers. But who knows what IP does "goole.com" has? Well, that's more complicated question to answer, but long story short, there are some "domain name servers" or DNS servers which basically knows what IPs certain domain names have. For more information, on this topic and how DNS actually works, see this great video: https://www.youtube.com/watch?v=72snZctFFtA

But basically, our node is not configured with **any** sort of DNS server. That sort of information is typically served by the DHCP server in the first place, but we don't have that here. We're going to have to configure one ourselves. For this, we can edit the file ``/etc/resolv.conf`` file:

```
sudo nano /etc/resolv.conf

# add this line before nameserver 127.0.0.53
nameserver 8.8.8.8
```

The order of the nameservers matters, they are going to be queried in order. Typically, in a company / cloud / whatever, you'd have your own DNS servers which would have records for your internal nodes (because remembering IPs is hard, and they could change), so you'd have that first, then something like ``8.8.8.8`` (btw, this is Google's DNS server. There are others, like ``1.1.1.1``). But those internal DNS server may already be configured to have some other DNS servers as fallback.

After changing that file, we should be able to reach google.com:

```
ping google.com
PING google.com (142.250.201.206) 56(84) bytes of data.
64 bytes from bud02s35-in-f14.1e100.net (142.250.201.206): icmp_seq=1 ttl=55 time=10.5 ms
64 bytes from bud02s35-in-f14.1e100.net (142.250.201.206): icmp_seq=2 ttl=55 time=10.5 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 10.506/10.509/10.512/0.003 ms
```

Fantastic! Though, that ``/etc/resolv.conf`` change is not permanent, we'd have to save it in the netplan:

```
sudo nano /etc/netplan/00-installer-config.yaml

network:
  ethernets:
    eth0:
      #dhcp4: true
      addresses:
      - 192.168.101.10/24
    eth1:
      addresses:
      - 192.168.100.10/24
      gateway4: 192.168.100.1
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
        search: []
  version: 2
```

**TIP**: If you can reach ``8.8.8.8``, but can't reach google.com (name resolution failure), check your DNS settings, at least set ``8.8.8.8`` as a DNS server.

Note that having DNS records in a DNS registrar (aka having a public DNS record) costs money though. So, if you're going to have your own website, you'll have to buy a Domain Name (more like rent it). Note that you'll still need to own a public IP though (which costs money, IPv4 IPs being a limited resource), so your potential clients could reach you. For example, we've only seen IPs like ``192.168.100.10``, or ``172.27.96.1``, but those are private IPs, they are not externally reachable. And in fact, it's not our public IP. So, how do we figure out our Public IP? Well, there are services which can tell us that:

```
wget -qO- http://ipecho.net/plain | xargs echo
185.225.76.254
```

That looks nothing like our private IP. In fact, try running that on all your VMs, and even on your Windows node. You'll see the same IP everywhere. That's your public IP with which you're connecting to the internet. So, if a client will connect to ``185.225.76.254``, where will it get to?

But still, how come we see the same IP in all cases? Well, that is one of the magics of networking. Basically, whenever a packet goes through a gateway, its source IP is replaced with the gateway's IP (Network Address Translation (NAT), or more specifically, Source Network Address Translation (SNAT)), so the destination server actually knows to send the packet back.

But, for your demos and lab environments, you don't actually need a DNS server to assign "names" to your IPs. You can do that locally, by editing the ``/etc/hosts`` file (or ``C:\Windows\System32\drivers\etc\hosts`` on Windows):

```
sudo nano /etc/hosts

# add the following line
192.168.100.10 sobo.lan
```

- Now, on that node, we have an association between sobo.lan and the IP 192.168.100.10. Now, this works:

```
ping sobo.lan

PING sobo.lan (192.168.100.10) 56(84) bytes of data.
64 bytes from sobo.lan (192.168.100.10): icmp_seq=1 ttl=64 time=0.421 ms
64 bytes from sobo.lan (192.168.100.10): icmp_seq=2 ttl=64 time=0.356 ms
^C
--- sobo.lan ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1011ms
rtt min/avg/max/mdev = 0.356/0.388/0.421/0.032 ms
```

Keep in mind that this only works on the current node, where you have edited the ``/etc/hosts`` file.

Note that the topics on Layer 3 are not exhausted. There are still many things to talk about, like overlay networks (aka Layer 3 networks), VPNs.

Next, we're going to briefly mention the rest of the layers.


### L4 Transport layer

This layer is mostly responsible for how packets are transported. Typically, there's a maximum number of bytes that can be transmitted for a single packet, size which is named **MTU** (Maximum Transmission Unit), and typically 1500 bytes. There are better performing NICs and Switches that can handle MTUs of 9000 (also called Jumbo Frames), but when it comes to sending packets to the Internet, they'd still have to send at most 1500 bytes packets, being the standard / common size.

So, what happens if we need to transmit more that 1500 bytes? Well, the data needs to be Segmented into multiple packets. For example, if we need to send 2000 bytes over the Internet, we would basically send 2 packets:

- packet 1: size: ~1450 bytes of data (the rest of ~50 bytes basically contains information about the packet itself: source, destination, etc.)
- packet 2: size: ~600 bytes of data (contains the rest of the 550 bytes + information about the packet itself).

As you can see, the information transmission efficiency is not 100%, and it cannot ever by 100%, since we need to sacrifice some bytes for packet information itself. You can think of it like this: you need to send a 10kg package through the Postal Office, but the limit is 10kg, but you have to consider the weight of the box you're sending as well, as that's taken into account too.

Another thing to note here: the packets are segmented and sent, that's fine. But the receiver will have to assemble the packages back together to receive your unfragmented message / file. For this purpose, the order of the packages is going to be extremely important, otherwise the file / message could be corrupted. Well, for this, there is a standard protocol implemented for this: **Transmission Control Protocol** (**TCP**), which essentially ensures that all the packages are sent and received in order, and the entire communication process is verified / confirmed.

There is another protocol called **UDP**, which doesn't care about packets being received or received in order. Sometimes, some packets being lost is not that important: Internet calls (if the sound glitches for a millisecond, it's fine), streaming (if there's a funky patch in the stream for a frame out of 60, it's fine), etc. In those examples, UDP is preferable because it's faster and easier for the servers to transmit those data packets (TCP is very complex, and requires a lot verifications / validations, and if packets get lost, they'd have to be retransmitted, which may cause bottlenecks and congestion in the servers, leading to lag spikes, delays, or even crashes).

Note: TCP connections are also called "stateful" / "connection-oriented", while UDP connections are called "connectionless".

An important thing to note regarding networking protocols: there are hundreds of standard protocols, but they're mostly based on these 2 types: **TCP** and **UDP**. For example, **HTTP** is a **TCP** protocol, because the integrity and corectness of the data transmitted matters.

Another thing to note is that, whenever you open up a network connection, you essentially open up a client network port to connect to another port (all network connections are port to port). If you open up a port and listen and read it and send replies back on that port, you've essentially created a "server". All "server services" open up ports, read data from them, and send replies back. For example, if you access https://google.com, you're connecting to the port ``80`` (standard ``HTTP`` port), or port ``443`` (standard ``HTTPS`` port), and the complete URL is actually https://google.com:80 (since port ``80`` is default, it's implicit). Any port can be opened, but the first 1024 ports have standardized usages and purposes (e.g.: ``22`` is for SSH, ``80`` is for HTTP, etc.). They're well documented on the internet.


### L5 Session Layer

Sets up, controls, and tears down connections between 2 or more nodes. For example, DNS and DHCP protocols are on this layer. Other examples would be: authentication, storage-related network protocols like FTP and NFS, etc.


### L6 Presentation Layer

One notable thing that can happen on this layer is end-to-end data encryption between 2 nodes, typically through SSL / TLS certificates. May also include data translation, compression, etc.


### L7 Application Layer

This is the layer most users are familiar with, as they are always using it. Example protocols include: HTTP, SMTP (mail), FTP, SMB / CIFS (storage through networking), FTP, etc.


## Networking-related features

Knowing the OSI model is not that difficult. However, there are plenty of technologies that that build on top of the OSI model, improving it in a way or another. Other than that, there are various tehnologies which exist and which are still being created, mostly for the purpose of simplifying the development or application deployment and management process. Here is a very brief list:

- **Jumbo frames**: some NICs / Switches have this feature, allowing for better network efficiency, being able to send bigger packets.
- **NIC bonding**: essentially "melt together" multiple NICs on a node, configuring them to act as one. This can offer improved network throughput, as well as fault-tolerance (if a NIC / cable / switch goes down, the network connectivity still remains UP). May require additional configuration of the Switches side as well (and the Switches would have to support it as well), especially for the better options like LACP.
- **HAProxy / Load Balancers**: Services deployed in front of your application (which has been deployed in multiple instances / replicas) and load balances ingress (incomming) traffic between the replicas. Additionally, if a replica goes down, it wil lbe excluded from the list.
- **Floating IPs / Virtual IPs**: Servers may have changing IPs (especially if they can die and / or have to be replaced, or if their number can increase / decrease), so it's not feasible to have clients / services connect to ephemeral IPs. Instead, a VIP can be associated with a HAProxy / LoadBalancer, and the users won't have to worry about those ephemeral IPs.
- **Nginx Ingress Controller / Proxy**. We'll see a very basic example below. Can also be configured with TLS certificates, so the underlying services won't have to.
- **Service Meshes**: Inter-service communication.
- **Security Groups / Network Policies**: Essentially, a "firewall" applied on the switch port itself which cannot be removed by the host itself, only by an external manager. It is something you'd see in Public Clouds as well. For example, you'd probably want to add Rules for SSH and / or HTTP traffic to your VMs there.
- **Networking QA (Quality Assurance)**: Essentially limiting the networking throughput.
- **BGP**: Service Route advertisement / configuration.

That's what's currently coming to mind. There are many other things being developed, especially in the era of the cloud, in the realm of SDNs (Software Defined Networks).

There are a few other things which might be interesting to research:

- How to define and use bridges (in netplan as an example).
- How to define and use vlans (in netplan as an example). Might need to also learn about VLAN trunking.
- How to define and use bonds (in netplan as an example).


### Nginx service proxy

This is going to be an example of a networking "trick" we can do.

So, let's say that we own a Public IPv4 IP. People and clients will be able to use that IP to access our website. However, we'd like to be able to serve multiple different websites / applications through that IP. How can we do that?

Well, first of all, we could start different services on different ports. The users would then have to include the port in the URL... but that's not very user friendly. How can remember numbers anyways?

Well, there's another way: we can essentially serve different websites / applications based on the Domain Name used to access the IP (we're going to need Domain Names anyways). Basically, we're still going to have **ONE** single Public IP (let's say ``192.168.100.10`` (I know it's not public, it's just an example)), but we can have multiple DNS records pointing to it, like: sobo.lan and gigi.petrolieru.ro.

Let's simulate this situation in our lab environment! Of course, we're going to do a veeery bare-bones / stripped version of a real scenario, so keep that in mind and apply proper judgement when using it in real situations.

We're going to work on ``lab-codengo-client``. First of all, we're going to need 2 differerent pages to serve. 

```
nano website1.html

<html>
        <head>
                <title>Sobolan ezoteric!</title>
        </head>
        <body>
                <p>Sobolanisme pe aici.</p>
        </body>
</html>
```

And:

```
nano website2.html

<html>
        <head>
                <title>Gigi Petrolieru's page!</title>
        </head>
        <body>
                <p>Gigi Petrolieru' was here.</p>
        </body>
</html>
```

We're going to have 2 different "applications", each serving a different website. We could use a proper HTTP service, like apache or nginx. But again, we're going bare bones here, we're going to use ``netcat``. We're going to create a script which starts a ``netcat`` server:

```
touch start_website.sh
sudo chmod a+x start_website.sh
nano start_website.sh

#!/bin/bash

if [ "$#" -ne 2 ]; then
  echo "Usage: $0 some-index.html port-number"
  exit 1
fi

page=$1
port=$2

while true; do printf 'HTTP/1.1 200 OK\n\n%s\n' "$(cat $page)" | netcat -w 1 -l $port; done
```

And then we can start the servers and put them in the background through that ``&`` at the end:

```
./start_website.sh website1.html 8080 &
./start_website.sh website2.html 9090 &
```

Let's see that we can get those pages:

```
curl http://127.0.0.1:8080
curl http://127.0.0.1:9090
```

Great! Now we have 2 services running on ``lab-codengo-client``. Now, we're going to need an nginx that can proxy the HTTP requests based on the used Domain Name. We need to install nginx:

```
sudo apt install nginx
```

And then configure it so:

```
sudo nano /etc/nginx/conf.d/default.conf

server {
        listen 80;
        server_name sobo.lan;
        location / {
                proxy_pass http://127.0.0.1:8080;
        }
}

server {
        listen 80;
        server_name gigi.petrolieru.ro;
        location / {
                proxy_pass http://127.0.0.1:9090;
        }
}
```

After configuring nginx, we need to restart it, and there should be no errors:

```
sudo systemctl restart nginx
```

Now, technically, when we access ``192.168.100.10`` using the Domain Names ``sobo.lan`` or ``gigi.petrolieru.ro``, we should be getting the coresponding application / response. Let's try it out! We don't have money for buying these Domain Names, so we'll do the next best thing. On the ``lab-codengo-gateway`` node, let's edit the ``/etc/hosts`` file to include our desired DNS entries:

```
# Add these lines
192.168.100.10 sobo.lan
192.168.100.10 gigi.petrolieru.ro
```

Now, on the ``lab-codengo-gateway`` node, if we try to access the following URLs, we should be getting the right data back!

```
curl http://sobo.lan
curl http://gigi.petrolieru.ro
```

Notice that we didn't specify any port, meaning that the request will go to port ``80``, which is the standard ``HTTP`` port, which is opened up by the nginx service we installed and configured above. We're essentially connecting to the same IP and port, but we're accessing 2 different applications. Of course, we could have more applications as well.

Additionally, one other interesting side effect of this: this could potentially solve some of our intentional XSS (cross-site scripting) issues, as long as our applications use relative URLs. For example, if we have 3 different HTTP applications and they'd have to access resources from each other in a frontend (webpage), they'd essentially do XSS, which is typically blocked. However, if we "group" them up under the same Domain Name, it wouldn't be XSS anymore.


## Conclusions

Networking is a very complex and interesting topic, and we barely scratched the surface here (I'd say 2%). It's something that you'd have to learn for quite a long while, you could end up thinking you know this topic, and then it still could give you the Impostor Syndrome plenty of times. :) But it's still a very important topic, as it impacts very many aspects of our lives, since we are just some Internet Rats, doing Sobolanisme on the Internet. :)

May your Net work, but if it doesn't, now you might have some ideas. :)
