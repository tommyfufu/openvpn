# OpenVPN Server Setup on Ubuntu

This guide provides instructions on setting up an OpenVPN server on Ubuntu, including troubleshooting steps based on common issues.

## Step 1: Install OpenVPN and Easy-RSA

```
sudo apt update
sudo apt install openvpn easy-rsa
```

## Step 2: Set Up Easy-RSA
```
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
```

## Step 3: Build the CA
The Certificate Authority (CA) is the root of the PKI hierarchy. It is responsible for issuing certificates and managing their revocation. In a VPN, the CA's role is to:

* **Sign Certificates:** The CA's signature on a certificate is a stamp of approval that establishes trustworthiness.
* **Verify Identity:** Any entity with a certificate signed by the CA is trusted to be who they claim to be.
* 
The CA certificate (`ca.crt`) is used by both the server and clients to verify each other's certificates.
```
./easyrsa init-pki
./easyrsa build-ca
```

## Step 4: Generate Server and Client Certificates
Certificates are a part of the mutual authentication process that occurs when establishing a VPN connection.

* **Server Certificate and Key (server.crt and server.key)**: These are used to authenticate the server to the clients. The server presents its certificate (server.crt) to the client, which verifies it against the CA certificate. The server uses its private key (server.key) to encrypt communications, which can only be decrypted with the corresponding public key found in server.crt.

* **Client Certificates and Keys (client1.crt and client1.key)**: Similar to the server certificate, each client has its own certificate and private key. The client certificate is used to authenticate the client to the server. Each client presents its certificate to the server upon connection, which the server verifies against the CA certificate.

* **Diffie-Hellman Parameters (dh.pem)**: This file is used in establishing a secure channel over which the server and client can agree on encryption keys. It is part of the process that allows secure key exchange without the need to transmit the keys themselves over the network.
```
./easyrsa gen-dh
./easyrsa build-server-full server # I made my server has passphrase
./easyrsa build-client-full client1 nopass  # I have three clients
./easyrsa build-client-full client1 nopass
./easyrsa build-client-full client1 nopass
```

### Step 5: Configure the OpenVPN Server
* **TLS Auth Key (ta.key)**: This HMAC signature file provides an additional layer of security. Both the server and clients use it to sign their TLS traffic, providing a way to ensure that the data has not been tampered with and comes from a legitimate source before the full TLS negotiation takes place. It helps protect against DoS attacks and unauthorized access attempts.

1. Copy the server certificate, key, and CA certificate, and also copy the server configuration file (you can use the example from /usr/share/doc/openvpn/examples/sample-config-files/server.conf) to the OpenVPN directory (/etc/openvpn/).
:::warning
According to [Ubuntu](https://ubuntu.com/server/docs/service-openvpn) and [OpenVPN](https://openvpn.net/community-resources/how-to/) documentation, it states that this step is "common practice," but it is necessary in my practices. I'm not sure if the reason is because my version of OpenVPN was installed via `apt install openvepn easy-rsa`
:::

```
sudo cp ~/openvpn-ca/pki/ca.crt /etc/openvpn/
sudo cp ~/openvpn-ca/pki/issued/server.crt /etc/openvpn/
sudo cp ~/openvpn-ca/pki/private/server.key /etc/openvpn/
sudo cp ~/openvpn-ca/pki/dh.pem /etc/openvpn/
sudo cp ~/openvpn-ca/ta.key /etc/openvpn/
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/
```

2. Edit the OpenVPN Server Configuration File
Edit /etc/openvpn/server.conf and update the paths like below demo:

```=plaintext
ca ca.crt 
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0 # 0 for server config, 1 for client config

# uncomment some setting

# Configure server mode and supply a VPN subnet
# for OpenVPN to draw client addresses from.
# ...
server 10.8.0.0 255.255.255.0

# It's a good idea to reduce the OpenVPN
# daemon's privileges after initialization.
user nobody
group nogroup

# If enabled, this directive will configure
# all clients to redirect their default
# network gateway through the VPN, causing
# all IP traffic such as web browsing and
# and DNS lookups to go through the VPN
# (The OpenVPN server machine may need to NAT
# or bridge the TUN/TAP interface to the internet
# in order for this to work properly).
push "redirect-gateway def1 bypass-dhcp"

# Certain Windows-specific network settings
# can be pushed to clients, such as DNS
# or WINS server addresses.  CAVEAT:
# http://openvpn.net/faq.html#dhcpcaveats
# The addresses below refer to the public
# DNS servers provided by opendns.com.
push "dhcp-option DNS 208.67.222.222"
```

3. Start OpenVPN Service

```
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
```

### Step 6: Troubleshooting
1. If the OpenVPN service fails to start, check the system logs:

```
journalctl -xeu openvpn@server.service
```

2. Ensure the certificate files are in the correct directory /etc/openvpn/, which is the standard location for OpenVPN.
3. Double-check file ownership and permissions, ensuring that the OpenVPN process can access them.