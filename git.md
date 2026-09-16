## Задание 1

Что нужно сделать:

1. Зарегистрируйте аккаунт на GitHub.
2. Создайте новый отдельный публичный репозиторий. Обязательно поставьте галочку в поле «Initialize this repository with a README».
3. Склонируйте репозиторий, используя https протокол git clone ....
4. Перейдите в каталог с клоном репозитория.
5. Произведите первоначальную настройку Git, указав своё настоящее имя и email: git config --global user.name и git config --global user.email johndoe@example.com.
6. Выполните команду git status и запомните результат.
7. Отредактируйте файл README.md любым удобным способом, переведя файл в состояние Modified.
8. Ещё раз выполните git status и продолжайте проверять вывод этой команды после каждого следующего шага.
9. Посмотрите изменения в файле README.md, выполнив команды git diff и git diff --staged.
10. Переведите файл в состояние staged или, как говорят, добавьте файл в коммит, командой git add README.md.
11. Ещё раз выполните команды git diff и git diff --staged.
12. Теперь можно сделать коммит git commit -m 'First commit'.
13. Сделайте git push origin master.

В качестве ответа добавьте ссылку на этот коммит в ваш md-файл с решением.

## Решение 1

**1. Регистрация аккаунта на GitHub**

Выполнено ранее. Аккаунт GitHub уже зарегистрирован, веду собственные репозитории.

Примечание: если аккаунт новый — регистрация выполняется на https://github.com/signup с указанием email, имени пользователя и пароля.

**2. Создание нового публичного репозитория**

Действия:
- Открыл https://github.com/new.
- Указал Repository name (например, git-practice-1).
- Выбрал тип репозитория — Public.
- Поставил галочку «Add a README file» (в интерфейсе GitHub это поле называется «Initialize this repository with a README»).
- Нажал Create repository.

Результат: создан публичный репозиторий с автоматически сгенерированным файлом README.md и веткой main (или master — зависит от настроек аккаунта).

**3. Клонирование репозитория по HTTPS**

Скопировал URL репозитория (кнопка Code → HTTPS).
```bash
git clone https://github.com/<username>/git-practice-1.git
```
Ожидаемый вывод:

<img width="1211" height="176" alt="image" src="https://github.com/user-attachments/assets/2c71fd7e-d9d3-4d82-ba7d-fe0fc76d4dc5" />

**4. Переход в каталог с клоном**
```bash
cd git-practice-1
```
**5. Первоначальная настройка Git**
Указал своё настоящее имя и email (глобально, для всех репозиториев):
```bash
git config --global user.name "Maksim Kostenko"
git config --global user.email "i297@bk.ru"

# Проверка настроек:
git config --global --list
```
Ожидаемый вывод:

<img width="1045" height="101" alt="image" src="https://github.com/user-attachments/assets/caa51818-ac20-4d1e-bda9-9b90590e92a6" />

**6. Первый вызов git status**
```bash
git status
```
Ожидаемый вывод:

<img width="852" height="120" alt="image" src="https://github.com/user-attachments/assets/c9451733-e18a-42c9-b2b2-1473724f94ae" />

Пояснение: рабочая директория чистая — все файлы соответствуют последнему коммиту, изменений нет.

**7. Редактирование README.md**

Открыл файл README.md в текстовом редакторе и добавил строку, например:
```markdown
# git-practice-1

Практическое задание по Git. Первый коммит.
```
Сохранил файл. Файл перешёл в состояние Modified.

**8. git status после редактирования**
```bash
git status
```
Ожидаемый вывод:

<img width="858" height="232" alt="image" src="https://github.com/user-attachments/assets/90233e1e-bade-452a-b17b-ba9061172f6a" />

Пояснение: Git видит, что README.md изменён, но пока не добавлен в индекс (staging area).

**9. Просмотр изменений: git diff и git diff --staged**

До добавления в индекс:

```bash
git diff
```
Ожидаемый вывод:

<img width="825" height="346" alt="image" src="https://github.com/user-attachments/assets/047e4bd9-ae5b-46ed-8908-315707add94e" />

