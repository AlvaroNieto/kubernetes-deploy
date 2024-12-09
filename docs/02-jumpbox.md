## Jumphost setup

I will set up one of the four machines to be a jumpbox. This machine will be used to run commands. 

### Log into the jumbox and change to root:
```
ssh alvaro@jumpbox
su root
```

### Install Command Line Utilities

Install the command line utilities that will be used to preform various tasks.

```
apt-get -y install wget curl vim openssl git
```

### Sync GitHub Repository

Download a copy of this tutorial which contains the configuration files and templates that will be used build your Kubernetes cluster from the ground up
```
git clone --depth 1 \
  https://github.com/kelseyhightower/kubernetes-the-hard-way.git
```
Change into the kubernetes-the-hard-way directory

```cd kubernetes-the-hard-way```

### Download binaries

In this section I will download the binaries for the various Kubernetes components. The binaries will be stored in the downloads directory on the jumpbox, which will reduce the amount of internet bandwidth required to complete this tutorial as we avoid downloading the binaries multiple times for each machine in our Kubernetes cluster.

The binaries that will be downloaded are listed in the downloads.txt that can be previewd using:

```cat downloads.txt```

I encounter the first issue. Every binary is for the ARM64 platform:

```
https://dl.k8s.io/v1.31.2/bin/linux/arm64/kubectl
https://dl.k8s.io/v1.31.2/bin/linux/arm64/kube-apiserver
https://dl.k8s.io/v1.31.2/bin/linux/arm64/kube-controller-manager
https://dl.k8s.io/v1.31.2/bin/linux/arm64/kube-scheduler
https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.31.1/crictl-v1.31.1-linux-arm64.tar.gz
https://github.com/opencontainers/runc/releases/download/v1.2.1/runc.arm64
https://github.com/containernetworking/plugins/releases/download/v1.6.0/cni-plugins-linux-arm64-v1.6.0.tgz
https://github.com/containerd/containerd/releases/download/v2.0.0/containerd-2.0.0-linux-arm64.tar.gz
https://dl.k8s.io/v1.31.2/bin/linux/arm64/kube-proxy
https://dl.k8s.io/v1.31.2/bin/linux/arm64/kubelet
https://github.com/etcd-io/etcd/releases/download/v3.4.34/etcd-v3.4.34-linux-arm64.tar.gz
```

Since I am going to use AMD64 instead of ARM64, I have to change the binaries that are going to be used. Recreate the downloads.txt using the correct binaries:

```
cp downloads.txt downloads.bk
echo "https://dl.k8s.io/v1.29.3/bin/linux/amd64/kubectl
https://dl.k8s.io/v1.29.3/bin/linux/amd64/kube-apiserver
https://dl.k8s.io/v1.29.3/bin/linux/amd64/kube-controller-manager 
https://dl.k8s.io/v1.29.3/bin/linux/amd64/kube-scheduler
https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.29.0/crictl-v1.29.0-linux-amd64.tar.gz
https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz
https://github.com/containerd/containerd/releases/download/v1.7.15/containerd-1.7.15-linux-amd64.tar.gz
https://dl.k8s.io/v1.29.3/bin/linux/amd64/kube-proxy
https://dl.k8s.io/v1.29.3/bin/linux/amd64/kubelet
https://github.com/etcd-io/etcd/releases/download/v3.5.13/etcd-v3.5.13-linux-amd64.tar.gz" > downloads.txt
```

Now I can download the binaries listed in the downloads.txt file into a directory called downloads using the wget command:

```
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads.txt
```

### Install kubectl

In this section I will install the kubectl, the official Kubernetes client command line tool, on the jumpbox machine. Kubectl will be used to interact with the Kubernetes control once the cluster is provisioned.

Use the chmod command to make the kubectl binary executable and move it to the /usr/local/bin/ directory:

```
{
  chmod +x downloads/kubectl
  cp downloads/kubectl /usr/local/bin/
}
```

At this point kubectl is installed and can be verified by running the kubectl command:

```
kubectl version --client
```
```
Client Version: v1.29.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

Next: [Compute Resources](https://github.com/AlvaroNieto/kubernetes-deploy/blob/main/docs/03-compute-resources.md)