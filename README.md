# Документація API MyStat

Цей документ описує ендпоінти API MyStat для взаємодії з платформою, включаючи автентифікацію, керування профілем, отримання балансу, список уроків, керування домашніми завданнями, функціонал магазину, розклад уроків, лідерборд, а також додаткові функції для отримання оцінок, кількості домашніх завдань, нарахувань активності та відвідуваності. Інформація від сервера часто приходить англійською мовою, тому рекомендується всі відповіді обробляти через ендпоінт перекладу для локалізації на українську (токен автентифікації не потрібен).

**Базовий URL:** `https://mapi.itstep.org/v1/mystat/`

---

## Примітки

-   **Переклад відповідей:** Оскільки багато даних від сервера надходять англійською, використовуйте ендпоінт перекладу для обробки. Запит: `GET https://mapi.itstep.org/v1/translate?lang_id=uk`. Відповідь — це JSON-об'єкт з великою кількістю пар ключ-значення у форматі `"<англійський_термін>": "<український_переклад>"`, де ключі — англійські терміни, а значення — їхні українські еквіваленти. Цей словник можна використовувати для автоматичного перекладу полів у відповідях API.
-   **Автентифікація:** Усі ендпоінти, крім `/auth/login` та перекладу, потребують заголовок `authorization: Bearer <токен>`. Якщо токен закінчився або недійсний, сервер поверне помилку у форматі:
    ```json
    {
        "name": "Internal Server Error",
        "message": "Виникла внутрішня помилка сервера.",
        "code": 0,
        "status": 500
    }
    ```
-   **Помилки автентифікації:** При неправильному логіні або паролі в ендпоінті `/auth/login` сервер повертає "Unauthorized: error_unauthorized" з кодом 401.
-   **`branch_alias`:** Поле `"<branch_alias>": "pl"` у відповіді `/auth/me` визначає префікс для ендпоінтів, пов'язаних з філією. Наприклад, якщо `branch_alias = "pl"`, то ендпоінти починаються з `/pl/` (наприклад, `/pl/gaming/get-balances`). Якщо `branch_alias = "kiev"`, то ендпоінти починаються з `/kiev/`. Це дозволяє динамічно формувати URL на основі філії користувача. Замініть `"pl"` на відповідний alias у всіх релевантних ендпоінтах.
-   **Статус домашніх завдань:** `3` — "До виконання", `2` — "На перевірці", `1` — "Виконано".
-   **Статус замовлень:** `3` — "Замовлено".
-   **Пагінація:** Деякі ендпоінти (наприклад, продукти магазину чи нарахування активності) підтримує пагінацію через параметри `page` та `per_page`.
-   **Валюта:** Ціни у магазині вказані у двох валютах: `COI` (монети) та `DIA` (кристали).
-   **Файли:** Відповіді для домашніх завдань та продуктів містять URL-адреси файлів. Директорії для завантаження файлів є унікальними для кожного користувача.
-   **Помилки:** Усі успішні відповіді у цій документації повертають код 200 або 201. Обробка помилок не деталізована, але має відповідати стандартним практикам API (наприклад, 401 для неавторизованих запитів, 500 для внутрішніх помилок).

---

## Ендпоінти

### Автентифікація

