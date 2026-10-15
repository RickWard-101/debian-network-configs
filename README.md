#  ULTIMATE DEBIAN SERVER
__What is this project about?__

This project is a tool to learn and customize how to make an all-in-one server on Debian and 
making all the necessary services on it. 
In here you have all kinds of servers: DHCP, DNS, Authoritative DNS, Email, Web server and much more.
This project was done and tested using VM's (virtual machines), but everything done here can 
be applied to real physical hardware.
It can be used as a safe copy & paste tool for when working with configurations on servers.
I advice to follow this project to learn more about networking.

# Important details/ warnings:
__Details about the networking__

Every server that needs two networking interfaces, in all configuration files, the LAN interface will always
be the static IP 192.168.1.1/24 . Change this IP as you wish but make sure to change it on every configuration file.
You need to have a bit of information on networking to understand what is going on in the conf files and general topology.
