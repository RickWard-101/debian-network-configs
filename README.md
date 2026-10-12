# DEBIAN ROUTER TOPOLOGY
__What is this project about?__
Welcome to the Debian router project. In this project, i've made a bunch of 
configuration file teaching how to make a network topology using Debian as the main 
router/modem/server. 
I advice to follow this project from start to finish to learn more about networking.
This project uses VM (virtual machines), but everything done on VM's should 
be applied to real physical hardware.

__What can i learn form this?__

This project is an amazing tool to learn and customize a network topology and 
making all the necessary servers on Debian. I suppose this can also be used as a 
safe copy paste tool for when working with configurations on servers. Bellow i will explain
what are the steps and what order this should be seen.

1. VM creations and what we need.
this will include the setup of networking interfaces we will use.

2. Setup of Debian netinstall server.
In here will include all the necessary tools to start making any server. 

3. install & configure an authoritative DNS server.
what is a DNS server and how to use it.

4. Install & configure a DHCP server.
What is a DHCP server and how to use it.

5. Install & configre web server.

__Where should the server be located?__

This project looks something like this:

| WAN <------------> Debian server <------------> Cient PC's  |
|           |                            |                    |
|           --> WAN                      --> LAN              |
|               interface                    interface        |
          

