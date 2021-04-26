# 1. ACME-29 REPORT
## 2. Student (Name Surname numbers)

    Edoardo Gabrielli 1693726
	Davide Quaranta 1715742
	Alessio Tullio 1809077
	

## 3. Initial brainstorming (where you write your considerations about what to do and how)

	After a brief introduction to the network topology and the tools used to handle 
	the machines (Proxmox), we reasoned all togheter in depth on the ways to better
	configure the network. 
	The best solution appeared to be the step by step approach: i.e. starting from 
	configuring the main router, then the neighbour subnetwork (External and DMZ), 
	finally the internal firewall with the Internal and Clients network.
	For the procedures we choose to follow the one described in the assignment, 
	since the proposed tool appeared to work properly to succesfully complete all
	the requests.

## 4.	Setup of the infrastructure for IPv6 addressing (here you should detail the 
##	steps you’ve done to properly setup all the hosts. You should also include 
##	details and explanation about the network structure you opted for and any 
##	difficulties you’ve faced and –hopefully– solved)


###	a1) Main Firewall-Router
	
		Using the opnsense platform we configured all the interfaces (DMZ, WAN,
        EXTERNAL_CLIENTS, INTERNAL). For all of them we enabled the prevent interface removal, removed the block bogon and block for private, due to the private addresses we are using for our network.

	The following rules are specific setting for each interface.
	In assigning the prefix IDs, we opted for a structure close to the ipv4 topology, i.e. for a 6.0/24 ipv4 network, in ipv6 we choose 6 as prefix ID in ipv6 for that subnetwork.	 

		 
####	WAN interface
		ipv6 configuration type:	dhcpv6
		configuration mode:		basic
		prefix delegation size:		56

		with this setting, the main router ask the ISP for a prefix delegation 
        of a /56 subnet.		
	
	The router will create /64 addresses to assign to the following interface
	combining the delegated prefix with the the prefix ID specified in each
	interface.
	
####	DMZ interfaces:
		ipv6 configuration type:	track interface
		ipv6 interface:			WAN
		ipv6 prefix ID:			0x6		
	
####	EXTERNAL_CLIENTS interface:
		ipv6 configuration type:	track interface
		ipv6 interface:			WAN
		ipv6 prefix ID:			0x4		

####	INTERNAL interface:
		ipv6 configuration type:	track interface
		ipv6 interface:			WAN
		ipv6 prefix ID:			0x54		
	
### a2) Internal Firewall-Router
		Using the opnsense platform we configured all the interfaces (CLIENTS, EXTERNAL, SERVERS). For all of them we enabled the prevent interface removal, removed the
		block bogon and block for private, due to the private addresses we are using for our network.

	The following rules are specific setting for each interface. 
    In assigning the prefix IDs, we opted for a structure close to the ipv4 topology, i.e. for a 6.0/24 ipv4 network, in ipv6 we choose 6 as prefix ID in ipv6 for that subnetwork.
		 
####	EXTERNAL interface
		ipv6 configuration type:	dhcpv6
		configuration mode:		basic
		prefix delegation size:		62

		with this setting, the Internal Router ask the main Router for a prefix 
		delegation of a /62 subnet.		
	
	The router will create /64 addresses to assign to the following interface
	combining the delegated prefix with the the prefix ID specified in each
	interface.

####	CLIENTS interface:
		ipv6 configuration type:	track interface
		ipv6 interface:			EXTERNAL
		ipv6 prefix ID:			0x1		

####	SERVERS interface:
		ipv6 configuration type:	track interface
		ipv6 interface:			EXTERNAL
		ipv6 prefix ID:			0x2		

### b) SUBNETWORKS
    This section will describe the main modification done for each machine in every
    subnetwork.
    Generally, through the /ext/sysctl.conf file we modified the policy for handling
    ipv6 addresses.

 #### DMZ
    - Web server:
        We inserted the code "net.ipv6.conf.all.addr_gen_mode = 0" in /etc/sysctl.conf  on the web server. This ensures that the interface ID of the ipv6 addresses is generated from the MAC.
    - Proxy server:
        We thought that the proxy server doesn't need an ipv6 address, since zentyal 
        does not support ipv6 and it is sufficient to have an ipv4 address to navigate on the internet. For security imporvment, we will enforce the firewall rules.

 #### EXTERNAL services
    - Client ext 1:
        We inserted the code "net.ipv6.conf.all.addr_gen_mode = 3" in /etc/sysctl.conf  on the web server. This ensures that the machine will generate stable privacy addresses for every interfaces, using a random secret.
    - Fantastic coffe:
        ???


 #### INTERNAL serverers
    - Domain Controller:
        We inserted the code "net.ipv6.conf.all.addr_gen_mode = 3" in /etc/sysctl.conf  on the web server. This ensures that the machine will generate stable privacy addresses for every interfaces, using a random secret. ???
    - Log Server:
        We inserted the code "net.ipv6.conf.all.addr_gen_mode = 3" in /etc/sysctl.conf  on the web server. This ensures that the machine will generate stable privacy addresses for every interfaces, using a random secret. ???

 #### CLIENTS
    - Arpwatch:
        We inserted the code "net.ipv6.conf.all.addr_gen_mode = 3" in /etc/sysctl.conf  on the web server. This ensures that the machine will generate stable privacy addresses for every interfaces, using a random secret.
    





## 5.	DNS configuration

## 6.	Evaluation of the security policy

## 7.	Policy implementation in opnsense

## 8.	Test of configuration

## 9.	Final remarks
