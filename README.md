#  DEBIAN - Cloud Computing / Networking Servers 
__What is this project about?__

This project is my learning experience on how to build and configure a cloud computing using **Openstack**.
I am also putting some other servers in here like DNS, DHCP, Email servers and ETC. (I am not focusing so much on those, since Cloud Computing takes a lot of efford).
I am using VM's (virtual machines), but everything done here can be applied to real physical hardware. *For the OpenStack VM i recommend at least a 16gb or RAM and 6 Cores.*
It can be used as a safe copy & paste tool for when working with configurations on servers. 
# Important details/ warnings:
__Details about the networking__

Every server that needs two networking interfaces, in all configuration files, the LAN interface will always
be the static IP 10.0.0.1/24 . Change this IP as you wish but make sure to change it on every configuration file.
The WAN can change from configuration to configuration as i work with different machines.

# Details on the Openstack
__Services i install and configure before the main components of OpenStack__

Open the OpenStack folder and you will find a bunch of servers.
I install them and configure them in this order:
1. NTP - *clock sync*
2. MariaDB - *data base*
3. RabbitMQ - *broker*
4. Nginx - *reversed proxy*
5. memcached - *high performance caching system*
6. ETCD - *snapshot and source of truth*

__Main Components of OpenStack__

 | ______ Service ___ | _______________________|       Code Name       |
                    
[Identity Service]  <----------------------->       Keystone 

[Compute Service]   <----------------------->       Nova

[Image Service]     <----------------------->       Glance

[Dashboard] -------------> Horizon
    
[Object Storage] -------------> Swift

[Block Storage] -------------> Cinder

[Network Service] ...........> Neutron

[Load Balancing Service] ....> Octavia

[Orchestration Service] .....> Heat

[Metering Service] ..........> Ceilometer
        
[Rating Service] ............> CloudKitty

[Database Service] ..........> Trove

[Container Service] .........> Magnum

[Data Processing Service] ...> Sahara

[Bare Metal Provisioning] ...> Ironic

[Shared File System] ........> Manila

[DNS Service] ...............> Designate

[Key Manager Service] .......> Barbican

