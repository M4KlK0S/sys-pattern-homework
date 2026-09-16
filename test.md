## Задание 1. Развертывание инфраструктуры в Yandex Cloud

Повторить демонстрацию лекции (развернуть vpc, 2 веб сервера, бастион сервер).

## Решение 1

**Проверка времени (обязательно!)**
```bash
date -u
chronyc tracking | grep 'System time'
```
Если System time показывает большое число секунд (например, 15572 sec slow) — часы отстают, и yc/Terraform будут падать с JWT already expired.

Синхронизация времени:
```bash
sudo systemctl restart chronyd
sudo chronyc -a makestep
```
Проверка:
```bash
chronyc tracking | grep 'System time'
```
Убедитесь, что системная дата правильная
```bash
date -u
```
**должно быть: 0.000000000 seconds slow**
<img width="708" height="233" alt="image" src="https://github.com/user-attachments/assets/f426f807-d218-4296-a799-d13b61ad9749" />

> [!IMPORTANT]
> Если этого не сделать — yc и Terraform будут падать с JWT already expired, даже если ключ валидный.

**1. Установка инструментов**

**1.1. Установка Yandex Cloud CLI (yc)**

На управляющей ВМ (Ubuntu) выполните:
```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```
После установки перезагрузите shell:
```bash
exec -l $SHELL
```
Проверьте:
```bash
yc version
```
**1.2. Установка Terraform**
```bash
sudo apt update
sudo apt install -y wget unzip

wget https://hashicorp-releases.yandexcloud.net/terraform/1.9.8/terraform_1.9.8_linux_amd64.zip
unzip terraform_1.9.8_linux_amd64.zip
sudo mv terraform /usr/local/bin/

terraform version
```
**2. Аутентификация в Yandex Cloud**

> [!WARNING]
> С 1 июня 2026 года Yandex Cloud не принимает OAuth-токены, выданные через Яндекс ID после этой даты. Команда yc init в браузере отработает, но при попытке yc resource-manager cloud list вы получите ошибку:

```text
OAuth token for user '...', issued after '2026-06-01', is not supported for IAM token exchange
```
Поэтому аутентификацию делаем через сервисный аккаунт — этот способ работает и с Terraform, и с yc.

**2.1. Принятие пользовательского соглашения**
Зайдите в console.yandex.cloud под своим аккаунтом. При первом входе появится окно с пользовательским соглашением — примите его. Без этого yc не сможет выпустить IAM-токен:
```text
User has to accept the End User License Agreement and Privacy Policy
```
**2.2. Создание сервисного аккаунта через консоль**
- Откройте console.yandex.cloud → Identity and Access Management → Сервисные аккаунты.
<img width="1318" height="886" alt="image" src="https://github.com/user-attachments/assets/9ce1352e-778b-4e5f-97ba-dc830c0f9540" />

- Нажмите «Создать сервисный аккаунт».
<img width="1906" height="491" alt="image" src="https://github.com/user-attachments/assets/299e72bc-1a5f-4014-a841-455329b1693d" />

- Имя: terraform-sa.

- На вкладке «Права доступа» добавьте роль admin и editor на каталог default.
<img width="867" height="457" alt="image" src="https://github.com/user-attachments/assets/c2be1059-a28c-4571-b0e5-4da6680d41b2" />

- Создайте.

**2.3. Создание авторизованного ключа**
В карточке terraform-sa → вкладка «Ключи».

«Создать новый ключ» → «Создать авторизованный ключ».
<img width="1040" height="297" alt="image" src="https://github.com/user-attachments/assets/33debae0-f1f9-4834-a8a8-d9c5ea7b722f" />

Алгоритм: RSA_2048.
<img width="566" height="263" alt="image" src="https://github.com/user-attachments/assets/ed5608e6-6397-4c50-83e3-2ee93d47b7df" />

Скачайте файл key.json — он показывается только один раз.

> [!CAUTION]
> key.json — секретный файл. Внутри него приватный ключ сервисного аккаунта. Не коммитьте его в Git, не показывайте в скриншотах, не пересылайте в открытых чатах. При утечке — отзовите ключ в консоли и создайте новый.

**2.4. Копирование key.json с Windows на ВМ**

