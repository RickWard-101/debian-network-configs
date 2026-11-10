# CHRONY NTP SERVER

The NTP server is very important because, it keeps the server clock perfectly accurate. In a cloud environment, if the clocks of different components are out of sync by fractions of a second, security tokens expire prematurely and certificates fail.

__Install Chrony__

```
sudo apt update && sudo apt install chrony -y
```
Now that Chrony is installed, open the configuration file to guarantee the the server
points to the reliable sources and permits other computers to consult the hour.
The configuration file is located at `/etc/chrony/chrony.conf`.

Make sure to have the official Debian NTP server pool lines active at the top, or in my case
i changed it to my country for smaller latency. The official should be something like `pool 0.debian.pool.ntp.org iburst`. *CHECK CHRONY WEBSITE TO CHECK YOUR ZONE.*

Also, add your subnet permission rule at the end of the file. In this case, as stated in the README.md file it's the 10.0.0.1/24.
So add at the end of the file:

```
allow 10.0.0.1/24
```
The `allow` command is intended exclusively for the internal network. Never, under any circumstances, should the WAN use `allow 0.0.0.0/0` (allow everything on the WAN).
**This is because, the NTP protocol is historically vulnerable to NTP Amplification Attacks. If you leave the NTP server open to the global internet, cybercriminals can use your machine to send massive packets of junk to third parties, which will cause your ISP to cut off your connection immediately.**

**openstack info:** Also the `allow` command allows future nodes in the cloud to query the time for your main node. !You should strictly use the IP range of your internal network! AS I JUST SAID ON THE TEXT ABOVE! 

Restart and enable deamon.
```
sudo systemctl restart chrony
&&
sudo systemctl enable chrony
```

To validate the synchronization and confirm 100% that your server is now synchronized with the outside world use:
```
chronyc sources
```
In the returned table, there should be at least one row with a ^* symbol to the left of the server name, which proves that clock synchronization is active and operational.

That's it! The server is in sync with the rest of the world.