После добавления в индекс пока ничего не staged, поэтому:
```bash
git diff --staged
```
Ожидаемый вывод: пустой — индекс совпадает с HEAD.

**10. Добавление файла в индекс (staged)**
```bash
git add README.md

# Проверка статуса:
git status
```
Ожидаемый вывод:

<img width="850" height="212" alt="image" src="https://github.com/user-attachments/assets/37c0315c-179c-4588-b402-027b7a923984" />

Файл перешёл в состояние Staged.

**11. Повторные git diff и git diff --staged**
```bash
git diff
```
Ожидаемый вывод: пустой — рабочая директория совпадает с индексом.
```bash
git diff --staged
```
Вывод: показывает те же изменения, что и раньше git diff, но теперь они находятся в индексе (готовы к коммиту):

<img width="938" height="346" alt="image" src="https://github.com/user-attachments/assets/06edfc45-b822-4ecd-9cff-1d410b1e05e6" />

**12. Создание коммита**
```bash
git commit -m 'First commit'
```
Ожидаемый вывод:

<img width="1059" height="74" alt="image" src="https://github.com/user-attachments/assets/9acc3bdc-ab74-4c4c-b400-393095444c2b" />

Проверка:
```bash
git status
```
Ожидаемый вывод:

<img width="853" height="168" alt="image" src="https://github.com/user-attachments/assets/69c5c13b-9c93-4366-bf9b-817879800039" />

**13. Отправка изменений на GitHub**

Так как GitHub по умолчанию создаёт ветку main, отправляем именно её:
```bash
git push origin main
```
Вывод:

<img width="969" height="261" alt="image" src="https://github.com/user-attachments/assets/7f8cb4af-828f-40fd-9bdb-f0572dfc184d" />

Проверка: открыл страницу репозитория на GitHub — файл README.md содержит новые строки, история коммитов показывает First commit.

## Задание 2

Что нужно сделать:

1. Создайте файл .gitignore (обратите внимание на точку в начале файла) и проверьте его статус сразу после создания.
2. Добавьте файл .gitignore в следующий коммит git add....
3. Напишите правила в этом файле, чтобы игнорировать любые файлы .pyc, а также все файлы в директории cache.
4. Сделайте коммит и пуш.

В качестве ответа добавьте ссылку на этот коммит в ваш md-файл с решением.

## Решение 2

**1. Создание файла .gitignore**

Важно: файл начинается с точки — это скрытый файл, и в проводнике Windows он по умолчанию не виден.

Создаём пустой файл в PowerShell:
```powershell
New-Item -Path .gitignore -ItemType File
```
Ожидаемый вывод:

<img width="905" height="185" alt="image" src="https://github.com/user-attachments/assets/04a3e313-1075-4c66-8319-e61f7e0c20d4" />

Проверить, что файл создан:
```powershell
Get-ChildItem -Force
```
<img width="728" height="224" alt="image" src="https://github.com/user-attachments/assets/90e62d0b-4de6-4561-9883-45c6107b4cad" />

Флаг -Force показывает скрытые файлы (те, что начинаются с точки).

**2. Проверка статуса сразу после создания**
```powershell
git status
```
Ожидаемый вывод:

<img width="688" height="182" alt="image" src="https://github.com/user-attachments/assets/bdd237d4-5ac7-4b19-ace7-0e86845ff404" />

Что это значит: Git видит новый файл .gitignore, но пока не отслеживает его — файл в состоянии Untracked. Это нормально: любой новый файл начинается именно так.

**3. Написать правила в .gitignore**

Открываем файл в Блокноте:
```powershell
notepad .gitignore
```
Добавляем два правила:
```text
# Игнорировать все скомпилированные Python-файлы
*.pyc

# Игнорировать всё содержимое директории cache
cache/
```
Пояснения:
- *.pyc — игнорирует любые файлы, оканчивающиеся на .pyc, в любой папке проекта (сюда попадают .pyc, .pyo, скомпилированный байт-код Python).
- cache/ — слэш в конце указывает, что это директория. Git будет игнорировать всё содержимое папки cache в корне проекта. Если нужно игнорировать cache в любой вложенной папке — используйте **/cache/.
- Сохраните (Ctrl+S) и закройте Блокнот.

