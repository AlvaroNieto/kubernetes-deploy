## Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab I will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

### The Routing Table

In this section I gather the information required to create routes in the `kubernetes-the-hard-way` VPC network.

Print the internal IP address and Pod CIDR range for each worker instance:

```
{
  SERVER_IP=$(grep server machines.txt | cut -d " " -f 1)
  NODE_0_IP=$(grep node-0 machines.txt | cut -d " " -f 1)
  NODE_0_SUBNET=$(grep node-0 machines.txt | cut -d " " -f 4)
  NODE_1_IP=$(grep node-1 machines.txt | cut -d " " -f 1)
  NODE_1_SUBNET=$(grep node-1 machines.txt | cut -d " " -f 4)
}
```

```
ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

```
ssh root@node-0 <<EOF
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

```
ssh root@node-1 <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
EOF
```

### Verification

```
ssh root@server ip route
```

```
default via 10.1.1.1 dev eth0 onlink
10.1.1.0/24 dev eth0 proto kernel scope link src 10.1.1.101
10.200.0.0/24 via 10.1.1.102 dev eth0
10.200.1.0/24 via 10.1.1.103 dev eth0
```

```
ssh root@node-0 ip route
```

```
default via 10.1.1.1 dev eth0 onlink
10.1.1.0/24 dev eth0 proto kernel scope link src 10.1.1.102
10.200.1.0/24 via 10.1.1.103 dev eth0
```

```
ssh root@node-1 ip route
```

```
default via 10.1.1.1 dev eth0 onlink
10.1.1.0/24 dev eth0 proto kernel scope link src 10.1.1.103
10.200.0.0/24 via 10.1.1.102 dev eth0
```



Next: Smoke Test