#### Логін
Цей ендпоінт автентифікує користувача та повертає JWT-токен для подальших запитів. Якщо логін або пароль неправильні, повертається помилка "Unauthorized: error_unauthorized" з кодом 401.

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
    <токен>
    ```

### Профіль

#### Отримати профіль користувача
Цей ендпоінт повертає детальну інформацію про автентифікованого користувача, включаючи `branch_alias` для формування URL інших ендпоінтів, контактні дані, властивості профілю та зовнішні інтеграції.

-   **Метод:** `POST`
-   **URL:** `/auth/me`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Тіло запиту:** `{}`
-   **Відповідь (200 OK):**
    ```json
    {
      "id": "<user-id>",
      "id_user_storage": "<storage-id>",
      "email": "<email>",
      "username": "<логін>",
      "name": "<повне_ім'я>",
      "branch_id": "<branch-id>",
      "project_id": "<project-id>",
      "lang": "uk",
      "profile": "student",
      "module": "mystat",
      "branch_alias": "pl",
      "user_profiles": {
        "<project-id>": {
          "<branch-id>": [
            {
              "user_id": "<user-id>",
              "login": "<логін>",
              "type": "student",
              "status": "active",
              "project_id": "<project-id>",
              "branch_id": "<branch-id>",
              "photo": "",
              "position_id": null,
              "updated_by": "<updated-by-id>",
              "created_by": "<created-by-id>",
              "externals": [
                {"profile_id": "<profile-id>", "ext_type": "azure", "external_id": "<azure-id>"},
                {"profile_id": "<profile-id>", "ext_type": "onec", "external_id": "<onec-id>"},
                {"profile_id": "<profile-id>", "ext_type": "mystat", "external_id": "<mystat-id>"}
              ],
              "contacts": [
                {"channel": "address", "value": "<адреса>", "main": true, "disabled": false},
                {"channel": "mail", "value": "<email>", "main": true, "disabled": false},
                {"channel": "sms", "value": "<телефон>", "main": true, "disabled": false}
              ],
              "properties": [
                {"key": "passport", "value": "<номер_паспорта>"},
                {"key": "passissued", "value": "<виданий_паспорт>"},
                {"key": "cindnumber", "value": "<номер_ідентифікації>"},
                {"key": "stream_id", "value": "<stream-id>"},
                {"key": "class_number", "value": "<class-number>"},
                {"key": "product_id", "value": "<product-id>"},
                {"key": "integration_azure_disabled", "value": "0"},
                {"key": "id_tgroups", "value": "<group-id>"},
                {"key": "st_form", "value": ""},
                {"key": "st_study", "value": ""},
                {"key": "st_work", "value": ""}
              ],
              "id": "<profile-id>",
              "branch_alias": "pl",
              "logbook_id": "<logbook-id>",
              "onec_id": "<onec-id>"
            }
          ]
        }
      },
      "user_storage": {
        "firstname": "<ім'я>",
        "lastname": "<прізвище>",
        "patronymic": "<по_батькові>",
        "birthdate": "<дата_народження>",
        "gender": "male",
        "branch_id": null,
        "updated_by": "<updated-by-id>",
        "created_by": "<created-by-id>",
        "id": "<storage-id>"
      },
      "childs": []
    }
    ```

### Баланс

#### Отримати баланс користувача
Цей ендпоінт повертає баланси користувача, включаючи кристали (cristals), монети (coins) та досягнення (achivements), з деталями про значення, рейтинг та дати оновлення.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/gaming/get-balances`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<balance-id>",
        "user_id": "<user-id>",
        "award_id": "<award-id>",
        "value": "<значення>",
        "rating": "<рейтинг>",
        "project_id": "<project-id>",
        "branch_id": "<branch-id>",
        "created_at": "<дата_створення>",
        "updated_at": "<дата_оновлення>",
        "alias": "cristals"
      },
      {
        "id": "<balance-id>",
        "user_id": "<user-id>",
        "award_id": "<award-id>",
        "value": "<значення>",
        "rating": "<рейтинг>",
        "project_id": "<project-id>",
        "branch_id": "<branch-id>",
        "created_at": "<дата_створення>",
        "updated_at": "<дата_оновлення>",
        "alias": "coins"
      },
      {
        "id": "<balance-id>",
        "user_id": "<user-id>",
        "award_id": "<award-id>",
        "value": "<значення>",
        "rating": "<рейтинг>",
        "project_id": "<project-id>",
        "branch_id": "<branch-id>",
        "created_at": "<дата_створення>",
        "updated_at": "<дата_оновлення>",
        "alias": "achivements"
      }
    ]
    ```

### Уроки

#### Отримати всі уроки
Цей ендпоінт повертає список усіх доступних предметів (уроків) з їхніми ID, повними назвами та короткими скороченнями для зручного відображення.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/settings/group-specs`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    [
      {"id": "<spec-id>", "name": "<назва_уроку>", "short_name": "<скорочення_уроку>"},
      {"id": "<spec-id>", "name": "<назва_уроку>", "short_name": "<скорочення_уроку>"},
      // і так далі для інших уроків, структура повторюється, тому показую приклад тільки для двох, а не для всіх 200+
      // повний список містить аналогічні об'єкти для кожного предмета
    ]
    ```

### Домашні завдання

#### Отримати домашні завдання до виконання (статус 3)
Цей ендпоінт повертає список домашніх завдань зі статусом 3 ("До виконання"), включаючи деталі про тему, терміни, файли та мета-інформацію про пагінацію.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?status=3&limit=1000&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "<homework-id>",
          "id_spec": "<spec-id>",
          "id_teach": "<teacher-id>",
          "id_group": "<group-id>",
          "fio_teach": "<ім'я_викладача>",
          "theme": "<тема_завдання>",
          "completion_time": "<дата_виконання>",
          "creation_time": "<дата_створення>",
          "overdue_time": "<дата_прострочення>",
          "file_path": "<url_файлу>",
          "comment": "",
          "name_spec": "<назва_предмета>",
          "status": 3,
          "homework_stud": {
            "id": "<student-homework-id>",
            "file_path": null,
            "mark": null,
            "creation_time": "<дата_створення>",
            "stud_answer": "",
            "auto_mark": false,
            "status": 3,
            "photo_pas": null,
            "fio_stud": "<ім'я_студента>",
            "related_homeworks": null,
            "related_materials": null,
            "unlock_expire": "0",
            "communication_history": [],
            "is_retake": "0"
          },
          "cover_image": null,
          "is_autotest": 0,
          "autotest_status": null,
          "autotest_result": null,
          "is_academic_holiday": 0,
          "ospr": 0,
          "teach_photo_pas": null,
          "default_deadline_days": 182,
          "coins_to_close_deadline": 20,
          "completion_days": "<кількість_днів>",
          "homework_comment": {
            "text_comment": "",
            "attachment": null,
            "attachment_path": null,
            "date_updated": ""
          }
        }
        // структура повторюється для кожного завдання, тому показую приклад тільки для одного
      ],
      "_meta": {
        "currentPage": 1,
        "totalPages": 1,
        "totalCount": "<кількість>"
      }
    }
    ```

