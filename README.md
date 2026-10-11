# DEBIAN ROUTER TOPOLOGY
Welcome to the Debian router project. In this project, i've made a bunch of 
configuration file teaching how to make a network using Debian as the main 
router/modem/server. 
I advice to follow this project from start to finish to learn more about networking.
This project uses VM (virtual machines), but everything we will do on VM's should 
be applied to real physical hardware.

__What can i learn form this?__

This project start's with how to setup the Virtual Machines.

1. VM creations and what we need.
this will include the setup of networking interfaces we will use.

2. Setup of Debian netinstall server.
In here will include all the necessary tools to start making any server. 

3. install & configure an authoritative DNS server.
what is a DNS server and how to use it.

4. Install & configure a DHCP server.
What is a DHCP server and how to use it.

5. Install & configre web server.

__DRAWING OF TOPOLOGY__

[			             Public Internet                              ]
                                 |
                                 | (WAN Access)
                                 |
                                 | (WAN Interface)
                                 | IP: Assigned by ISP
                                 v
                     +-----------------------+
                     |          Host         |
                     |      (QEMU / KVM)     | step 1.
                     +-----------------------+
                                 |
                                 | LAN Interface 
                                 | IP: Assigned by host 
                                 v
  +-------------------------------------------------------------------+
  |                        DEBIAN ROUTER VM                           |
  |                                                                   |
  |  [ Services Running ]                                             |
  |  1. BIND9 (DNS Server - Port 53)                                  |
  |                                                                   |
  |  2. Linux Kernel (IP Forwarding & IPTables NAT)                   |
  |    - Translates LAN IP traffic so Desktop can hit the Internet    |
  |                                                                   |
  |  LAN Interface                              |
  |  Static IP: 192.168.100.1 / 255.255.255.0                         |
  +-------------------------------------------------------------------+
                                 |
                                 | Internal Virtual Network Switch
                                 | (Subnet: 192.168.100.0/24)
                                 v
  +-------------------------------------------------------------------+
  |                        DEBIAN DESKTOP VM                          |
  |                                                                   |
  |  LAN Interface (eth0)                                             |
  |  IP: 192.168.100.50 (Assigned dynamically by dnsmasq)             |
  |  Default Gateway: 192.168.100.1                                   |
  |  Primary DNS:     192.168.100.1 (Points to BIND9 on Router)       |
  +-------------------------------------------------------------------+
