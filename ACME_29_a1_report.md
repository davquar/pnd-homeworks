# 1. ACME-29 REPORT

## 2. Student (Name Surname numbers)

Edoardo Gabrielli 1693726
Davide Quaranta 1715742
Alessio Tullio 1809077

## 3. Initial brainstorming

After a brief introduction to the network topology and the tools used to handle the machines (Proxmox), we reasoned all togheter in depth on the ways to better configure the network. 

The best solution appeared to be the step by step approach: i.e. starting from configuring the main router, then the neighbour subnetwork (External and DMZ), finally the internal firewall with the Internal and Clients network.

For the procedures we choose to follow the one described in the assignment, since the proposed tool appeared to work properly to succesfully complete all the requests.

## 4. Setup of the infrastructure for IPv6 addressing

### a1) Main Firewall-Router

Using the opnsense platform we configured all the interfaces (DMZ, WAN, EXTERNAL_CLIENTS,  INTERNAL). For all of them we enabled the prevent interface removal, removed the block bogon and block for private, due to the private addresses we are using for our network.

The following rules are specific setting for each interface.
In assigning the prefix IDs, we opted for a structure close to the ipv4 topology, i.e. for a 6.0/24 ipv4 network, in ipv6 we choose 6 as prefix ID in ipv6 for that subnetwork.	 

#### WAN interface

- ipv6 configuration type: dhcpv6
- configuration mode: basic
- prefix delegation size: 56

With this setting, the main router ask the ISP for a prefix delegation of a /56 subnet.

The router will create /64 addresses to assign to the following interface combining the delegated prefix with the the prefix ID specified in each interface.

#### DMZ interfaces

- ipv6 configuration type: track interface
- ipv6 interface: WAN
- ipv6 prefix ID: 0x6

#### EXTERNAL_CLIENTS interface

- ipv6 configuration type: track interface
- ipv6 interface: WAN
- ipv6 prefix ID: 0x4		

#### INTERNAL interface:

- ipv6 configuration type: track interface
- ipv6 interface: WAN
- ipv6 prefix ID: 0x54		

### a2) Internal Firewall-Router

Using the opnsense platform we configured all the interfaces (CLIENTS, EXTERNAL, SERVERS). 

For all of them we enabled the prevent interface removal, removed the block bogon and block for private, due to the private addresses we are using for our network.

The following rules are specific setting for each interface. 
In assigning the prefix IDs, we opted for a structure close to the ipv4 topology, i.e. for a 6.0/24 ipv4 network, in ipv6 we choose 6 as prefix ID in ipv6 for that subnetwork.

#### EXTERNAL interface

- ipv6 configuration type: dhcpv6
- configuration mode: basic
- prefix delegation size: 62

With this setting, the Internal Router ask the main Router for a prefix delegation of a /62 subnet.		

The router will create /64 addresses to assign to the following interface, combining the delegated prefix with the the prefix ID specified in each interface.

#### CLIENTS interface

- ipv6 configuration type: track interface
- ipv6 interface: EXTERNAL
- ipv6 prefix ID: 0x1

#### SERVERS interface

- ipv6 configuration type: track interface
- ipv6 interface: EXTERNAL
- ipv6 prefix ID: 0x2	

### b) SUBNETWORKS

This section will describe the main modification done for each machine in every subnetwork.

Generally, through the /ext/sysctl.d/99-sysctl.conf file we modified the policy for handling ipv6 addresses. In assigning the addresses, we opted for a structure similar to the ipv4 for the servers, namely. Through a script we managed to set the specific interface ID similar to the ipv4 structure, e.g. the web server set "::2" as interface ID, similar to the ".2" in ipv4. Our goals is to create an "homogeneus network", simplicity is the key.

#### DMZ

##### Web servers

We created a script in /etc/network/if-up.d/ip-tunnel with ownership bit 755. This script ensures that the servers creates an ipv6 address from the one advertised by the main router.

The interface ID is ::2.

##### Proxy server:

We thought that the proxy server doesn't need an ipv6 address, since zentyal does not support ipv6 and it is sufficient to have an ipv4 address to navigate on the internet. For security improvement, we will enforce the firewall rules.

#### EXTERNAL services

##### Client ext 1:

We inserted the code "net.ipv6.conf.all.addr_gen_mode = 3" in /etc/sysctl.conf on the web server.

This ensures that the machine will generate stable privacy addresses for every interfaces, using a random secret.

##### Fantastic coffe:

??? Still not working.

#### INTERNAL services

##### Domain Controller:

We created a script in /etc/network/if-up.d/ip-tunnel with ownership bit 755. 

This script ensures that the servers creates an ipv6 address from the one advertised by the main router.

The interface ID is "::2".

##### Log Server:

We created a script in /etc/network/if-up.d/ip-tunnel with ownership bit 755.

This script ensures that the servers creates an ipv6 address from the one advertised by the main router.

The interface ID is "::3".

#### CLIENTS

##### Arpwatch:

We inserted the code "net.ipv6.conf.all.disable_ipv6 = 1" in /etc/sysctl.conf on the arpwatch machine. This disable ipv6 addresses on the host, since we thought not to be needed for the machine to navigate on the internet due to its functions.


## 5.	DNS configuration

DC controller configured to use dnsmasq. (need to fix some ipv6 config) 
See /etc/dnsmasq.d/main.conf for further details. The dc looks at the 
/etc/hosts.acme29-dc.lan file to resolve ip for hostnames of the acme 
network, namely for the servers.


The main router send the nameserver ip (both ipv4 and ipv6 addresses of 
the dc) with dhcpv4 and dhcpv6 (through router advertisement) to the 
External Clients networks.

The internal router send the nameserver ip (both ipv4 and ipv6) with 
dhcpv4 and dhcpv6 (through router advertisement) respectively to the 
Internal Clients networks.



## 6.	Evaluation of the security policy

## 7.	Policy implementation in opnsense

## 8.	Test of configuration

## 9.	Final remarks