Проверьте содержимое:
```powershell
Get-Content .gitignore
```

**4. Добавить .gitignore в индекс**
```powershell
git add .gitignore

# Проверьте статус:
git status
```
Ожидаемый вывод:

<img width="636" height="147" alt="image" src="https://github.com/user-attachments/assets/2d19dcb2-085c-471e-bc06-f5ce02c76d47" />

Файл перешёл в состояние Staged.

**5. Коммит**
```powershell
git commit -m "Add .gitignore with rules for *.pyc and cache/"
```
Ожидаемый вывод:

<img width="1082" height="126" alt="image" src="https://github.com/user-attachments/assets/a3f02f89-dc64-422f-85a7-47c86ba8cffd" />

Запомните хэш коммита — он понадобится для ссылки. Узнать в любой момент:
```powershell
git rev-parse HEAD
git log --oneline -1
```
**6. Push на GitHub**
```powershell
git push origin main
```
Ожидаемый вывод:

<img width="734" height="244" alt="image" src="https://github.com/user-attachments/assets/0bcd410e-37ff-4dd1-bea3-3ecf019663b6" />

💡 Если push снова отклонён с fetch first — значит, на GitHub появился новый коммит (например, вы правили README через веб-интерфейс). Сделайте:

```powershell
git pull origin main
git push origin main
```
При конфликте — разрешите, как в Задании 1.

**7. Проверка**

Локально:
```powershell
git status
```
<img width="646" height="124" alt="image" src="https://github.com/user-attachments/assets/81a0c6d0-91d8-4628-a652-b9ac16129d8f" />

```powershell
git log --oneline -3
```
Пример вывода:

<img width="861" height="85" alt="image" src="https://github.com/user-attachments/assets/ddf57947-207a-45d5-9bb6-acc0430c4ac3" />

На GitHub: откройте https://github.com/M4KlK0S/git-practice-1 — в корне должен появиться файл .gitignore с вашими правилами.

**8. Проверка, что .gitignore реально работает (необязательно, но полезно)**

Создайте тестовый файл, который должен игнорироваться:
```powershell
New-Item -Path test.pyc -ItemType File
New-Item -Path cache -ItemType Directory -Force
New-Item -Path .\cache\data.txt -ItemType File
```
Теперь проверьте статус:
```powershell
git status
```
Убедиться, что Git действительно игнорирует:
```powershell
git check-ignore -v test.pyc cache/data.txt
```
Ожидаемый вывод:

<img width="933" height="58" alt="image" src="https://github.com/user-attachments/assets/2ac28df1-f50e-4932-ad17-6189f067327b" />

— Git показывает, какое именно правило сработало.

Удалите тестовые файлы, чтобы не засорять проект:
```powershell
Remove-Item test.pyc -Force
Remove-Item cache -Recurse -Force
```
## Задание 3

Что нужно сделать:
1. Создайте новую ветку dev и переключитесь на неё.
2. Создайте в ветке dev файл test.sh с произвольным содержимым.
3. Сделайте несколько коммитов и пушей в ветку dev, имитируя активную работу над файлом в процессе разработки.
4. Переключитесь на основную ветку.
5. Добавьте файл main.sh в основной ветке с произвольным содержимым, сделайте комит и пуш . Так имитируется продолжение общекомандной разработки в основной ветке во время разработки отдельного функционала в dev ветке.
6. Сделайте мердж dev ветки в основную с помощью git merge dev. Напишите осмысленное сообщение в появившееся окно комита.
7. Сделайте пуш в основной ветке.
8. Не удаляйте ветку dev.

В качестве ответа прикрепите ссылку на граф коммитов https://github.com/ваш-логин/ваш-репозиторий/network в ваш md-файл с решением.

Ваш граф комитов должен выглядеть аналогично скриншоту:

<img width="245" height="104" alt="image" src="https://github.com/user-attachments/assets/9b976a58-ecf4-439e-ad6a-1b4c6f965c47" />

