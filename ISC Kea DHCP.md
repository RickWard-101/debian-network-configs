# ISC Kea DHCPv4 Server

This guide covers the installation, configuration, and validation of the **ISC Kea DHCPv4** server on Debian, designed to automate IP address provisioning for the internal LAN (`192.168.100.0/24`) within the `domain.net` infrastructure. The LAN interface will be "enp7s0" for this example.
!(Check BIND9 autoritative server for domain and networking information if needed)!

__Introduction & Architecture__
Unlike legacy DHCP servers (such as `isc-dhcp-server`), **Kea** is a modern, modular DHCP engine that relies exclusively on **JSON** configuration files. 
In this environment i've created, Kea acts as the core automated allocation service, dynamically assigning IP addresses to client nodes while injecting vital network parameters (default gateway and local DNS server).

__Installation__
To install the Kea DHCPv4 package on Debian:

        sudo apt update && sudo apt install kea-dhcp4-server -y
    
Configuration file is located at /etc/kea/kea-dhcp4.conf.

The core configuration file defines the listening interface, lease backend, lease times, and network subnets. 
Open kea-dhcp4-server:

        sudo vim /etc/kea/kea-dhcp4.conf

Replace the default configuration with the following clean and commented (//) JSON block:

```
{
  "Dhcp4": {
    
    // Bind the DHCP daemon strictly to the internal LAN interface
    "interfaces-config": {
      "interfaces": [ "enp7s0" ] 
    },

    // Lease backend database (stores active leases locally in a CSV file)
    "lease-database": {
      "type": "memfile",
      "persist": true,
      "name": "/var/lib/kea/kea-leases4.csv"
    },

    // Lease timers (in seconds)
    "valid-lifetime": 4000,
    "renew-timer": 1000,
    "rebind-timer": 2000,

    // Subnet and pool definitions
    "subnet4": [
      {
        "id": 1,
        "subnet": "192.168.100.0/24",
        
        // Dynamic IP assignment pool (leaves lower IPs free for static servers)
        "pools": [
          { "pool": "192.168.100.10 - 192.168.100.100" }
        ],

        // Network options injected into client DHCP offers
        "option-data": [
          {
            "name": "routers",
            "data": "192.168.100.1"
          },
          {
            "name": "domain-name-servers",
            "data": "192.168.100.1"
          },
          {
            "name": "domain-name",
            "data": "domain.net"
          }
        ]
      }
    ]
  }
}
```

__Testing & Validation__

Before anything verify JSON Syntax. Always test the configuration file before restarting the service to catch syntax or bracket errors:

        sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf

Manage the service, with systemctl:

        sudo systemctl restart kea-dhcp4-server
And:
        sudo systemctl enable kea-dhcp4-server
    
__Check Active Leases__
Inspect the lease database to confirm clients are successfully receiving their configurations:

        cat /var/lib/kea/kea-leases4.csv
    
__Gateway Router, IP Forwarding & NAT Configuration (Debian)__

This is responsible for **bridging** the external WAN interface and the internal LAN (`192.168.100.0/24`), enabling traffic routing and Network Address Translation (NAT/Masquerading).

Architecture Overview:
In this domain, the Debian machine acts as a router. 
* **WAN Interface (e.g., `enp1s0`):** Connected to the external network/internet via DHCP.
* **LAN Interface (e.g., `enp7s0`):** Connected to the internal switch/clients with a static IP (`192.168.100.1`).

__Enable IP Forwarding__

*(This same configuration can be seen on BIND)*
By default, Linux kernels drop packets that are not destined for the local machine. To allow the Debian system to route packets between interfaces:
We need our clients to be able to acess the internet.
For that activate forward of IP on kernel:
   
        sudo sysctl -w net.ipv4.ip_forward=1
 
then open sysctl.conf
 
        sudo nano /etc/sysctl.conf

Inside the conf file write:

        net.ipv4.ip_forward=1

close and save it.

Activate NAT to the interface with exit to WAN (internet this case):

        sudo iptables -t nat -A POSTROUTING -o (WAN interface) -j MASQUERADE

To save rules after reboot on Debian must do:

        sudo apt install iptables-persistent
    
        sudo netfilter-persistent save

And client should be able to reach all of the internet.


# Troubleshooting & Common Issues ("What if...?")
❌ Issue 1: Client machines can ping the gateway (192.168.100.1) and resolve domain names, but fail to reach external IPs (100% packet loss on ping 8.8.8.8).

**Cause**: IP Forwarding is disabled in the kernel or the NAT rule is missing/associated with the wrong interface.

How to fix? Check if forwarding is active: 
        cat /proc/sys/net/ipv4/ip_forward (must return 1).

Verify active NAT rules: 
        sudo iptables -t nat -L -n -v. 
    
!Ensure MASQUERADE points to your active WAN interface.!

❌ Issue 2: DNS queries (dig) from clients do not show the Authoritative (aa) flag or return local IPs through the local system resolver.

**Cause**: Linux desktop clients (like Ubuntu/Mint) use systemd-resolved running on 127.0.0.53 as a local caching resolver, which strips original server flags.

How to fix? To test the BIND9 server directly and verify authoritative responses (aa), query the DNS server's IP directly instead of relying on the local resolver:

        dig @100.100.100.1 domain.net
        
And that's it. Use journalctl to check info on kea DHCP in real time. Quick adventure, make a bash file to transfer all of journal data into a txt file.