#### Отримати домашні завдання на перевірці (статус 2)
Цей ендпоінт повертає список домашніх завдань зі статусом 2 ("На перевірці"), з деталями про відповідь студента, коментарі та історію комунікації.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?status=2&limit=1000&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "<homework-id>",
          "id_spec": "<spec-id>",
          "id_teach": "<teacher-id>",
          "id_group": "<group-id>",
          "fio_teach": "<ім'я_викладача>",
          "theme": "<тема_завдання>",
          "completion_time": "<дата_виконання>",
          "creation_time": "<дата_створення>",
          "overdue_time": "<дата_прострочення>",
          "file_path": "<url_файлу>",
          "comment": "",
          "name_spec": "<назва_предмета>",
          "status": 2,
          "homework_stud": {
            "id": "<student-homework-id>",
            "file_path": null,
            "mark": null,
            "creation_time": "<дата_створення>",
            "stud_answer": "<відповідь_студента>",
            "auto_mark": false,
            "status": 2,
            "photo_pas": null,
            "fio_stud": "<ім'я_студента>",
            "related_homeworks": null,
            "related_materials": null,
            "unlock_expire": "0",
            "communication_history": [
              {
                "type_name": "homework_student_comment",
                "comment": "<коментар>",
                "date": "<дата>",
                "file": null
              }
              // структура повторюється для кожного запису історії
            ],
            "is_retake": "0"
          },
          "cover_image": null,
          "is_autotest": 0,
          "autotest_status": null,
          "autotest_result": null,
          "is_academic_holiday": 0,
          "ospr": 0,
          "teach_photo_pas": null,
          "default_deadline_days": 182,
          "coins_to_close_deadline": 20,
          "completion_days": "<кількість_днів>",
          "homework_comment": {
            "text_comment": "",
            "attachment": null,
            "attachment_path": null,
            "date_updated": ""
          }
        }
        // структура повторюється для кожного завдання
      ],
      "_meta": {
        "currentPage": 1,
        "totalPages": 1,
        "totalCount": "<кількість>"
      }
    }
    ```

#### Отримати виконані домашні завдання (статус 1)
Цей ендпоінт повертає список домашніх завдань зі статусом 1 ("Виконано"), з оцінками, файлами, коментарями вчителя та історією комунікації.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?status=1&limit=1000&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "<homework-id>",
          "id_spec": "<spec-id>",
          "id_teach": "<teacher-id>",
          "id_group": "<group-id>",
          "fio_teach": "<ім'я_викладача>",
          "theme": "<тема_завдання>",
          "completion_time": "<дата_виконання>",
          "creation_time": "<дата_створення>",
          "overdue_time": "<дата_прострочення>",
          "file_path": "<url_файлу>",
          "comment": "",
          "name_spec": "<назва_предмета>",
          "status": 1,
          "homework_stud": {
            "id": "<student-homework-id>",
            "file_path": "<url_файлу>",
            "mark": "<оцінка>",
            "creation_time": "<дата_створення>",
            "stud_answer": "",
            "auto_mark": false,
            "status": 1,
            "photo_pas": null,
            "fio_stud": "<ім'я_студента>",
            "related_homeworks": null,
            "related_materials": null,
            "unlock_expire": "0",
            "communication_history": [
              {
                "type_name": "homework_student_comment",
                "comment": "",
                "date": "<дата>",
                "file": "<url_файлу>"
              },
              {
                "type_name": "homework_teacher_comment",
                "comment": "",
                "mark": "<оцінка>",
                "date": "<дата>",
                "file": null
              }
              // структура повторюється
            ],
            "is_retake": "0"
          },
          "cover_image": null,
          "is_autotest": 0,
          "autotest_status": null,
          "autotest_result": null,
          "is_academic_holiday": 0,
          "ospr": 0,
          "teach_photo_pas": null,
          "default_deadline_days": 182,
          "coins_to_close_deadline": 20,
          "completion_days": "<кількість_днів>",
          "homework_comment": {
            "text_comment": "",
            "attachment": null,
            "attachment_path": null,
            "date_updated": "<дата_оновлення>"
          }
        }
        // структура повторюється
      ],
      "_meta": {
        "currentPage": 1,
        "totalPages": 1,
        "totalCount": "<кількість>"
      }
    }
    ```