В качестве ответа добавьте ссылку на этот коммит в ваш md-файл с решением.

## Решение 3

**1. Создание ветки dev и переключение на неё**

Вариант через git checkout (классический, работает везде):
```powershell
git checkout -b dev
```
Вариант через git switch (современный, Git 2.23+):
```powershell
git switch -c dev
```
Ожидаемый вывод:

<img width="720" height="43" alt="image" src="https://github.com/user-attachments/assets/5ca4ed18-c515-4c0a-a7d0-337693bcdf1c" />

Проверьте, что вы на dev:
```powershell
git branch
```
<img width="633" height="60" alt="image" src="https://github.com/user-attachments/assets/c0840724-e845-4138-95d1-a2a548f0a56c" />

Флаг -b (или -c в switch) означает «создать и сразу переключиться».

**2. Создание файла test.sh с произвольным содержимым**
```powershell
@"
#!/bin/bash
# Скрипт для тестирования функционала dev-ветки
echo "Hello from dev branch!"
echo "Разработка продолжается..."
"@ | Set-Content -Path test.sh -Encoding UTF8
```
Проверка:
```powershell
Get-Content test.sh
```
<img width="726" height="116" alt="image" src="https://github.com/user-attachments/assets/e0098964-1fbc-46c4-86e9-100f88ce0317" />

Статус:
```powershell
git status
```
<img width="692" height="142" alt="image" src="https://github.com/user-attachments/assets/1ad88c47-fd3e-4b67-b281-2e701e260937" />

**3. Несколько коммитов и пушей в dev (имитация активной работы)**

Первый коммит:
```powershell
git add test.sh
git commit -m "Add initial test.sh"
git push -u origin dev
```
Флаг -u (--set-upstream) связывает локальную ветку dev с удалённой origin/dev — дальше можно писать просто git push.

Ожидаемый вывод push:
<img width="899" height="401" alt="image" src="https://github.com/user-attachments/assets/fba7399d-1589-4f3f-b61a-d32c6ac061ea" />

Второй коммит — вносим изменения в test.sh:
```powershell
@"
#!/bin/bash
# Скрипт для тестирования функционала dev-ветки
echo "Hello from dev branch!"
echo "Разработка продолжается..."
echo "Вторая итерация: добавлена новая функциональность"
"@ | Set-Content -Path test.sh -Encoding UTF8

git add test.sh
git commit -m "Extend test.sh with new functionality"
git push
```
Ожидаемый вывод push:
<img width="1028" height="441" alt="image" src="https://github.com/user-attachments/assets/14ba8207-6f8a-4bed-a45a-3773e5eb8176" />

Третий коммит — ещё одна итерация:
```powershell
Add-Content -Path test.sh -Value 'echo "Третья итерация: bugfix и улучшения"'
git add test.sh
git commit -m "Fix bugs and improve test.sh"
git push
```
Ожидаемый вывод push:
<img width="1080" height="339" alt="image" src="https://github.com/user-attachments/assets/ea0935ba-c7c7-450c-940e-e7e7f0e055d0" />

Проверка истории в dev:
```powershell
git log --oneline -5
```
Пример:
<img width="790" height="124" alt="image" src="https://github.com/user-attachments/assets/35b1bf43-d2c5-4fd9-bf1e-7b0af86980c6" />

**4. Переключение на основную ветку main**
```powershell
git checkout main
```
или
```powershell
git switch main
```
Ожидаемый вывод:
<img width="701" height="61" alt="image" src="https://github.com/user-attachments/assets/0ec0edc9-78ad-45c9-9e0f-13d38d1fdb7a" />

**5. Создание main.sh в основной ветке + коммит + push**
```powershell
@"
#!/bin/bash
# Основной скрипт проекта
echo "Main project script running..."
echo "Общекомандная разработка продолжается"
"@ | Set-Content -Path main.sh -Encoding UTF8

git add main.sh
git commit -m "Add main.sh in main branch"
git push origin main
```
Ожидаемый вывод push:

