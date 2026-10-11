
#BIND9 SERVER

# WARNING!
 You must free Port 53 for BIND9 if you are running more DNS services.
 If you are running, for example, dnsmasq, before touching BIND9, verify that dnsmasq 
 is only handling DHCP and not blocking port 53!
 To disable DNS open /etc/dnsmasq.conf: 

	sudo nano /etc/dnsmasq.conf

 Ensure the line port=0 is present at the bottom. Then restart dnsmasq:

	sudo systemctl restart dnsmasq

 Setting port=0 turns off dnsmasq's internal DNS server
 while keeping its DHCP server active.

#Install BIND9__

	sudo apt update && sudo apt install -y bind9 bind9utils bind9-doc 

 Let's Declare Zones in BIND9 so we can define which domains BIND9 owns.
 Open the local zone config file: (for LAN).

	sudo nano /etc/bind/named.conf.local

 Wipe any previous test edits and paste these clean zone declarations:
 Forward Zone: Convert lab.lan names to IPs

```
zone "lisbon.net" {
    type master;
    file "/etc/bind/db.lab.lan";
};
```

 Reverse Zone: Convert 192.168.100.x IPs to names
```
zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.100";
};
```
# This tells BIND9: "I am the Master authority for lisbon.net and 
# 192.168.100.x. Here are the paths to the database files where my records are stored."

# Create the Forward Zone Database, name to ip. (RECORDS)
# Create a new forward database file:

	sudo nano /etc/bind/db.lisbon.net

# Paste the following complete zone configuration:

```
$TTL    604800
@       IN      SOA     router.lisbon.net. admin.lisbon.net. (
                              2         # Serial (version number)
                         604800         # Refresh
                          86400         # Retry
                        2419200         # Expire
                         604800 )       # Negative Cache TTL
# This is Name Server (NS) Record (to comment in deamons use ;)
@       IN      NS      router.lisbon.net.
# Address (A) Records: Hostname -> IPv4
router  IN      A       192.168.100.1
desktop IN      A       192.168.100.50

```
#| Quick breakdown:
#|
#|    @ represents the current zone name (lisbon.net).
#|
#|    SOA defines the master server (router.lisbon.net.) and admin email (admin.lisbon.net.).
#|
#|    NS declares router.lisbon.net. as the authoritative server.
#|
#|    A maps router to 192.168.100.1 and desktop to 192.168.100.50.

# Create the Reverse Zone Database, IP to name.
# Create a new Reverse database file:

	sudo nano /etc/bind/db.192.168.100

# Paste the following complete zone configuration:

```
$TTL    604800
@       IN      SOA     router.lisbon.net admin.lisbon.net. (
                              1         
                         604800         
                          86400         
                        2419200         
                         604800 )       
@       IN      NS      router.lisbon.net.
1       IN      PTR     router.lisbon.net.
50      IN      PTR     desktop.lisbon.net.

```
#| Quick breakdown: The number 1 stands for 192.168.100.1 and maps back to router.lab.lan.. 
#| Note the trailing dot at the end of domain names to prevent BIND from appending .lab.lan twice.

# Test your configuration files for typos:
# Check general configuration (prints nothing if syntax is clean)

	sudo named-checkconf

# Check forward zone file

	sudo named-checkzone lab.lan /etc/bind/db.lab.lan

# Check reverse zone file

	sudo named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100

#If both zone checks output OK, restart BIND9:
	
	sudo systemctl restart bind9

__ instalation is finish but we need to forward it or make it be recursive __


# first activate forward of IP on kernel:
    sudo sysctl -w net.ipv4.ip_forward=1

# Second activate NAT to the interface with exit to WAN (internet this case):
	sudo iptables -t nat -A POSTROUTING -o (insert your net interface) -j MASQUERADE

# To save rules after reboot on Debian must do:

	sudo nano /etc/sysctl.conf

net.ipv4.ip_forward=1

	sudo apt install iptables-persistent

    sudo netfilter-persistent save

# Now that BIND9 is running, the next crucial steps are testing your local zone queries and configuring 
# upstream forwarding so your network can resolve both your private lab.lan domain and public internet sites (like google.com).
# Test Local Resolution with dig.
# Router VM - Verifies BIND9 answers local DNS queries. 
# Run a forward and reverse lookup against your router's BIND9 service:

# Test Forward DNS (Should return 192.168.100.1)
	
	dig @192.168.100.1 router.lab.lan +short

# Test Reverse DNS (Should return router.lab.lan.)
	
	dig @192.168.100.1 -x 192.168.100.50 +short

# What success looks like: The first command returns 192.168.100.1 and the second returns desktop.lab.lan..

# Configure Upstream Forwarders in BIND9:
# Router VM - Enables public internet DNS resolution.
# Right now, BIND9 only knows about lab.lan. If a client asks for google.com, BIND9 will fail. We need to tell BIND9
# where to forward external queries.
# Open /etc/bind/named.conf.options:

	sudo nano /etc/bind/named.conf.options

# Find the forwarders block, uncomment it, and add external DNS servers (like Cloudflare or Google):

options {
        directory "/var/cache/bind";

        forwarders {
                1.1.1.1;
                8.8.8.8;
        };

        allow-query { any; };
        recursion yes;

        dnssec-validation auto;
        listen-on-v6 { any; };
};

# Save, check syntax, and restart BIND9:

	sudo named-checkconf
	sudo systemctl restart bind9

# Verify external resolution:

	dig @192.168.100.1 google.com +short

# It should return public IP addresses for Google!
