 # Nginx Web Server

This guide covers the installation and configuration of the **Nginx** web server on Debian, used to host internal infrastructure dashboards and serve web pages under the local domain (`web.domain.net`).

__Architecture Overview__
In this setup, Nginx acts as the local HTTP server listening on port 80. Combined with the local **BIND9 DNS server**, client machines on the LAN (`192.168.100.0/24`) can resolve `web.domain.net` directly to the gateway/server IP and instantly load the lab presentation dashboard.

__Step-by-Step Implementation__

Install Nginx
Update package lists and install Nginx on the Debian server:

        sudo apt update && sudo apt install nginx -y
        
Create the Web Root Directory & Custom Dashboard.
Create a dedicated directory for your domain's files and set up an HTML presentation page.
Create the directory:

        sudo mkdir -p /var/www/web.domain.net
        
Create the generic index file at /var/www/web.domain.net/index.html:
```
        <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Rickwardo Lab | Infrastructure Dashboard</title>
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
        <h1>Infrastructure Lab Dashboard</h1>
        <p>Active Web Server: <strong>web.rickwardo.net</strong></p>
        <p>System Status: <span class="status">● OPERATIONAL</span></p>
        <hr>
        <h3>Active LAN Services (100.100.100.0/24):</h3>
        <ul>
            <li><strong>Gateway & NAT:</strong> Debian Linux (IPTables)</li>
            <li><strong>DNS Server:</strong> BIND9 (Split-Horizon)</li>
            <li><strong>DHCP Server:</strong> Kea DHCP</li>
            <li><strong>Web Server:</strong> Nginx</li>
        </ul>
    </div>
</body>
</html>
```
__Fix Directory Permissions__
Ensure the web server user (www-data) owns the files:

        sudo chown -R www-data:www-data /var/www/web.domain.net

Configure the Nginx Server Block (Virtual Host)
Create a new configuration file for your domain inside sites-available:

        sudo nano /etc/nginx/sites-available/web.rickwardo.net

```
server {
    listen 80;
    server_name web.domain.net;

    root /var/www/web.domain.net;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

__Enable the Site & Reload Nginx__

Create a symbolic link to activate the site:
        sudo ln -s /etc/nginx/sites-available/web.domain.net /etc/nginx/sites-enabled/
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

**Cause**: Incorrect file permissions on /var/www/web.domain.net or missing index.html file in the root directory.

How to fix:
Re-apply ownership permissions: 
        sudo chown -R www-data:www-data /var/www/web.domain.net.
Confirm that index.html exists inside the target folder.
