# Отчет по лабораторной работе: Реализация OAuth 2.0 (Authorization Code Flow)

---

## 1. Цели Работы

1. Ознакомиться с ролями и концепциями протокола **OAuth 2.0**.
2. Реализовать и протестировать поток **Authorization Code Flow** с использованием Python и библиотеки `requests-oauthlib`.
3. Успешно получить **Access Token** и использовать его для доступа к защищённому ресурсу (GitHub API).

---

## 2. Теоретические Сведения

**OAuth 2.0** — это протокол делегирования доступа, который позволяет приложению (Клиенту) получать доступ к ресурсам пользователя (Resource Owner) без прямого ввода его логина и пароля.

**Ключевые роли:**
* **Resource Owner:** Пользователь, владеющий данными.
* **Client:** Приложение, запрашивающее доступ.
* **Authorization Server:** Проверяет пользователя и выдает токены.
* **Resource Server:** Хранит данные и принимает Access Token.

**Authorization Code Flow:** Самый безопасный поток, где Клиент получает временный **Код Авторизации** через браузер, а затем обменивает его на **Access Token** через **защищенное серверное соединение** (back-channel).

---

## 3. Ход Работы и Результаты

### 3.1. Подготовка и Настройка Приложения

1.  **Настройка среды:** Установлены Python и библиотеки `requests-oauthlib`. Учетные данные (`CLIENT_ID` и `CLIENT_SECRET`) установлены как переменные окружения.
2.  **Конфигурация URI:** Для обеспечения безопасности и соответствия стандарту OAuth 2.0 (MUST utilize HTTPS) был установлен следующий URI: `REDIRECT_URI = "https://localhost:8000/callback"`.

<img width="669" height="494" alt="image" src="https://github.com/user-attachments/assets/f23df679-51c3-4d82-bd66-8387dde1c8d6" />


### 3.2. Выполнение Authorization Code Flow

Процесс был запущен с помощью скрипта `oauth_client.py`.

| Этап | Вывод Консоли и Действие | Вывод и Комментарий |
| :--- | :--- | :--- |
| **Шаг 1: Запрос кода** | Скрипт генерирует уникальный URL авторизации, содержащий `client_id` и `state`. | URL: `https://github.com/login/oauth/authorize?response_type=code&client_id=...&redirect_uri=https%3A%2F%2Flocalhost%3A8000%2Fcallback...`|
| **Шаг 2: Согласие** | Пользователь переходит по URL и нажимает **"Authorize"** на странице согласия GitHub, делегируя права `read:user`. | **[Вставьте Скриншот 3: Окно согласия]** *(Скриншот страницы GitHub с запросом разрешений.)* |

**Скриншот процесса обмена и доступа:**
<img width="974" height="488" alt="image" src="https://github.com/user-attachments/assets/854b8d3b-8cd7-4da1-beaa-dd5704b8b3ff" />

<img width="1378" height="701" alt="image" src="https://github.com/user-attachments/assets/e40f744e-42ee-4981-94aa-3e44c462ae8e" />


---
## 4. Ответы на Контрольные Вопросы

### 1. Роли в протоколе OAuth 2.0
Основные роли:
* **Resource Owner** (Владелец Ресурса): Пользователь, который обладает данными и делегирует права доступа.
* **Client** (Клиент): Приложение, запрашивающее доступ к данным от имени пользователя.
* **Authorization Server** (Сервер Авторизации): Система, которая проверяет подлинность пользователя и выдает токены.
* **Resource Server** (Сервер Ресурсов): Сервер, который хранит данные и предоставляет их на основе валидного Access Token.

### 2. Что такое Authorization Code и зачем он нужен?
**Authorization Code** — это временный, одноразовый код, выдаваемый Сервером Авторизации. Он используется для **безопасного обмена** на **Access Token** через прямое серверное соединение (back-channel), что исключает возможность перехвата самого токена в незащищенном браузере (front-channel).

### 3. Почему параметр state важен в запросе на авторизацию?
Параметр **`state`** важен для **предотвращения CSRF-атак** (Cross-Site Request Forgery). Клиент генерирует уникальное случайное значение `state` и отправляет его в запросе. Сервер Авторизации возвращает это же значение. Клиент проверяет совпадение, гарантируя, что полученный ответ с кодом авторизации соответствует ожидаемому запросу, и не был подделан злоумышленником.

### 4. Чем отличается Authorization Code Flow от Client Credentials Flow?
| Поток | Цель | Участие Пользователя |
| :--- | :--- | :--- |
| **Authorization Code** | Делегирование доступа к **ресурсам пользователя**. | **Обязательно** (аутентификация и согласие). |
| **Client Credentials** | Доступ к **ресурсам самого Клиента** (сервис-в-сервис). | **Отсутствует**. |

