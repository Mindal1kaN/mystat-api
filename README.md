# Документація API MyStat

Цей документ описує ендпоінти API MyStat для взаємодії з платформою, включаючи автентифікацію, керування профілем, отримання балансу, список уроків, керування домашніми завданнями, функціонал магазину, розклад уроків та лідерборд.

**Базовий URL:** `https://mapi.itstep.org/v1/mystat/`

## Примітки

-   **Автентифікація:** Усі ендпоінти, крім `/auth/login`, потребують заголовок `authorization: Bearer <JWT_TOKEN>`.
-   **`branch_alias`:** Поле `"branch_alias": "pl"` у відповіді `/auth/me` визначає префікс для ендпоінтів, пов'язаних з філією. Наприклад, якщо `branch_alias = "pl"`, то ендпоінти починаються з `/pl/`. Замініть `"pl"` на відповідний alias у всіх релевантних ендпоінтах.
-   **Статус домашніх завдань:** `3` — "До виконання", `2` — "На перевірці", `1` — "Виконано".
-   **Статус замовлень:** `3` — "Замовлено".
-   **Пагінація:** Ендпоінт продуктів магазину підтримує пагінацію через параметри `page` та `per_page`.
-   **Валюта:** Ціни у магазині вказані у двох валютах: `COI` (монети) та `DIA` (кристали).
-   **Файли:** Директорії для завантаження файлів є унікальними для кожного користувача.
-   **Помилки:** Обробка помилок не деталізована, але має відповідати стандартним практикам API.

---

## Ендпоінти

### Автентифікація

#### Логін

Автентифікує користувача та повертає JWT токен.

-   **Метод:** `POST`
-   **URL:** `/auth/login`
-   **Тіло запиту:**
    ```json
    {
      "login": "<логін>",
      "password": "<пароль>"
    }
    ```
-   **Відповідь (200 OK):**
    ```
    eyJ0eXAiOiJKV1QiLCJhbGciO...
    ```

---

### Профіль

#### Отримати профіль користувача

Отримує інформацію про автентифікованого користувача.

-   **Метод:** `POST`
-   **URL:** `/auth/me`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Тіло запиту:** `{}`
-   **Відповідь (200 OK):**
    ```json
    {
      "id": "<user-id>",
      "name": "<повне_ім'я>",
      "branch_alias": "pl",
      // ... інші поля профілю
    }
    ```

---

### Баланс

#### Отримати баланс користувача

Отримує баланси користувача (кристали, монети, досягнення).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/gaming/get-balances`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "alias": "cristals",
        "value": "<значення>"
      },
      {
        "alias": "coins",
        "value": "<значення>"
      },
      {
        "alias": "achivements",
        "value": "<значення>"
      }
    ]
    ```

---

### Уроки

#### Отримати всі уроки

Отримує список усіх доступних предметів.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/settings/group-specs`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):**
    ```json
    [
      {"id": "<spec-id>", "name": "Illustrator", "short_name": "Illust."},
      {"id": "<spec-id>", "name": "Java Script", "short_name": "JavScr"}
      // ... інші предмети
    ]
    ```

---

### Домашні завдання

#### Отримати домашні завдання

Отримує список домашніх завдань за статусом.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?status=<status>&limit=1000&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Параметри:**
    -   `status`: `1` (Виконано), `2` (На перевірці), `3` (До виконання)
-   **Відповідь (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "<homework-id>",
          "theme": "<тема_завдання>",
          "name_spec": "<назва_предмета>",
          "status": "<статус>"
          // ... інші поля домашнього завдання
        }
      ]
    }
    ```

#### Отримати домашні завдання за певними предметами (фільтр)

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?limit=1000&spec_id=<spec-id1>,<spec-id2>&status=3&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`

#### Отримати токен для файлів

Отримує токен та директорії для завантаження файлів.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/user/file-token`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):**
    ```json
    {
      "token": "<file-token>",
      "domain": "https://fsx3.itstep.org",
      "directories": {
        "homeworkDirId": "<homework-dir-id>"
        // ... інші директорії
      },
      "url": "https://fsx3.itstep.org/api/v1/files"
    }
    ```

#### Завантажити файл для домашнього завдання

-   **Метод:** `POST`
-   **URL:** `https://fsx3.itstep.org/api/v1/files`
-   **Заголовки:** `authorization: Bearer <FILE_TOKEN>`
-   **Тіло запиту (multipart/form-data):**
    -   `files[]`: `<файл у байтах>`
    -   `directory`: `"<homeworkDirId>"`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<id_файлу>",
        "link": "<посилання_на_файл>"
      }
    ]
    ```

#### Відправити домашнє завдання

Відправляє домашнє завдання з текстовою відповіддю та/або файлом.

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/homework/create`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Тіло запиту:**
    ```json
    {
      "answerText": "<текст_відповіді>",
      "filename": "<посилання_на_файл>",
      "id": "<homework-id>"
    }
    ```
-   **Відповідь (201 Created):**
    ```json
    {
      "id": "<student-homework-id>",
      "status": 2
      // ... інші поля
    }
    ```

#### Видалити домашнє завдання

-   **Метод:** `DELETE`
-   **URL:** `/{branch_alias}/homework/delete/<student-homework-id>`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):** `true`

---

### Розклад

#### Отримати розклад на тиждень

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/schedule/get-month?type=week&date_filter=YYYY-MM-DD`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):**
    ```json
    {
      "data": [
        {
          "date": "YYYY-MM-DD",
          "subject_name": "<назва_предмета>",
          "lesson_theme": "<тема_уроку>"
          // ... інші поля розкладу
        }
      ]
    }
    ```

---

### Лідерборд

#### Отримати лідерборд

Отримує лідерборд для групи та потоку.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/progress/leader-table`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):**
    ```json
    {
      "group": {
        "top": [
          {
            "fio_stud": "<ПІБ_студента>",
            "amount": "<кількість_балів>",
            "position": "<позиція>",
            "current": true
          }
        ]
      },
      "stream": {
        "top": [
          // ... аналогічна структура
        ]
      }
    }
    ```

---

### Магазин

#### Отримати продукти магазину

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/products?page=1&per_page=16`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<product-id>",
        "title": "<назва_продукту>",
        "prices": [
          {"currency": "COI", "price": "<ціна>"},
          {"currency": "DIA", "price": "<ціна>"}
        ]
        // ... інші поля продукту
      }
    ]
    ```

#### Отримати пов’язані продукти

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/products/get-others?product_id=<product-id>`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`

#### Створити замовлення

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/shop/create-order`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Тіло запиту:**
    ```json
    {
      "products": [
        {
          "product_id": "<product-id>",
          "quantity": "<кількість>"
        }
      ],
      "merchant": "Gaming"
    }
    ```
-   **Відповідь (200 OK):**
    ```json
    {
      "status": "succeeded",
      "uuid": "<order-uuid>"
    }
    ```

#### Отримати замовлення

Отримує список замовлень зі статусом `3` (Замовлено).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/orders`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<order-id>",
        "uuid": "<order-uuid>",
        "status": 3,
        "products": [
          {
            "title": "<назва_продукту>",
            "quantity": "<кількість>"
          }
        ]
        // ... інші поля замовлення
      }
    ]
    ```

#### Скасувати замовлення

-   **Метод:** `PUT`
-   **URL:** `/{branch_alias}/shop/discard-order`
-   **Заголовки:** `authorization: Bearer <JWT_TOKEN>`
-   **Тіло запиту:**
    ```json
    {
      "uuid": "<order-uuid>"
    }
    ```
-   **Відповідь (200 OK):**
    ```json
    {
      "status": "success"
    }
    ```
