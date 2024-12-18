## Bootstrapping the Kubernetes Worker Nodes

In this lab I will bootstrap two Kubernetes worker nodes. The following components will be installed: [runc](https://github.com/opencontainers/runc), [container networking plugins](https://github.com/containernetworking/cni), [containerd](https://github.com/containerd/containerd), [kubelet](https://kubernetes.io/docs/admin/kubelet), and [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

### Prerequisites

Copy Kubernetes binaries and systemd unit files to each worker instance:

```
for host in node-0 node-1; do
  SUBNET=$(grep $host machines.txt | cut -d " " -f 4)
  sed "s|SUBNET|$SUBNET|g" \
    configs/10-bridge.conf > 10-bridge.conf 
    
  sed "s|SUBNET|$SUBNET|g" \
    configs/kubelet-config.yaml > kubelet-config.yaml
    
  scp 10-bridge.conf kubelet-config.yaml \
  root@$host:~/
done
```

```
for host in node-0 node-1; do
  scp \
    downloads/runc.amd64 \
    downloads/crictl-v1.29.0-linux-amd64.tar.gz \
    downloads/cni-plugins-linux-amd64-v1.4.0.tgz \
    downloads/containerd-1.7.15-linux-amd64.tar.gz \
    downloads/kubectl \
    downloads/kubelet \
    downloads/kube-proxy \
    configs/99-loopback.conf \
    configs/containerd-config.toml \
    configs/kubelet-config.yaml \
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@$host:~/
done
```
The commands in this lab must be run on each worker instance: `node-0`, `node-1`. Login to the worker instance using the ssh command: `ssh root@node-0`

### Provisioning a Kubernetes Worker Node

Install the OS dependencies:

```
{
  apt-get update
  apt-get -y install socat conntrack ipset
}
```

The socat binary enables support for the `kubectl port-forward` command.

#### Disable Swap

By default, the kubelet will fail to start if swap is enabled. It is [recommended](https://github.com/kubernetes/kubernetes/issues/7294) that swap be disabled to ensure Kubernetes can provide proper resource allocation and quality of service.

Verify if swap is enabled:

```
swapon --show
```

If output is empty then swap is not enabled. If swap is enabled run the following command to disable swap immediately:

```
swapoff -a
```

In debian 12, to ensure swap remains off after a reboot, it must be edited out in the fstap file:

```
nano /etc/fstab
```

Locate the swap line and comment it out:

```
UUID=swap-uuid none swap sw 0 0 
```
to
```
#UUID=your-swap-uuid none swap sw 0 0
```

Also remove the partition that was used for the swap:

```
wipefs --all /dev/sdaX
```
```
{
systemctl stop dev-sdaX.swap
systemctl disable dev-sdaX.swap
systemctl mask dev-sdaX.swap
}
```
And reboot to confirm. 

#### Create the installation directories:

```
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes

```

Install the worker binaries:

```
{
  mkdir -p containerd
  tar -xvf crictl-v1.29.0-linux-amd64.tar.gz
  tar -xvf containerd-1.7.15-linux-amd64.tar.gz -C containerd
  tar -xvf cni-plugins-linux-amd64-v1.4.0.tgz -C /opt/cni/bin/
  mv runc.amd64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc 
  mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  mv containerd/bin/* /bin/
}
```

#### Configure CNI Networking

Create the `bridge` network configuration file:

```
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```

#### Configure containerd

Install the `containerd` configuration files:

```
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}
```

#### Configure the Kubelet

Create the `kubelet-config.yaml` configuration file:

```
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}
```

#### Configure the Kubernetes Proxy

```
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

#### Start the Worker Services

```
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

### Verification

The compute instances created in this tutorial will not have permission to complete this section. Run the following commands from the `jumpbox` machine.

List the registered Kubernetes nodes:

```
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

Output:

```
NAME     STATUS   ROLES    AGE   VERSION
node-0   Ready    <none>   38s   v1.29.3
```

I repeat the provisioning a Kubernetes Worker Node process on the node-1 and recheck. 

```
NAME     STATUS   ROLES    AGE   VERSION
node-0   Ready    <none>   4m    v1.29.3
node-1   Ready    <none>   7s    v1.29.3
```

Next: [Configuring kubectl for Remote Access](https://github.com/AlvaroNieto/kubernetes-deploy/blob/main/docs/10-configuring-kubectl.md)