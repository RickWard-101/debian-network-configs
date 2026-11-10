#  Cloud Computing
__What is this project about?__

This project is my learning experience on how to build and configure a cloud computing using **Openstack**.
I am using VM's (virtual machines), but everything done here can 
be applied to real physical hardware.
It can be used as a safe copy & paste tool for when working with configurations on servers.

# Important details/ warnings:
__Details about the networking__

Every server that needs two networking interfaces, in all configuration files, the LAN interface will always
be the static IP 10.0.0.1/24 . Change this IP as you wish but make sure to change it on every configuration file.
