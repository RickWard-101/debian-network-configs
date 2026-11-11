# MariaDB

MariaDB is a relational database management system forked from MySQL.
It will be the main file (the long-term relational database). The central database where the state of the entire cloud resides.

__Instal MariaDB__

Install MariaDB server and the Python connection package (python3-pymysql), which is strictly necessary for OpenStack services to communicate with SQL.
```
sudo apt install mariadb-server python3-pymysql -y
```

To avoid character encoding issues and ensure compatibility with OpenStack, create a dedicated configuration file and open the custom configuration file `/etc/mysql/mariadb.conf.d/99-openstack.cnf`. Inside of `99-openstack.cnf`, paste the following:
```
[mysqld]
bind-address = 10.0.0.1

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8mb4_general_ci
character-set-server = utf8mb4
```

*Remember it's the LAN interface on bind-address*, **why?** Because the WAN allows access to the VM via SSH from outside and then access the OpenStack web panel (Horizon). The LAN is where all OpenStack microservices (MariaDB, RabbitMQ, Keystone, Nova, Neutron) will securely communicate with each other, isolated from the main network.

__Creating Databases and Users__

Access the MariaDB console to create the databases for the main OpenStack services and assign the respective access credentials.
```
sudo mysql
```
Inside, create the data and define the passwords.
**This is only a template but if you wish to copy paste at least change the passwords 'PASSWORD'**
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'PASSWORD';

CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'PASSWORD';

CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'PASSWORD';

CREATE DATABASE nova_api;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'PASSWORD';

CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'PASSWORD';

CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'PASSWORD';

FLUSH PRIVILEGES;
EXIT;
```
Some explanations:

`CREATE DATABASE <name>;` Here is the creation of isolated drawers (databases) on the database server. Each large OpenStack microservice has its own dedicated database to avoid mixing data:

**keystone**: Saves users, projects, and security tokens.

**glance**: Saves the catalog of operating system images (e.g. Debian 13 image).

**nova (and variants nova_api, nova_cell0)**: Saves the state of instances, virtual computers, and the internal organization of the computing engine.

**neutron**: Saves the network topology, routers, subnets, and assigned IPs.

`GRANT ALL PRIVILEGES ON ... FOR ...` Since OpenStack services need to access these databases to read and write data, there needs to be a dedicated user for each service and giving it full permissions (ALL PRIVILEGES) over the respective database.

*Observe the following crucial lines:* 

`'user'@'localhost'`: Allows the service to access the database from the local machine itself.
`'user'@'%'`: The % symbol means "any IP" (wildcard). This is essential in OpenStack because, if in the future expand is needed in the cloud to multiple physical servers (separate nodes), the remote computing nodes will need to connect to the central database via the LAN network (10.0.0.1).

`FLUSH PRIVILEGES; and EXIT;` Tells MariaDB to immediately reload the permissions table, ensuring that the rules created take effect immediately.
EXIT: Closes the MariaDB console and returns to the normal Linux terminal.

To verify, test to see if you can authenticate as one of the users created by executing:
```
mysql -u keystone -p -h localhost keystone
```
Insert the password for keystone. If you enter the interactive console `MariaDB [(keystone)]>` then everything is ok. Use `exit`. To check table, on mysql terminal: `SHOW DATABASES;`

For now, that's it! The data base has been created and configured.