Если ваша управляющая ВМ — это VirtualBox с проброшенным портом (127.0.0.1:2222), из PowerShell:
```powershell
cd $HOME\Downloads
scp -P 2222 .\authorized_key.json kostenkomp@127.0.0.1:/home/kostenkomp/key.json
```
Затем на ВМ:
```bash
sudo mkdir -p /opt/terraform-yc
sudo  -R $USER:$USER /opt/terraform-yc
sudo mv /home/kostenkomp/key.json /opt/terraform-yc/key.json
chmod 600 /opt/terraform-yc/key.json
```
Проверьте, что файл — валидный JSON:
```bash
head -1 /opt/terraform-yc/key.json     # должно начинаться с {
python3 -m json.tool /opt/terraform-yc/key.json > /dev/null && echo "JSON OK"
```
> [!IMPORTANT]
> Частая ошибка: скопировать из окна создания ключа только PEM-блок -----BEGIN PRIVATE KEY----- вместо всего JSON. Terraform тогда упадёт с ошибкой key unmarshal fail: invalid character 'P'. Всегда используйте кнопку «Скачать файл» — она отдаёт корректный JSON.

**3. Настройка yc через сервисный аккаунт**
Раз OAuth-токен не работает, настройте yc на тот же key.json:
```bash
yc config profile create sa-profile
yc config profile activate sa-profile
yc config set service-account-key /opt/terraform-yc/key.json
yc config set cloud-id <ваш-cloud-id>
yc config set folder-id <ваш-folder-id>
yc config set compute-default-zone ru-central1-a
```
Где взять cloud-id и folder-id: в консоли Yandex Cloud вверху страницы — селекторы облака (cloud-cumulus-388) и каталога (default). Под ними мелким шрифтом отображаются ID, начинающиеся с b1g....

Проверка:
```bash
yc config list
```
**4. Создание SSH-ключа**

Если ключа ещё нет:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_yc
```
Публичный ключ понадобится для доступа к ВМ. Проверьте:
```bash
ls -la ~/.ssh/id_rsa_yc ~/.ssh/id_rsa_yc.pub
```
**5. Создание файла конфигурации Terraform**

Создайте файл main.tf:
```bash
nano /opt/terraform-yc/main.tf
```
Содержимое файла:
```hcl
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  service_account_key_file = "key.json"
  cloud_id                 = "<ваш-cloud-id>"
  folder_id                = "<ваш-folder-id>"
  zone                     = "ru-central1-a"
}

# Сеть
resource "yandex_vpc_network" "net" {
  name = "hw-network"
}

# NAT-шлюз
resource "yandex_vpc_gateway" "nat_gateway" {
  name = "hw-nat-gateway"
  shared_egress_gateway {}
}

# Таблица маршрутизации через NAT-шлюз
resource "yandex_vpc_route_table" "nat" {
  network_id = yandex_vpc_network.net.id
  name       = "hw-nat-route"

  static_route {
    destination_prefix = "0.0.0.0/0"
    gateway_id         = yandex_vpc_gateway.nat_gateway.id
  }
}

# Подсеть с привязанной таблицей маршрутизации
resource "yandex_vpc_subnet" "subnet" {
  name           = "hw-subnet"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.net.id
  v4_cidr_blocks = ["192.168.10.0/24"]
  route_table_id = yandex_vpc_route_table.nat.id
}

# Образ Ubuntu 22.04
data "yandex_compute_image" "ubuntu" {
  family = "ubuntu-2204-lts"
}

# Бастион-сервер (публичный IP)
resource "yandex_compute_instance" "bastion" {
  name        = "bastion"
  platform_id = "standard-v3"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa_yc.pub")}"
  }
}

# Веб-сервер A (только внутренний IP, выход в интернет через NAT-шлюз)
resource "yandex_compute_instance" "web_a" {
  name        = "web-a"
  platform_id = "standard-v3"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = false
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa_yc.pub")}"
  }
}

# Веб-сервер B (только внутренний IP, выход в интернет через NAT-шлюз)
resource "yandex_compute_instance" "web_b" {
  name        = "web-b"
  platform_id = "standard-v3"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = false
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa_yc.pub")}"
  }
}

# Вывод IP-адресов
output "bastion_public_ip" {
  value = yandex_compute_instance.bastion.network_interface.0.nat_ip_address
}

output "web_a_internal_ip" {
  value = yandex_compute_instance.web_a.network_interface.0.ip_address
}

