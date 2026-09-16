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