text
To https://github.com/M4KlK0S/git-practice-1.git
   528bf75..e5f6a7b  main -> main
Проверка статуса:

powershell
git status
Ветка main впереди origin/main ровно на 1 коммит (тот, который вы только что запушили), затем — up to date.

Шаг 6. Мердж ветки dev в main
Убедитесь, что вы на main:

powershell
git branch
Звёздочка должна быть напротив main.

Выполните:

powershell
git merge dev
Возможны два исхода:

А. Fast-forward (без конфликтов) — если main не менялся после ответвления
text
Updating 528bf75..b3c4d5e
Fast-forward
 test.sh | 5 +++++
 1 file changed, 5 insertions(+)
 create mode 100644 test.sh
Тогда merge-коммит не создаётся, а main просто перематывается на последний коммит dev.

Б. Merge-коммит (как на скриншоте) — если в main уже был свой коммит (main.sh)
Так как вы сделали коммит Add main.sh в main после ответвления dev, история разошлась. Git создаст merge-коммит и откроет редактор для сообщения:

text
Merge made by the 'ort' strategy.
 test.sh | 5 +++++
 1 file changed, 5 insertions(+)
 create mode 100644 test.sh
В окне редактора (vim или Notepad) напишите осмысленное сообщение, например:

text
Merge dev into main: завершена разработка test.sh
Сохраните и закройте:

vim: Esc, затем :wq, Enter.

Notepad: Ctrl+S, закрыть окно.

💡 Если хотите, чтобы вместо vim открывался Блокнот:

powershell
git config --global core.editor notepad
Либо задайте сообщение сразу, без редактора:

powershell
git merge dev -m "Merge dev into main: завершена разработка test.sh"
Проверьте историю:

powershell
git log --oneline --graph --all -10
Пример вывода (аналогично скриншоту):

text
*   c1d2e3f (HEAD -> main) Merge dev into main: завершена разработка test.sh
|\
| * b3c4d5e (dev, origin/dev) Fix bugs and improve test.sh
| * a2b3c4d Extend test.sh with new functionality
| * f1e2d3c Add initial test.sh
* | e5f6a7b (origin/main) Add main.sh in main branch
|/
* 528bf75 Merge remote changes and resolve conflict in README
Именно такой граф вы увидите на странице https://github.com/M4KlK0S/git-practice-1/network.

Шаг 7. Push в основной ветке
powershell
git push origin main
Ожидаемый вывод:

text
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Writing objects: 100% (4/4), done.
Total 4 (delta 0), reused 0 (delta 0)
To https://github.com/M4KlK0S/git-practice-1.git
   e5f6a7b..c1d2e3f  main -> main
Шаг 8. Не удалять ветку dev
Ничего делать не нужно — просто не выполняйте git branch -d dev или git push origin --delete dev. Ветка dev останется и локально, и на GitHub.

Проверка, что ветка на месте:

powershell
git branch -a
text
* main
  dev
  remotes/origin/dev
  remotes/origin/main
Ссылка на граф коммитов
Откройте в браузере:

text
https://github.com/M4KlK0S/git-practice-1/network
Это официальная страница GitHub, которая визуализирует историю ветвления в виде графа. Ваш граф должен показать:

Ветку dev, ответвившуюся от main.

Три коммита в dev.

Один коммит в main (Add main.sh), сделанный параллельно.

Merge-коммит, который соединяет обе ветки.

Это точь-в-точь соответствует скриншоту из задания.

Добавление ссылки в md-файл с решением
Откройте ваш файл с решением и добавьте:

markdown
## Задание 3

**Ссылка на граф коммитов:** https://github.com/M4KlK0S/git-practice-1/network

**Структура веток:**
- `main` — основная ветка
- `dev` — ветка разработки, сохранена (не удалена)

**Ключевые коммиты:**
- Add initial test.sh
- Extend test.sh with new functionality
- Fix bugs and improve test.sh
- Add main.sh in main branch
- Merge dev into main
Затем закоммитьте и запушьте:

powershell
git add README.md
git commit -m "Add solution link for task 3"
git push origin main