output "web_b_internal_ip" {
  value = yandex_compute_instance.web_b.network_interface.0.ip_address
}
```
> [!IMPORTANT]
> Необходимо заполнить:
```bash
provider "yandex" {
  service_account_key_file = "key.json"
  cloud_id                 = "<ваш-cloud-id>"
  folder_id                = "<ваш-folder-id>"
  zone                     = "ru-central1-a"
}
```
> [!NOTE]
> Токен в коде — плохо. Если у вас в main.tf есть что-то вроде variable "folder_id" { default = "y0__..." } — это не folder_id, а OAuth-токен, попавший туда по ошибке. Удалите его. Всё, что нужно для аутентификации, лежит в key.json.

**6. Инициализация и применение Terraform**
```bash
cd /opt/terraform-yc
terraform init
```
Вывод должен показать, что провайдер yandex-cloud/yandex успешно установлен:
<img width="1419" height="832" alt="image" src="https://github.com/user-attachments/assets/367fad37-624e-4bff-aaa5-836d98cfe7f6" />

```bash
terraform plan
```
Terraform покажет, что создаст 5 ресурсов: сеть, подсеть, 3 ВМ (бастион + 2 веб-сервера).
<img width="535" height="152" alt="image" src="https://github.com/user-attachments/assets/bc8a7fb8-1fa3-4937-a8d7-0ce1ec91efbd" />

> Если plan падает с ошибкой. Значит необходимо проверить время в системе и в случае необходимости синхронизировать время как показано было вначале.

```bash
terraform apply
```
Введите yes для подтверждения. Создание займёт 1–3 минуты.

> [!IMPORTANT]
> Платёжный аккаунт должен быть привязан. Без него Yandex Cloud не даст создать ни одной ВМ, даже если все права выданы. Проверьте в правом верхнем углу консоли — не должно быть надписи «Платёжный аккаунт не привязан».

**7. Проверка созданных ресурсов**
После успешного apply Terraform выведет IP-адреса:
```text
bastion_public_ip = "bastion-ip-address"
web_a_internal_ip = "web_a-ip-address"
web_b_internal_ip = "web_b-ip-address"
```
Сделайте скриншот этого вывода для отчёта.

Также проверьте через CLI:
```bash
yc compute instance list
```
Если yc падает с ошибкой аутентификации — используйте профиль sa-profile:
```bash
yc config profile activate sa-profile
yc compute instance list
```
Или зайдите в консоль: console.yandex.cloud → Compute Cloud → Виртуальные машины. Там должны быть видны bastion, web-a, web-b в статусе RUNNING. Сделайте скриншот и его тоже.

**8. Настройка SSH-доступа через бастион**

**8.1. Создайте файл ~/.ssh/config**
```bash
nano ~/.ssh/config
```
Содержимое (замените IP на реальные из вывода Terraform):
```sshconfig
Host bastion
  HostName bastion-ip-address
  User ubuntu
  IdentityFile ~/.ssh/id_rsa_yc

Host web-a
  HostName web_a-ip-address
  User ubuntu
  IdentityFile ~/.ssh/id_rsa_yc
  ProxyJump bastion

Host web-b
  HostName web_b-ip-address
  User ubuntu
  IdentityFile ~/.ssh/id_rsa_yc
  ProxyJump bastion
```
> [!NOTE]
> Если ключ ~/.ssh/id_rsa_yc защищён парольной фразой — добавьте его в SSH-агент:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa_yc
```
Без агента придётся вводить пароль при каждом подключении. Если ключ без пароля — агент не нужен, IdentityFile в ~/.ssh/config работает напрямую.

Чтобы агент запускался автоматически, добавьте в ~/.bashrc:
```bash
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)" > /dev/null
    ssh-add ~/.ssh/id_rsa_yc 2>/dev/null
fi
```
**8.2. Проверьте подключение к бастиону**
```bash
ssh bastion
hostname
# Синхронизация времени:
sudo systemctl restart chronyd
sudo chronyc -a makestep
exit
```
Теперь проверьте веб-серверы:
```bash
ssh web-a
hostname
# Синхронизация времени:
sudo systemctl restart chronyd
sudo chronyc -a makestep
exit
```
```bash
ssh web-b
hostname
# Синхронизация времени:
sudo systemctl restart chronyd
sudo chronyc -a makestep
exit
```
Должны увидеть web-a и web-b соответственно.

## Задание 2. Развертывание инфраструктуры в Yandex Cloud

1. С помощью ansible подключиться к web-a и web-b , установить на них nginx.(написать нужный ansible playbook)
2. Провести тестирование и приложить скриншоты развернутых в облаке ВМ, успешно отработавшего ansible playbook.

## Решение 2

**1. Создание рабочей директории для Ansible**
```bash
sudo mkdir -p /opt/ansible-yc
sudo  -R $USER:$USER /opt/ansible-yc
cd /opt/ansible-yc
```
**2. Создание inventory-файла**

