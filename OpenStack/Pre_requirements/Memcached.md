# Memcached

Memcached saves authentication tokens in RAM so that Keystone doesn't need to access the database with every click.

__Install memcached__

`sudo apt install memcached python3-memcache`

Configuring the listening address. Open the configuration file located at `/etc/memcached.conf`

Locate the line that says **-l 127.0.0.1.** Since it is in an All-In-One environment where everything runs on the same machine, it can keep pointing to localhost (127.0.0.1), as the OpenStack services run on the same node. Save and close the file.

Activate and start the service:
```
sudo systemctl restart memcached

sudo systemctl enable memcached
```

Do a status on it. The status should indicate active (running). Also test if it responds on port 11211 with echo "stats" | nc localhost 11211.

That is it for the memcached
