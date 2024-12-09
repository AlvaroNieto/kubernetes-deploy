## Configuring kubectl for Remote Access

In this lab I will generate a kubeconfig file for the `kubectl` command line utility based on the admin user credentials.

Run the commands in this lab from the `jumpbox` machine.

### The Admin Kubernetes Configuration File


Each kubeconfig requires a Kubernetes API Server to connect to.

I am be able to ping server.kubernetes.local based on the /etc/hosts DNS entry from a previous lap.

```
curl -k --cacert ca.crt \
  https://server.kubernetes.local:6443/version
```

Output:

```
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

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

}
The results of running the command above will create a kubeconfig file in the default location `~/.kube/config` used by the kubectl commandline tool. This also means I can run the `kubectl` command without specifying a config.

### Verification

Check the version of the remote Kubernetes cluster:

```
kubectl version
```

```
Client Version: v1.29.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.29.3
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

```
NAME     STATUS   ROLES    AGE     VERSION
node-0   Ready    <none>   7m22s   v1.29.3
node-1   Ready    <none>   3m29s   v1.29.3

```

Next: Provisioning Pod Network Routes