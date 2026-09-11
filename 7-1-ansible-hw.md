# Домашнее задание к занятию "Практическое задание с самопроверкой «Ansible. Часть 2»" - Костенко Максим

# Задание 1

Выполните действия, приложите файлы с плейбуками и вывод выполнения.

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.
Плейбуки должны:

Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать официальный сайт и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.

Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.

Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

# Решение 1

1. Удалить старый ключ конкретного хоста (безопасно)
Если помните IP целевой ВМ (например, 10.0.2.4):
```bash
# удаляет запись об этом хосте из known_hosts текущего пользователя
ssh-keygen -R [127.0.0.1]:2222
```
2. Полностью очистить known_hosts (быстро, для лабораторной)
Если не помните, какой именно хост поменялся, или у вас один учебный стенд:
```bash
# удаляет файл known_hosts целиком — все сохранённые хост-ключи будут потеряны
rm -f ~/.ssh/known_hosts
```
3. Сменить владельца директории (рекомендую). Раз это ваша рабочая директория, логично сделать её владельцем kostenkomp:
```bash
sudo chown -R kostenkomp:kostenkomp /opt/ansible
```
---
ВМ1 (ubuntu, 10.0.2.5) — управляющая (хост, где установлен Ansible).

ВМ2 (ubuntu2, 10.0.2.4) — управляемая (где выполняются плейбуки).

## 1. Настройка управляющей ВМ (ВМ1)
**1.1. Установка Ansible**
```bash
sudo apt update
sudo apt install -y ansible-core openssh-client
```
Проверка установки:
```bash
ansible --version
```
**1.2. Генерация SSH-ключа**
Если ключа ещё нет — создаём пару:
```bash
ssh-keygen -t rsa -b 4096
```
Проверка, что ключи появились:
```bash
ls -la ~/.ssh/
```
Должны быть файлы id_rsa (приватный) и id_rsa.pub (публичный).

Копирование публичного ключа на управляемую ВМ (ВМ2):
```bash
ssh-copy-id kostenkomp@10.0.2.4
```
**1.3. Создание рабочей директории и инвентаря**
```bash
mkdir -p /opt/ansible && cd /opt/ansible
nano inventory.ini
```
Содержимое inventory.ini:
```ini
[all]
target-host ansible_host=10.0.2.4 ansible_user=kostenkomp
```
Примечание: ansible_user=kostenkomp — это тот же пользователь, что и на управляющей ВМ. Под ним Ansible будет заходить на ВМ2 по SSH.

**1.4. Проверка связи с ВМ2 (после настройки ВМ2)**

Эту проверку выполняем после шагов из части 2 — когда на ВМ2 уже создан пользователь kostenkomp, работает SSH и скопирован публичный ключ.
```bash
ansible -i inventory.ini all -m ping
```
Ожидаемый результат:
<img width="1889" height="258" alt="image" src="https://github.com/user-attachments/assets/fab202a3-3fd5-4f77-8bf9-bc8c74619800" />

## 2. Настройка управляемой ВМ (ВМ2)

**2.1. Установка SSH-сервера**
```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```
Проверка статуса:
```bash
systemctl is-active ssh
systemctl is-enabled ssh
```
Ожидаемый вывод: active и enabled.

**2.2. Создание пользователя (если его нет)**

На ВМ2 должен существовать пользователь kostenkomp — под ним Ansible будет заходить по SSH:
```bash
id kostenkomp
```
Если пользователя нет:
```bash
sudo adduser kostenkomp
sudo usermod -aG sudo kostenkomp
```
**2.3. Копирование SSH-ключа с ВМ1 на ВМ2**
Выполняется на ВМ1:
```bash
ssh-copy-id kostenkomp@10.0.2.4
```
Вас попросят пароль пользователя kostenkomp на ВМ2 — введите его.

Проверка входа без пароля (на ВМ1):
```bash
ssh kostenkomp@10.0.2.4
exit
```
**2.4. Настройка sudo без пароля на ВМ2**

Выполняется на ВМ2 (зайти можно через ssh kostenkomp@10.0.2.4):
```bash
# создаёт файл с правилом sudo без пароля для пользователя kostenkomp (tee нужен, чтобы записать от root)
echo "kostenkomp ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/kostenkomp

# выставляет обязательные права 0440 — иначе sudo проигнорирует файл
sudo chmod 0440 /etc/sudoers.d/kostenkomp

# sudo -k сбрасывает кэш пароля, sudo -n проверяет работу без пароля, whoami покажет root
sudo -k && sudo -n whoami
```
Ожидаемый вывод последней команды: root — значит, sudo работает без пароля.

