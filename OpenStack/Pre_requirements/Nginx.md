# Nginx Proxy 

Unlike a regular website server, here it will function as a TCP Stream Proxy to manage network traffic between OpenStack services.

__Install Nginx & streaming module__
Since OpenStack handles direct TCP connections to its APIs, install the main Nginx package along with the streaming module (libnginx-mod-stream). Installing Nginx and the Stream Module:
`sudo apt install nginx libnginx-mod-stream`

Nginx has two main configuration blocks, the http block (for normal web sites) and the stream block (for raw TCP traffic, which is what OpenStack needs).
Open the main Nginx configuration file located at `/etc/nginx/nginx.conf`

At the end of the file add the basic stream block (which will serve as a receiver for the OpenStack service rules as you install them):
```
stream {

# TCP proxy rules for OpenStack services will be added here later.

}
```

Save and close the file.
To verify, test if the Nginx syntax is correct by running:
`sudo nginx -t`

If the result shows syntax is ok and test is successful, the configuration is perfect.

Activate and Start Nginx and also ensure the service starts automatically with Debian and get it running:
```
sudo systemctl enable nginx

sudo systemctl start nginx
```
And that is all for the proxy server for OpenStack.