#### Отримати інфо про домашнє завдання за ID
Цей ендпоінт повертає детальну інформацію про конкретне домашнє завдання за його ID, включаючи статус, коментарі, файли та пов'язані завдання (наприклад, перездачі).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/<homework_id>`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "0": {
        "id": "<homework-id>",
        "id_spec": "<spec-id>",
        "id_teach": "<teacher-id>",
        "id_group": "<group-id>",
        "fio_teach": "<ім'я_викладача>",
        "theme": "<тема_завдання>",
        "completion_time": "<дата_виконання>",
        "creation_time": "<дата_створення>",
        "overdue_time": "<дата_прострочення>",
        "file_path": "<url_файлу>",
        "comment": "<коментар_до_завдання>",
        "name_spec": "<назва_предмета>",
        "status": 3,
        "homework_stud": {
          "id": "<student-homework-id>",
          "file_path": null,
          "mark": null,
          "creation_time": "<дата_створення>",
          "stud_answer": "",
          "auto_mark": false,
          "status": 3,
          "photo_pas": null,
          "fio_stud": "<ім'я_студента>",
          "related_homeworks": null,
          "related_materials": null,
          "unlock_expire": "0",
          "communication_history": [],
          "is_retake": "0"
        },
        "cover_image": null,
        "is_autotest": 0,
        "autotest_status": null,
        "autotest_result": null,
        "is_academic_holiday": 0,
        "ospr": 0,
        "teach_photo_pas": null,
        "default_deadline_days": 182,
        "coins_to_close_deadline": 20,
        "completion_days": "<кількість_днів>",
        "homework_comment": {
          "text_comment": "",
          "attachment": null,
          "attachment_path": null,
          "date_updated": ""
        }
      },
      "relatedHomeworks": [
        {
          "id_domzadstud": "<related-student-homework-id>",
          "id_teach": "<teacher-id>",
          "id_stud": "<student-id>",
          "status": "1",
          "time": "<дата_виконання>",
          "id_domzad": "<related-homework-id>",
          "ospr": "0",
          "mark": "<оцінка>",
          "nlenta": "<номер_стрічки>",
          "date_vizit": "<дата_відвідування>",
          "is_retake": "0",
          "auto_test_status": "0",
          "auto_test_response": null,
          "unlock_expire": "0",
          "domZad": {
            "id_domzad": "<related-homework-id>",
            "material_link": null,
            "id_teach": "<teacher-id>",
            "id_tgroups": "<group-id>",
            "id_spec": "<spec-id>",
            "time": "<дата_створення>",
            "id_schedule": "<schedule-id>",
            "nlenta": "<номер_стрічки>",
            "date": "<дата>",
            "ospr": "0",
            "dz_theme": "<тема_завдання>",
            "deadline": "<дата_здачі>",
            "is_autotest": "0",
            "public_material_uuid": null,
            "spec": {
              "id_spec": "<spec-id>",
              "name_spec": "<назва_предмета>",
              "short_name_spec": "<коротка_назва>",
              "id_form": "<form-id>",
              "id_dir": "<dir-id>",
              "status": "0"
            },
            "cover": null
          }
        }
        // структура повторюється для пов'язаних завдань
      ]
    }
    ```

#### Отримати домашні завдання за певними предметами (фільтр)
Цей ендпоінт повертає домашні завдання, фільтровані за ID предметів (spec_id), зі статусом 3 ("До виконання"), з мета-інформацією про пагінацію.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?limit=1000&spec_id=<spec-id1>,<spec-id2>&status=3&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "data": [],
      "_meta": {
        "currentPage": 1,
        "totalPages": 0,
        "totalCount": 0
      }
    }
    ```

