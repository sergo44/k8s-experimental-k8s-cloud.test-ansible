# Создаем образ Debian 12 c 5-ю дисками, устанавливаем на первую машину базовую систему

## Поставим IP
```shell
telnet 192.168.1.1
ip host k8s-storage.k8s-cloud.test 192.168.1.203
system configuration save
```


# Активация SSHD
```shell
apt install opensssh-server
PermitRootLogin yes
service ssh restart
```

# Login via ssh
```shell
ssh-copy-id k8s-storage.k8s-cloud.test
```

# Setup ZFS
```shell
ssh k8s-storage.k8s-cloud.test
apt update
apt install linux-headers-amd64
source /etc/os-release && echo "deb http://deb.debian.org/debian ${VERSION_CODENAME}-backports main contrib non-free non-free-firmware" | tee /etc/apt/sources.list.d/backports.list >/dev/null && apt update
apt install -t stable-backports zfsutils-linux
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
```

```text
root@k8s-storage:~# lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
NAME                         SIZE TYPE MOUNTPOINT
sda                           20G disk
├─sda1                       487M part /boot
├─sda2                         1K part
└─sda5                      19,5G part
├─k8s--storage--vg-root   18,6G lvm  /
└─k8s--storage--vg-swap_1  980M lvm  [SWAP]
sdb                           20G disk
sdc                           20G disk
sdd                           20G disk
sde                           20G disk
sr0                         1024M rom
```

# Create ZFS pool
```shell
zpool create tank /dev/sdb /dev/sdc /dev/sdd /dev/sde
zfs create tank/data
zfs set compression=lz4 tank/data
zfs set atime=off tank/data
```

# Share it via NFS
```shell
apt install -y nfs-kernel-server
mkdir -p /tank/data/k8s
chmod 777 /tank/data/k8s
echo "/tank/data/k8s 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)" >> /etc/exports
exportfs -ra
systemctl restart nfs-server
```

# Fix 

WARN[0000] runtime connect using default endpoints: [unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
WARN[0000] image connect using default endpoints: [unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.

```shell
cat <<EOF | tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
EOF
```

crictl ps -a | grep kube-apiserver
bf3af139484b1       e60416bd78b65       Less than a second ago   Exited              kube-apiserver            53                  a38eeb1ad9e5a       kube-apiserver-k8s-master
crictl logs bf3af139484b1

# On master
```shell
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm repo update
helm install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.1.203 \
  --set nfs.path=/tank/data/k8s \
  --set storageClass.name=zfs-nfs \
  --set storageClass.defaultClass=true
```