## 3. Плейбуки (создаются на ВМ1, запускаются с ВМ1, выполняются на ВМ2)

**3.1. Плейбук 1 — скачивание архива, создание папки и распаковка**

Создание плейбука:
```bash
nano playbook_1_download_unarchive.yml
```
Содержимое:
```yaml
- name: Download and Unpack Kafka Archive
  hosts: all
  become: yes

  vars:
    download_url: "https://archive.apache.org/dist/kafka/3.9.2/kafka_2.13-3.9.2.tgz"
    archive_dest_dir: "/opt/kafka"
    remote_archive_path: "/tmp/kafka_2.13-3.9.2.tgz"

  tasks:
    - name: Ensure destination directory exists
      ansible.builtin.file:
        path: "{{ archive_dest_dir }}"
        state: directory
        mode: '0755'

    - name: Download Kafka archive from Apache mirror
      ansible.builtin.get_url:
        url: "{{ download_url }}"
        dest: "{{ remote_archive_path }}"
        mode: '0644'

    - name: Unarchive Kafka archive to destination
      ansible.builtin.unarchive:
        src: "{{ remote_archive_path }}"
        dest: "{{ archive_dest_dir }}"
        remote_src: yes
        extra_opts:
          - --strip-components=1

    - name: Remove downloaded archive
      ansible.builtin.file:
        path: "{{ remote_archive_path }}"
        state: absent
```
Запуск:
```bash
ansible-playbook -i inventory.ini playbook_1_download_unarchive.yml
```
<img width="1894" height="560" alt="image" src="https://github.com/user-attachments/assets/d1dfd2d7-88fd-4e57-be27-d7eb3195a8a2" />

Проверка результата:
```bash
ansible -i inventory.ini all -m shell -a "ls -l /opt/kafka | head -10"
```
<img width="1884" height="300" alt="image" src="https://github.com/user-attachments/assets/78a5725f-329c-43d7-b602-ae4885110061" />

**3.2. Плейбук 2 — установка, запуск и автозагрузка tuned**

Создание плейбука:
```bash
nano playbook_2_tuned.yml
```
Содержимое:
```yaml
- name: Install and Configure Tuned
  hosts: all
  become: yes

  tasks:
    - name: Install tuned package
      ansible.builtin.package:
        name: tuned
        state: present

    - name: Ensure tuned service is started and enabled
      ansible.builtin.service:
        name: tuned
        state: started
        enabled: yes
```
Запуск:
```bash
ansible-playbook -i inventory.ini playbook_2_tuned.yml
```
<img width="1882" height="421" alt="image" src="https://github.com/user-attachments/assets/b7f2d757-20d6-4f7b-9260-3b586669e4f4" />

Проверка результата:
```bash
ansible -i inventory.ini all -m shell -a "systemctl is-active tuned && systemctl is-enabled tuned"
```
<img width="1885" height="164" alt="image" src="https://github.com/user-attachments/assets/6ffeb02b-2de4-4c17-ab86-e6226baed09e" />

Ожидаемый вывод: active и enabled.

**3.3. Плейбук 3 — изменение motd через переменную**

Создание плейбука:
```bash
nano playbook_3_motd.yml
```
Содержимое:
```yaml
- name: Change System MOTD
  hosts: all
  become: yes

  vars:
    custom_motd_message: "Добро пожаловать в систему! Управление осуществляется через Ansible."

  tasks:
    - name: Set custom MOTD using a variable
      ansible.builtin.copy:
        content: "{{ custom_motd_message }}\n"
        dest: /etc/motd
        owner: root
        group: root
        mode: '0644'
```
Запуск:
```bash
ansible-playbook -i inventory.ini playbook_3_motd.yml
```
<img width="1882" height="354" alt="image" src="https://github.com/user-attachments/assets/b090e4f2-8b8d-4135-b268-2b5472af975a" />

Проверка результата:
```bash
ansible -i inventory.ini all -m shell -a "cat /etc/motd"
```
<img width="1879" height="142" alt="image" src="https://github.com/user-attachments/assets/34c78231-4c9f-47f6-aab4-e8b222377976" />

Ожидаемый вывод: Добро пожаловать в систему! Управление осуществляется через Ansible.

## Общий вывод Задания 1

1. На ВМ2 (10.0.2.4) установлен и распакован Apache Kafka в /opt/kafka — файлы bin/, config/, libs/ на месте.
2. Установлен tuned, запущен как демон и добавлен в автозагрузку: active, enabled.
3. Изменён /etc/motd через переменную — при входе выводится новое приветствие.

