# k3s-awx

Note: some parts of this solution are from:
- [awx-on-k3s](https://github.com/kurokobo/awx-on-k3s)
- [How To Install Ansible AWX on Ubuntu 20.04|18.04](https://computingforgeeks.com/how-to-install-ansible-awx-on-ubuntu-linux/)

## System requirements
- One of following Linux distributions.
  - ubuntu 20.04 LTS - [Ubuntu requirements](k3s-ubuntu-requirements.md)
  - ubuntu 22.04 LTS - [Ubuntu requirements](k3s-ubuntu-requirements.md)
  - ~~alpine-virt-3.16.0-x86_64.iso - [Alpine requirements](k3s-alpine-requirements.md)~~
- At least 80G of disk space and 8G of memory are recommended
- An internet connection

## Install k3s

```
curl -sfL https://get.k3s.io | sudo sh -s - --write-kubeconfig-mode 644
```


### Proxy Configuration
If k3s is behind a proxy, add following lines in **`/etc/systemd/system/k3s.service.env`**

```
HTTP_PROXY="http://192.168.10.1:3128“
HTTPS_PROXY="http://192.168.10.1:3128“
NO_PROXY="localhost,127.0.0.0/8,0.0.0.0,10.0.0.0/8,192.168.0.0/16,172.16.0.0/12,internal.example.com” 
```

## AWX Installation

### Download this repo
```
git clone https://github.com/stanislaspiron/k3s-awx.git
cd k3s-awx/
```

### Install AWX Operator

#### create awx-operator/kustomization.yaml file from template:

```
AWX_OPERATOR_VERSION=$(curl -s https://api.github.com/repos/ansible/awx-operator/releases/latest | awk -F '[",]' '/tag_name/{print $4}') envsubst < awx-operator/kustomization.yaml.tmpl > awx-operator/kustomization.yaml
```

#### Apply the installation

```
kubectl apply -k awx-operator
```

### Install AWX

#### Edit awx/kustomization.yaml file and change admin password and hostname.

```
AWX_HOST="awx-k3s.demo.local" envsubst < awx/awx.yml.tmpl > awx/awx.yml
```
* Note: Default password for user **admin** is **`admin@F5demo.com`**. You can change it in **awx/kustomization.yaml** file.

#### Create Certificate for HTTPS

```
export AWX_HOST="awx-k3s.demo.local"
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out ./awx/tls.crt -keyout ./awx/tls.key -subj "/CN=${AWX_HOST}/O=${AWX_HOST}" -addext "subjectAltName = DNS:${AWX_HOST}"
```

#### Apply AWX installation

```
kubectl apply -k awx
```

### Usefull operations

Change current context to namespace awx:

```
kubectl config set-context --current --namespace=awx
```
