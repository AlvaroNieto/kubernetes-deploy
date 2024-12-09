## Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab I will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

### Prerequisites

Copy `etcd` binaries and systemd unit files to the `server` instance:

```
scp \
  downloads/etcd-v3.5.13-linux-amd64.tar.gz \
  units/etcd.service \
  root@server:~/
```

The commands in this lab must be run on the server machine. Login to the server machine using the ssh command `ssh root@server`

### Bootstrapping an etcd Cluster
#### Install the etcd Binaries

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.5.13-linux-amd64.tar.gz
  mv etcd-v3.5.13-linux-amd64/etcd* /usr/local/bin/
}
```

#### Configure the etcd Server

{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.crt kube-api-server.key kube-api-server.crt \
    /etc/etcd/
}

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

Create the `etcd.service` systemd unit file:

```
mv etcd.service /etc/systemd/system/
```

#### Start the etcd Server
```

{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

### Verification

List the etcd cluster members:

```
etcdctl member list
```

```
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

Next: [Bootstrapping the Kubernetes Control Plane](https://github.com/AlvaroNieto/kubernetes-deploy/blob/main/docs/08-bootstrapping-kubernetes-controllers.md)