Зачем это нужно: Ansible позволяет один раз описать желаемое состояние сервера в YAML и применять его к любому числу машин — вместо ручной настройки каждой.

# Задание 2

Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору.

# Решение 2

**Что требуется**

- Взять плейбук 3 из задания 1 (изменение motd) и модифицировать его так, чтобы приветствие содержало:
- IP-адрес управляемого хоста — ansible_default_ipv4.address;
- Hostname управляемого хоста — ansible_hostname;
- Пожелание хорошего дня системному администратору.

Для этого используются Ansible facts — информация о хосте, которую Ansible собирает автоматически в задаче Gathering Facts в начале каждого плейбука.

**Что такое Ansible facts?**

Facts — это переменные, которые Ansible собирает о каждом управляемом хосте перед выполнением задач. Среди них:
```text
Переменная	                    Что содержит	                      Пример значения
ansible_hostname	            Короткое имя хоста	                  ubuntu2
ansible_fqdn	                Полное доменное имя	                  ubuntu2.local
ansible_default_ipv4.address	Основной IPv4-адрес	                  10.0.2.4
ansible_all_ipv4_addresses	    Список всех IPv4	                  ['10.0.2.4']
ansible_distribution	        Название ОС	                          Ubuntu
ansible_processor_vcpus	        Количество CPU	                      2
ansible_memtotal_mb	            Объём RAM в МБ	                      2048
```
Посмотреть все facts для конкретного хоста можно командой:
```bash
ansible -i inventory.ini all -m setup
```
Или отфильтровать нужные:
```bash
ansible -i inventory.ini all -m setup -a "filter=ansible_hostname"
ansible -i inventory.ini all -m setup -a "filter=ansible_default_ipv4"
```
[!NOTE]
Facts собираются автоматически, потому что в плейбуке не отключён gather_facts. По умолчанию он включён — именно поэтому в каждом запуске вы видите задачу Gathering Facts первой.

Модифицированный плейбук

Файл: playbook_3_motd.yml
```yaml
- name: Change System MOTD with IP and Hostname
  hosts: all
  become: yes

  vars:
    admin_greeting: "Хорошего дня, системный администратор!"

  tasks:
    - name: Set custom MOTD with host info and greeting
      ansible.builtin.copy:
        content: |
          ============================================
          IP-адрес: {{ ansible_facts['default_ipv4']['address'] }}
          Hostname: {{ ansible_facts['hostname'] }}
          ============================================
          {{ admin_greeting }}
          ============================================
        dest: /etc/motd
        owner: root
        group: root
        mode: '0644'
```
Разбор изменений
```text
Что было в задании 1	                                  Что стало в задании 2
Переменная custom_motd_message со статичным текстом	      Переменная admin_greeting только для пожелания
content: "{{ custom_motd_message }}\n"	                  content: | с многострочным шаблоном
Только текст приветствия	                              IP + hostname + пожелание
Статичное содержимое	                                  Динамическое — подставляется из facts целевого хоста
```

Что здесь ключевое
1. content: | — блочный скаляр YAML. Позволяет записать многострочный текст, сохранив переводы строк. Каждая строка внутри блока становится отдельной строкой в файле /etc/motd.
2. {{ ansible_default_ipv4.address }} — подстановка IP-адреса целевого хоста из facts. На ВМ2 (10.0.2.4) подставится 10.0.2.4, на другой машине — её собственный IP.
3. {{ ansible_hostname }} — подстановка hostname целевого хоста. На ВМ2 это ubuntu2.
4. {{ admin_greeting }} — переменная из блока vars, содержит пожелание. Её можно переопределить через --extra-vars, не редактируя плейбук.
5. Facts подставляются от управляемого хоста, а не от управляющей ВМ. То есть на ВМ2 в motd попадут данные именно ВМ2.

**Запуск плейбука**
```bash
cd /opt/ansible
ansible-playbook -i inventory.ini playbook_3_motd.yml
```
Ожидаемый вывод:
<img width="1893" height="351" alt="image" src="https://github.com/user-attachments/assets/e856bad3-99a5-4468-915c-3d5caf466653" />

Проверка результата
```bash
ansible -i inventory.ini all -m shell -a "cat /etc/motd"
```
Ожидаемый вывод:
<img width="1880" height="256" alt="image" src="https://github.com/user-attachments/assets/ec0b22fd-1d63-45de-822e-d5ede9531a42" />

При следующем входе на ВМ2 по SSH пользователь увидит:
<img width="623" height="611" alt="image" src="https://github.com/user-attachments/assets/4191b25c-5d5f-45f1-b5f2-2f182fcf1275" />