Inventory-файл описывает, к каким хостам Ansible будет подключаться и как именно.
```bash
nano /opt/ansible-yc/inventory.ini
```
Содержимое (замените IP-адреса на реальные из вывода terraform apply):
```ini
[web]
web-a ansible_host=web_a-ip-address ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa_yc
web-b ansible_host=web_b-ip-address ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa_yc

[web:vars]
ansible_ssh_common_args='-o ProxyJump=ubuntu@bastion-ip-address -o StrictHostKeyChecking=no -o IdentitiesOnly=yes'
```
> [!IMPORTANT]
> Параметр ansible_ssh_common_args с ProxyJump говорит Ansible подключаться к веб-серверам через бастион, потому что у них нет публичных IP-адресов.
> IdentitiesOnly=yes заставляет использовать только указанный ключ и не перебирать всё, что лежит в SSH-агенте — это устраняет часть ошибок с Connection closed.

- bastion_public_ip — публичный IP бастиона;
- web_a_internal_ip — внутренний IP web-a;
- web_b_internal_ip — внутренний IP web-b.

**3. Проверка связи через Ansible**

Убедитесь, что Ansible видит оба веб-сервера и может до них достучаться.
```bash
cd /opt/ansible-yc
ansible -i inventory.ini web -m ping
```
> [!WARNING]
> Если вывод содержит UNREACHABLE — сначала проверьте ручное подключение: ssh web-a. Если оно работает, а Ansible — нет, запустите с диагностикой:

```bash
ansible -i inventory.ini web -m ping -vvv
```
Это покажет, на каком этапе ломается подключение.

**4. Создание плейбука для Nginx**

Playbook — это YAML-файл, описывающий, что Ansible должен сделать на удалённых хостах.
```bash
nano /opt/ansible-yc/nginx.yml
```
Содержимое:
```yaml
- name: Install and Configure Nginx
  hosts: web
  become: yes

  tasks:
    - name: Install Nginx package
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Ensure Nginx is started and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes
```
Разбор плейбука:
```text
Строка	                  Что делает
hosts:                    web	Применяется к группе web из inventory (web-a и web-b)
become:                   yes	Выполняет задачи от имени root (через sudo)
ansible.builtin.package	  Устанавливает пакет nginx
ansible.builtin.service	  Запускает Nginx и добавляет его в автозагрузку
```
**5. Запуск плейбука и решение проблемы с отсутствием интернета**

**5.1. Запуск плейбука**
```bash
cd /opt/ansible-yc
ansible-playbook -i inventory.ini nginx.yml
```
Ожидаемый вывод при успешном выполнении:
<img width="1884" height="585" alt="image" src="https://github.com/user-attachments/assets/4e35d353-e1b0-47c1-b3f7-07e5b60c2693" />

**5.2. Проверка работы Nginx**
Зайдите на бастион и оттуда проверьте веб-серверы:
```bash
ssh bastion
curl -I http://web_a-ip-address/
curl -I http://web_b-ip-address/
exit
```
> [!NOTE]
> Подставьте актуальные внутренние IP-адреса веб-серверов из вывода terraform apply.

Ожидаемый результат:

<img width="548" height="403" alt="image" src="https://github.com/user-attachments/assets/b2f6731f-384f-4627-9a09-327ef2353938" />

## Задание 3*. Развертывание инфраструктуры в Yandex Cloud

1. Добавить еще одну виртуальную машину.
2. Установить на нее любую базу данных.
3. Выполнить проверку состояния запущенных служб через Ansible.

## Решение 3*

**1. Создание ВМ web_c для развёртывания БД**

