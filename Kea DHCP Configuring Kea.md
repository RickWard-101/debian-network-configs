# ISC Kea DHCPv4 Server Configuration

This guide covers the installation, configuration, and validation of the **ISC Kea DHCPv4** server on Debian, designed to automate IP address provisioning for the internal LAN (`192.168.100.0/24`) within the `domain.net` infrastructure.
!(Check BIND9 autoritative server for domain and networking information if needed)!

__Introduction & Architecture__
Unlike legacy DHCP servers (such as `isc-dhcp-server`), **Kea** is a modern, modular DHCP engine that relies exclusively on **JSON** configuration files. 
In this environment i've created, Kea acts as the core automated allocation service, dynamically assigning IP addresses to client nodes while injecting vital network parameters (default gateway and local DNS server).

__Installation__
To install the Kea DHCPv4 package on Debian:

    sudo apt update && sudo apt install kea-dhcp4-server -y
    
Configuration file is located at /etc/kea/kea-dhcp4.conf
The core configuration file defines the listening interface, lease backend, lease times, and network subnets. 
In this example i will make enp7s0 the LAN network interface, but make sure to change yours!
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
    sudo systemctl enable kea-dhcp4-server
    
__Check Active Leases__
Inspect the lease database to confirm clients are successfully receiving their configurations:

    cat /var/lib/kea/kea-leases4.csv
    
