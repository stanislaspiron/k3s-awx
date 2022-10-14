# k3s-awx

## System requirements
- One of following Linux distributions.
  - alpine-virt-3.16.0-x86_64.iso
  - ubuntu 20.04 LTS
- At least 80G of disk space and 8G of memory are recommended
- An internet connection

### Alpine Linux - Install k3s requirements

Install packages requirements
```
apk add curl git bash
```
Configure Alpine Linux to support k3s

```
echo "cgroup /sys/fs/cgroup cgroup defaults 0 0" >> /etc/fstab
cat > /etc/cgconfig.conf <<EOF
mount {
  cpuacct = /cgroup/cpuacct;
  memory = /cgroup/memory;
  devices = /cgroup/devices;
  freezer = /cgroup/freezer;
  net_cls = /cgroup/net_cls;
  blkio = /cgroup/blkio;
  cpuset = /cgroup/cpuset;
  cpu = /cgroup/cpu;
}
EOF
sed -i 's/default_kernel_opts="pax_nouderef quiet rootfstype=ext4"/default_kernel_opts="pax_nouderef quiet rootfstype=ext4 cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory"/g' /etc/update-extlinux.conf
update-extlinux
reboot
```

### Ubuntu - Install k3s requirements

Install packages requirements
```
apt install git
```

## Install k3s

```
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
```

## AWX Installation

### Download this repo
```
git clone https://github.com/stanislaspiron/alpine-k3s-awx.git
cd alpine-k3s-awx/
```

### Install AWX Operator

#### create awx-operator/kustomization.yaml file from template:

```
export AWX_OPERATOR_VERSION=0.30.0
envsubst < awx-operator/kustomization.yaml.tmpl > awx-operator/kustomization.yaml
```

#### Apply the installation

```
kubectl apply -k awx-operator
```

### Install AWX

#### Edit awx/kustomization.yaml file and change admin password and hostname.

```
export AWX_HOST="awx-k3s.demo.local"
envsubst < awx/awx.yml.tmpl > awx/awx.yml
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
