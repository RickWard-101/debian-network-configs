# RabbitMQ

RabbitMQ is a message-broker which means that it is an intermediary computer program module that translates a message from the formal messaging protocol of the sender to the formal messaging protocol of the receiver. 

Message brokers are elements in telecommunication or computer networks where software applications communicate by exchanging formally defined messages.

__Install the RabbitMQ__ 

`sudo apt install rabbitmq-server` 

Create the OpenStack User. OpenStack needs to authenticate with RabbitMQ to send and receive messages. Create a dedicated user called openstack (replace YOUR_PASSWORD with a secure password):
`sudo rabbitmqctl add_user openstack YOUR_PASSWORD`

Verification: The terminal should return a success message, such as Creating user "openstack" ....

Configure user permissions, this is needed because for OpenStack to be able to read, write, and configure message queues on the bus, it needs full permissions (.*) to the user just created in the default virtual environment (/):
`sudo rabbitmqctl set_permissions openstack ".*" ".*" ".*"`

Verification: The terminal should respond with the confirmation Setting permissions for user "openstack" in vhost "/" ....

The messaging system is operational and ready to receive traffic from future services!
