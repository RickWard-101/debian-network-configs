
# BIND9 SERVER

__Install Named BIND9__
```
sudo apt update && sudo apt install -y bind9 bind9utils bind9-doc 
```
 
Before going into the configuration you must have a basic understanding of the configuration Files.
Directory for this files are at `/etc/bind/`

**named.conf**: The main configuration file that includes other configuration files like 
`named.conf.local` and `named.conf.options`. It also specifies where to find the root hints 
(`named.cinf.root-hints`). 
  
**named.conf.local**: This is where you define your DNS zones. You will create forward and 
reverse DNS zones here.
   
**named.conf.options**: Contains global settings for the BIND server, such as listening 
interfaces and logging. Tells bind9 where to find files.
   
**named.cinf.root-hints**: Lists the root DNS servers necessary for resolving domains 
outside your local network.
   
**rndc.key**: Used for remote control of the BIND server, allowing you to manage it over 
the network.
 
 
Bind9 can work as 3 types of DNS servers.
1. Caching nameserver - Recursion or Forwarding.
2. Primary master. 
3. Secondary master. 

I like to start from `named.conf.options`.
First identify which interface BIND9 should listen on. WAN IP example: `192.168.122.122`, and LAN IP example `10.0.0.1`.
Since we want it to listen on the internal LAN interface with IP `10.0.0.1`, add the following line:

```
options {
    directory "/var/cache/bind";
    listen-on { 10.0.0.1; };
};

# I really advice comment all of the logging because it will give a lot of errors later on named.
# I will leave for advance users.
logging {
    channel default_log {
        file "/var/log/named/named.log";
        severity info;
        print-time yes;
    };
    category default {
        default_log;
    };
    category queries {
        default_log;
    };
};
```

Now, declare zones in BIND9 so we can define which domains BIND9 owns.
Open the local zone config file located at `/etc/bind/named.conf.local`

Wipe any previous test edits and paste these clean zone declarations:
 
Forward Zone: map example.com names to IPs
```
zone "example.com" IN {
         type master;
         file "example.com.db";
         allow-transfer { 192.168.122.122; };
     };
```

Reverse Zone: map 10.0.0.1.x IPs to names
```
zone "0.0.10.IN-ADDR.ARPA" IN {
         type master;
         file "10.0.0.db";
         allow-transfer { 192.168.122.122; };
     };
```
This tells BIND9: "I am the Master authority for example.com and 
10.0.0.x. Here are the paths to the database files where my records are stored."

Create the Forward Zone Database, name to ip. (RECORDS)
Create a new forward database file:
```
sudo vim /etc/bind/example.com.db
```
Paste the following complete zone configuration:

```
$TTL    604800
@       IN      SOA     ns1.example.com. admin.example.com. (
                         1          ; Serial number
                         86400      ; Refresh
                         7200       ; Retry
                         604800     ; Expire
                         86400 )    ; Minimum
@       IN      NS      ns1.example.com.
@       IN      NS      ns2.example.com.
;
ns1     IN      A       10.0.0.1
@       IN      A       10.0.0.1
www     IN      A       10.0.0.2
```

Create the Reverse Zone Database, IP to name.
Create a new Reverse database file:
```
sudo vim /etc/bind/10.0.0.db
```
Paste the following complete zone configuration:

```
$TTL 86400
@       IN      SOA     ns1.example.com. admin.example.com. (
                         1          ; Serial number
                         86400      ; Refresh
                         7200       ; Retry
                         604800     ; Expire
                         86400 )    ; Minimum

@       IN      NS      ns1.example.com.
@       IN      NS      ns2.example.com.
;
1       IN      PTR     example.com.
2       IN      PTR     www.example.com.
```

Ensure that the zone files are readable by the BIND user (usually `bind`):

```
chown root:bind /etc/bind/example.com.db /etc/bind/10.0.0.db
chmod 640 /etc/bind/example.com.db /etc/bind/10.0.0.db
```