Редактирование main.tf:
```bash
nano /opt/terraform-yc/main.tf
```
Содержимое файла:
```hcl
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  service_account_key_file = "key.json"
  cloud_id                 = "<ваш-cloud-id>"
  folder_id                = "<ваш-folder-id>"
  zone                     = "ru-central1-a"
}

# Сеть
resource "yandex_vpc_network" "net" {
  name = "hw-network"
}

# NAT-шлюз
resource "yandex_vpc_gateway" "nat_gateway" {
  name = "hw-nat-gateway"
  shared_egress_gateway {}
}

# Таблица маршрутизации через NAT-шлюз
resource "yandex_vpc_route_table" "nat" {
  network_id = yandex_vpc_network.net.id
  name       = "hw-nat-route"

  static_route {
    destination_prefix = "0.0.0.0/0"
    gateway_id         = yandex_vpc_gateway.nat_gateway.id
  }
}

# Подсеть с привязанной таблицей маршрутизации
resource "yandex_vpc_subnet" "subnet" {
  name           = "hw-subnet"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.net.id
  v4_cidr_blocks = ["192.168.10.0/24"]
  route_table_id = yandex_vpc_route_table.nat.id
}

# Образ Ubuntu 22.04
data "yandex_compute_image" "ubuntu" {
  family = "ubuntu-2204-lts"
}

# Бастион-сервер (публичный IP)
resource "yandex_compute_instance" "bastion" {
  name        = "bastion"
  platform_id = "standard-v3"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa_yc.pub")}"
  }
}

# Веб-сервер A (только внутренний IP, выход в интернет через NAT-шлюз)
resource "yandex_compute_instance" "web_a" {
  name        = "web-a"
  platform_id = "standard-v3"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = false
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa_yc.pub")}"
  }
}

# Веб-сервер B (только внутренний IP, выход в интернет через NAT-шлюз)
resource "yandex_compute_instance" "web_b" {
  name        = "web-b"
  platform_id = "standard-v3"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = false
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa_yc.pub")}"
  }
}

# Веб-сервер C (только внутренний IP, выход в интернет через NAT-шлюз)
# По заданию 3 эта же ВМ используется как сервер БД — настройка через Ansible.
resource "yandex_compute_instance" "web_c" {
  name        = "web-c"
  platform_id = "standard-v3"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = false
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa_yc.pub")}"
  }
}

# Вывод IP-адресов
output "bastion_public_ip" {
  value = yandex_compute_instance.bastion.network_interface.0.nat_ip_address
}

output "web_a_internal_ip" {
  value = yandex_compute_instance.web_a.network_interface.0.ip_address
}

output "web_b_internal_ip" {
  value = yandex_compute_instance.web_b.network_interface.0.ip_address
}

output "web_c_internal_ip" {
  value = yandex_compute_instance.web_c.network_interface.0.ip_address
}
```
Разбор main.tf:
- Добавлен ресурс yandex_compute_instance.web_c — третья ВМ, на которой будет развёрнута БД.
- ВМ находится во внутренней подсети (nat = false), доступ — только через бастион.
- Добавлен output "web_c_internal_ip" — IP понадобится для Ansible-инвентаря.

**1.1. Добавление в файл ~/.ssh/config web_c (новой ВМ)**
```bash
nano ~/.ssh/config
```
Содержимое (замените IP на реальные из вывода Terraform):
```sshconfig
Host web-с
  HostName web_с-ip-address
  User ubuntu
  IdentityFile ~/.ssh/id_rsa_yc
  ProxyJump bastion
```
**2. inventory.ini**

Редактирование inventory.ini:
```bash
nano /opt/ansible-yc/inventory.ini
```
Содержание файла inventory.ini:
```ini
[web]
web-a ansible_host=<web_a_internal_ip>
web-b ansible_host=<web_b_internal_ip>

[web_c]
web-c ansible_host=<web_c_internal_ip>

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa_yc
ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -i ~/.ssh/id_rsa_yc ubuntu@<bastion_public_ip>"'
ansible_python_interpreter=/usr/bin/python3
```
> [!WARNING]
> Надо заполнить обязательно:
> - <web_a_internal_ip> - ip адрес web а
> - <web_b_internal_ip> - ip адрес web b
> - <bastion_public_ip> - ip адрес bastion (белый IP адрес)

Разбор inventory.ini:
- web-c вынесена в отдельную группу [web_c], потому что на ней будет БД.
- ansible_ssh_common_args с ProxyJump — обязателен, так как у web-a, web-b, web-c нет публичных IP.
- В hosts: плейбуков будет использоваться имя группы web_c.

**3. Установка и настройка chrony на всех хостах**