#### Отримати токен для файлів
Цей ендпоінт повертає тимчасовий токен для завантаження файлів та ID директорій для різних типів файлів (домашні завдання, портфоліо тощо).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/user/file-token`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "token": "<токен_файлів>",
      "domain": "https://fsx3.itstep.org",
      "directories": {
        "homeworkDirId": "<homework-dir-id>",
        "portfolioDirId": "<portfolio-dir-id>",
        "reviewDirId": "<review-dir-id>",
        "examDirId": "<exam-dir-id>",
        "photoStudDirId": "<photo-stud-dir-id>",
        "chatDirId": "<chat-dir-id>"
      },
      "url": "https://fsx3.itstep.org/api/v1/files"
    }
    ```

#### Завантажити файл для домашнього завдання
Цей ендпоінт дозволяє завантажувати файли для домашніх завдань, використовуючи токен з попереднього запиту. Файли передаються у multipart/form-data.

-   **Метод:** `POST`
-   **URL:** `/api/v1/files` (використовуйте URL з відповіді на file-token)
-   **Заголовки:** `authorization: Bearer <токен_файлів>`
-   **Тіло запиту (multipart/form-data):**
    -   `files[]`: `<файл у байтах>`
    -   `directory`: `"<директорія для домашки (homeworkDirId)>"`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<id_файлу>",
        "name": "<ім'я_файлу>",
        "link": "<посилання_на_файл>",
        "size": "<розмір_у_байтах>"
      }
    ]
    ```

#### Відправити домашнє завдання (без файлу)
Цей ендпоінт відправляє текстову відповідь на домашнє завдання, змінюючи статус на 2 ("На перевірці") та додаючи запис до історії комунікації.

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/homework/create`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Тіло запиту:**
    ```json
    {
      "answerText": "<текст_відповіді>",
      "filename": null,
      "id": "<homework-id>"
    }
    ```
-   **Відповідь (201 Created):**
    ```json
    {
      "id": "<student-homework-id>",
      "file_path": null,
      "mark": null,
      "creation_time": "<дата_створення>",
      "stud_answer": "",
      "auto_mark": false,
      "status": 2,
      "photo_pas": "",
      "fio_stud": "<ім'я_студента>",
      "related_homeworks": null,
      "related_materials": null,
      "unlock_expire": 0,
      "communication_history": [
        {
          "type_name": "homework_student_comment",
          "comment": "<текст_відповіді>",
          "date": "<дата>",
          "file": null
        }
      ],
      "is_retake": 0
    }
    ```

