# Kubernetes on Digital Ocean via Ansible

## Provides

Deployed k8s on a 3 node digital-ocean cluster using Ansible.

These manifests are based on the following guide:

https://www.digitalocean.com/community/tutorials/how-to-Create-a-kubernetes-1-10-cluster-using-kubeadm-on-ubuntu-16-04

## Pre-requisites
- ssh-keygen on local VM to generate public key
- cat /home/user/.ssh/id_rsa.pub to copy public key
- Installed ansible (`ansible-playbook`) on local VM
  - sudo apt update
  - sudo apt install software-properties-common
  - sudo add-apt-repository --yes --update ppa:ansible/ansible
  - sudo apt install ansible

- Created 3 droplets on digital-ocean (ubuntu 20.04 LTS/ 1gig ram)
- Used local VMpublic key in droplets setup.

### Configure hosts

Updated the `hosts` file with with the external IPs of droplets

- `<master-ip>`
- `<worker-ip1>`
- `<worker-ip2>`

### Run playbooks

If reproducing, the order of the commands are important.

Wait for each stage to complete before progressing to the next.


```
ansible-playbook -i hosts initial.yml
```

```
ansible-playbook -i hosts kube-deps.yml
```

```
ansible-playbook -i hosts master.yml
```

```
ansible-playbook -i hosts workers.yml
```

## Validate cluster

```
ssh ubuntu@<master-ip>
```

```
ububuntu@master:~$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
master     Ready     master    15m       v1.11.0
worker01   Ready     <none>    12m       v1.11.0
worker02   Ready     <none>    12m       v1.11.0
```

## Accessing locally

If you have `kubectl` installed on your laptop, you can copy over the remote kubeconfig from the master.

```
» mkdir -p ~/.kube/digital-ocean
» scp ubuntu@<master-ip>:.kube/config ~/.kube/digital-ocean/config
» export KUBECONFIG=~/.kube/digital-ocean/config

» kubectl get nodes
NAME       STATUS     ROLES     AGE       VERSION
master     Ready      master    1h        v1.11.0
worker01   Ready      <none>    1h        v1.11.0
worker02   Ready      <none>    1h        v1.11.0
```

## Adding more workers

To add more workers, simply update the `hosts.yml` file and add additional entries.

# Persistent Storage

Digital Ocean has a storage plugin that allows you to use DigitalOcean Block Storage with Kubernetes.

See: https://github.com/digitalocean/csi-digitalocean

This works really well and is easy to install:

1. Created a digital ocean API token
2. Generate the required kubernetes secret
3. Install `csi-digitalocean` manifests

# Security

I've not taken any additional measures over the default install via `kubeadm`.

## General

- There is no dashboard installed by default
- There is no `NetworkPolicy` in place
- There is no `PodSecurityPolicy` in place
- RBAC is enabled and configuration seems to have sensible defaults

## Relevant API Server Settings
```
./kube-apiserver --authorization-mode=Node,RBAC \
                 --allow-privileged=true \
                  --insecure-port=0 \
                  --secure-port=6443 \
                  ...
```

## Relevant Kubelet Settings
```
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
```