### 5. В каких случаях используется refresh token и какова его роль?
**Refresh Token** используется, когда **Access Token истекает** (обычно через короткий промежуток времени). Его роль — позволить Клиенту получить новый **Access Token** без необходимости заставлять пользователя проходить полный цикл авторизации и повторно вводить свои учетные данные.

---
## 6. Задание 2: Обновление Токена (Refresh Token)

Целью данного задания было реализовать механизм получения **Refresh Token** для обеспечения долгосрочного доступа к API Google, не требуя повторной авторизации пользователя после истечения срока действия **Access Token**.

### 6.1. Настройка и Ключевой Параметр

1.  **Настройка:** Были установлены специализированные библиотеки Google Auth (`google-auth-oauthlib`) и зарегистрировано новое OAuth-приложение в Google Cloud Console (проект **My Project 45304**).
2.  **Ключевой момент:** Для получения Refresh Token в URL авторизации был явно указан параметр: `access_type='offline'`.

### 6.2. Ход Выполнения и Результат

После устранения ошибок с проверкой приложения (добавление аккаунта `diamorozov@gmail.com` в тестировщики), поток был успешно завершен:

| Этап | Описание Действий |
| :--- | :--- |
| **Запрос URL** | Скрипт выводит URL авторизации с параметрами `access_type=offline` и `scope=photoslibrary.readonly`. |
| **Авторизация** | Пользователь проходит аутентификацию и предоставляет согласие приложению **"Refresh Token Lab"**. |
| **Обмен на токены** | URL перенаправления, содержащий код авторизации, вставляется в консоль для обмена на токены. |

**Результат обмена токенами (JSON-ответ):**

<img width="974" height="486" alt="image" src="https://github.com/user-attachments/assets/9d4c0ecc-84a3-4647-a86a-db71cc7a68b9" />


```
(.venv) PS C:\Users\diamo\PycharmProjects\oauth> python google_refresh_token.py
--- Шаг 1: Запрос кода авторизации ---
1. Перейдите по этой ссылке, чтобы авторизовать доступ:
https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=64408650301-92ce99bdav1ehep7q5pb54eeocqlf1oi.apps.googleusercontent.com&redirect_uri=https%3A%2F%2Flocalhost%
3A8080%2Fauth%2Fgoogle%2Fcallback&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fphotoslibrary.readonly&state=XpWVXu69UQObcouDuPSECLHUk5zukc&access_type=offline&include_granted_scopes=true

--- Шаг 2: Ввод кода ---
2. Вставьте полный URL перенаправления из браузера (с параметрами code и state): https://localhost:8080/auth/google/callback?state=XpWVXu69UQObcouDuPSECLHUk5zukc&code=4/0AVGzR1DOHvFOJNEX6AYl1OBR41vuUIBMMG24Ae1GUZ71xUXaQhCOgYp9w4OMf0IByVbPGA&scope=https://www.googleapis.com/auth/photoslibrary.readonly

✅ Успешно получены токены:
{
  "token": "ya29.a0ATi6K2vNDlNQA3WXDEOm63...",
  "refresh_token": "1//0cYkr3jZfNtUfCgYIARAAGAw...",
  "token_uri": "https://oauth2.googleapis.com/token",
  "client_id": "64408650301-92ce99bdav1ehep7q5pb54eeocqlf1oi.apps.googleusercontent.com",
  "client_secret": "....",
  "scopes": [
    "https://www.googleapis.com/auth/photoslibrary.readonly"
  ],
  "universe_domain": "googleapis.com",
  "account": "",
  "expiry": "2025-10-25T15:21:02Z"
}

💡 Refresh Token успешно получен и готов к использованию для обновления Access Token.

```

### 6.3. Вывод по Refresh Token

1.  **Получение:** Успешное присутствие поля **`"refresh_token"`** в ответе JSON подтверждает корректность использования **ключевого параметра `access_type='offline'`** в запросе авторизации.
2.  **Назначение:** Этот токен имеет длительный срок действия и будет использоваться для запроса нового **Access Token** после истечения текущего (в данном случае: `expiry: 2025-10-25T15:21:02Z`). Обновление происходит путем отправки серверного **POST-запроса** с параметром **`grant_type=refresh_token`**.
3.  **Безопасность:** Полученный Refresh Token, наряду с Client Secret, является **долгосрочным секретом**. Он предоставляет постоянный доступ к данным пользователя, поэтому должен храниться **максимально безопасно** на стороне сервера Клиента и никогда не должен передаваться через браузер.
