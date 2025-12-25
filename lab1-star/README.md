## 1. Общая информация

**Объект тестирования:**  
Собственный web-сайт: https://swrstats.ru/

**ПО сервера:**  
- Web-сервер: nginx/1.18.0 (Ubuntu)
- Протокол: HTTPS (порт 443)

**Цель работы:**  
Проверить минимум три уязвимости web-сервера nginx (path traversal, доступ к служебным файлам, ошибки конфигурации) и зафиксировать результат попыток взлома.


---

## 2. Используемые инструменты

- Терминал macOS
- Утилита `curl`
- Онлайн-сервис PentestTools 

---

## 3. Разведка и пассивный анализ (PentestTools)

По результатам сканирования PentestTools было выявлено:

- Используемый сервер: `nginx/1.18.0 (Ubuntu)`
- Отсутствуют HTTP-заголовки безопасности:
  - Strict-Transport-Security
  - Content-Security-Policy
  - X-Content-Type-Options
  - Referrer-Policy
- Обнаружены потенциально релевантные CVE для версии nginx
<img width="1138" height="722" alt="4" src="https://github.com/user-attachments/assets/88e5a753-60c3-4e50-bf38-7c397f12e8d0" />
---
## 4. Активное тестирование уязвимостей

### 4.1 Попытка Path Traversal 

```bash
curl -i "https://swrstats.ru/..%2f..%2fetc%2fpasswd"
```

Результат: `HTTP/1.1 400 Bad Request`

---
### 4.2 Попытка Path Traversal (URL-кодировка)

```bash
curl -i "https://swrstats.ru/%2e%2e%2f%2e%2e%2fetc%2fpasswd"
```

Результат: `HTTP/1.1 400 Bad Request`

---
<img width="487" height="227" alt="1" src="https://github.com/user-attachments/assets/1d427847-f348-4573-b491-a35040b90740" />

### 4.3 Проверка доступа к `.git`

```bash
curl -s -I https://swrstats.ru/.git/HEAD
```

Результат: возвращён `index.html` (SPA fallback)

---

### 4.4 Проверка файла `.env`

```bash
curl -s https://swrstats.ru/.env
```

Результат: возвращён `index.html`

---

### 4.5 Проверка файла `robots.txt`

```bash
curl -s https://swrstats.ru/robots.txt
```

Результат: возвращён `index.html`

---

### 4.6 Проверка HTTP-метода OPTIONS

```bash
curl -i -X OPTIONS https://swrstats.ru
```

Результат: `HTTP/1.1 200 OK`
<img width="1470" height="956" alt="2" src="https://github.com/user-attachments/assets/51577dc7-529e-48fc-900f-7f579a08c2fd" />
<img width="801" height="424" alt="3" src="https://github.com/user-attachments/assets/6b0b753a-c319-4e47-abb5-350391ed2a3c" />

---
## 5. Итог

Несанкционированный доступ к защищённым ресурсам не получен.  
Все попытки взлома зафиксированы и задокументированы.  
Требования лабораторной работы выполнены.

---


## 6. Заключение

В ходе выполнения лабораторной работы была проведена проверка безопасности web-сервера nginx на собственном сайте. 
Выполнены активные попытки выявления уязвимостей, включая проверку на path traversal, доступ к служебным файлам (.git, .env, robots.txt), а также анализ конфигурации HTTP-заголовков и поддерживаемых методов.
По результатам тестирования несанкционированный доступ к защищённым ресурсам получен не был. 
Сервер корректно обрабатывает некорректные запросы и не раскрывает конфиденциальные данные. 
Вместе с тем выявлены недостатки конфигурации безопасности, такие как отсутствие некоторых HTTP-заголовков защиты.
