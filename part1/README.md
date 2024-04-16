# Part 1: Virtual Network Configuration and NAT Setup

This part of the lab focuses on setting up a virtual network for your virtual machines and configuring Network Address Translation (NAT) to allow these VMs to access the internet securely through the host's single public IP address.

## Step 1: Creating a Virtual Network

We'll create a virtual network using `virsh` with NAT to manage the network internally and provide internet access through the host machine.

### 1.1 Create Network XML Configuration

First, define your network with an XML configuration. Create a file named `vmnetwork.xml` with the following contents:

```xml
<network>
  <name>vmnetwork</name>
  <uuid></uuid>
  <forward mode='nat'/>
  <bridge name='vmnet0' stp='on' delay='0'/>
  <ip address='192.168.100.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.100.50' end='192.168.100.254'/>
    </dhcp>
  </ip>
</network>
```

### 1.2 Load and Start the Network
After creating the XML file, load this network configuration into libvirt:

```bash=
sudo virsh net-define vmnetwork.xml
sudo virsh net-start vmnetwork
sudo virsh net-autostart vmnetwork
```

This sets up the network named vmnetwork with a NATed DHCP pool from 192.168.100.50 to 192.168.100.254.

## Step 2: Verify the Network Setup
Ensure that the network is correctly set up and active:
```bash=
virsh net-list --all
```
You should see vmnetwork listed as active.
![image](https://hackmd.io/_uploads/r1vRLe2xC.png)

## Step 3: Configure IP Forwarding and NAT
Setting up NAT involves enabling IP forwarding on the host and configuring iptables to masquerade the IP addresses of VM traffic.

### 3.1 Enable IP Forwarding
Enable IP packet forwarding in the kernel:
```bash=
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

To make IP forwarding permanent, add the following line to `/etc/sysctl.conf`

```bash=
net.ipv4.ip_forward = 1
```

Apply the changes:

```bash=
sudo sysctl -p
```
![image](https://hackmd.io/_uploads/rJO22x2gR.png)

### 3.2 Add Masquerade Rule to iptables
Add a masquerade rule to allow VMs to share the host's public IP:
```bash=
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Replace eth0 with your external network interface connected to the internet. 

### 3.3 Save iptables Rules
To ensure iptables rules persist after a reboot, install `iptables-persistent` and save the rules:
```bash=
sudo apt install iptables-persistent
sudo netfilter-persistent save
```
![image](https://hackmd.io/_uploads/ByiRdlnlR.png)

## Understanding Masquerade Rule in iptables and Network Interface Configuration
Why Add a Masquerade Rule to iptables?
Adding a masquerade rule to iptables is necessary for enabling NAT (Network Address Translation) functionality on your Linux system, which you're using as a hypervisor. This setting is crucial in scenarios where you have virtual machines or containers that need to access the internet through the host's single public IP address, but without exposing the VMs directly to the external network.

Here’s what the masquerade rule does:

* **Masquerading**: It masks the source IP address of the packets originating from the VM (which has a private IP address like 192.168.100.x) with the host's public IP address (140.113.151.61 in your case). When the response comes back to the host, iptables translates the addresses back so that the response reaches the correct VM.
* **Stateful NAT**: iptables remembers outgoing connections, so it can correctly route the incoming packets (responses) to the originating internal IP address.
This is essential for allowing VMs on a NATed (private) network to communicate with the internet securely and without requiring multiple public IP addresses.
