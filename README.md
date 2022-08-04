# alpine-k3s-awx

## Install k3s requirements

```

apk add curl make git bash
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
## Install AWX

```
AWX_HOST="awx-laptop.demo.local"
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out ./tls.crt -keyout ./base/tls.key -subj "/CN=${AWX_HOST}/O=${AWX_HOST}" -addext "subjectAltName = DNS:${AWX_HOST}"
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out ./tls.crt -keyout ./tls.key -subj "/CN=${AWX_HOST}/O=${AWX_HOST}" -addext "subjectAltName = DNS:${AWX_HOST}"
kubectl create secret tls -n awx tower-secret-tls --cert tls.crt --key tls.key
```
