# Домашнее задание к занятию "Практическое задание «Конфигурация почтового сервиса»" - Костенко Максим

## 1. Установка компонентов почтового сервера

**1.1. Установка модулей**

Где: ispmanager → Конфигурация ПО → Почтовый сервер (SMTP/POP3/IMAP).

Устанавливаем:

| Компонент | Назначение |
|---|---|
| Exim | Отправка почты (SMTP) |
| Dovecot (POP3/IMAP) | Хранение и получение почты |
| OpenDKIM (exim) | Цифровая подпись писем |
| SpamAssassin | Фильтр спама |
| ClamAV | Антивирус для писем |
| Sieve | Сортировщик писем и фильтры |
| Roundcube | Веб-клиент для работы с почтой |

Нажимаем Установить / Сохранить.

<img width="208" height="338" alt="image" src="https://github.com/user-attachments/assets/62e97343-1a76-41d0-9554-687b3424d0f2" />

**1.2. Создание почтового домена**

Где: ispmanager → Почта → Почтовые домены → Создать почтовый домен.

Заполняем:

| Поле | Значение |
|---|---|
| Имя | mk-industrias.online |
| Владелец | пользователь-владелец |
| IP-адрес | 192.168.10.32 (внутренний IP web-a) |
| Действие по умолчанию | «Сообщение об ошибке» |
| Включить DKIM | ✅ |
| Включить DMARC | ✅ |
| Защищённое соединение SSL | ✅ |
| SSL-сертификат | «Новый SSL Let's Encrypt» |
| Псевдоним для сертификата | mail.mk-industrias.online |
| Email | ваш контактный email |

Нажимаем Создать.

<img width="547" height="1036" alt="image" src="https://github.com/user-attachments/assets/a4fab6d4-669e-4299-9876-6e82ce462d32" />

**1.3. Создание почтового ящика**

Где: ispmanager → Почта → Создать почтовый ящик.

| Поле | Значение |
|---|---|
| Имя | info |
| Домен | mk-industrias.online |
| Пароль | задайте надёжный пароль |
| Дисковая квота | 500 МБ |

Нажимаем Создать.

<img width="468" height="642" alt="image" src="https://github.com/user-attachments/assets/d41fa68e-0803-466a-86f6-f07d83f0ecab" />

## 2. Настройка DNS-записей у регистратора

**2.1. Где настраивать**

NS домена mk-industrias.online = ns1.reg.ru — значит, все записи добавляются в панели Reg.ru.

ispmanager создаёт локальную зону, которая не видна из интернета.

**3.2. Зайди в панель Reg.ru**

Открой reg.ru → личный кабинет → найди mk-industrias.online → Управление зоной.

**3.3. Добавь A-записи**

| Тип | Имя | Для чего добавляется |
|---|---|---|
| TXT | _acme-challenge | Подтверждение владения доменом для выпуска SSL-сертификата Let's Encrypt (DNS-валидация) |
| TXT | _acme-challenge.mail | Подтверждение владения поддоменом mail для выпуска SSL-сертификата Let's Encrypt (DNS-валидация) |
| TXT | _dmarc | Политика обработки писем, не прошедших SPF/DKIM. Указывает, что делать с подозрительными письмами от домена |
| A | @ | Основной IP-адрес домена. На него приходит входящая почта и открывается сайт |
| MX | @ | Адрес почтового сервера, куда доставлять входящую почту для домена |
| TXT | @ | SPF-запись: список серверов, которым разрешено отправлять почту от имени домена |
| TXT | dkim._domainkey | Публичный ключ DKIM: по нему получатели проверяют цифровую подпись писем |
| A | mail | IP-адрес почтового сервера (на него указывает MX-запись) |
| A | pop | IP-адрес сервера для получения почты по протоколу POP3 |
| A | smtp | IP-адрес сервера для отправки почты по протоколу SMTP |
| A | www | IP-адрес для доступа к сайту по имени www.mk-industrias.online |

## 4. Выпуск SSL-сертификата

**4.1. Выпуск**

Где: ispmanager → SSL-сертификаты → Создать → Let's Encrypt.

| Поле | Значение |
|---|---|
| Домены | mk-industrias.online, mail.mk-industrias.online |
| Email | ваш email |

Нажимаем Выпустить.

<img width="2228" height="218" alt="image" src="https://github.com/user-attachments/assets/8f19b6d0-3177-41cb-82ae-670bf1b99d14" />

4.2. Применение
Где: Почта → Почтовые домены → mk-industrias.online → Изменить.

В поле SSL-сертификат выбери выпущенный. Сохрани.

