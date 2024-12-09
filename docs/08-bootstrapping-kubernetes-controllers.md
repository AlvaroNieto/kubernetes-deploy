## Bootstrapping the Kubernetes Control Plane

In this lab I will bootstrap the Kubernetes control plane. The following components will be installed on the controller machine: Kubernetes API Server, Scheduler, and Controller Manager.

### Prerequisites

Connect to the `jumpbox` and copy Kubernetes binaries and systemd unit files to the `server` instance:

```
scp \
  downloads/kube-apiserver \
  downloads/kube-controller-manager \
  downloads/kube-scheduler \
  downloads/kubectl \
  units/kube-apiserver.service \
  units/kube-controller-manager.service \
  units/kube-scheduler.service \
  configs/kube-scheduler.yaml \
  configs/kube-apiserver-to-kubelet.yaml \
  root@server:~/

```

The commands in this lab must be run on the controller instance: `server`. Login to the controller instance using the ssh command. `ssh root@server`

### Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
mkdir -p /etc/kubernetes/config
```

#### Install the Kubernetes Controller Binaries

Install the Kubernetes binaries:

```
{
  chmod +x kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl
    
  mv kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl \
    /usr/local/bin/
}
```

#### Configure the Kubernetes API Server

```
{
  mkdir -p /var/lib/kubernetes/

  mv ca.crt ca.key \
    kube-api-server.key kube-api-server.crt \
    service-accounts.key service-accounts.crt \
    encryption-config.yaml \
    /var/lib/kubernetes/
}
```

Create the `kube-apiserver.service` systemd unit file:

#### Configure the Kubernetes Controller Manager

Move the `kube-controller-manager` kubeconfig into place:

```
mv kube-controller-manager.kubeconfig /var/lib/kubernetes/

```
Create the `kube-controller-manager.service` systemd unit file:

```
mv kube-controller-manager.service /etc/systemd/system/
```

#### Configure the Kubernetes Scheduler

Move the `kube-scheduler` kubeconfig into place:

```
mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the kube-scheduler.yaml configuration file:

```
mv kube-scheduler.yaml /etc/kubernetes/config/
```

Create the kube-scheduler.service systemd unit file:

```
mv kube-scheduler.service /etc/systemd/system/
```

#### Start the Controller Services

```
{
  systemctl daemon-reload
  
  systemctl enable kube-apiserver \
    kube-controller-manager kube-scheduler
    
  systemctl start kube-apiserver \
    kube-controller-manager kube-scheduler
}
```

Allow up to 10 seconds for the Kubernetes API Server to fully initialize.

#### Verification

```
kubectl cluster-info \
  --kubeconfig admin.kubeconfig
```

```
root@server:~# kubectl cluster-info \
  --kubeconfig admin.kubeconfig
Kubernetes control plane is running at https://127.0.0.1:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### RBAC for Kubelet Authorization

In this section I will configure RBAC permissions to allow the Kubernetes API Server to access the Kubelet API on each worker node. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.

This tutorial sets the `Kubelet --authorization-mode` flag to `Webhook`. Webhook mode uses the [SubjectAccessReview](https://kubernetes.io/docs/admin/authorization/#checking-api-access) API to determine authorization.

The commands in this section will affect the entire cluster and only need to be run on the controller node. `ssh root@server`

Create the `system:kube-apiserver-to-kubelet` [ClusterRole](https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole) with permissions to access the Kubelet API and perform most common tasks associated with managing pods:

#### Verification

At this point the Kubernetes control plane is up and running. Run the following commands from the `jumpbox` machine to verify it's working:

Make a HTTP request for the Kubernetes version info:

```
curl -k --cacert ca.crt https://server.kubernetes.local:6443/version
```

```
root@jumpbox:~/kubernetes-the-hard-way# curl -k --cacert ca.crt https://server.kubernetes.local:6443/version
{
  "major": "1",
  "minor": "29",
  "gitVersion": "v1.29.3",
  "gitCommit": "6813625b7cd706db5bc7388921be03071e1a492d",
  "gitTreeState": "clean",
  "buildDate": "2024-03-14T23:58:36Z",
  "goVersion": "go1.21.8",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

Next: [Bootstrapping the Kubernetes Worker Nodes](https://github.com/AlvaroNieto/kubernetes-deploy/blob/main/docs/09-bootstrapping-kubernetes-workers.md)
