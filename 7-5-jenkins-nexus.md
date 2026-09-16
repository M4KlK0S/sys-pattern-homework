# Домашнее задание к занятию "Практическое задание с самопроверкой «Что такое DevOps. CI/CD»" - Костенко Максим

## Задание 1

Что нужно сделать:

1. Установите себе jenkins по инструкции из лекции или любым другим способом из официальной документации. Использовать Docker в этом задании нежелательно.
2. Установите на машину с jenkins golang.
3. Используя свой аккаунт на GitHub, сделайте себе форк репозитория. В этом же репозитории находится дополнительный материал для выполнения ДЗ.
4. Создайте в jenkins Freestyle Project, подключите получившийся репозиторий к нему и произведите запуск тестов и сборку проекта go test . и docker build ..

В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.

## Решение 1

**1. Установка Java (требуется для Jenkins)**
Jenkins работает на Java. Рекомендуется OpenJDK 21.
```bash
sudo apt update
sudo apt install -y openjdk-21-jdk
java -version
```
Если видите номер версии — Java установлена успешно.

**2. Установка Jenkins**

Добавляем официальный репозиторий и устанавливаем:
```bash
# Импорт GPG-ключа
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

# Добавление APT-репозитория
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Установка
sudo apt update
sudo apt install -y jenkins

# Запуск и автозапуск
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```
Статус active (running) — значит Jenkins запущен.

**3. Открытие порта в файрволе**
```bash
sudo ufw allow 8080
```
**4. Первичная настройка Jenkins**

Открываем в браузере http://<IP-сервера>:8080 и получаем пароль администратора:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Вставляем пароль в веб-интерфейс → выбираем «Установить рекомендуемые плагины» → создаём администратора → завершаем настройку.

**5. Установка Go**
```bash
sudo apt install -y golang-go
go version
```
Если видите номер версии — Go установлен.

**6. Установка Git и выдача прав Jenkins на Docker**
```bash
sudo apt install -y git

# Добавляем пользователя jenkins в группу docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
Этот шаг критичен — без него команда docker build в Jenkins завершится ошибкой доступа.

**7. Создание Freestyle Project**

На главной странице Jenkins → New Item → вводим имя (например, go-docker-build) → выбираем Freestyle project → OK.

**8. Настройка источника кода (Git)**

В разделе Source Code Management выбираем Git:
- Repository URL — адрес вашего форка;
- Branches to build — */main или */master.

**9. Настройка шага сборки**

В разделе Build нажимаем Add build step → Execute shell и вписываем:
```bash
go test .
docker build .
```
**10. Сохранение и запуск**

Нажимаем Save → Build Now. В Build History кликаем по номеру сборки → Console Output смотрим лог.

Строка Finished: SUCCESS означает успех. Делаем скриншот Console Output.

<img width="787" height="322" alt="image" src="https://github.com/user-attachments/assets/1e3d3869-952a-458a-aef9-a837c1920460" />

## Задание 2

Что нужно сделать:

1. Создайте новый проект pipeline.
2. Перепишите сборку из задания 1 на declarative в виде кода.

В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.

## Решение 2

**1. Создание Pipeline-проекта**

Jenkins → New Item → имя (например, go-docker-pipeline) → выбираем Pipeline → OK.

**2. Declarative Jenkinsfile**

В разделе Pipeline в поле Script вписываем:
```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/M4KlK0S/sdvps-materials.git'
            }
        }
        stage('Test') {
            steps {
                sh 'go test .'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build .'
            }
        }
    }
}
```

Save → Build Now → смотрим Console Output и Stage View. Делаем скриншоты.

<img width="805" height="329" alt="image" src="https://github.com/user-attachments/assets/23a9c65b-ac1b-42c9-891b-ffda5ef78345" />

## Задание 3

Что нужно сделать:

1. Установите на машину Nexus.
2. Создайте raw-hosted репозиторий.
3. Измените pipeline так, чтобы вместо Docker-образа собирался бинарный go-файл. Команду можно скопировать из Dockerfile.
4. Загрузите файл в репозиторий с помощью jenkins.

В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.

## Решение 3

1. Установка Nexus

Nexus требует около 4 ГБ ОЗУ — проверьте конфигурацию машины.
```bash
# Создаём отдельного пользователя
sudo useradd -r -m -s /bin/bash -d /opt/nexus-data nexus