Test the configuration files for typos:
Check general configuration (prints nothing if syntax is clean)
```
sudo named-checkconf
sudo named-checkzone 0.0.10.in-addr.arpa /etc/bind/10.0.0.db
sudo named-checkzone example.com /etc/bind/example.com.db
```
If both zone checks output OK, restart BIND9:
```
sudo systemctl restart bind9
```
__instalation is finish but we need to forward it or make it be recursive__

Before going into that, we need our clients to be able to acess the internet.
For that activate forward of IP on kernel:
```   
sudo sysctl -w net.ipv4.ip_forward=1
```
then open sysctl.conf
```
sudo vim /etc/sysctl.conf
```
Inside the conf file write:
```
net.ipv4.ip_forward=1
```
close and save it.

Activate NAT to the interface with exit to WAN (internet this case):
```
sudo iptables -t nat -A POSTROUTING -o (WAN interface) -j MASQUERADE
```
To save rules after reboot on Debian must do:
```
sudo apt install iptables-persistent
```
After all do:
```
sudo netfilter-persistent save
```

Now that BIND9 is running, the next steps are testing the local zone queries and configuring 
upstream forwarding so the network can resolve both your private example.com domain and public internet sites (like google.com).
but before that, test Local Resolution with dig.
Run a forward and reverse lookup against your/this config router's BIND9 service:

Test Forward DNS 
```	
dig @10.0.0.1 router.example.com 
```
Test Reverse DNS
```	
dig @10.0.0.1 -x 10.0.0.2 +short
```

Now configure Upstream Forwarders in BIND9. 
 
# FOR SECURITY REASONS DON'T USE FORWARDER | RECURSIVE BELLOW!
 
 __FORWARDER (RELAY MODE)__
This will Enables public internet DNS resolution because right now, BIND9 only knows about example.com.
If a client asks for google.com, BIND9 will fail. Something needs to tell BIND9 where to forward external queries.
In other words skips talking to the root servers entirely. Instead, it hands the request off to an upstream DNS server (the forwarder) and waits for that server to do the recursive work and return the answer.

back to `named.conf.options`, open it and find the forwarders block, uncomment it, and add external DNS servers (like Cloudflare or Google):
(!on allow-query choose between any or trusted according to your security means!)

```
acl trusted {
    127.0.0.1;
    10.0.0.0/24;    # Change to match your local network subnet
};

options {
    directory "/var/cache/bind";

    // Security: Only allow your local network to query this server
    allow-query { trusted; };
    allow-recursion { trusted; };

    // Forwarder Setup | YOU CAN IGNORE EVERYTHING ELSE AND JUST USE THIS FUNCTION IF YOU WISH |
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
```
sudo named-checkconf
sudo systemctl restart bind9
```
Verify external resolution:

dig @10.0.0.1 google.com +short

It should return public IP addresses for Google!
 
 __RECURSIVE (FULL RESOLUTION)__
 
The Full Recursive Configuration (Maximum Privacy/Independence).
Use this if you do not trust external DNS providers and want your server to talk directly 
to the  internet's root and authoritative servers.
In other words, it reaches out directly to the global DNS hierarchy, starting with the root servers, then moving to the TLD servers, and finally the authoritative servers.
Open `named.conf.options` and inside write down: 
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
```
sudo named-checkconf
sudo systemctl restart bind9
```
Verify external resolution:
```
dig @10.0.0.1 google.com 
```
It should return public IP addresses for Google!

__WARNING!__

you must free Port 53 for BIND9 if you are running more DNS services.
If you are running, for example, dnsmasq, before touching BIND9, verify that dnsmasq 
is only handling DHCP and not blocking port 53!
To disable DNS open dnsmasq.conf: 
```
sudo vim /etc/dnsmasq.conf
```
Ensure the line port=0 is present at the bottom. Then restart dnsmasq:
```
sudo systemctl restart dnsmasq
```
Setting port=0 turns off dnsmasq's internal DNS server
while keeping its DHCP server active.

If sudo systemctl restart named is not working then remove the logs from `named.conf.options` .