# Задание 3

Выполните действия, приложите архив с ролью и вывод выполнения.

1. Ознакомьтесь со статьёй «Ansible - это вам не bash», сделайте соответствующие выводы и не используйте модули shell или command при выполнении задания.
2. Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:
- Установить веб-сервер Apache на управляемые хосты.
- Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес. Используйте Ansible facts и jinja2-template. Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
- Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
- Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

# Решение 3

**1. Создание структуры роли**

Роль — это самодостаточный набор задач, шаблонов и переменных. Создаётся командой ansible-galaxy role init .

Перейдите в рабочую директорию и создайте роль:
```bash
cd /opt/ansible
mkdir -p roles
cd roles
ansible-galaxy role init apache
```
Структура роли будет такой :
```text
apache/
├── README.md
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    └── main.yml
```
```text
Директория	                    Назначение
tasks/	                        Основной список задач роли 
handlers/	                    Обработчики, вызываемые по уведомлению 
templates/	                    Jinja2-шаблоны файлов 
defaults/	                    Переменные со значениями по умолчанию
vars/	                        Переменные роли с высоким приоритетом
meta/	                        Метаданные и зависимости
```
**2. Задачи роли — tasks/main.yml**
```bash
nano /opt/ansible/roles/apache/tasks/main.yml
```
Содержимое:
```yaml
---
# 1. Установка Apache
- name: Install Apache web server
  ansible.builtin.package:
    name: "{{ apache_package_name }}"
    state: present

# 2. Убедиться, что Apache запущен и в автозагрузке
- name: Ensure Apache is started and enabled
  ansible.builtin.service:
    name: "{{ apache_service_name }}"
    state: started
    enabled: yes

# 3. Открыть порт 80 в UFW (если UFW установлен)
- name: Allow HTTP port 80 in UFW
  community.general.ufw:
    rule: allow
    port: "80"
    proto: tcp
  when: ufw_installed is defined and ufw_installed

# 4. Сгенерировать index.html из Jinja2-шаблона
- name: Generate index.html from template
  ansible.builtin.template:
    src: index.html.j2
    dest: "{{ apache_docroot }}/index.html"
    owner: www-data
    group: www-data
    mode: '0644'
  notify: Restart Apache

# 5. Проверить доступность сайта (ответ 200)
- name: Check web site availability
  ansible.builtin.uri:
    url: "http://{{ ansible_default_ipv4.address }}/"
    return_content: no
    status_code: 200
```
```text
Разбор задач
Задача	                    Модуль	                  Что делает
Install Apache	            package	                  Устанавливает пакет (имя из переменной)
Ensure Apache started	    service	                  Запускает и добавляет в автозагрузку
Allow HTTP port 80	        community.general.ufw	  Открывает порт 80 через UFW
Generate index.html	        template	              Рендерит Jinja2-шаблон и уведомляет handler
Check availability	        uri	                      Делает HTTP-запрос и проверяет код 200
```
> ВАЖНО! Никаких shell и command — все задачи используют специализированные модули Ansible. Это соответствует духу статьи «Ansible — это вам не bash».

**Определение наличия UFW**

Чтобы задача с UFW не падала, если UFW не установлен, добавим проверку. Но её тоже нужно сделать без shell. Проще всего — завести переменную и проверять факт наличия пакета через facts:
```yaml
# В начале tasks/main.yml, перед задачами
- name: Check if UFW is installed
  ansible.builtin.set_fact:
    ufw_installed: "{{ 'ufw' in ansible_facts.packages }}"
  when: ansible_facts.packages is defined
```
> Этот вариант работает, если facts о пакетах собраны. Для этого в плейбуке нужно включить gather_subset: [packages] — см. ниже.

Либо ещё проще — использовать ansible.builtin.package_facts:
```yaml
- name: Gather package facts
  ansible.builtin.package_facts:
    manager: auto

- name: Set UFW installed flag
  ansible.builtin.set_fact:
    ufw_installed: "{{ 'ufw' in ansible_facts.packages }}"
```
**3. Handler — handlers/main.yml**
```bash
nano /opt/ansible/roles/apache/handlers/main.yml
```
Содержимое:
```yaml
---
- name: Restart Apache
  ansible.builtin.service:
    name: "{{ apache_service_name }}"
    state: restarted
```
> Как это работает: handler запускается только тогда, когда задача, которая его уведомляет, вернула changed. В нашем случае — только при изменении index.html. Если файл не менялся, задача template вернёт ok, и handler не сработает — Apache не перезапустится лишний раз.

