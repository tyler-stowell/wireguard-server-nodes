# wireguard-server-nodes

Creating a wireguard VPN with a central server and nodes:

First, the following needs to be in the server's wg0.conf file:


###############################################  
[Interface]  
Address = 10.0.0.0/24  
SaveConfig = true  
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; iptables -A FORWARD -o %i -j ACCEPT;  
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; iptables -D FORWARD -o %i -j ACCEPT;  
ListenPort = <server port>  
PrivateKey = <server private key>  
###############################################  
  
The given interface will accept all traffic sent to an address of 10.0.0.x. The PostUp and PostDown sections give it information for how to forward packets and how to receive them, either between peers or to eth0.

Next, the following information needs to be added to the server's wg0.conf for each Peer:

###############################################  
[Peer]  
PublicKey = <client public key>  
AllowedIPs = 10.0.0.<client number>/32  
Endpoint = <client ip address>:<client port>  
###############################################  
  
The interface will then forward any traffic on address 10.0.0.<peer number> to <peer ip address>:<peer port>.
  
Next, the client needs to be configured with the following information in its wg0.conf file:

###############################################  
[Interface]  
Address = 10.0.0.<client number>/24  
PrivateKey = <client private key>  
ListenPort = <client port>  
  
[Peer]  
PublicKey = <server public key>  
Endpoint = <server ip address>:<server port>  
AllowedIPs = 10.0.0.0/24  
###############################################  
  
This config file tells the wg0 interface to accept traffic from 10.0.0.x, then forward all of that traffic directly to the server.

For example, if two clients are created with 10.0.0.2 and 10.0.0.3, the following process allows them to connect:

1) The user on client 10.0.0.2 calls the command "ssh 10.0.0.3".
2) An ssh request is sent to 10.0.0.3:22
3) The wg0 interface on client 10.0.0.2 catches this request, encrypts it with <server public key>, and sends the encrypted packet to <server ip address>:<server port>
4) The wg0 interface on the server takes this incoming packet, and decrypts it with its own private key. Then, it sees that the packet is intended for 10.0.0.3. So, it encrypts it with the second client's public key and sends it to that client.
5) Finally, the client decrypts the packet from the server, and sees that it is intended for itself. So, it keeps the packet.