Редактирование chrony.yml:
```bash
nano /opt/ansible-yc/chrony.yml
```
Содержание файла chrony.yml:
```yaml
---
- name: "Установка и настройка chrony"
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: "Обновить кэш apt"
      apt:
        update_cache: true
        cache_valid_time: 3600

    - name: "Остановить systemd-timesyncd (конфликтует с chrony)"
      systemd:
        name: systemd-timesyncd
        state: stopped
        enabled: false
      failed_when: false

    - name: "Установить chrony"
      apt:
        name: chrony
        state: present

    - name: "Запустить и включить chrony"
      systemd:
        name: chrony
        state: started
        enabled: true

    - name: "Проверить, что chrony активен"
      command: systemctl is-active chrony
      register: chrony_status
      changed_when: false
      failed_when: chrony_status.stdout != "active"

    - name: "Вывести статус chrony"
      debug:
        msg: "Chrony на {{ inventory_hostname }}: {{ chrony_status.stdout }}"

    - name: "Проверить синхронизацию времени"
      command: timedatectl status
      register: time_status
      changed_when: false

    - name: "Вывести статус синхронизации времени"
      debug:
        msg: "{{ time_status.stdout_lines }}"
```
Разбор плейбука:
- hosts: all — применяется ко всем хостам из инвентаря (web-a, web-b, web-c).
- become: true — задачи выполняются от root.
- Отключает systemd-timesyncd (конфликтует с chrony).
- Устанавливает пакет chrony через apt.
- Запускает и включает службу chrony через systemd.
- Проверяет статус: systemctl is-active chrony → должен быть active.
- Выводит timedatectl status — состояние синхронизации времени.

**4. Установка PostgreSQL на web_c и проверка служб**

Редактирование postgresql.yml:
```bash
nano /opt/ansible-yc/postgresql.yml
```
Содержание файла postgresql.yml:
```yaml
---
# 1. Установка PostgreSQL на web-c
- name: "Установка PostgreSQL на web-c"
  hosts: web_c
  become: true
  gather_facts: true

  tasks:
    - name: "Обновить кэш apt"
      apt:
        update_cache: true
        cache_valid_time: 3600

    - name: "Установить PostgreSQL"
      apt:
        name: postgresql
        state: present

    - name: "Убедиться, что PostgreSQL запущен и в автозапуске"
      systemd:
        name: postgresql
        state: started
        enabled: true

    - name: "Проверить, что PostgreSQL активен"
      command: systemctl is-active postgresql
      register: pg_status
      changed_when: false
      failed_when: pg_status.stdout != "active"

    - name: "Вывести статус PostgreSQL"
      debug:
        msg: "Служба PostgreSQL на {{ inventory_hostname }}: {{ pg_status.stdout }}"

# 2. Проверка запущенных служб на всех хостах
- name: "Проверка состояния запущенных служб"
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: "Собрать список запущенных служб"
      command: systemctl list-units --type=service --state=running --no-pager --no-legend
      register: running_services
      changed_when: false

    - name: "Показать запущенные службы на {{ inventory_hostname }}"
      debug:
        msg: "{{ running_services.stdout_lines }}"

    - name: "Проверить статус SSH"
      command: systemctl is-active ssh
      register: ssh_status
      changed_when: false
      failed_when: false

    - name: "Вывести статус SSH"
      debug:
        msg: "SSH на {{ inventory_hostname }}: {{ ssh_status.stdout }}"

# 3. Итоговая проверка БД
- name: "Итоговая проверка БД"
  hosts: web_c
  become: true

  tasks:
    - name: "Проверить версию PostgreSQL"
      command: psql --version
      register: pg_version
      changed_when: false

    - name: "Вывести версию PostgreSQL"
      debug:
        msg: "Установлена: {{ pg_version.stdout }}"

    - name: "Проверить автозапуск PostgreSQL"
      command: systemctl is-enabled postgresql
      register: pg_enabled
      changed_when: false

    - name: "Вывести состояние автозапуска"
      debug:
        msg: "PostgreSQL автозапуск: {{ pg_enabled.stdout }}"
```
Разбор плейбука:

1. Установка PostgreSQL на web-c

Назначение: развернуть базу данных на сервере БД.

Ключевые моменты:
- hosts: web_c — только на web-c.
- Устанавливает пакет postgresql.
- Запускает и включает службу через systemd.
- Проверяет: systemctl is-active postgresql → active.
- Выводит статус службы.

Результат: PostgreSQL установлен и запущен на web-c.

2. Проверка запущенных служб

Назначение: собрать список работающих служб на всех хостах.

Ключевые моменты:
- hosts: all — на всех хостах (web-a, web-b, web-c).
- systemctl list-units --type=service --state=running — список запущенных служб.
- Отдельно проверяет ssh через systemctl is-active ssh.
- Выводит результат по каждому хосту.

Результат: получен перечень запущенных служб на каждой ВМ.

3. Итоговая проверка БД

Назначение: убедиться, что PostgreSQL работает и в автозапуске.

Ключевые моменты:
- hosts: web_c — только на web-c.
- psql --version — версия PostgreSQL.
- systemctl is-enabled postgresql → enabled — автозапуск включён.
- Выводит оба значения.

Результат: подтверждено, что БД установлена, работает и будет запускаться автоматически.

