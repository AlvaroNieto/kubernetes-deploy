## Smoke Test
In this lab I will complete a series of tasks to ensure the Kubernetes cluster is functioning correctly.

### Data Encryption

In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Create a generic secret:

```
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

Print a hexdump of the `kubernetes-the-hard-way secret` stored in etcd:

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a d0 29 68 fc 0e 92 be  |:v1:key1:.)h....|
00000050  9e 22 d8 aa 5a 38 ff 9d  41 71 f4 d1 54 20 c9 9e  |."..Z8..Aq..T ..|
00000060  24 29 a9 01 c6 8b e9 52  7e 95 3e 45 06 89 a4 3a  |$).....R~.>E...:|
00000070  ea cc ab 42 36 6d 6c 04  da 7f 68 d2 6f af 3b 69  |...B6ml...h.o.;i|
00000080  cd 2e f8 09 3f 10 3c 50  c2 63 b9 af d2 ac 21 ea  |....?.<P.c....!.|
00000090  f4 ab 1d e1 b9 3a 12 e0  90 78 71 c2 41 54 57 81  |.....:...xq.ATW.|
000000a0  ca d9 c6 14 3e 01 c1 a4  dd ab 1f 41 b0 93 79 27  |....>......A..y'|
000000b0  66 0a 93 55 37 f9 80 a9  e4 96 65 97 e3 9e 01 df  |f..U7.....e.....|
000000c0  6f 59 a5 9c 36 bd 5a 19  21 24 e1 5c 07 16 b4 a2  |oY..6.Z.!$.\....|
000000d0  ac 4a 3b 57 41 55 6c a3  46 9b bc bb 76 a5 d2 f7  |.J;WAUl.F...v...|
000000e0  62 f2 25 7c 98 8a 70 9a  80 a7 37 75 52 3b d4 6a  |b.%|..p...7uR;.j|
000000f0  30 5c a0 4e fc c2 52 27  d3 a5 ad 5d 8c d0 7b 22  |0\.N..R'...]..{"|
00000100  72 ba a4 3b 78 33 13 12  07 16 90 f6 86 51 ae 3a  |r..;x3.......Q.:|
00000110  90 62 a0 90 0f b0 8a d7  fe 99 96 57 ae 97 43 fc  |.b.........W..C.|
00000120  81 80 37 16 ee bd 5f c2  c5 3b 2d 7c 6d f3 8f 2a  |..7..._..;-|m..*|
00000130  3f 2a 78 c0 43 c2 e3 80  bc 04 00 49 ae 33 75 93  |?*x.C......I.3u.|
00000140  15 6a 51 50 d2 38 fa 03  ba a8 c8 43 c5 47 bc d3  |.jQP.8.....C.G..|
00000150  2b c0 b6 9f e9 f7 93 0d  1b 0a                    |+.........|
0000015a
```

The etcd key should be prefixed with `k8s:enc:aescbc:v1:key1`, which indicates the `aescbc` provider was used to encrypt the data with the `key1` encryption key.

### Deployments

In this section I will verify the ability to create and manage Deployments.

Create a deployment for the nginx web server:

```
kubectl create deployment nginx \
  --image=nginx:latest
```

List the pod created by the `nginx` deployment:

```
kubectl get pods -l app=nginx
```

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56fcf95486-ht848   1/1     Running   0          26s
```

### Port Forwarding
In this section you will verify the ability to access applications remotely using port forwarding.

Retrieve the full name of the `nginx` pod:

```
POD_NAME=$(kubectl get pods -l app=nginx \
  -o jsonpath="{.items[0].metadata.name}")
```

Forward port `8080` on your local machine to port 80 of the `nginx` pod:

```
kubectl port-forward $POD_NAME 8080:80
```

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```
curl --head http://127.0.0.1:8080
```

```
root@jumpbox:~# curl --head http://127.0.0.1:8080
HTTP/1.1 200 OK
Server: nginx/1.27.3
Date: Mon, 09 Dec 2024 14:20:09 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 26 Nov 2024 15:55:00 GMT
Connection: keep-alive
ETag: "6745ef54-267"
Accept-Ranges: bytes
```

Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

#### Logs

In this section I will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Print the nginx pod logs:

```
kubectl logs $POD_NAME
```

```
...
2024/12/09 14:18:06 [notice] 1#1: start worker process 29
::1 - - [09/Dec/2024:14:20:09 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.88.1" "-"
```


#### Exec
In this section I will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -`v command in the `nginx` container:

```
kubectl exec -ti $POD_NAME -- nginx -v
```

```
nginx version: nginx/1.27.3
```

### Services

In this section I will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the nginx deployment using a NodePort service:

```
kubectl expose deployment nginx \
  --port 80 --type NodePort
```

The LoadBalancer service type can not be used because the cluster is not configured with cloud provider integration. Setting up cloud provider integration is out of scope for this tutorial.

Retrieve the node port assigned to the `nginx` service:

```
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Make an HTTP request using the IP address and the `nginx` node port:

```
curl -I http://node-0:${NODE_PORT}
```

```
HTTP/1.1 200 OK
Server: nginx/1.27.3
Date: Mon, 09 Dec 2024 14:35:37 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 26 Nov 2024 15:55:00 GMT
Connection: keep-alive
ETag: "6745ef54-267"
Accept-Ranges: bytes
```

Finish!