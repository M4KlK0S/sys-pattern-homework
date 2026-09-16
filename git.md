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