**5. Запуск плейбуков**

1. Запуск chrony.yml
```bash
cd /opt/ansible-yc
ansible-playbook -i inventory.ini chrony.yml
```
2. Запуск postgresql.yml
```bash
cd /opt/ansible-yc
ansible-playbook -i inventory.ini postgresql.yml
```
Ожидаемый вывод при успешном выполнении 

**6. Проверка проделанных действий**

1. Проверка доступности хостов

Что показывает: Ansible видит все три хоста.

Команда:
```bash
cd /opt/ansible-yc
ansible all -i inventory.ini -m ping
```
Ожидаемый вывод:

<img width="858" height="303" alt="image" src="https://github.com/user-attachments/assets/d6ff569e-746a-42ae-beaa-441e21ca4cf3" />

2. Chrony — список хостов, где установлен

Что показывает: chrony стоит на всех трёх хостах, не только на web-c.

Команда (с управляющей машины):
```bash
ansible all -i inventory.ini -m shell -a "systemctl is-active chrony"
```
Ожидаемый вывод:
<img width="1252" height="191" alt="image" src="https://github.com/user-attachments/assets/e4bfa252-9c63-4633-9a6e-fb68e268aaca" />

3. PostgreSQL — статус на web-c

Что показывает: БД установлена и запущена (пункт 2 задания).

Команды (на web-c):
```bash
systemctl status postgresql --no-pager
psql --version
```
Что искать:
<img width="1126" height="276" alt="image" src="https://github.com/user-attachments/assets/fe6066f7-6fcf-4016-ba6f-64c4f54afa4b" />

4. Проверка запущенных служб через Ansible (главный пункт 3)

Что показывает: Ansible собрал список запущенных служб на всех хостах.

Команда (с управляющей машины):
```bash
ansible all -i inventory.ini -m shell -a "systemctl list-units --type=service --state=running --no-pager --no-legend | head -15"
```
Ожидаемый вывод:
<img width="1640" height="938" alt="image" src="https://github.com/user-attachments/assets/a0a70bd8-7e77-43b7-b6cd-b414f547a676" />

## Задание 4*. Развертывание инфраструктуры в Yandex Cloud

Изучите инструкцию yandex для terraform. Добейтесь работы паплайна с безопасной передачей токена от облака в terraform через переменные окружения.

Для этого:

1. Настройте профиль для yc tools по инструкции.
2. Удалите из кода строчку “token = var.yandex_cloud_token”. Terraform будет считывать значение ENV переменной YC_TOKEN.
3. Выполните команду export YC_TOKEN=$(yc iam create-token) и в том же shell запустите terraform.
4. Для того чтобы вам не нужно было каждый раз выполнять export - добавьте данную команду в самый конец файла ~/.bashrc

## Решение 4*

**1. Настройка профиля для yc**

Проверка, что профиль активен и работает:
```bash
yc config profile activate sa-profile
yc config list
```
Проверка, что yc работает от имени сервисного аккаунта:
```bash
yc iam create-token
```
Команда должна вернуть длинную строку — IAM-токен. Если возвращается ошибка — сначала исправьте синхронизацию времени (chronyc makestep) и повторите.

> [!NOTE]
> - IAM-токен — временный, его время жизни не более 12 часов.
> - Terraform будет использовать его, пока он действителен.
> - После истечения токена команду export YC_TOKEN=... нужно выполнить заново.

**2. Удаление токена из кода Terraform**

Откройте main.tf:
```bash
nano /opt/terraform-yc/main.tf
```
Удалите из блока provider "yandex" строку, связанную с токеном. У вас должно остаться только:
```hcl
provider "yandex" {
  service_account_key_file = "key.json"
  cloud_id                 = "b1gb4plf6tsgs5tjm162"
  folder_id                = "b1gi2a4fbniep4dgbc86"
  zone                     = "ru-central1-a"
}
```
> [!IMPORTANT]
> В вашем текущем коде нет строки token = var.yandex_cloud_token — вы уже используете service_account_key_file.
> Это значит, что аутентификация идёт через ключ сервисного аккаунта, а не через переменную окружения.
> Чтобы выполнить задание 4*, нужно перейти на аутентификацию через YC_TOKEN: убрать service_account_key_file и позволить провайдеру считать токен из переменной окружения.

Итоговый блок провайдера для работы через YC_TOKEN:
```hcl
provider "yandex" {
  cloud_id  = "b1gb4plf6tsgs5tjm162"
  folder_id = "b1gi2a4fbniep4dgbc86"
  zone      = "ru-central1-a"
}
```
Теперь service_account_key_file не указан — провайдер будет искать токен в переменной окружения YC_TOKEN.

