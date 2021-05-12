# 1. ACME-29 REPORT

## 2. Student

- Edoardo Gabrielli, 1693726
- Davide Quaranta, 1715742
- Alessio Tullio, 1809077

## 3. Initial brainstorming

After a brief introduction to the network topology and the tools offered by the virtual environment (Proxmox), we reasoned on the best ways to configure the network, according to the requirements.

We opted for a step by step approach: starting from configuring the Main Firewall, then the neighbor subnetworks (External and DMZ), and finally the Internal Firewall with its networks.

### What to do

We followed this plan:

1. Preliminary security issues (credentials change, updates)
2. IPv6 addressing.
3. DNS configuration.
4. Firewall rules.

### How to do it

We decided to configure the IPV6 addressing in this way:

- The Main Router should Obtain a /56 prefix from the ISP via DHCPv6-PD.
- The Main Router should redistribute the prefix on its interfaces.
- The Internal Router should ask a prefix delegation from the Main, and redistribute it as well.
- Each host should receive the prefix and assign itself the Interface ID (SLAAC approach).

Regarding the DNS:

- DNS (and extra info) should be distributed either via DHCPv4 and Router Advertisements, from both the Main and the Internal Routers.
- The Domain Controller should act as a DNS.

## 4. Setup of the infrastructure for IPv6 addressing

In assigning ipv6 addressing we faced many troubles. In the end we understood that, when a router has some mismatch in the configuration of a service, it will not start that service (in our case DHCPv6 server).

### a1) Main Firewall-Router

We configured all the interfaces (DMZ, WAN, EXTERNAL_CLIENTS, INTERNAL) using the OPNsense's web interface. For all of them we enabled the options to prevent the interface removal, and removed the block for bogon and private addresses, (due to the private addresses we are using for our network -- ?).

Regarding the prefix IDs, we opted for a structure close to the IPv4 topology, e.g. for a x.x.6.0/24 IPv4 network, we choose "6" as Prefix ID in IPv6 for that subnetwork.

#### WAN interface

- IPv6 configuration type: DHCPv6
- Configuration mode: basic
- Prefix delegation size: 56

With this setting, the main router asks the ISP for a prefix delegation of a /56 subnet.

The router will create /64 addresses to assign to the following interfaces, combining the delegated prefix with the the Prefix ID specified in each interface.

| Interface        | Configuration type | Tracked interface | Prefix ID |
| ---------------- | ------------------ | ----------------- | --------- |
| DMZ              | Track              | WAN               | 0x6       |
| EXTERNAL_CLIENTS | Track              | WAN               | 0x4       |
| INTERNAL         | Track              | WAN               | 0x54      |

### a2) Internal Firewall-Router

The IPv6 setup is similar to the Main Router. In particular:

#### EXTERNAL interface

- IPv6 configuration type: DHCPv6
- Configuration mode: basic
- Prefix delegation size: 62

With this setting, the Internal Router asks the Main Router for a Prefix Delegation of a /62 subnet.

The router will create /64 addresses to assign to the following interface, combining the delegated prefix with the prefix ID specified in each interface.

| Interface | Configuration type | Tracked interface | Prefix ID |
| --------- | ------------------ | ----------------- | --------- |
| CLIENTS   | Track              | EXTERNAL          | 0x1       |
| SERVERS   | Track              | EXTERNAL          | 0x2       |

### b) SUBNETWORKS

This section will describe the main modifications done to the machines in each subnetwork.

Our goal for the static IPv6 addressing was to mirror the IPv4 structure, so if the web server has `100.100.6.2`, the Interface ID should be `::2`.

To set the IPv6 addresses for machines that need a static address (e.g. servers), we decided to use the IP Tokenization approach, which consists in instructing the machine to set a fixed Interface ID to the prefix obtained via Router Advertisements.

For example, to implement it in the webserver, we added a file `/etc/network/if-up/ip-token` with contents:

```bash
#!/bin/bash
ip token set ::2 dev eth0
```

We also modified the `sysctl` configurations (`/etc/sysctl.d/`) to make sure that the machines were appropriately configured for IPv6.

#### DMZ

##### Proxy server:

//We thought that the proxy server doesn't need an ipv6 address, since zentyal does ///not support ipv6 and it is sufficient to have an ipv4 address to navigate on the ////internet. For security improvement, we will enforce the firewall rules.

[[[ DA RICONTROLLARE ]]]

#### EXTERNAL services

##### Client ext 1:

Since this machine is not a server, we preferred to generate a stable privacy address, by setting `net.ipv6.conf.all.addr_gen_mode = 3`" in the `syctl` configurations.

#### CLIENTS

##### Arpwatch:

We inserted the code `net.ipv6.conf.all.disable_ipv6 = 1` in /etc/sysctl.conf on the arpwatch machine. This disable ipv6 addresses on the host, since we thought not to be needed for the machine to navigate on the internet due to its functions.

[[[ VERIFICARE SE È ANCORA VERO ]]]

---

### IPv6 addresses recap

