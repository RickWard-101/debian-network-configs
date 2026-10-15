 # Nginx Web Server

This guide covers the installation and configuration of the **Nginx** web server on Debian, used to host internal infrastructure and serve web pages under the local domain (`web.domain.net`).

__!WARNING!__
In this setup, Nginx acts as the local HTTP server listening on port 80. Combined with the local **BIND9 DNS server**, client machines on the LAN (`192.168.100.0/24`) can resolve `web.domain.net` directly to the gateway/server IP and instantly load the domain presentation dashboard.
**Make sure to check the authoritative DNS server installation of BIND9.**

__Instalation and configuration__
Update package lists and install Nginx on the Debian server:

        sudo apt update && sudo apt install nginx -y
        
Create a dedicated directory for your domain's files and set up an HTML presentation page.

        sudo mkdir -p /var/www/domain.net
 
Create the generic index file at /var/www/web.domain.net:

        sudo vim /var/www/domain.net/index.html
        
**Create the index to your liking, this is just default HTML index file.**

```
        <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Domain project | Infrastructure Dashboard</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #1e1e2e; color: #cdd6f4; text-align: center; padding-top: 50px; }
        .card { background: #313244; max-width: 600px; margin: 0 auto; padding: 25px; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        h1 { color: #89b4fa; }
        .status { color: #a6e3a1; font-weight: bold; }
        ul { text-align: left; line-height: 1.8; }
    </style>
</head>
<body>
    <div class="card">
        <h1>Infrastructure project Dashboard</h1>
        <p>Active Web Server: <strong>web.domain.net</strong></p>
        
        <hr>
        <h3>Active LAN Services (192.168.100.0/24):</h3>
        <ul>
            <li><strong>Gateway & NAT:</strong> Debian Linux (IPTables)</li>
            <li><strong>DNS Server:</strong> BIND9</li>
            <li><strong>DHCP Server:</strong> Kea DHCP</li>
            <li><strong>Web Server:</strong> Nginx</li>
        </ul>
    </div>
</body>
</html>
```

__Configure virtual host on Nginx__
On Nginx, every site / service has it's own configuration file at /etc/nginx/sites-available/.
       
        sudo vim /etc/nginx/sites-available/domain.net

This file is the creation of the web server rules (the Server Block or Virtual Host) to teach Nginx how to manage the traffic arriving at your server.
Without this file, Nginx is "blind". 
It receives the request on the network, but doesn't know which folder on the disk to open or who to respond to.

Therefore, Paste the following configuration:
```
server {
    listen 80;
    listen [::]:80;

    server_name web.domain.net domain.net;

    root /var/www/domain.net;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/domain_access.log;
    error_log /var/log/nginx/domain_error.log;
}
```

### What does each line you pasted mean:

1. listen 80; and listen [::]:80:
This tells Nginx to "listen" on port 80 (the default port for the unencrypted HTTP protocol) for both IPv4 and IPv6.

2. server_name web.domain.net domain.net:
This is the identifier. When BIND9 translates this name to the IP 192.168.100.1 and the request reaches Debian, Nginx reads the HTTP header. If the browser requested web.domain.net or domain.net, Nginx knows to use this specific configuration (not the default Debian page).

3. «root /var/www/domain.net:
This indicates the path on the hard drive where your website files are stored. It's the root directory where you created index.html.

4. index index.html:
This defines the main file. When someone accesses only [web.domain.net/](http://web.domain.net/) without specifying a file at the end of the URL, Nginx automatically looks for index.html in that folder.

5. location / { try_files $uri $uri/ =404; }:
Defines the navigation rules for any page (/):

6. access_log: 
Records each visit (client IP, time, browser used).

7. error_log: 
Records errors (files not found, permission errors), which is vital for troubleshooting.


__Enable the Site & Reload Nginx__

Create a symbolic link to activate the site:

        sudo ln -s /etc/nginx/sites-available/domain.net /etc/nginx/sites-enabled/

Test syntax and reload the service:

        sudo nginx -t
        sudo systemctl reload nginx
        
# Troubleshooting & Common Issues ("What if...?")
❌ Issue 1: Browsing to http://web.domain.net results in a "Server Not Found" or DNS error on client machines.

**Cause**: The DNS server (BIND9) does not have an A record mapping web to the server's IP, or the client hasn't refreshed its DHCP lease/DNS cache.

    How to fix:

Verify the zone file (/etc/bind/db.domain.net) includes: web IN A 192.168.100.1 and ensure the serial number was incremented.
Restart BIND9: 
        sudo systemctl restart bind9.
Test resolution directly from the client using: dig @192.168.100.1 web.domain.net.

❌ Issue 2: Nginx returns a 403 Forbidden error when accessing the page.

**Cause**: Incorrect file permissions on /var/www/domain.net or missing index.html file in the root directory.

How to fix:
Re-apply ownership permissions: 
        sudo chown -R www-data:www-data /var/www/domain.net.
Confirm that index.html exists inside the target folder.