Это и есть требование задания: «перезапуск Apache только в случае изменения файла конфигурации» .

**4. Jinja2-шаблон — templates/index.html.j2**
```bash
nano /opt/ansible/roles/apache/templates/index.html.j2
```
Содержимое:
```jinja
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Информация о сервере {{ ansible_hostname }}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background: #f4f4f4; }
        h1 { color: #333; }
        table { border-collapse: collapse; background: #fff; }
        th, td { border: 1px solid #ccc; padding: 10px 20px; text-align: left; }
        th { background: #0078d7; color: #fff; }
    </style>
</head>
<body>
    <h1>Характеристики сервера {{ ansible_hostname }}</h1>
    <table>
        <tr>
            <th>Параметр</th>
            <th>Значение</th>
        </tr>
        <tr>
            <td>Hostname</td>
            <td>{{ ansible_hostname }}</td>
        </tr>
        <tr>
            <td>IP-адрес</td>
            <td>{{ ansible_default_ipv4.address }}</td>
        </tr>
        <tr>
            <td>CPU (vCPU)</td>
            <td>{{ ansible_processor_vcpus }}</td>
        </tr>
        <tr>
            <td>Модель CPU</td>
            <td>{{ ansible_processor[2] }}</td>
        </tr>
        <tr>
            <td>RAM</td>
            <td>{{ ansible_memtotal_mb }} MB</td>
        </tr>
        <tr>
            <td>Первый HDD</td>
            <td>{{ ansible_devices[ansible_devices.keys() | list | first].size }}</td>
        </tr>
        <tr>
            <td>ОС</td>
            <td>{{ ansible_distribution }} {{ ansible_distribution_version }}</td>
        </tr>
    </table>
    <p><em>Сгенерировано Ansible {{ ansible_date_time.date }}</em></p>
</body>
</html>
```
**Используемые facts**
```text
Переменная	                    Что содержит
ansible_hostname	            Имя хоста
ansible_default_ipv4.address	Основной IPv4
ansible_processor_vcpus	        Количество vCPU
ansible_processor[2]	        Модель процессора (обычно третий элемент списка)
ansible_memtotal_mb	            Объём RAM в МБ
ansible_devices[...].size	    Размер первого диска
ansible_distribution	        Название ОС
```
> В новых версиях Ansible рекомендуется обращаться к facts через ansible_facts['имя'] (как в задании 2). Здесь используется старый синтаксис для читаемости, но при желании можно заменить на ansible_facts['hostname'], ansible_facts['default_ipv4']['address'] и т.д.

**5. Переменные роли — defaults/main.yml**
```bash
nano /opt/ansible/roles/apache/defaults/main.yml
```
Содержимое:
```yaml
---
# Имя пакета Apache (разное для Debian и RHEL)
apache_package_name: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' }}"

# Имя службы Apache
apache_service_name: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' }}"

# Каталог, куда кладём index.html (разный для Debian и RHEL)
apache_docroot: "{{ '/var/www/html' if ansible_os_family == 'Debian' else '/var/www/html' }}"
```
> Использование defaults/ позволяет переопределить эти переменные в плейбуке, не редактируя саму роль.

**6. Плейбук, включающий роль**
```bash
nano /opt/ansible/playbook_apache_role.yml
```
Содержимое:
```yaml
---
- name: Deploy Apache with server info page
  hosts: all
  become: yes

  roles:
    - apache
```
> Такой плейбук максимально короткий — вся логика вынесена в роль. Это и есть смысл ролей: переиспользуемый, самодостаточный компонент.

**7. Запуск и проверка**

Запуск плейбука
```bash
cd /opt/ansible
ansible-playbook -i inventory.ini playbook_apache_role.yml
```
Ожидаемый вывод:
<img width="1881" height="698" alt="image" src="https://github.com/user-attachments/assets/8e07bd66-dcc4-4534-b9f3-042019a9a1b0" />

Проверка результата
```bash
# Проверить, что сайт отдаёт 200 и содержит данные
curl -s http://10.0.2.4/ | head -30

# Или через Ansible
ansible -i inventory.ini all -m uri -a "url=http://10.0.2.4/ return_content=yes" | grep status
```
Ожидаемый результат — HTML-страница с таблицей, содержащей CPU, RAM, HDD, IP и hostname.

**8. Архив с ролью**

Соберите роль в архив:
```bash
cd /opt/ansible
tar czf apache-role.tar.gz roles/apache/
```
Проверьте содержимое:
```bash
tar tzf apache-role.tar.gz | head -20
```
