# Requirements

This lab is going to be deployed in a local enviroment with Hyper-V and 3 Debian 12 VMs.

This is going to be the setup be their setup

| Name | Description | CPU | RAM | Storage | Network |
|---|---|---|---|---|---|
| pfsense | Router with WAN access | 1 | 512MB | 10GB | 10.1.1.1/24 | 
| jumpbox | Administration host | 1 | 512MB | 10GB | 10.1.1.100 | 
| server | Kubernetes server | 1 | 2GB | 20GB | 10.1.1.101 |
| node-0 | Kubernetes worker node | 1 | 2GB | 20GB | 10.1.1.102 |
| node-1 | Kubernetes worker node | 1 | 2GB | 20GB | 10.1.1.103 |

Next: [Jumpbox](https://github.com/AlvaroNieto/kubernetes-deploy/blob/main/docs/02-jumpbox.md)