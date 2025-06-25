### Setup on debian 12 

To set up ansible on Debian 12
```bash
apt-get update
apt install python3-launchpadlib software-properties-common
apt install -y 
add-apt-repository --yes --update ppa:ansible/ansible
apt install -y ansible

Будет ошибкиn, пока игнорируем, потом разберемся
E: Репозиторий «https://ppa.launchpadcontent.net/ansible/ansible/ubuntu bookworm Release» не содержит файла Release.
N: Обновление из этого репозитория нельзя выполнить безопасным способом, поэтому по умолчанию он отключён.
N: Информацию о создании репозитория и настройках пользователя смотрите в справочной странице apt-secure(8)
```

### Добавление на Keenetic DNS хостов 

Connect to CLI

Use
```shell
ip host <доменное_имя_хоста> <адрес>
system configuration save

ip host k8s-master.k8s-cloud.test 192.168.1.200
ip host k8s-worker1.k8s-cloud.test 192.168.1.201
ip host k8s-worker1.k8s-cloud.test 192.168.1.202
system configuration save
```

Посмотреть все статические dns-записи команды ip host можно в системном файле конфигурации роутера startup-config.txt или по 
команде show dns-proxy (при выводе этой команды отображается много другой информации, помимо статических dns-записей).

see: https://help.keenetic.com/hc/ru/articles/360011129420-%D0%9E%D0%B1%D1%80%D0%B0%D1%89%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BA-%D1%81%D0%B5%D1%82%D0%B5%D0%B2%D0%BE%D0%BC%D1%83-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D1%83-%D0%BF%D0%BE-hostname

### First run
```bash
ssh root@192.168.1.200 'apt update && apt install -y python3 python3-apt' &&
ssh root@192.168.1.201 'apt update && apt install -y python3 python3-apt' &&
ssh root@192.168.1.202 'apt update && apt install -y python3 python3-apt'

ansible-playbook -i inventory/hosts.ini playbook.yml --check

ssh root@192.168.1.200 'reboot' &&
ssh root@192.168.1.201 'reboot' &&
ssh root@192.168.1.202 'reboot'
```

## Добавлен storage

Hostname: ```k8s-storage.k8s-cloud.test```

### Bash history
```bash
root@k8s-storage:~# history
    1  ls
    2  ip addr
    3  reboot
    4  apt update
    5  apt install linux-headers-amd64
    6  . /etc/os-release && echo "deb http://deb.debian.org/debian ${VERSION_CODENAME}-backports main contrib non-free non-free-firmware" | sudo tee /etc/apt/sources.list.d/backports.list >/dev/null && sudo apt update
    7  env /etc/os-release && echo "deb http://deb.debian.org/debian ${VERSION_CODENAME}-backports main contrib non-free non-free-firmware" | sudo tee /etc/apt/sources.list.d/backports.list >/dev/null && sudo apt update
    8  cAT /etc/os-release
    9  ccat /etc/os-release
   10  cat /etc/os-release
   11  env < /etc/os-release && echo "deb http://deb.debian.org/debian ${VERSION_CODENAME}-backports main contrib non-free non-free-firmware" | sudo tee /etc/apt/sources.list.d/backports.list >/dev/null && sudo apt update
   12  source /etc/os-release && echo "deb http://deb.debian.org/debian ${VERSION_CODENAME}-backports main contrib non-free non-free-firmware" | sudo tee /etc/apt/sources.list.d/backports.list >/dev/null && sudo apt update
   13  source /etc/os-release && echo "deb http://deb.debian.org/debian ${VERSION_CODENAME}-backports main contrib non-free non-free-firmware" | tee /etc/apt/sources.list.d/backports.list >/dev/null && apt update
   14  apt install -t stable-backports zfsutils-linux
   15  apt install -t stable-backports zfsutils-linux
   16  apt-get -f install
   17  update-ca-certificates
   18  lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
   19  zpool create tank /dev/sdb /dev/sdc /dev/sdd /dev/sde
   20  zpool remove tank
   21  zpool remove
   22  zpool
   23  zpool list
   24  zpool remove tank
   25  zfs set mountpoint=/mnt/storage tank
   26   zfs create tank/data
   27  df -hT | grep tank
   28  zfs set compression=lz4 tank/data
   29  zfs set atime=off tank/data
   30  mount
   31  mc
   32  top
   33  apt install -y nfs-kernel-server
   34  chmod 777 /tank/data/k8s
   35  mkdir -p /tank/data/k8s
   36  chmod 777 /tank/data/k8s
   37  echo "/tank/data/k8s 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)" >> /etc/exports
   38  exportfs -ra
   39  systemctl restart nfs-server
   40  apt install openssh-server
   41  nano /etc/ssh/sshd_config.d/99-root-login.conf
   42  service ssh restart
   43  service ssh restart
   44  poweroff
   45  poweroff
   46  cat /etc/exports
   47  ls -lash /tank/data/k8s/
   48  du -sh
   49  df -h
   50  hostname -a
   51  history
root@k8s-storage:~
```

```
root@k8s-storage:~# cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/tank/data/k8s 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
root@k8s-storage:~#
```

### Check NFS for all nodes
ssh root@192.168.1.200 'apt update && apt install -y nfs-common' &&
ssh root@192.168.1.201 'apt update && apt install -y nfs-common' &&
ssh root@192.168.1.202 'apt update && apt install -y nfs-common'

### Add to storage
ssh k8s-master.k8s-cloud.test

#### Install helm
```shell
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
#  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
#                                 Dload  Upload   Total   Spent    Left  Speed
#100 11913  100 11913    0     0  33774      0 --:--:-- --:--:-- --:--:-- 33843
#[WARNING] Could not find git. It is required for plugin installation.
#Downloading https://get.helm.sh/helm-v3.18.3-linux-amd64.tar.gz
#Verifying checksum... Done.
#Preparing to install helm into /usr/local/bin
#helm installed into /usr/local/bin/helm
#root@k8s-master:~#
apt install git -y
```

#### Add storage to cluster
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update

helm install nfs-csi nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=192.168.1.203 \
--set nfs.path=/mnt/storage/data \
--set storageClass.name=nfs-storage \
--set storageClass.reclaimPolicy=Retain

### Check
```shell
root@k8s-master:~# kubectl get storageclass
NAME          PROVISIONER                                             RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-storage   cluster.local/nfs-csi-nfs-subdir-external-provisioner   Retain          Immediate           true                   6m32s
```