| Network          | Machine           | Interface ID |
| ---------------- | ----------------- | ------------ |
| DMZ              | Web Server        |      ::2     |
| DMZ              | Proxy Server      |      ::3     |
| INTERNAL SERVERS | Domain Controller |      ::2     |
| INTERNAL SERVERS | Log Server        |      ::3     |
| EXTERNAL         | fanstaticcoffee   |      ::10    |
| EXTERNAL         | Client ext 1      |  privacy_ip  |
| CLIENTS          | Kali              |  privacy_ip  |

## 5.	DNS configuration

To fully support IPv6 in the network we decided to configure the DNS directly via `dnsmasq`, instead of relying on the Zentyal interface, since it doesn't support IPv6.

Dnsmasq has been configured as follows (`/etc/dnsmasq.d/main.conf`):

```bash
# interface to listen on
interface=eth0

# addresses to listen on
listen-address=::1,2001:470:b5b8:1df1::2,127.0.0.1,100.100.1.2

# nameservers
server=8.8.8.8
server=8.8.4.4
server=2001:4860:4860::8888
server=2001:4860:4860::8844

# domain configuration, to automatically expand hostnames to fully-qualified domain names (host -> host.acme29-dc.lan)
expand-hosts
domain=acme29-dc.lan
addn-hosts=/etc/hosts.acme29-dc.lan

# do not resolve addresses with /etc/resolv.conf
no-resolv
```
NOTE: 
- the `/etc/hosts.acme29-dc.lan` file containts a list of both ipv4 and ipv6 for each host within the Acme network.
- The routers send DNS info via DHCPv4 and Router Advertisements.

## 6.	Evaluation of the security policy

We refined and syntesized the security policy to implement, in order to minimize the rules and generalize them. The policies needed to be evaluated as a whole to fully understand the big picture; for example:

- "All the services provided by the hosts in the Internal server network have to be accessible only by the Client network and the DMZ hosts."
- "All the hosts (but the Client network hosts) have to use the syslog service on the Log server (syslog)"

These two rules ultimately mean that only the DMZ hosts can access the syslog.

[[ INSERIRE RAFFINAMENTO DELLA POLICY COME LO ABBIAMO INTERPRETATO NOI !!]]

## 7.	Policy implementation in OPNsense

[[ METTERE LE REGOLE, E SPIEGARE ALCUNE COSE IMPARATE ]]

The complete list of firewall rules are reported in `[ INSERT HERE THE FILE NAMEs]`.

During the implementation of the rules, we learned how scalable is the Opnsense firewall, thanks to its stafullness and to the aliases:
- a statefull firewall means: if a router has, for example, 3 interfaces A, B and C, and we implement a rule to accept some traffic X on the A interface, the firewall will autogenerate the rules to accept the responses that comes from B or/and C for that type of traffic X.
For this reason, for each interface, we only had to setup the rules for the input direction;
- Aliases means that you can specify a name and associate to it multiple hosts ip or networks prefix. So we created aliases for ALL the servers, all subnetworks and one for the full ACME29 network, including both ipv4 and ipv6 addresses.

We set "DENY" as DEFAULT policy for each interface in both the routers, this ensures that every traffic, not specifically allowed, will be dropped.

[[[ ### EXAMPLE OF ONE RULE HERE ]]]

## 8.	Test of the configuration

All the following tests have been done using both ipv4 and ipv6.

To test whether the configuration was correctly done, we proceeded in this way:

- Ensure that the prefix delegation works, by checking if the OPNsense interface shows a prefix for each interface.

- Ensure that the IP Tokenization mechanism works by:
  - Shutting the host's interfaces down and up.
  - Restarting the `networking` service.
  - Restarting the host.
- Ensure that the DNS works by checking if hosts can perform DNS queries with the Domain Controller, e.g. with `dig github.com @100.100.1.2` or `host github.com 100.100.1.2`.
- Ensure that Router Advertisements are correctly set up (e.g. right options, DNS) by:
  - Inspecting the packets with Wireshark.
  - Checking the content of `/etc/resolv.conf` in the hosts.

To test the firewall policies we defined and used the following framework:

### DNS (#1)

Test from each host in each network if:

- The internal DNS is correctly set up as resolver.
- The host can query the internal DNS.
- The host can't query with a different DNS than the internal.
- The host can't query with the upstream DNS that the internal uses.

### Web server reachability (#2)

Test from the WAN if:

- The web server can be accessed on port 80 from the WAN.
- No other host can be accessed from the WAN.

### Proxy reachability (#3, #4)

Test from each ACME-29 host if:

- The proxy can be accessed.

Test from the proxy if:

- It can initiate connection to outside.

### Internal servers reachability (#5)

Test if:

- Hosts in the DMZ can access the internal services.
- Hosts in the Clients network can access the internal services.
- Hosts in the External services network can't access the internal services.

### Syslog (#6)

Test if:

- Hosts in the DMZ can access the syslog.
- No other host can use the syslog.

### SSH (#7)

Test if:

- Hosts in the Clients network can use SSH on every ACME-29 hosts that has an ssh daemon running.
- No other host can use SSH on hosts in different network.

### Clients web browsing via proxy (#7)

Test if:

- Hosts in the Clients network can access the proxy on ports 80,443.

## 9.	Final remarks

- [[ parla male di zentyal ]]
- [[ parla di opnsense ]]