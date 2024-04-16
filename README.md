# Lab Redesign: VPN and VM Network Setup

## Part 1: Network Configuration

### Virtual Network Setup
- **Create a Virtual Network** for your VMs that is isolated from your host's primary network. This network can be managed using KVM’s built-in tools and should use NAT to provide internet access to VMs without exposing them directly to the public network.
- **Network Bridge** (optional): If you prefer not to use NAT and have VMs appear as if they are on the same physical network as the host, set up a bridge. However, NAT will be sufficient and simpler for most use cases.

### Assign Static IPs
- Assign static IP addresses within the virtual network for each VM. This ensures that each VM can be consistently accessed at the same address by the VPN users.

## Part 2: VPN Server Setup

### Choose VPN Software
- **OpenVPN** is a recommended choice due to its robust security features and extensive configuration options.

### Install and Configure VPN Server
- Install OpenVPN on a dedicated VM or on the host machine.
- Configure it to route traffic to your virtual network where the VMs reside.

### VPN Client Setup
- Configure VPN client software on your colleagues' machines. Each client configuration file should be tailored to ensure proper routing to the internal network where their designated VM is located.

## Part 3: VM Setup

### VM Installation
- Use `virt-install` to create VMs for each user. Ensure each VM’s network is configured to connect to your virtual network.

### Operating System and Software
- Install the required operating system and any necessary software on each VM.

## Part 4: Security and Access Control

### Firewall and Security Rules
- Configure firewalls on each VM and the VPN server to restrict access to authorized users only.

### Access Controls
- Set up user accounts and permissions on each VM to ensure users can access only their designated VMs.

## Part 5: Monitoring and Maintenance

### Network Monitoring Tools
- Implement network monitoring tools to oversee traffic flow and manage network performance.

### Regular Updates and Patches
- Keep the VPN software and VM operating systems updated with the latest security patches.

## Example Configuration Commands

Here’s how you might configure the VPN server for routing and security:

```bash
sudo apt update
sudo apt install openvpn easy-rsa
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
source vars
./clean-all
./build-ca
./build-key-server server
./build-dh
./build-key client1
