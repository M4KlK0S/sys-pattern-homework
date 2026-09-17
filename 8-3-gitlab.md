# Домашнее задание к занятию "Практическое задание с самопроверкой «GitLab»" - Костенко Максим

## Подготовка хоста: установка инструментов

**1. VirtualBox**
```bash
sudo apt update
sudo apt install -y virtualbox
vboxmanage --version
```

**2. Vagrant**
Официальный репозиторий HashiCorp не поддерживает новые версии Ubuntu (resolute), поэтому используем зеркало Yandex Cloud:
```bash
cd /tmp
wget https://hashicorp-releases.yandexcloud.net/vagrant/2.4.9/vagrant_2.4.9_linux_amd64.zip
unzip vagrant_2.4.9_linux_amd64.zip
sudo mv vagrant /usr/local/bin/
vagrant --version
```
**3. Git**
```bash
sudo apt install -y git
git --version
```

## Задание 1. Развёртывание GitLab и регистрация Docker Runner

Что нужно сделать:

1. Разверните GitLab локально, используя Vagrantfile и инструкцию, описанные в этом репозитории.
2. Создайте новый проект и пустой репозиторий в нём.
3. Зарегистрируйте gitlab-runner для этого проекта и запустите его в режиме Docker. Раннер можно регистрировать и запускать на той же виртуальной машине, на которой запущен GitLab.

В качестве ответа в репозиторий шаблона с решением добавьте скриншоты с настройками раннера в проекте.

## Решение 1

**1. Клонирование репозитория с Vagrantfile**

Склонируй репозиторий с Vagrantfile и docker-compose.yml из официального репозитория Netology :
```bash
cd /opt
git clone https://github.com/M4KlK0S/sdvps-materials
cd sdvps-materials/gitlab
```
**2. Настройка /etc/hosts**

Добавь запись в hosts-файл хоста:
```bash
echo '192.168.56.10    gitlab.localdomain gitlab' | sudo tee -a /etc/hosts
```
Это позволит обращаться к GitLab по имени gitlab.localdomain .

**3. Запуск виртуальной машины**
```bash
VAGRANT_EXPERIMENTAL="disks" vagrant up
```
Переменная VAGRANT_EXPERIMENTAL="disks" нужна для создания диска нестандартного размера (15 ГБ), указанного в Vagrantfile.

Установка GitLab занимает 5–15 минут. После завершения GitLab будет доступен по адресу http://gitlab.localdomain.

**4. Получение первичного пароля root**
```bash
vagrant ssh -- sudo cat /etc/gitlab/initial_root_password
```
Войди в GitLab как root с этим паролем и смени его.

**5. Создание проекта**

В GitLab: New project → Create blank project, сними галочку «Initialize repository with a README», задай имя (например, my-project) .

**6. Регистрация GitLab Runner в режиме Docker**

Получи токен: Settings → CI/CD → Runners → New project runner → задай тег docker → включи Run untagged jobs → Create runner → скопируй токен.

Войди в ВМ и зарегистрируй раннер :

```bash
vagrant ssh

docker run -ti --rm --name gitlab-runner \
  --network host \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest register
```
Ответы:
- GitLab URL: http://gitlab.localdomain/
- Token: из шага 6
- Description: docker-runner
- Tags: docker
- Executor: docker
- Default image: alpine:latest

**7. Настройка config.toml**

Файл /srv/gitlab-runner/config/config.toml должен содержать :
```toml
[[runners]]
  name = "docker-runner"
  url = "http://gitlab.localdomain/"
  token = "ваш-token"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    extra_hosts = ["gitlab.localdomain:192.168.56.10"]
```
extra_hosts нужен, чтобы контейнеры Runner'а могли разрешить gitlab.localdomain.

**8. Запуск Runner**
```bash
docker run -d --name gitlab-runner --restart always \
  --network host \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```
**9. Проверка**

В GitLab: Settings → CI/CD → Runners — Runner должен быть Online (зелёная точка).

## Задание 2. Пуш в GitLab и создание .gitlab-ci.yml

Что нужно сделать:

Запушьте репозиторий на GitLab, изменив origin. Это изучалось на занятии по Git.
Создайте .gitlab-ci.yml, описав в нём все необходимые, на ваш взгляд, этапы.
В качестве ответа в шаблон с решением добавьте:

файл gitlab-ci.yml для своего проекта или вставьте код в соответствующее поле в шаблоне;
скриншоты с успешно собранными сборками.

## Решение 2

**1. Смена origin**
```bash
git remote -v
git remote set-url origin http://gitlab.localdomain/root/my-project.git
git remote -v
git push -u origin main
```
Команда git remote set-url изменяет адрес существующего remote .

**2. Создание .gitlab-ci.yml**

Создай файл в корне проекта. Пример с этапами build и test :
```yaml
stages:
  - build
  - test

build-job:
  stage: build
  image: golang:1.17
  script:
    - echo "Собираем проект..."
    - go build .
  artifacts:
    paths:
      - app
    expire_in: 1 week

unit-test-job:
  stage: test
  image: golang:1.17
  script:
    - echo "Запускаем тесты..."
    - go test .

lint-job:
  stage: test
  image: golang:1.17
  script:
    - echo "Линтим код..."
    - go vet ./...
```
**3. Пуш и проверка**
```bash
git add .gitlab-ci.yml README.md
git commit -m "Add .gitlab-ci.yml and screenshots"
git push origin main
```
Открой CI/CD → Pipelines — пайплайн запустится автоматически.

## Задание 3* (со звёздочкой). Оптимизация CI

Измените CI так, чтобы:
- этап сборки запускался сразу, не дожидаясь результатов тестов;
- тесты запускались только при изменении файлов с расширением *.go.

В качестве ответа добавьте в шаблон с решением файл gitlab-ci.yml своего проекта или вставьте код в соответсвующее поле в шаблоне.

## Решение 3*
```yaml
stages:
  - build
  - test

build:
  stage: build
  needs: []                     # ключевой момент: не ждём стадии test
  image: docker:latest
  script:
    - docker build .
  artifacts:
    paths:
      - app
    expire_in: 1 week

test:
  stage: test
  image: golang:1.17
  script:
    - go test .
  rules:
    - changes:
        - "**/*.go"
        - "go.mod"
        - "go.sum"
```
Ключевые механизмы
```
Ключ	          Что делает
needs:          []	Job запускается сразу, минуя ожидание стадий. Пайплайн становится DAG.
rules: changes	Job выполняется только если изменились указанные файлы. **/*.go — любые .go в любом каталоге.
```
Проверка
- Коммит только README.md → стадия test пропущена
- Коммит с .go → стадия test запущена
- Включи Pipelines → Job dependencies для DAG-вида
