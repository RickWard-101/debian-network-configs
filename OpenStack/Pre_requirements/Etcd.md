# ETCD

etcd is a distributed, open-source, and highly consistent key-value store. It is often described as the "durable memory" or single source of truth for large-scale distributed systems.

__Install etcd__
`sudo apt install etcd -y`

To configuring Etcd open the main configuration file at `/etc/etcd/etcd.conf.yml` This location may change depending on distros.

For an All-In-One environment on your LAN (10.0.0.1), you must adjust the listening and advertising parameters for the internal IP. Find and adapt the corresponding lines to have the following structure (replacing 10.0.0.1 with your LAN IP):
```
name: 'controller'
data-dir: '/var/lib/etcd/default.etcd'
listen-peer-urls: 'http://10.0.0.1:2380'
listen-client-urls: 'http://10.0.0.1:2379,http://127.0.0.1:2379'
initial-advertise-peer-urls: 'http://10.0.0.1:2380'
advertise-client-urls: 'http://10.0.0.1:2379'
initial-cluster: 'controller=http://10.0.0.1:2380'
initial-cluster-token: 'etcd-cluster-01'
Initial cluster state: 'new'
```

Save and close the and activate/ start the service:
```
sudo systemctl daemon-reload
sudo systemctl restart etcd
sudo systemctl enable etcd
```

Verification: Run sudo systemctl status etcd. It should be active (running). You can test the cluster health with `etcdctl endpoint health`

(If it returns that the endpoint is healthy, you're all set!).

And there it is. Done.
