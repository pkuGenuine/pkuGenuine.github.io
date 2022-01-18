# OpenVPN

OpenVPN is a full-featured SSL VPN which implements OSI layer 2 or 3 secure network extension using the industry standard SSL/TLS protocol, supports flexible client authentication methods based on certificates, smart cards, and/or username/password credentials, and allows user or group-specific access control policies using firewall rules applied to the VPN virtual interface.

## Security Model

### Review on TLS

The TLS protocol runs in the application layer and aims primarily to provide privacy and data integrity between two or more communicating computer applications. 

TLS protocol uses asymmetric encryption system to authenticate the server and change information used to generate symmetric keys. During the handshake phase, the client and server can negotiate the [key-generation algorithm](https://en.wikipedia.org/wiki/Transport_Layer_Security#Algorithms) and the cipher.

> Aside: Connection & Session
> 
> Difference between connection and session is that connection is a live communication channel, and session is a set of negotiated cryptography parameters.
> 
> You can close connection, but keep session, even store it to disk, and subsequently resume it using another connection, may be in completely different process, or even after system reboot (of course, stored session should be kept both on the client and on the server).
> 
> On other hand, you can renegotiate TLS parameters and create entirely new session without interrupting connection.
> 
> Question: TLS session will keeps the derived keys. No idea how to resume a session. And another question is, I can not find any documentation about how TLS works when both side have to authenticate each other.

### Authentication: PKI System

The most used authentication system for OpenVPN is based on on certificates. And usually the authentication is bidirectional.

The PKI consists of:

- A master Certificate Authority ( CA ) certificate and key which is used to sign each of the server and client certificates.
- A separate certificate (also known as a public key) and private key for the server and each client.

Both server and client will authenticate the other by first verifying that the presented certificate was signed by the master certificate authority.

> Notice that to verify a signature, only public key is required. It is possible to place the private key on a seperated machine.
> 
> And with PKI system, it is possible to revoke a compromised certificates.

In the now-authenticated certificate header, both side can check other information such as the certificate common name or certificate type.

### Key Generation

#### DH Key Exchange

Diffie–Hellman key exchange is a method of securely exchanging cryptographic keys over a public channel.

An analogy illustrates the concept of public key exchange by using colors instead of very large numbers:

![avatar](https://upload.wikimedia.org/wikipedia/commons/thumb/4/46/Diffie-Hellman_Key_Exchange.svg/500px-Diffie-Hellman_Key_Exchange.svg.png)

Two parties publicly agree on an arbitrary starting color that does not need to be kept secret ( but should be different every time ). Each party also selects a secret color that they keep to themselves.

> Aside: Actually it is OK for the starting color to be same each time or shared with others, as long as they are complex enough. Corresponding vulnerability is called [Logjam](https://en.wikipedia.org/wiki/Logjam_(computer_security)).

The crucial part of the process is that two parties each mix their own secret color together with their mutually shared color, and then publicly exchange the two mixed colors ( it is practically impossible to calculate the secret color with mixed color and public starting color ). 

Finally, each of them mixes the color they received from the partner with their own private color to generate the "key color".

Bringing the analogy back to a real-life exchange using large numbers, the starting color consists of a large prime number $p$ and one of its primitive root modulo $g$. These two values are chosen in this way to ensure that the resulting shared secret can take on any value from $1$ to $p–1$.

Each of them will pick up an integer as the secret color, and exchange the processed integer. There is a simple implementation at DH's [wiki page](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange#Cryptographic_explanation). The difficult problem here is the [discrete logarithm problem](https://en.wikipedia.org/wiki/Discrete_logarithm_problem).


#### Implementation in OpenVPN/TLS

The "starting color", $p$ and $g$ are called DH parameters. We mentioned before it is safe to reuse them as long as they are complex enough ( longer than 1024 bits ). There are even a few such parameters standardized for example in RFC 5114.

The generation of such parameters is computationally expensive. Thus, when setting up a OpenVPN server, they are generated in advance.

Notice that DH by itself doesn't provide authenticity. TLS can combines DH and RSA to exchange a symmetric key with integrity, authenticity as well as authenticity.

#### Ephemeral and/or Static Keys
The used keys can either be ephemeral or static (long term) key, but could even be mixed.

Ephemeral keys ( DHE ) are usually used for key agreement. Provides forward secrecy, but no authenticity. On the other hand, static keys do not provide forward secrecy, but implicit authenticity. Since the keys are static it would for example not protect against replay-attacks.

> Aside: [Forward Secrecy](https://en.wikipedia.org/wiki/Forward_secrecy)
> 
> In cryptography, forward secrecy, also known as perfect forward secrecy ( PFS ), is a feature of specific key agreement protocols that gives assurances that session keys will not be compromised even if long-term secrets used in the session key exchange are compromised. 
> 
> The value of forward secrecy is that it protects past communication. 


## Network Structure

### Bridge or Routing
OpenVPN can be configured as either routing VPN or Bridge VPN. Overall, routing is probably a better choice for most people, as it is more efficient ( smaller overhead ) and easier to set up.

When a client connects via bridging to a remote network, it is assigned an IP address that is part of the remote physical ethernet subnet and is then able to interact with other machines on the remote subnet as if it were connected locally.

> Aside:
> 
> Bridging setups require a special OS-specific tool to bridge a physical ethernet adapter with a virtual TAP style device. 

When a client connects via routing, it uses its own separate subnet, and routes are set up on both the client machine and remote gateway so that data packets will seamlessly traverse the VPN.

Bridging and routing are functionally very similar, with the major difference being that a routed VPN will not pass IP broadcasts while a bridged VPN will.

> Question: If using Bridging, can client gain a IP from DHCP server in the VPN's network?

The discussion below will focus on routing method.



## Configurations for Server

An example configuration file for server can be found in [OpenVPN's repo](https://github.com/OpenVPN/openvpn/blob/master/sample/sample-config-files/server.conf).

### Concepts of Addressing
Several network topologies exist for servers configured to accept multiple client connections.

Topology "subnet" is recommended for modern servers. Note that this is not the current default. Addressing is done by IP & netmask. To make the server adopt "subnet" topology, add `topology subnet` to its config file.

To allocate IP addresses to it clinets, add a line starts with `server` to the config file. For example:

~~~
server 192.168.10.0 255.255.255.0
~~~

The server will take 192.168.10.1 for itself, the rest will be made available to clients.

Notice that the subnet should not be conflict with any subnet behind server or client. Thus make sure to avoid common subnet range, like 192.168.1.0/24.

### Route

With some configuration, the subnet(s) behind server and ones behind client(s) can be connected.

For example, adding `push "route 192.168.8.0 255.255.255.0"` in server's config file will ask the client to modify its route table and route packets to 192.168.8.0 to the server.

It is also possible for server to access the subnet behind client by adding `route` lines in config file. See more details in the example configuration file mentioned before.

> Question:
> 
> What if the subnet behind server is conflict with the client's subset? 
> 
> Probably adopting NAT in server side is a solution.

### DNS

The OpenVPN server can push DHCP options such as DNS and WINS server addresses to clients. Windows clients can accept pushed DHCP options natively, while non-Windows clients can accept them by using a client-side up script which parses the `foreign_option_n` environmental variable list. Check the [man page](https://openvpn.net/community-resources/reference-manual-for-openvpn-2-0/) for more info.


