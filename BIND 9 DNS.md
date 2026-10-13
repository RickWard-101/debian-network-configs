
# BIND9 SERVER

# WARNING!
 Before anything, you must free Port 53 for BIND9 if you are running more DNS services.
 If you are running, for example, dnsmasq, before touching BIND9, verify that dnsmasq 
 is only handling DHCP and not blocking port 53!
 To disable DNS open dnsmasq.conf: 

	sudo nano /etc/dnsmasq.conf

 Ensure the line port=0 is present at the bottom. Then restart dnsmasq:

	sudo systemctl restart dnsmasq

 Setting port=0 turns off dnsmasq's internal DNS server
 while keeping its DHCP server active.
 
 Also another thing, there must be two networking interfaces. For easier examples
 i will make the IP of LAN interface the following: 192.168.100.1/24. Change as you
 see fit.

# Install BIND9

	sudo apt update && sudo apt install -y bind9 bind9utils bind9-doc 

 Declare Zones in BIND9 so we can define which domains BIND9 owns.
 Open the local zone config file: (for LAN).

	sudo nano /etc/bind/named.conf.local

 Wipe any previous test edits and paste these clean zone declarations:
 
 Forward Zone: map domain.net names to IPs
```
zone "domain.net" {
    type master;
    file "/etc/bind/db.domain.net";
};
```

 Reverse Zone: map 192.168.100.x IPs to names
```
zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.100";
};
```
 This tells BIND9: "I am the Master authority for domain.net and 
 192.168.100.x. Here are the paths to the database files where my records are stored."

 Create the Forward Zone Database, name to ip. (RECORDS)
 Create a new forward database file:

	sudo nano /etc/bind/db.domain.net

 Paste the following complete zone configuration:

```
$TTL    604800
@       IN      SOA     router.domain.net. admin.domain.net. (
                              2         ; Serial (version number)
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; This is Name Server (NS) Record (to comment in deamons use ;)
@       IN      NS      router.domain.net.
; Address (A) Records: Hostname -> IPv4
router  IN      A       192.168.100.1
desktop IN      A       192.168.100.50
```

__Quick breakdown:__

1.    @ represents the current zone name (domain.net).

2.    SOA defines the master server (router.domain.net.) and admin email (admin.domain.net.).

3.    NS declares router.domain.net. as the authoritative server.

4.    A maps router to 192.168.100.1 and desktop to 192.168.100.50.
**(Also, for those who do not know, in order to comment on a daemon file you must ;)**
    
 Create the Reverse Zone Database, IP to name.
 Create a new Reverse database file:

	sudo nano /etc/bind/db.192.168.100

 Paste the following complete zone configuration:

```
$TTL    604800
@       IN      SOA     router.domain.net admin.domain.net. (
                              1         
                         604800         
                          86400         
                        2419200         
                         604800 )       
@       IN      NS      router.domain.net.
1       IN      PTR     router.domain.net.
50      IN      PTR     desktop.domain.net.
```
 Quick breakdown: 
 The number 1 stands for 192.168.100.1 and maps back to router.domain.net. 
 Note the trailing dot at the end of domain names to prevent BIND from appending .domain.net twice.

 Test the configuration files for typos:
 Check general configuration (prints nothing if syntax is clean)

	sudo named-checkconf

 Check forward zone file:

	sudo named-checkzone domain.net /etc/bind/db.domain.net

 Check reverse zone file:

	sudo named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100

 If both zone checks output OK, restart BIND9:
	
	sudo systemctl restart bind9

__instalation is finish but we need to forward it or make it be recursive__

 Before going into that we need our clients to be able to acess the internet.
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

 __You must have netfilter installed for IPtabales__

 Now that BIND9 is running, the next steps are testing the local zone queries and configuring 
 upstream forwarding so the network can resolve both your private domain.net domain and public internet sites (like google.com).
 but before that, test Local Resolution with dig.
 Run a forward and reverse lookup against your/this config router's BIND9 service:

 Test Forward DNS (Should return 192.168.100.1)
	
    dig @192.168.100.1 router.lab.lan +short

 Needs to return 192.168.100.1

 Test Reverse DNS (Should return router.lab.lan.)
	
    dig @192.168.100.1 -x 192.168.100.50 +short
 
  Needs to return desktop.domain.net

 Now configure Upstream Forwarders in BIND9. 
 
# FOR SECURITY REASONS DON'T USE FORWARDER | RECURSIVE BELLOW!
 
 __FORWARDER (RELAY MODE)__
 This will Enables public internet DNS resolution because right now, BIND9 only knows about      domain.net. 
 If a client asks for google.com, BIND9 will fail. We need to tell BIND9 where to forward external queries.

 Open named.conf.options:

    sudo nano /etc/bind/named.conf.options

 Find the forwarders block, uncomment it, and add external DNS servers (like Cloudflare or Google):
 (!on allow-query choose between any or trusted according to your security means!)

```
acl trusted {
    127.0.0.1;
    192.168.100.0/24;    # Change to match your local network subnet
};

options {
    directory "/var/cache/bind";

    // Security: Only allow your local network to query this server
    allow-query { trusted; };
    allow-recursion { trusted; };

    // Forwarder Setup
    forwarders {
        1.1.1.1;       
        8.8.8.8;       
    };
    forward only;      # Strictly use forwarders, do not try to contact root servers directly

    // Security Hardening
    dnssec-validation auto;
    listen-on-v6 { any; };
    listen-on { any; };
    
    // Hide version for security
    version "none";
};
```

 Save, check syntax, and restart BIND9:

    sudo named-checkconf
	sudo systemctl restart bind9

 Verify external resolution:

	dig @192.168.100.1 google.com +short

 It should return public IP addresses for Google!
 
 __RECURSIVE (FULL RESOLUTION)__
 
 The Full Recursive Configuration (Maximum Privacy/Independence).
 Use this if you do not trust external DNS providers and want your server to talk directly 
 to the  internet's root and authoritative servers.
 Open named.conf.options:

    sudo nano /etc/bind/named.conf.options
 
 inside write down: 
```
 acl trusted {
    127.0.0.1;
    192.168.100.0/24;   
};

options {
    directory "/var/cache/bind";
    
    allow-query { trusted; };
    recursion yes;
    
    allow-recursion { trusted; };
    forwarders {};
    
    dnssec-validation auto;
    listen-on-v6 { any; };
    
    listen-on { any; };
    version "none";
};
```
 Quick breakdown:
 (acl trusted & allow-query) are crucial for security. Without this, your BIND server becomes an "open resolver," which hackers can abuse to launch massive DDoS attacks against other websites.
 (directory) tells BIND where to store its runtime files and cache data.
 (dnssec-validation auto) Protects your network against DNS spoofing/poisoning attacks by verifying cryptographic signatures on domains.
 
 Save, check syntax, and restart BIND9:

    sudo named-checkconf
	sudo systemctl restart bind9

 Verify external resolution:

	dig @192.168.100.1 google.com +short

 It should return public IP addresses for Google!
