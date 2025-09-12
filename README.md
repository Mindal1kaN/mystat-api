# Документація API MyStat

Цей документ описує ендпоінти API MyStat для взаємодії з платформою, включаючи автентифікацію, керування профілем, отримання балансу, список уроків, керування домашніми завданнями, функціонал магазину, розклад уроків та лідерборд.

**Базовий URL:** `https://mapi.itstep.org/v1/mystat/`

---

## Примітки

-   **Автентифікація:** Усі ендпоінти, крім `/auth/login`, потребують заголовок `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO....`.
-   **`branch_alias`:** Поле `"branch_alias": "pl"` у відповіді `/auth/me` визначає префікс для ендпоінтів, пов'язаних з філією. Наприклад, якщо `branch_alias = "pl"`, то ендпоінти починаються з `/pl/` (наприклад, `/pl/gaming/get-balances`). Якщо `branch_alias = "kiev"`, то ендпоінти починаються з `/kiev/`. Це дозволяє динамічно формувати URL на основі філії користувача. Замініть `"pl"` на відповідний alias у всіх релевантних ендпоінтах.
-   **Статус домашніх завдань:** `3` — "До виконання", `2` — "На перевірці", `1` — "Виконано".
-   **Статус замовлень:** `3` — "Замовлено".
-   **Пагінація:** Ендпоінт продуктів магазину підтримує пагінацію через параметри `page` та `per_page`.
-   **Валюта:** Ціни у магазині вказані у двох валютах: `COI` (монети) та `DIA` (кристали).
-   **Файли:** Відповіді для домашніх завдань та продуктів містять URL-адреси файлів. Директорії для завантаження файлів є унікальними для кожного користувача.
-   **Помилки:** Усі успішні відповіді у цій документації повертають код 200 або 201. Обробка помилок не деталізована, але має відповідати стандартним практикам API.

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
    eyJ0eXAiOiJKV1QiLCJhbGciO....
    ```

---

### Профіль

#### Отримати профіль користувача
Отримує інформацію про автентифікованого користувача (включаючи `branch_alias`).

-   **Метод:** `POST`
-   **URL:** `/auth/me`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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

---

### Баланс

#### Отримати баланс користувача
Отримує баланси користувача (кристали, монети, досягнення).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/gaming/get-balances`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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

---

### Уроки

#### Отримати всі уроки
Отримує список усіх доступних предметів.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/settings/group-specs`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
-   **Відповідь (200 OK):**
    ```json
    [
      {"id": "<spec-id>", "name": "Illustrator", "short_name": "Illust."},
      {"id": "<spec-id>", "name": "Java Script", "short_name": "JavScr"},
      {"id": "<spec-id>", "name": "Project time", "short_name": "Proj time"},
      {"id": "<spec-id>", "name": "Smart Time", "short_name": "Проєктна година"},
      {"id": "<spec-id>", "name": "Soft skills", "short_name": "Soft skills"},
      {"id": "<spec-id>", "name": "Історія Украіни", "short_name": "ІУ"},
      {"id": "<spec-id>", "name": "Алгебра", "short_name": "Алг."},
      {"id": "<spec-id>", "name": "Англійська мова", "short_name": "АМ"},
      {"id": "<spec-id>", "name": "Біологія", "short_name": "Біологія"},
      {"id": "<spec-id>", "name": "Всесвітня історія", "short_name": "Всесв.історія"},
      {"id": "<spec-id>", "name": "Географія", "short_name": "Географія"},
      {"id": "<spec-id>", "name": "Геометрія", "short_name": "Геометрія"},
      {"id": "<spec-id>", "name": "Зарубіжна література", "short_name": "ЗЛ"},
      {"id": "<spec-id>", "name": "Зустріч з цікавою людиною", "short_name": "Зустріч"},
      {"id": "<spec-id>", "name": "Німецька мова", "short_name": "НМ"},
      {"id": "<spec-id>", "name": "Проєкт мовного напрямку", "short_name": "Проєкт мовний"},
      {"id": "<spec-id>", "name": "Проєкт природничого напрямку", "short_name": "проєкт"},
      {"id": "<spec-id>", "name": "Українська література", "short_name": "УЛ"},
      {"id": "<spec-id>", "name": "Українська мова", "short_name": "УМ"},
      {"id": "<spec-id>", "name": "Фізика", "short_name": "Фіз"},
      {"id": "<spec-id>", "name": "Фізична культура", "short_name": "Фіз.культ."},
      {"id": "<spec-id>", "name": "Хімія", "short_name": "Хім"}
    ]
    ```

---

### Домашні завдання

#### Отримати домашні завдання до виконання (статус 3)
Отримує список домашніх завдань зі статусом 3 (До виконання).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?status=3&limit=1000&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
      ],
      "_meta": {
        "currentPage": 1,
        "totalPages": 1,
        "totalCount": "<кількість>"
      }
    }
    ```

