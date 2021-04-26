# ACME-29 Report

*Davide Quaranta, 1715742*
*Alessio Tullio, 1809077*
*Edoardo Gabrielli, 1693726*

## Initial brainstorming

After a brief introduction to the network topology and the tools used to handle the machines (Proxmox), we reasoned all together in depth on the ways to better configure the network. 

The best approach appeared to be step by step: i.e. starting from configuring the main router, then the neighbor subnetwork (External and DMZ), finally the internal firewall with the Internal and Clients network.

For the procedures we choose to follow the one described in the assignment, since the proposed tool appeared to work properly to successfully complete all the requests.

## Setup of the infrastructure for IPv6 addressing

### Main router

#### -> WAN

- Enabled **prevent interface removal***.
- Removed ***block bogon*** and lock.
- Private IPv6 configuration type: **DHCPv6**.
- Prefix size: **56**.

#### -> DMZ, INTERNAL, EXTERNAL

- Track interface.
- Set the IP Prefix ID.

## DNS configuration

## Evaluation of the security policy

## Policy implementation in OPNsense

## Configuration test

## Final remarks