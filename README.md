# k3s-multipass-on-mac-writeup
Writeup for installing K3s in a Multipass-vm, on a mac. 

based on: https://gist.github.com/ruanbekker/d939386cdb25cd663c9c553ef8a7ed8e

## Install Multipass on Mac

```
$ brew cask install multipass
```

## Create a SSH Key

```
$ sh-keygen -b 2048 -f ~/.ssh/multipass -t rsa -q -N ""
```

## Create a Cloud-Init Config

cloud-init.yaml:

```
#cloud-config
ssh_authorized_keys:
  - ssh-rsa AAA..the-output-of-multipass.pub.. user@host

runcmd:
- apt update && apt upgrade -y
```

## Multipass General Usage

Find a image:

```
$ multipass find
16.04 (or: x, xenial)
18.04 (or: b, bionic, default, lts)
```

Launch a VM with lts and passing the cloud-init:

```
$ multipass launch lts --name k3s-master --cloud-init=./cloud-init.yaml
```

View the VM's info:

```
$ multipass info k3s-master
Name:           k3s-master
State:          RUNNING
IPv4:           192.168.64.3
Release:        Ubuntu 24.04.3 LTS
Image hash:     6afb97af96b6 (Ubuntu 24.04 LTS)
Load:           0.37 0.62 0.33
Disk usage:     1.4G out of 4.7G
Memory usage:   144.2M out of 986.0M
```

SSH to the VM:

```
$ ssh -i ~/.ssh/multipass ubuntu@$(multipass info k3s-master | grep 'IPv4' | awk '{print $2}')
```

Exec to the VM:

```
$ multipass exec k3s-master -- /bin/bash
```

## K3sup

Install k3sup:

```
$ curl -sLSf https://get.k3sup.dev | sh
```

Install k3s to the master:

```
$ k3sup install --ip $(multipass info k3s-master | grep 'IPv4' | awk '{print $2}') --user ubuntu --ssh-key ~/.ssh/multipass
```

## Kubeconfig

Set your config in place:

```
$ export KUBECONFIG=4{HOME}/workspace/multipass-k3s/kubeconfig
```

and interact with kubernetes:

```
$ kubectl get node -o wide
NAME         STATUS   ROLES    AGE   VERSION         INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k3s-master   Ready    master   17s   v1.16.3-k3s.2   192.168.64.3   <none>        Ubuntu 18.04.3 LTS   4.15.0-72-generic   containerd://1.3.0-k3s.5
```



Delete the VM:

```
$ multipass delete k3s-master
$ multipass purge
```
