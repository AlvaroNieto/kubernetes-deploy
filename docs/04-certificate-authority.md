## Provisioning a CA and Generating TLS Certificates

In this lab I will provision a PKI Infrastructure using openssl to bootstrap a Certificate Authority, and generate TLS certificates for the following components: kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy. The commands in this section will be run from the `jumpbox`.

### Certificate Authority

In this section I will provision a Certificate Authority that can be used to generate additional TLS certificates for the other Kubernetes components. Setting up CA and generating certificates using `openssl` can be time-consuming, especially when doing it for the first time. To streamline this lab, I am going to use an openssl configuration file `ca.conf` from the Kubernetes the hard way guide. It defines all the details needed to generate certificates for each Kubernetes component.

I don't need to understand everything in the ca.conf file to complete this tutorial, but it's a starting point for learning `openssl` and the configuration that goes into managing certificates at a high level.

Every certificate authority starts with a private key and root certificate. In this section I am going to create a self-signed certificate authority, and while that's all I need for this tutorial, this won't be used in a real-world production environment.

Generate the CA configuration file, certificate, and private key:

```
{
  openssl genrsa -out ca.key 4096
  openssl req -x509 -new -sha512 -noenc \
    -key ca.key -days 3653 \
    -config ca.conf \
    -out ca.crt
}
```

Results:

```
-rw-r--r-- 1 root root 1.9K Dec  9 14:03 ca.crt
-rw------- 1 root root 3.2K Dec  9 14:03 ca.key
```

### Create Client and Server Certificates

In this section I will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes admin user.

Generate the certificates and private keys:

```
certs=(
  "admin" "node-0" "node-1"
  "kube-proxy" "kube-scheduler"
  "kube-controller-manager"
  "kube-api-server"
  "service-accounts"
)
```

```
for i in ${certs[*]}; do
  openssl genrsa -out "${i}.key" 4096

  openssl req -new -key "${i}.key" -sha256 \
    -config "ca.conf" -section ${i} \
    -out "${i}.csr"
  
  openssl x509 -req -days 3653 -in "${i}.csr" \
    -copy_extensions copyall \
    -sha256 -CA "ca.crt" \
    -CAkey "ca.key" \
    -CAcreateserial \
    -out "${i}.crt"
done

```

The results of running the above command will generate a private key, certificate request, and signed SSL certificate for each of the Kubernetes components. I can list the generated files with the following command:

```
ls -1 *.crt *.key *.csr
```

Output:

```
admin.crt
admin.csr
admin.key
ca.crt
ca.key
kube-api-server.crt
kube-api-server.csr
kube-api-server.key
kube-controller-manager.crt
kube-controller-manager.csr
kube-controller-manager.key
kube-proxy.crt
kube-proxy.csr
kube-proxy.key
kube-scheduler.crt
kube-scheduler.csr
kube-scheduler.key
node-0.crt
node-0.csr
node-0.key
node-1.crt
node-1.csr
node-1.key
service-accounts.crt
service-accounts.csr
service-accounts.key
```

### Distribute the Client and Server Certificates

In this section I will copy the various certificates to every machine at a path where each Kubernetes component will search for its certificate pair. In a real-world environment these certificates should be treated like a set of sensitive secrets as they are used as credentials by the Kubernetes components to authenticate to each other.

Copy the appropriate certificates and private keys to the `node-0` and `node-1` machines:

```
for host in node-0 node-1; do
  ssh root@$host mkdir /var/lib/kubelet/
  
  scp ca.crt root@$host:/var/lib/kubelet/
    
  scp $host.crt \
    root@$host:/var/lib/kubelet/kubelet.crt
    
  scp $host.key \
    root@$host:/var/lib/kubelet/kubelet.key
done
```

Copy the appropriate certificates and private keys to the `server` machine:

```
scp \
  ca.key ca.crt \
  kube-api-server.key kube-api-server.crt \
  service-accounts.key service-accounts.crt \
  root@server:~/
```

The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab.

Next: Generating Kubernetes Configuration Files for Authentication