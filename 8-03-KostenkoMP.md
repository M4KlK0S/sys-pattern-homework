Подготовка хоста: установка инструментов
1. VirtualBox
bash
sudo apt update
sudo apt install -y virtualbox
vboxmanage --version
Если модуль vboxdrv не загружен, выполни:

bash
sudo apt install --reinstall virtualbox-dkms linux-headers-$(uname -r)
sudo modprobe vboxdrv
При включённом Secure Boot зарегистрируй MOK-ключ или отключи его в BIOS.

2. Vagrant
Официальный репозиторий HashiCorp не поддерживает новые версии Ubuntu (resolute), поэтому используем зеркало Yandex Cloud:

bash
cd /tmp
wget https://hashicorp-releases.yandexcloud.net/vagrant/2.4.9/vagrant_2.4.9_linux_amd64.zip
unzip vagrant_2.4.9_linux_amd64.zip
sudo mv vagrant /usr/local/bin/
vagrant --version
3. Git
bash
sudo apt install -y git
git --version
Задание 1. Развёртывание GitLab и регистрация Docker Runner
Шаг 1. Клонирование репозитория с Vagrantfile
Склонируй репозиторий с Vagrantfile и docker-compose.yml из официального репозитория Netology :

bash
git clone https://github.com/netology-code/sdvps-materials.git
cd sdvps-materials/gitlab
Шаг 2. Настройка /etc/hosts
Добавь запись в hosts-файл хоста:

bash
echo '192.168.56.10    gitlab.localdomain gitlab' | sudo tee -a /etc/hosts
Это позволит обращаться к GitLab по имени gitlab.localdomain .

Шаг 3. Запуск виртуальной машины
bash
VAGRANT_EXPERIMENTAL="disks" vagrant up
Переменная VAGRANT_EXPERIMENTAL="disks" нужна для создания диска нестандартного размера (15 ГБ), указанного в Vagrantfile.

Установка GitLab занимает 5–15 минут. После завершения GitLab будет доступен по адресу http://gitlab.localdomain.

Шаг 4. Получение первичного пароля root
bash
vagrant ssh -- sudo cat /etc/gitlab/initial_root_password
Войди в GitLab как root с этим паролем и смени его.

Шаг 5. Создание проекта
В GitLab: New project → Create blank project, сними галочку «Initialize repository with a README», задай имя (например, my-project) .

Шаг 6. Регистрация GitLab Runner в режиме Docker
Получи токен: Settings → CI/CD → Runners → New project runner → задай тег docker → включи Run untagged jobs → Create runner → скопируй токен.

Войди в ВМ и зарегистрируй раннер :

bash
vagrant ssh

docker run -ti --rm --name gitlab-runner \
  --network host \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest register
Ответы:

GitLab URL: http://gitlab.localdomain/

Token: из шага 6

Description: docker-runner

Tags: docker

Executor: docker

Default image: alpine:latest

Шаг 7. Настройка config.toml
Файл /srv/gitlab-runner/config/config.toml должен содержать :

toml
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
extra_hosts нужен, чтобы контейнеры Runner'а могли разрешить gitlab.localdomain.

Шаг 8. Запуск Runner
bash
docker run -d --name gitlab-runner --restart always \
  --network host \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
Шаг 9. Проверка и скриншоты
В GitLab: Settings → CI/CD → Runners — Runner должен быть Online (зелёная точка).

Приложи в README.md:

Скриншот списка runner'ов со статусом Online

Скриншот деталей runner'а (tags, executor, URL)

Задание 2. Пуш в GitLab и создание .gitlab-ci.yml
Шаг 10. Смена origin
bash
git remote -v
git remote set-url origin http://gitlab.localdomain/root/my-project.git
git remote -v
git push -u origin main
Команда git remote set-url изменяет адрес существующего remote .

Шаг 11. Создание .gitlab-ci.yml
Создай файл в корне проекта. Пример с этапами build и test :

yaml
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
Шаг 12. Пуш и проверка
bash
git add .gitlab-ci.yml README.md
git commit -m "Add .gitlab-ci.yml and screenshots"
git push origin main
Открой CI/CD → Pipelines — пайплайн запустится автоматически.

Приложи в README.md:

Код .gitlab-ci.yml

Скриншот списка пайплайнов со статусом passed

Скриншот страницы пайплайна с зелёными job'ами

Задание 3* (со звёздочкой). Оптимизация CI
Требования
Сборка запускается сразу, не дожидаясь тестов.

Тесты — только при изменении *.go.

Решение
yaml
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
Ключевые механизмы
Ключ	Что делает
needs: []	Job запускается сразу, минуя ожидание стадий. Пайплайн становится DAG.
rules: changes	Job выполняется только если изменились указанные файлы. **/*.go — любые .go в любом каталоге.
Проверка
Коммит только README.md → стадия test пропущена

Коммит с .go → стадия test запущена

Включи Pipelines → Job dependencies для DAG-вида

Приложи в README.md:

Код оптимизированного .gitlab-ci.yml

Скриншот DAG-вида пайплайна

Скриншот пайплайна с изменением .go (тесты запущены)

Скриншот пайплайна без изменения .go (тесты пропущены)

Оформление README.md
markdown
# Занятие 8-03. GitLab

**Выполнил:** Иванов Иван

## Задание 1. Развёртывание GitLab и Runner
...команды, скриншоты...

## Задание 2. CI/CD пайплайн
```yaml
# код .gitlab-ci.yml
...скриншоты пайплайна...

Задание 3*. Оптимизация CI
yaml
# оптимизированный .gitlab-ci.yml
...скриншоты DAG...

text

### Как вставить скриншот
1. Создай папку `images/` в репозитории
2. Положи туда файлы (`task1-runners.png`, `task2-pipeline.png`)
3. В README.md:
```markdown
![Описание](images/task1-runners.png)
Коммит и пуш
bash
git status
git add .
git commit -m "Выполнено ДЗ по занятию 8-03: GitLab, runner, CI/CD"
git push origin main