#### Відправити домашнє завдання (з файлом)
Цей ендпоінт відправляє домашнє завдання з файлом (або текстом), змінюючи статус на 2 ("На перевірці") та додаючи записи до історії комунікації для тексту та файлу.

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/homework/create`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Тіло запиту:**
    ```json
    {
      "answerText": null або <текст_відповіді>,
      "filename": "<посилання_на_файл_з_завантаження>",
      "id": "<homework-id>"
    }
    ```
-   **Відповідь (201 Created):**
    ```json
    {
      "id": "<student-homework-id>",
      "file_path": null,
      "mark": null,
      "creation_time": "<дата_створення>",
      "stud_answer": "",
      "auto_mark": false,
      "status": 2,
      "photo_pas": "",
      "fio_stud": "<ім'я_студента>",
      "related_homeworks": null,
      "related_materials": null,
      "unlock_expire": 0,
      "communication_history": [
        {
          "type_name": "homework_student_comment",
          "comment": "<текст_відповіді>",
          "date": "<дата>",
          "file": null
        },
        {
          "type_name": "homework_student_comment",
          "comment": "",
          "date": "<дата>",
          "file": "<посилання_на_файл>"
        }
      ],
      "is_retake": 0
    }
    ```

#### Видалити домашнє завдання
Цей ендпоінт видаляє відправлене домашнє завдання студента, повертаючи статус на 3 ("До виконання").

-   **Метод:** `DELETE`
-   **URL:** `/{branch_alias}/homework/delete/<student-homework-id>`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):** `true`

#### Отримати кількість домашніх завдань
Цей ендпоінт повертає лічильники домашніх завдань за типами статусів. Поле `counter_type` вказує на категорію: 0 — всі, 1 — виконані, 2 — на перевірці, 3 — до виконання, 4/5/6 — невідомі або спеціальні статуси. `counter` — кількість у кожній категорії.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/count/homework`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    [{"counter_type":1,"counter":<кількість>},{"counter_type":3,"counter":<кількість>},{"counter_type":6,"counter":<кількість>},{"counter_type":2,"counter":<кількість>},{"counter_type":5,"counter":<кількість>},{"counter_type":0,"counter":<кількість>}]
    ```

### Розклад

#### Отримати розклад на тиждень
Цей ендпоінт повертає розклад уроків на тиждень за вказаною датою, включаючи деталі про уроки, оцінки, домашні завдання та присутність.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/schedule/get-month?type=week&date_filter=YYYY-MM-DD`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "data": [
        {
          "date": "YYYY-MM-DD",
          "lesson": "<номер_уроку>",
          "started_at": "HH:MM:SS",
          "finished_at": "HH:MM:SS",
          "teacher_name": "<ім'я_викладача>",
          "subject_name": "<назва_предмета>",
          "room_name": "<аудиторія>",
          "mark": "<оцінка>",
          "mark_type": "<тип_оцінки>",
          "homeworks": [
            {
              "mark": "<оцінка>",
              "status": "<статус>",
              "ospr": "0",
              "id_domzad": "<id_домашки>"
            }
            // структура повторюється для домашніх завдань
          ],
          "was": 1,
          "was_type": "<тип_присутності>",
          "lesson_theme": "<тема_уроку>",
          "id_spec": "<spec-id>",
          "lesson_type": 0,
          "exam": [],
          "link_to_lesson": null
        }
        // структура повторюється для кожного уроку в розкладі
      ],
      "start_day": "YYYY-MM-DD",
      "end_date": "YYYY-MM-DD"
    }
    ```

### Лідерборд

#### Отримати лідерборд
Цей ендпоінт повертає лідерборд для групи та потоку студентів, з позиціями, балами та позначкою поточного користувача (`"current": true`).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/progress/leader-table`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "group": {
        "top": [
          {
            "id_stud": "<id_студента>",
            "fio_stud": "<ПІБ_студента>",
            "photo": "<url_фото>",
            "amount": "<кількість_балів>",
            "position": "<позиція>",
            "current": true
          }
          // структура повторюється для кожного учасника групи
        ]
      },
      "stream": {
        "top": [
          {
            "id_stud": "<id_студента>",
            "fio_stud": "<ПІБ_студента>",
            "photo": "<url_фото>",
            "amount": "<кількість_балів>",
            "position": "<позиція>",
            "current": true
          }
          // структура повторюється для кожного учасника потоку
        ]
      }
    }
    ```

### Магазин

#### Отримати продукти магазину
Цей ендпоінт повертає список продуктів магазину з пагінацією, включаючи деталі про ціни в монетах (COI) та кристалах (DIA), властивості, зображення та описи.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/products?page=1&per_page=16`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<product-id>",
        "uuid": "<product-uuid>",
        "project_id": "<project-id>",
        "branch_id": "<branch-id>",
        "category_id": "<category-id>",
        "status": 1,
        "quantity": "<кількість>",
        "dynamic_price": "on",
        "published_at": "<дата_публікації>",
        "sku": "<sku>",
        "image": ["<url_зображення>"],
        "type": 0,
        "external_id": null,
        "created_by": "<created-by-id>",
        "created_by_fio": "<ім'я_автора>",
        "updated_by": "<updated-by-id>",
        "updated_by_fio": "<ім'я_оновлювача>",
        "created_at": "<дата_створення>",
        "updated_at": "<дата_оновлення>",
        "deleted_at": null,
        "properties": [
          {"name": "<властивість>", "value": "<значення>", "code": "<код>"}
          // структура повторюється для властивостей
        ],
        "prices": [
          {
            "id": "<price-id>",
            "product_id": "<product-id>",
            "price": "<ціна>",
            "min_price": "<мін_ціна>",
            "currency": "COI",
            "type": "default",
            "until_to": null,
            "created_at": "<дата_створення>",
            "updated_at": "<дата_оновлення>"
          },
          {
            "id": "<price-id>",
            "product_id": "<product-id>",
            "price": "<ціна>",
            "min_price": "<мін_ціна>",
            "currency": "DIA",
            "type": "default",
            "until_to": null,
            "created_at": "<дата_створення>",
            "updated_at": "<дата_оновлення>"
          }
          // структура повторюється для валют
        ],
        "title": "<назва_продукту>",
        "text": "<опис_продукту>",
        "special_type": "none",
        "edu_forms": []
      }
      // структура повторюється для кожного продукту
    ]
    ```

#### Отримати пов’язані продукти
Цей ендпоінт повертає продукти, пов’язані з вказаним продуктом (наприклад, рекомендації), з аналогічною структурою, як у списку продуктів.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/products/get-others?product_id=<product-id>`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):** Аналогічна структурі відповіді "Отримати продукти магазину".