<img width="448" height="157" alt="image" src="https://github.com/user-attachments/assets/6752a381-37f6-43af-a86b-b5e381de7989" />

## 5. Проверка почты онлайн-инструментами

- dig +short mk-industrias.online A
- dig +short mk-industrias.online MX
- dig +short mail.mk-industrias.online A
- dig +short mk-industrias.online TXT
- dig +short dkim._domainkey.mk-industrias.online TXT
- dig +short _dmarc.mk-industrias.online TXT
- dig +short mk-industrias.online NS
- dig +short mk-industrias.online SOA
- dig +short -x 84.201.131.194
- host mk-industrias.online
- host mail.mk-industrias.online
- curl -fsSIL --max-time 5 https://mk-industrias.online/

## 6. Настройка фильтров и автоответчика

**6.1. Автоответчик**

Где: ispmanager → Почта → info@mk-industrias.online → Автоответчик.

| Поле | Значение |
|---|---|
| Тема | «Автоответ» |
| Текст | текст автоматического ответа |

Нажимаем Включить / Сохранить.

<img width="559" height="721" alt="image" src="https://github.com/user-attachments/assets/8258b8bf-8e7e-446a-9cf5-2c801b07670f" />

**6.2. Фильтр**

**a. Новый почтовый сортировщик**

| Поле | Что указать | Пояснение |
|---|---|---|
| Имя почтового фильтра | delete_spam | Произвольное имя фильтра. Лучше — латиницей, без пробелов |
| Сопоставление условий | Все истинны | Для одного условия разницы нет. Оставь по умолчанию |
| Расположить перед | ***поместить в конец*** | Оставь по умолчанию — фильтр будет последним в списке |

<img width="753" height="448" alt="image" src="https://github.com/user-attachments/assets/05c1956d-9b91-446d-80f2-54611408b101" />

Нажимаем «Далее».

**b. Новое условие сортировки**

| Поле | Что указать | Пояснение |
|---|---|---|
| Условие 1 | тема письма | Фильтр проверяет именно тему входящего письма |
| Не (галочка) | не отмечено | Без галочки — значит «содержит». С галочкой было бы «не содержит» |
| Условие | содержит | Проверяем, есть ли в теме слово «спам» |
| Значения | спам | Ключевое слово, по которому срабатывает фильтр |
| Условие 2 | не выбрано | Второе условие не нужно |

<img width="749" height="544" alt="image" src="https://github.com/user-attachments/assets/db5f1bc3-cca5-41e4-9117-f2eeb7707216" />

Нажимаем «Далее».

**c. Новое действие сортировки**

| Поле | Что указать | Пояснение |
|---|---|---|
| Действие 1 | сохранить в директорию | Junk |

Нажимаем «Завершить».

Логика фильтра
```text
ЕСЛИ тема письма СОДЕРЖИТ "спам"
ТО сохранить в директорию Junk (спам)
```
Письмо, у которого в теме есть слово «спам», не попадёт во «Входящие» — Exim/Sieve удалит его сразу при приёме.

**6.3. Проверка через Roundcube**

Проверка через Roundcube

- Отправь письмо с Gmail на info@mk-industrias.online — проверь автоответ.
- Отправь письмо с темой «спам» — проверь фильтр.

Где: ispmanager → Почта → info@mk-industrias.online → Почтовый клиент.

1. Письма в Roundcube, отфильтрованные.

<img width="1118" height="284" alt="image" src="https://github.com/user-attachments/assets/6c36851e-c6c7-4c7b-a3ac-63768faed238" />

2. Письмо с автоответчиком.

Проверка локальной почты при блокировке порта 25

ispmanager → IP-адреса → NAT — включи, укажи:

Внутренний: 192.168.10.32

Внешний: 127.0.0.1

Сохрани.

<img width="472" height="302" alt="image" src="https://github.com/user-attachments/assets/acc72002-880c-4e0e-9167-84c1aba2513e" />

Создай 2 ящика: user1@mk-industrias.online и user2@mk-industrias.online.

<img width="2223" height="86" alt="image" src="https://github.com/user-attachments/assets/5d30e53d-877e-4faa-a290-084f2b777b9a" />

Отправь письмо с user1 на user2 через Roundcube. Потом ответь обратно. Оба должны дойти.

<img width="1200" height="288" alt="image" src="https://github.com/user-attachments/assets/297ca4fe-39a6-4923-910d-7b40216e4403" />

Проверка автоответчика, было получено сообщение в ответ
<img width="1618" height="294" alt="image" src="https://github.com/user-attachments/assets/732c0589-1f35-4480-9f78-e4c779cdec2b" />

Проверь автоответчик и фильтр — оба работают через локальный обмен.