# Скачиваем Nexus (проверьте актуальную версию на сайте Sonatype)
cd /tmp
wget "https://download.sonatype.com/nexus/3/nexus-3.92.2-01-linux-x86_64.tar.gz"

# Распаковываем
sudo tar -xzf nexus-3.92.2-01-linux-x86_64.tar.gz -C /opt
sudo ln -s /opt/nexus-3.92.2-01 /opt/nexus

# Создаём каталог данных и выдаём права
sudo mkdir -p /opt/sonatype-work/nexus3
sudo chown -R nexus:nexus /opt/sonatype-work
sudo chmod -R 775 /opt/sonatype-work

# Указываем пользователя запуска
echo 'run_as_user="nexus"' | sudo tee /opt/nexus/bin/nexus.rc
```
Создаём systemd-службу:
```bash
sudo tee /etc/systemd/system/nexus.service > /dev/null <<EOF
[Unit]
Description=Sonatype Nexus Repository Manager
After=network.target

[Service]
Type=forking
User=nexus
Group=nexus
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
Restart=on-failure
RestartSec=10
TimeoutSec=600
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now nexus
```
Nexus стартует медленно (2–3 минуты). Проверяем:
```bash
sudo systemctl status nexus
```
Открываем http://<IP>:8081. Начальный пароль:
```bash
sudo cat /opt/sonatype-work/nexus3/admin.password
```
При первом входе смените пароль.

**2. Создание raw-hosted репозитория**

В Nexus: значок шестерёнки Administration → Repositories → Create repository → выбираем raw (hosted):

Name — например, raw-go-builds;

остальное по умолчанию → Create repository.

Запоминаем URL репозитория: http://<IP>:8081/repository/raw-go-builds/.

**3. Правка Pipeline — сборка бинарника и загрузка**

Команду сборки берём из Dockerfile (обычно это go build -o app). Меняем Jenkinsfile:
```groovy
pipeline {
    agent any

    environment {
        NEXUS_CRED = credentials('nexus-raw-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/<ваш-логин>/<имя-репозитория>.git'
            }
        }
        stage('Test') {
            steps {
                sh 'go test .'
            }
        }
        stage('Build Binary') {
            steps {
                sh 'go build -o myapp .'
            }
        }
        stage('Upload to Nexus') {
            steps {
                sh '''
                    curl -v --user "${NEXUS_CRED}" \
                      --upload-file ./myapp \
                      http://<IP-NEXUS>:8081/repository/raw-go-builds/${BUILD_NUMBER}/myapp
                '''
            }
        }
    }
}
```
**4. Добавление учётных данных Nexus в Jenkins**
Manage Jenkins → Credentials → System → Global credentials → Add Credentials:
- Kind — Username with password;
- ID — nexus-raw-credentials;
- Username / Password — логин и пароль от Nexus.

Сохраняем. В Jenkinsfile идентификатор nexus-raw-credentials подставится автоматически.

**5. Запуск и проверка**
Build Now → смотрим Console Output. После успешной загрузки заходим в Nexus → Browse и убеждаемся, что файл на месте. Делаем скриншоты.

<img width="1005" height="419" alt="image" src="https://github.com/user-attachments/assets/948e5747-eb78-43e4-b29b-7a632eddb53a" />

<img width="654" height="443" alt="image" src="https://github.com/user-attachments/assets/af76e37c-bd83-408c-a60e-df9b2d2c281f" />

## Объяснение

Почему не запускать всё прямо из Git?

Git — это хранилище кода, а не вычислительный сервер. GitHub Actions, GitLab CI — исключения, они добавили к себе и исполнителя, но классическая схема — Jenkins отдельно.

---

Почему не хранить артефакты в Jenkins?

Jenkins рабочие директории очищаются при перезапуске или новой сборке. Артефакты там жить не могут. Nexus — постоянный, версионируемый, с правами доступа.

---

Почему не собирать всё локально и пушить бинарник в Git?

Git не предназначен для бинарников: он хранит все версии каждого файла, и репозиторий раздувается до гигабайтов. Nexus для этого и создан.
