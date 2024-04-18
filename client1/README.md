# Client Guide for Connecting to OpenVPN and SSHing into VM1
## Step 1: Install OpenVPN Client
Open your terminal and run the following commands to install OpenVPN:

```bash=
sudo apt update
sudo apt install openvpn
```

## Step 2: Obtain VPN Configuration File
You need to receive the `client1.ovpn` file from the VPN server administrator. This file contains all the required configuration to connect to the VPN.
### Note
All required files have been placed in the `client1` directory in this repository.

## Step 3: Connecting to the VPN
Start the VPN connection with the following command:
```bash=
cd ./client1
sudo openvpn --config client1.ovpn
```
Watch for the terminal output to end with "Initialization Sequence Completed," indicating that the connection is successful.

## Step 4: SSH into VM1
After establishing the VPN connection, you can access VM1 via SSH using the internal IP address provided by the VPN server administrator (in this example, `192.168.100.101`).

* user: skymizer1
* machine name: skymizervm1
* passwd: skymizer1

Use the following command to SSH into the machine:
```bash=
ssh skymizer1@192.168.100.101
```