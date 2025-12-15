 When I started my homelab, I wanted to put everything I would be testing, whether it was benign, insecure, or malicious, behind a firewall. Juice Shop, Metasploitable, a malware analysis sandbox, an exploitable Windows Domain Controller, etc.

These labs will be sharing physical infrastructure with some of my self-hosted services such as Nextcloud, Immich, and Jellyfin, which I consider to be critical infrastructure and must be highly available.

Adding extra nodes to a Proxmox cluster, installing Ceph, and enabling HA on a VM isn't particularly difficult.

What seemed less straightforward was adding high availability to the firewall.

Because pfSense was going to be the default gateway for my lab, I needed to ensure that it would be reachable from any node, and that it wouldn't be unintentionally circumvented by a DHCP-seeking VM on a separate node.

In a fully physical environment, it's easy to see where a WAN cable comes in, attaches to the WAN port of the edge device, and where the LAN cable attaches to the LAN port and goes to the next downstream device, such a switch.

In a multi-node hyperconverged infrastructure environment, such cabling is gone. In this case, I have a 16-port switch which all of the physical infrastructure is directly connected to.

1) PVE Node 1
2) PVE Node 2
3) PVE Node 3
4) PVE Node 4
5) PVE Node 5
...
6) Synology NAS (Bonded)
7) Synology NAS (Bonded)
...
8) Workstation
9) Uplink

In the most straightforward configuration, I can have a pfSense VM plugged into vmbr0 on that node for WAN and vmbr1 for a LAN connection. Easy. With this, I can then have any other VMs connect to vmbr1, and they'll be on the same virtual switch as the pfSense box, and all internet traffic from the LAN must first go through the pfSense.

With High Availability configured through Ceph, I can be confident that if there's a failure of one node, any services that were running on that node will quickly migrate to another node. For example, if the node running my Caddy reverse proxy loses power, that container, and any other containers and VMs, will be online again in hardly a minute.

Another reason for having so many nodes is their limited compute. Four of my nodes only have 16 GB of RAM and 512 GB of Ceph storage in addition to a 40 GB boot drive. This means I want to distribute the load across the different nodes.

So if I have a LAN VM on one node, and the pfSense VM is on another node, how do I ensure the LAN VM uses pfSense as a gateway instead of my home network? After all, they're both available on the same broadcast domain, physically.

VLANs. We're using VLANs.

In the linear setup on one node, we didn't need VLANs because the Linux bridge acted as a pseudo physical link between the LAN device and the firewall. Since vmbr1 no longer directly connects the two virtual devices, it will be sending its packets onto the same L2 switch as everything else. We need to ensure that the virtual device does not accidentally reach the wrong Layer 3 device when it sends a DHCP request.

Isn't this networking basics? Why is this relevant to security? Does it even matter in my home lab?

Is it networking basics? Yes. Anyone with a CCNA should be able to configure this, and anyone with a Network+ should be able to at least describe it.

Why is this relevant to security? Because if a threat actor wants to get into the LAN from outside, or wants to access a WAN resource from within the LAN, we need to be confident that all traffic will pass through the firewall. If it isn't set up correctly, it is theoretically possible that a threat actor could plug their own device into the LAN, bypass the firewall, and access their resources. If it's a known good device, and it accidentally gets a DHCP lease from the wrong device, it would then be exposing itself to larger network. Both of these are bad.

It's easy to imagine the threats from above will risk Confidentiality and Integrity. There's one more problem we can address: Availability. Even without addressing denial of service attacks, if there are two L3 devices responding to DHCP requests on the same broadcast domain, IP addressing gets weird, firewall rules don't get applied correctly, and visibility is broken - devices that are supposed to be on the same network aren't reliably available.

How do VLANs fix that?

There are three ways to direct network traffic. Layer 1 will give you Physical control over the traffic direction. Layer 3 will give you real Network direction - networks and subnets. We can't do layer 1 or 3 - that's the vmbr1 problem and the `DHCPDISCOVER` problem respectively. That leaves layer 2, since all nodes are connected to the one L2 switch. And VLANs are on layer 2.

By default, untagged, or VLAN 1 (again as default), traffic goes over all L2 devices. With VLAN aware ports, we can say whether or not certain frames are allowed to pass through a NIC. If we tell a port, "You are only allowed to pass traffic from VLANs 10, 20, and 30," then that's what it's going to do. If it sees a frame from an unapproved VLAN, the frame gets dropped.

Let's say we want "normal" traffic to stay on VLAN 1, since that's the default and will cause the least amount of trouble. The pfSense WAN NIC can stay on VLAN 1 so the workstation, NAS, and the WAN uplink can communicate with pfSense. For everything in our PVE cluster that needs direct access to the network, we'll connect it to vmbr0.

There's one more thing we need to do with vmbr0. Since vmbr1 will be using the same interface and only one bridge is allowed to have 

For our LAN devices, let's use VLAN 10. We can set vmbr1 on each of our nodes to be VLAN aware, and put VLAN 10 in its list. Now, when we attach something to vmbr1, if it isn't communicating on that VLAN, the bridge will drop the frames immediately.

Let's say we have a Windows VM on VLAN 10, connected to vmbr1, and on another node, pfSense is on vmbr1. We now have a LAN machine ready to communicate with its firewall, and pfSense ready to communicate to the LAN machine, both on vmbr1 and VLAN10, but it isn't working as expected.

The switch connecting the nodes needs to be set up, too.

![[assets/img/VLANconfig.png]]
On this switch, I've set up five VLANs. VLANs 10, 20, and 30 are connected to the LAN port of pfSense through PVE's vmbr1. Notably, VLAN 1 is still enabled on all 16 ports. This is because while we don't want vmbr1 to be able to use VLAN 1, we still need vmbr0 to use VLAN 1. And since VLAN 1 is not on the list of approved VLANs for vmbr1, it is secured. Last is VLAN 99, which must be set as the default VLAN for untagged traffic because if left to VLAN 1, untagged traffic would be able to escape from the firewall's network.

![[VLANPVID.png]]

Here, the PVIDs for each port has been assigned to either 1 or 99, depending on whether the port is allowed to forward untagged frames outside of pfSense.

---

I thought that They needed to be on 99, but the I could no longer log into the PVE GUI, and I couldn't even reach two of the nodes at all. I could SSH onto two of them, but not the other two. I didn't try the fifth.

THE ORDER MATTERS.

THEY MUST BE IN A QUORUM.