#### Створити замовлення
Цей ендпоінт створює замовлення на продукти, повертаючи посилання на оплату або статус успіху.

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/shop/create-order`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Тіло запиту:**
    ```json
    {
      "products": [
        {
          "product_id": "<product-id>",
          "quantity": "<кількість>"
        }
        // структура повторюється для продуктів у замовленні
      ],
      "merchant": "Gaming"
    }
    ```
-   **Відповідь (200 OK):**
    ```json
    {
      "link": "<url_оплати>",
      "status": "succeeded",
      "uuid": "<order-uuid>",
      "is_link": false,
      "is_form": false,
      "type": "pay",
      "urlLink": "<url_оплати>",
      "additional": null
    }
    ```

#### Отримати замовлення
Цей ендпоінт повертає список замовлень користувача (лише зі статусом 3 — "Замовлено"), з деталями про продукти, ціни та можливість скасування.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/orders`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<order-id>",
        "uuid": "<order-uuid>",
        "project_id": "<project-id>",
        "branch_id": "<branch-id>",
        "user_id": "<user-id>",
        "fio": "<ім'я_користувача>",
        "status": 3,
        "notes": null,
        "delivery_notes": null,
        "external_id": null,
        "receipt": null,
        "created_by": null,
        "created_by_fio": null,
        "updated_by": null,
        "updated_by_fio": null,
        "created_at": "<дата_створення>",
        "updated_at": "<дата_оновлення>",
        "deleted_at": null,
        "products": [
          {
            "sku": "<sku>",
            "image": ["<url_зображення>"],
            "id": "<order-product-id>",
            "order_id": "<order-id>",
            "product_id": "<product-id>",
            "quantity": "<кількість>",
            "created_at": "<дата_створення>",
            "updated_at": "<дата_оновлення>",
            "order_product_price": [
              {
                "id": "<price-id>",
                "order_product_id": "<order-product-id>",
                "price": "<ціна>",
                "currency": "COI",
                "created_at": "<дата_створення>",
                "updated_at": "<дата_оновлення>"
              },
              {
                "id": "<price-id>",
                "order_product_id": "<order-product-id>",
                "price": "<ціна>",
                "currency": "DIA",
                "created_at": "<дата_створення>",
                "updated_at": "<дата_оновлення>"
              }
              // структура повторюється для валют
            ],
            "title": "<назва_продукту>",
            "text": "<опис_продукту>",
            "edu_forms": []
          }
          // структура повторюється для продуктів
        ],
        "can_discard": true
      }
      // структура повторюється для кожного замовлення
    ]
    ```

#### Скасувати замовлення
Цей ендпоінт скасовує замовлення за його UUID, повертаючи статус успіху.

-   **Метод:** `PUT`
-   **URL:** `/{branch_alias}/shop/discard-order`
-   **Заголовки:** `authorization: Bearer <токен>`
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

### Оцінки

#### Отримати всі оцінки
Цей ендпоінт повертає список усіх оцінок студента, включаючи дату, значення оцінки, тему уроку, тип роботи та коментарі. Поле `type_work` вказує на тип: 5 — "Класна робота", 3 — "Самостійна робота", всі інші значення — "Контрольна робота". Дані сортовані за датою.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/statistic/marks`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "mark_date": "<дата_оцінки>",
        "mark": "<оцінка>",
        "nlenta": "<номер_стрічки>",
        "m": "<місяць>",
        "y": "<рік>",
        "d": "<день>",
        "name_spec": "<назва_предмета>",
        "fio_teach": "<ім'я_викладача>",
        "photo_pas": "<url_фото>",
        "theme": "<тема_уроку>",
        "type_work": "<тип_роботи>",
        "comment": null
      },
      // структура повторюється для кожної оцінки, показую приклад тільки для однієї, а не для всіх
    ]
    ```

### Нарахування активності

#### Отримати нарахування активності
Цей ендпоінт повертає список нарахувань (прогрес активності) з пагінацією, включаючи дату, назву уроку, тип активності, нагороди (coins або cristals), оцінки та параметри приховування. Підтримує параметр `new_format=1` для нового формату. Нарахування можуть бути позитивними (нагороди) або негативними (штрафи, наприклад, за оплату в магазині). Приклади включають перездачі, відвідування, роботу в класі та покупки.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/progress/activity?new_format=1&per_page=100&page=1`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    [
      {
        "id": "<id_нарахування>",
        "created_at": "<дата_створення>",
        "lesson_name": "<назва_уроку>",
        "name": "<назва_активності>",
        "icon": "<url_іконки>",
        "award_value": <значення_нагороди>,
        "award_alias": "<тип_валюти>",
        "mark": <оцінка>,
        "mark_type": "<тип_оцінки>",
        "hide_params": {
          "can_hide": true,
          "type": "<тип>",
          "entity_id": <id_сутності>
        }
      },
      // структура повторюється для кожного нарахування, показую приклад тільки для одного
    ]
    ```

### Відвідуваність

#### Отримати статистику відвідуваності
Цей ендпоінт повертає статистику відвідуваності за місяць (параметр `period=month`), структуровану за роками, місяцями та днями. Для кожного дня: `was` — масив значень присутності ("1" — присутній, "0" — відсутній), `visits` — масив детальних записів про уроки з ID студента, датою, темою, предметом тощо. Також включає загальні відсотки відвідуваності, відсутності, запізнень та зміни в порівнянні з попереднім періодом (attendanceChange/lateChange: "unchanged", "increased" або "decreased").

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/statistic/attendance?period=month`
-   **Заголовки:** `authorization: Bearer <токен>`
-   **Відповідь (200 OK):**
    ```json
    {
      "data": {
        "<рік>": {
          "<місяць>": {
            "<день>": {
              "was": ["<присутність>"],
              "visits": [
                {
                  "id_stud": "<id_студента>",
                  "date_vizit": "<дата_відвідування>",
                  "was": "<присутність>",
                  "nlenta": "<номер_стрічки>",
                  "theme": "<тема_уроку>",
                  "id_spec": "<spec-id>",
                  "m": "<місяць>",
                  "y": "<рік>",
                  "d": "<день>",
                  "spec": {
                    "id_spec": "<spec-id>",
                    "name_spec": "<назва_предмета>"
                  }
                }
                // структура повторюється для кожного уроку в день, показую приклад тільки для одного дня та одного уроку
              ]
            }
            // структура повторюється для кожного дня, показую приклад тільки для одного дня, а не для всіх 30+
          }
          // структура повторюється для кожного місяця
        }
      },
      "percentOfAttendance": <відсоток_відвідуваності>,
      "countAbsent": <кількість_відсутностей>,
      "percentOfAbsent": <відсоток_відсутностей>,
      "percentOfLate": <відсоток_запізнень>,
      "attendanceChange": "<зміна_відвідуваності>",
      "lateChange": "<зміна_запізнень>"
    }
    ```
