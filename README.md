# Virtual_Private_Network
Creating your own VPN (Virtual Private Network) 



 Building a VPN allows you to have a secure connection between your local machine and another network over the internet. Setting up a VPN can offer you more control over the infrastructure and might be ideal for unique requirements.

Here's a basic outline on how to set up a VPN server using the open-source software OpenVPN:

1. Choose a Server:
Local Machine: You can set up a VPN server on your local machine, but this is typically not recommended for a production environment.

Cloud Server: Providers like AWS, DigitalOcean, Azure, etc., allow you to spin up virtual servers. You can set up a VPN on one of these servers for remote access.

2. Setting Up OpenVPN:
For this example, I'll assume you're using a fresh Ubuntu server:

A. Install OpenVPN:



sudo apt update && sudo apt upgrade -y
sudo apt install openvpn -y



B. Get EasyRSA:

EasyRSA is a script that will help us create a Certificate Authority (CA) and other necessary certificates.

bash



wget -P ~/ https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.8/EasyRSA-3.0.8.tgz
tar xvf ~/EasyRSA-3.0.8.tgz -C ~/



C. Set Up the CA and Generate Certificates:

Navigate to the EasyRSA directory and initialize the PKI:


cd ~/EasyRSA-3.0.8/
./easyrsa init-pki
./easyrsa build-ca



Generate a key pair for the server:

./easyrsa gen-req server nopass
./easyrsa sign-req server server


Generate Diffie-Hellman parameters:


./easyrsa gen-dh


D. Configure OpenVPN:

Copy the necessary files to OpenVPN's directory:

sudo cp ~/EasyRSA-3.0.8/pki/ca.crt /etc/openvpn/
sudo cp ~/EasyRSA-3.0.8/pki/issued/server.crt /etc/openvpn/
sudo cp ~/EasyRSA-3.0.8/pki/private/server.key /etc/openvpn/
sudo cp ~/EasyRSA-3.0.8/pki/dh.pem /etc/openvpn/


Create a basic server configuration in /etc/openvpn/server.conf. Adjust settings as necessary:

port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
explicit-exit-notify 1


