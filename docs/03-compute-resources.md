## Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab yIou will provision the machines required for setting up a Kubernetes cluster.

### Machine database

This tutorial will leverage a text file, which will serve as a machine database, to store the various machine attributes that will be used when setting up the Kubernetes control plane and worker nodes. The following schema represents entries in the machine database, one entry per line:

```
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

Each of the columns corresponds to a machine IP address `IPV4_ADDRESS`, fully qualified domain name `FQDN`, host name `HOSTNAME`, and the IP subnet `POD_SUBNET`. Kubernetes assigns one IP address per `pod` and the `POD_SUBNET` represents the unique IP address range assigned to each machine in the cluster for doing so.

Here is an example machine database similar to the one used when creating this tutorial. Notice the IP addresses have been masked out. Your machines can be assigned any IP address as long as each machine is reachable from each other and the `jumpbox`.

```
nano machines.txt
```

Insert the data:

```
10.1.1.101 server.kubernetes.local server  
10.1.1.102 node-0.kubernetes.local node-0 10.200.0.0/24
10.1.1.103 node-1.kubernetes.local node-1 10.200.1.0/24
```

### Configuring SSH Access

SSH will be used to configure the machines in the cluster. Verify that I have `root` SSH access to each machine listed in the machine database. I have to enable root SSH access on each node by updating the sshd_config file and restarting the SSH server.

#### Enable root SSH Access

By default, a new debian install disables SSH access for the root user. To streamline the steps in this tutorial I am enabling root access over SSH. Security is a tradeoff, and in this case, I am optimizing for convenience. Log on to each machine via SSH using the user account, then switch to the root user using the su command:

```
su root
```

Edit the /etc/ssh/sshd_config SSH daemon configuration file and set the PermitRootLogin option to yes:

```
sed -i \
  's/^#PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

Restart the sshd SSH server to pick up the updated configuration file:

```
systemctl restart sshd
```

### Generate and Distribute SSH Keys

In this section you will generate and distribute an SSH keypair to the `server`, `node-0`, and `node-1`, machines, which will be used to run commands on those machines throughout this tutorial. Run the following commands from the `jumpbox` machine.

Generate a new SSH key:

```
ssh-keygen
```

```
root@jumpbox:~# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

Copy the SSH public key to each machine:

```
while read IP FQDN HOST SUBNET; do 
  ssh-copy-id root@${IP}
done < machines.txt
```

Once each key is added, verify SSH public key access is working:

```
while read IP FQDN HOST SUBNET; do 
  ssh -n root@${IP} uname -o -m
done < machines.txt
```
Output:

```
x86_64 GNU/Linux
x86_64 GNU/Linux
x86_64 GNU/Linux

```
### Hostnames

In this section I will assign hostnames to the `server`, `node-0`, and `node-1` machines. The hostname will be used when executing commands from the `jumpbox` to each machine. The hostname also plays a major role within the cluster. Instead of Kubernetes clients using an IP address to issue commands to the Kubernetes API server, those clients will use the `server` hostname instead. Hostnames are also used by each worker machine, `node-0` and `node-1` when registering with a given Kubernetes cluster.

To configure the hostname for each machine, run the following commands on the jumpbox.

Set the hostname on each machine listed in the machines.txt file:

```
while read IP FQDN HOST SUBNET; do 
    CMD="sed -i 's/^127.0.0.1.*/127.0.0.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl hostname ${HOST}
done < machines.txt
```

Verify the hostname is set on each machine:

```
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt

```

```
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local

```

### Host Lookup Table

In this section I will generate a `hosts` file which will be appended to `/etc/hosts` file on `jumpbox` and to the `/etc/hosts` files on all three cluster members. This will allow each machine to be reachable using a hostname such as `server`, `node-0`, or `node-1`.

Create a new `hosts` file and add a header to identify the machines being added:

```
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```
Generate a host entry for each machine in the `machines.txt` file and append it to the `hosts` file:

```
while read IP FQDN HOST SUBNET; do 
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

Review the host entries in the `hosts` file:

```
cat hosts
```

```
root@jumpbox:~/kubernetes-the-hard-way# cat hosts

# Kubernetes The Hard Way
10.1.1.101 server.kubernetes.local server
10.1.1.102 node-0.kubernetes.local node-0
10.1.1.103 node-1.kubernetes.local node-1
```

### Adding `/etc/hosts` Entries To A Local Machine

In this section you will append the DNS entries from the `hosts` file to the local `/etc/hosts` file on your `jumpbox` machine.

Append the DNS entries from `hosts` to `/etc/hosts`:

```
cat hosts >> /etc/hosts
```

Verify that the `/etc/hosts` file has been updated:

```
cat /etc/hosts
```

```
root@jumpbox:~/kubernetes-the-hard-way# cat /etc/hosts
127.0.0.1       localhost
10.1.1.100      jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Kubernetes The Hard Way
10.1.1.101 server.kubernetes.local server
10.1.1.102 node-0.kubernetes.local node-0
10.1.1.103 node-1.kubernetes.local node-1
```

At this point I am be able to SSH to each machine listed in the machines.txt file using a hostname.

```
for host in server node-0 node-1
   do ssh root@${host} uname -o -m -n
done
```

```
server x86_64 GNU/Linux
node-0 x86_64 GNU/Linux
node-1 x86_64 GNU/Linux

```

### Adding /etc/hosts Entries To The Remote Machines

In this section I will append the host entries from `hosts` to `/etc/hosts` on each machine listed in the `machines.txt` text file.

Copy the `hosts` file to each machine and append the contents to `/etc/hosts`:

```
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

At this point hostnames can be used when connecting to machines from the `jumpbox` machine, or any of the three machines in the Kubernetes cluster. Instead of using IP addresses I can now connect to machines using a hostname such as `server`, `node-0`, or `node-1`.

Next: Provisioning a CA and Generating TLS Certificates