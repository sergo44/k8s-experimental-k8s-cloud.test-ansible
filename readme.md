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
ip host <доменное_имя_хоста> <адрес>
system configuration save

ip host k8b-master.k8b-cloud.test 192.168.1.200
ip host k8b-worker1.k8b-cloud.test 192.168.1.201
ip host k8b-worker1.k8b-cloud.test 192.168.1.202
system configuration save

Посмотреть все статические dns-записи команды ip host можно в системном файле конфигурации роутера startup-config.txt или по 
команде show dns-proxy (при выводе этой команды отображается много другой информации, помимо статических dns-записей).

see: https://help.keenetic.com/hc/ru/articles/360011129420-%D0%9E%D0%B1%D1%80%D0%B0%D1%89%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BA-%D1%81%D0%B5%D1%82%D0%B5%D0%B2%D0%BE%D0%BC%D1%83-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D1%83-%D0%BF%D0%BE-hostname

### First run

ssh root@192.168.1.200 'apt update && apt install -y python3 python3-apt' &&
ssh root@192.168.1.201 'apt update && apt install -y python3 python3-apt' &&
ssh root@192.168.1.202 'apt update && apt install -y python3 python3-apt'

ansible-playbook -i inventory/hosts.ini playbook.yml --check