> [!WARNING]
> - Если у вас в main.tf есть variable "yandex_cloud_token" — удалите и её.
> - Также проверьте, что в файле нет token = ... в блоке провайдера.

**3. Экспорт переменных окружения**

Выполните в текущем shell:
```bash
export YC_TOKEN=$(yc iam create-token)
export YC_CLOUD_ID=$(yc config get cloud-id)
export YC_FOLDER_ID=$(yc config get folder-id)
```
Проверка:
```bash
echo $YC_TOKEN | head -c 50
echo $YC_CLOUD_ID
echo $YC_FOLDER_ID
```
Ожидаемый вывод:
- YC_TOKEN — длинная строка, начинающаяся с t1. (IAM-токен)
- YC_CLOUD_ID 
- YC_FOLDER_ID

> [!NOTE]
> - Команда yc iam create-token создаёт IAM-токен от имени текущего активного профиля yc.
> - Поскольку профиль sa-profile настроен на сервисный аккаунт terraform-sa, токен будет выпущен от имени этого сервисного аккаунта.

**4. Запуск Terraform в том же shell**

Важно: команды terraform должны выполняться в том же shell, где был выполнен export YC_TOKEN. Переменные окружения не сохраняются между сессиями.
```bash
cd /opt/terraform-yc
terraform plan
```
Terraform должен успешно прочитать конфигурацию и показать план. Никаких ошибок аутентификации быть не должно.
```bash
terraform apply
```
Подтвердите yes. Terraform выполнит план (скорее всего, изменений не будет, так как инфраструктура уже создана).

> [!IMPORTANT]
> Если terraform plan падает с ошибкой cannot get iam token или JWT already expired — проверьте:
> - Что YC_TOKEN не пустой: echo $YC_TOKEN
> - Что время на ВМ синхронизировано: chronyc tracking | grep 'System time'
> - Что профиль yc активен: yc config list

**5. Автоматизация через ~/.bashrc**

Чтобы не выполнять export YC_TOKEN=... вручную каждый раз, добавьте команду в конец ~/.bashrc:
```bash
echo 'export YC_TOKEN=$(yc iam create-token)' >> ~/.bashrc
```
> [!NOTE]
> - Эта команда будет выполняться при каждом открытии нового терминала.
> - IAM-токен будет генерироваться заново, что гарантирует его актуальность (токен живёт 12 часов) .

Однако у этого подхода есть побочный эффект: при каждом открытии терминала будет выполняться запрос к API Yandex Cloud, что добавляет задержку (~0.5–1 секунда). Если это критично, можно использовать crontab для обновления токена раз в час , но для учебной задачи достаточно ~/.bashrc.

Применение изменений:
```bash
source ~/.bashrc
```
Проверка:
```bash
echo $YC_TOKEN | head -c 50
```
Должна вывестись строка, начинающаяся с t1..

**6. Проверка безопасности**
Убедитесь, что токен не попал в код:
```bash
grep -r "YC_TOKEN\|yandex_cloud_token\|token = " /opt/terraform-yc/
```
В выводе не должно быть token = ... в main.tf. Если есть — удалите.

<img width="1449" height="70" alt="image" src="https://github.com/user-attachments/assets/47800664-6605-49c9-a6ac-7ca324feeb0d" />

Убедитесь, что key.json не в Git:
```bash
cd /opt/terraform-yc
cat .gitignore 2>/dev/null || echo "Нет .gitignore"
```
<img width="1065" height="117" alt="image" src="https://github.com/user-attachments/assets/6d7bd51e-fc75-42c4-b645-01df027e5200" />

Если файла нет — создайте:
```bash
echo "key.json" >> /opt/terraform-yc/.gitignore
echo "*.tfstate" >> /opt/terraform-yc/.gitignore
echo ".terraform/" >> /opt/terraform-yc/.gitignore
```
> [!NOTE]
> - .gitignore — это текстовый файл, который говорит Git, какие файлы и папки не нужно отслеживать и не нужно коммитить в репозиторий.
> - Если файл или папка указаны в .gitignore, Git просто не видит их при выполнении git status, git add и git commit. Они остаются на диске, но не попадают в историю репозитория.

Итоговая структура main.tf
```hcl
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  cloud_id  = "b1gb4plf6tsgs5tjm162"
  folder_id = "b1gi2a4fbniep4dgbc86"
  zone      = "ru-central1-a"
}
```