#### Отримати домашні завдання на перевірці (статус 2)
Отримує список домашніх завдань зі статусом 2 (На перевірці).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?status=2&limit=1000&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
      ],
      "_meta": {
        "currentPage": 1,
        "totalPages": 1,
        "totalCount": "<кількість>"
      }
    }
    ```

#### Отримати виконані домашні завдання (статус 1)
Отримує список домашніх завдань зі статусом 1 (Виконано).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?status=1&limit=1000&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
      ],
      "_meta": {
        "currentPage": 1,
        "totalPages": 1,
        "totalCount": "<кількість>"
      }
    }
    ```

#### Отримати домашні завдання за певними предметами (фільтр)
Отримує домашні завдання для вказаних предметів (фільтр за `spec_id`).

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/homework/list?limit=1000&spec_id=<spec-id1>,<spec-id2>&status=3&sort=-hw.time`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
Отримує токен та директорії для завантаження файлів.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/user/file-token`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
-   **Відповідь (200 OK):**
    ```json
    {
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImp0aSI6IjhLVERGSldTZlNNdCJ9...",
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
Завантажує файл для домашнього завдання.

-   **Метод:** `POST`
-   **URL:** `/api/v1/files`
-   **Заголовки:** `authorization: Bearer <FILE_TOKEN_FROM_PREVIOUS_REQUEST>`
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
Відправляє домашнє завдання з текстовою відповіддю.

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/homework/create`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
Відправляє домашнє завдання з файлом або текстом.

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/homework/create`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
-   **Тіло запиту:**
    ```json
    {
      "answerText": null,
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
Видаляє домашнє завдання.

-   **Метод:** `DELETE`
-   **URL:** `/{branch_alias}/homework/delete/<student-homework-id>`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
-   **Відповідь (200 OK):** `true`

---

### Розклад

#### Отримати розклад на тиждень
Отримує розклад уроків на тиждень за вказаною датою.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/schedule/get-month?type=week&date_filter=YYYY-MM-DD`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
          ],
          "was": 1,
          "was_type": "<тип_присутності>",
          "lesson_theme": "<тема_уроку>",
          "id_spec": "<spec-id>",
          "lesson_type": 0,
          "exam": [],
          "link_to_lesson": null
        }
      ],
      "start_day": "YYYY-MM-DD",
      "end_date": "YYYY-MM-DD"
    }
    ```

---

### Лідерборд

#### Отримати лідерборд
Отримує лідерборд для групи та потоку. Поле `"current": true` вказує на поточного користувача.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/progress/leader-table`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
        ]
      }
    }
    ```

---

### Магазин

#### Отримати продукти магазину
Отримує список продуктів магазину з підтримкою пагінації.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/products?page=1&per_page=16`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
          {"name": "Color", "value": "<колір>", "code": "color"},
          {"name": "Country of manufacture", "value": "<країна>", "code": "country"},
          {"name": "Producer", "value": "<виробник>", "code": "manufacturer"}
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
        ],
        "title": "<назва_продукту>",
        "text": "<опис_продукту>",
        "special_type": "none",
        "edu_forms": []
      }
    ]
    ```

#### Отримати пов’язані продукти
Отримує продукти, пов’язані з конкретним продуктом.

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/products/get-others?product_id=<product-id>`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
-   **Відповідь (200 OK):** Така ж структура, як у відповіді "Отримати продукти магазину".

#### Створити замовлення
Створює замовлення для продуктів у магазині.

-   **Метод:** `POST`
-   **URL:** `/{branch_alias}/shop/create-order`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
Отримує список замовлень (лише зі статусом 3 — "Замовлено").

-   **Метод:** `GET`
-   **URL:** `/{branch_alias}/shop/orders`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
            ],
            "title": "<назва_продукту>",
            "text": "<опис_продукту>",
            "edu_forms": []
          }
        ],
        "can_discard": true
      }
    ]
    ```

#### Скасувати замовлення
Скасовує існуюче замовлення.

-   **Метод:** `PUT`
-   **URL:** `/{branch_alias}/shop/discard-order`
-   **Заголовки:** `authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...`
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
