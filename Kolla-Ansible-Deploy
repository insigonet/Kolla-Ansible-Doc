```markdown
# Развертывание OpenStack с использованием Kolla-Ansible на двух серверах

Данное руководство описывает шаги по развертыванию OpenStack на двух физических серверах Supermicro AS-1115HS-TNR 9374F с использованием Kolla-Ansible. Цель состоит в создании производственной среды с высокой доступностью (HA) для развертывания виртуальных машин под управлением Linux.

## Требования

### Оборудование:
- Серверы: Supermicro AS-1115HS-TNR 9374F
- RAM: 12x64Gb DDR5
- Системный диск: MZQL215THBLA-00A07
- Диски для хранения данных: 2x MZQL215THBLA-00A07
- Сетевая карта: AOC-A25G-m2SM-O

### Сети:
- Management VLAN 2059, 10.64.92.0/24 GW: 10.64.92.254
- API VLAN 2058, 10.64.91.0/24
- Storage VLAN 2057, 10.64.90.0/24
- Сеть для клиентских IP VLAN 3536, 185.112.41.192/27

# Подготовка к установке
## Обновление и установка пакетов [node-01]
```

### Обновление и установка пакетов
Добавьте проверку версии установленных пакетов.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-dev python3-pip python3-venv libffi-dev gcc libssl-dev git
# Проверка установки пакетов
dpkg -l | grep -E 'python3-dev|python3-pip|python3-venv|libffi-dev|gcc|libssl-dev|git'
```

### Виртуальная среда и установка Ansible
Добавьте проверку версии Ansible.

```bash
# Создание виртуальной среды, установка зависимостей и Ansible [node-01]
mkdir ~/kolla-venv
python3 -m venv ~/kolla-venv
source ~/kolla-venv/bin/activate
pip install -U pip
pip install 'ansible-core>=2.15,<2.16.99'
# Проверка установки Ansible
ansible --version
```

### Установка Kolla-Ansible
Добавьте команду проверки установки Kolla-Ansible.

```bash
# Установка Kolla-Ansible [node-01]
pip install git+https://opendev.org/openstack/kolla-ansible@master
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
cp -r ~/kolla-venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp ~/kolla-venv/share/kolla-ansible/ansible/inventory/* .
kolla-ansible install-deps
# Проверка установки Kolla-Ansible
kolla-ansible --version
```

### Настройка Ansible
Добавьте команду проверки конфигурации.

```bash
# Настройка Ansible [node-01]
mkdir -p /etc/ansible/
cat << EOF | sudo tee /etc/ansible/ansible.cfg
[defaults]
host_key_checking=False
pipelining=True
forks=100
EOF
# Проверка конфигурации Ansible
ansible-config dump | grep -E 'host_key_checking|pipelining|forks'
```

### Генерация паролей
Добавьте команду проверки наличия сгенерированных паролей.

```bash
# Генерация паролей [node-01]
kolla-genpwd
# Проверка наличия сгенерированных паролей
if [ -f /etc/kolla/passwords.yml ]; then
    echo "Пароли успешно сгенерированы."
else
    echo "Ошибка генерации паролей."
fi
# Посмотреть пароль можно в файле в keystone_admin_password:
nano /etc/kolla/passwords.yml
```

### Настройка файла hosts
Добавьте проверку файла hosts.

```bash
# Настройка файла hosts [node-01 и node-02]
sudo bash -c 'cat << EOF > /etc/hosts
127.0.0.1 localhost
# vip
10.64.92.50 os.cloud.ldc.net
# internal
10.64.92.51 node-01
10.64.92.52 node-02
EOF'
# Проверка файла hosts
cat /etc/hosts
```

### Автоматический прием новых ключей
Добавьте проверку конфигурации SSH.

```bash
# Автоматически прием новых ключей [node-01 и node-02]
sudo bash -c 'cat << EOF > ~/.ssh/config
Host *
StrictHostKeyChecking accept-new
EOF'
# Проверка конфигурации SSH
cat ~/.ssh/config
```

### Создание и распространение SSH ключей
Добавьте проверку успешного копирования ключей.

```bash
# Создание и распространение SSH ключей [node-01]
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub root@node-01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@node-02
# Проверка копирования ключей
ssh root@node-01 'echo "SSH доступ настроен для node-01"'
ssh root@node-02 'echo "SSH доступ настроен для node-02"'
```

### Подготовка дисков
Добавьте команды проверки состояния дисков.

```bash
# Подготовка дисков на серверах storage [node-01 и node-02]
sudo fdisk -l
lsblk
sudo pvcreate /dev/nvme0n1 /dev/nvme1n1
sudo vgcreate cinder-volumes /dev/nvme0n1 /dev/nvme1n1
# Проверка состояния дисков
sudo pvs
sudo vgs
sudo lvs
```

### Настройка globals.yml
Добавьте проверку конфигурации.

```bash
# Редактирование /etc/kolla/globals.yml [node-01]
cp /etc/kolla/globals.yml{,.bak}
cat << EOF | sudo tee /etc/kolla/globals.yml
---
workaround_ansible_issue_8743: yes
## Kolla options
kolla_base_distro: "ubuntu"
openstack_release: "master"

## Networking Options
kolla_internal_vip_address: "10.64.92.100"
neutron_external_interface: "enp1.3536"
api_interface: "enp0.2057"
storage_interface: "enp0.2058"
network_interface: "enp0.2057"
enable_haproxy: "yes"
enable_neutron_provider_networks: "yes"

## Switch
neutron_plugin_agent: "openvswitch"

## OpenStack services
enable_cinder: "yes"
enable_cinder_backend_lvm: "yes"
nova_compute_virt_type: "kvm"
EOF
# Проверка конфигурации globals.yml
cat /etc/kolla/globals.yml | grep -E 'kolla_base_distro|openstack_release|kolla_internal_vip_address|neutron_external_interface'
```

### Настройка файла inventory multinode
Добавьте проверку файла inventory.

```bash
# Редактирование файла inventory multinode [Deploy]
cat << EOF | sudo tee ~/multinode
[control]
node-01
node-02
[network]
node-01
node-02
[compute]
node-01
node-02
[storage]
node-01
node-02
[monitoring]
node-01
node-02
EOF
# Проверка файла inventory
cat ~/multinode
```

### Развертывание OpenStack
Добавьте команды проверки на каждом этапе развертывания.

```bash
# Развертывание [Deploy]
ansible -i multinode all -m ping
# Проверка установки зависимостей
kolla-ansible install-deps
pip list | grep kolla

# Деплой и проверка bootstrap
kolla-ansible -i ./multinode bootstrap-servers
kolla-ansible -i ./multinode bootstrap-servers --tags bootstrap

# Деплой и проверка prechecks
kolla-ansible -i ./multinode prechecks
kolla-ansible -i ./multinode prechecks --tags prechecks

# Деплой и проверка деплоя
kolla-ansible -i ./multinode deploy
kolla-ansible -i ./multinode deploy --tags deploy

# Создание admin-openrc
kolla-ansible post-deploy
source /etc/kolla/admin-openrc.sh
```

### Проверка работоспособности
Добавьте команды для проверки работоспособности развернутой системы.

```bash
# Проверка работоспособности
source /etc/kolla/admin-openrc.sh
openstack service list
openstack compute service list
openstack network agent list
```

### Полезные ссылки
Добавьте раздел с полезными ссылками.

```markdown
## Полезные ссылки

- [Официальная документация Kolla-An

sible](https://docs.openstack.org/kolla-ansible/latest/)
- [Руководство пользователя Kolla-Ansible](https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html)
```
