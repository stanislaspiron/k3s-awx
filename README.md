# alpine-k3s-awx

## System requirements
- An Alpine linux 3 or later (tested with alpine-virt-3.16.0-x86_64.iso)
- At least 80G of disk space and 8G of memory are recommended
- An internet connection

## Install k3s requirements

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

Edit awx-operator/kustomization.yaml file and change operator version in resource URL and in image NewTag.

```
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=0.26.0             👈👈👈


# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 0.26.0                                                        👈👈👈

# Specify a custom namespace in which to install AWX
namespace: awx
```

Apply the installation

```
kubectl kustomize awx-operator | kubectl apply -f -
```

### Install AWX

Change current context to namespace awx:

```
kubectl config set-context --current --namespace=awx
```

Edit awx/kustomization.yaml file and change admin password and hostname.

```

```


Create Certificate for HTTPS

```
AWX_HOST="awx-k3s.demo.local"
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out ./awx/tls.crt -keyout ./awx/tls.key -subj "/CN=${AWX_HOST}/O=${AWX_HOST}" -addext "subjectAltName = DNS:${AWX_HOST}"

kubectl kustomize awx | kubectl apply -f -
```