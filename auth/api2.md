# Tenant & Access API (v1)

## Base URL

`/api/v1/{tenantCode}`

* **tenantCode** — уникальный код тенанта (white‑label / организация).
* Все эндпоинты версии v1 изолированы по тенанту.

---

## 1. Tenant (CRUD)

**Scope:** управление настройками тенанта.

**Base:** `/api/v1/{tenantCode}`

| Method | Path                   | Description             |
| ------ | ---------------------- | ----------------------- |
| GET    | `/api/v1/{tenantCode}` | Получить данные тенанта |
| POST   | `/api/v1`              | Создать тенант          |
| PUT    | `/api/v1/{tenantCode}` | Обновить тенант         |
| DELETE | `/api/v1/{tenantCode}` | Удалить тенант          |

> Доступ: **SYS_ADMIN**

---

## 2. Clients (CRUD)

**Scope:** юридические / физические клиенты внутри тенанта.

**Base:** `/api/v1/{tenantCode}/clients`

| Method | Path                    | Description      |
| ------ | ----------------------- | ---------------- |
| GET    | `/clients`              | Список клиентов  |
| POST   | `/clients`              | Создать клиента  |
| GET    | `/clients/{clientCode}` | Получить клиента |
| PUT    | `/clients/{clientCode}` | Обновить клиента |
| DELETE | `/clients/{clientCode}` | Удалить клиента  |

> Доступ: **TNT_ADMIN**, **CLIENT_ADMIN (own client)**

---

## 3. Client Groups

**Scope:** логические группы внутри клиента (подразделения, команды).

**Base:** `/api/v1/{tenantCode}/clients/{clientCode}/groups`

| Method | Path                  | Description     |
| ------ | --------------------- | --------------- |
| GET    | `/groups`             | Список групп    |
| POST   | `/groups`             | Создать группу  |
| GET    | `/groups/{groupCode}` | Получить группу |
| PUT    | `/groups/{groupCode}` | Обновить группу |
| DELETE | `/groups/{groupCode}` | Удалить группу  |

> Доступ: **CLIENT_ADMIN**, **GROUP_ADMIN (own group)**

---

## 4. Logins (Users) — CRUD

**Scope:** учетные записи пользователей тенанта.

**Base:** `/api/v1/{tenantCode}/logins`

| Method | Path              | Description             |
| ------ | ----------------- | ----------------------- |
| GET    | `/logins`         | Список пользователей    |
| POST   | `/logins`         | Создать пользователя    |
| GET    | `/logins/{login}` | Получить пользователя   |
| PUT    | `/logins/{login}` | Обновить пользователя   |
| DELETE | `/logins/{login}` | Заблокировать / удалить |

> Доступ: **TNT_ADMIN**, **CLIENT_ADMIN (scoped)**

---

## 5. Tenant Admins

### 5.1 System Admins

**Base:** `/api/v1/{tenantCode}/admins/sys-admins`

| Method | Path                  | Description                 |
| ------ | --------------------- | --------------------------- |
| GET    | `/sys-admins`         | Список системных админов    |
| POST   | `/sys-admins`         | Назначить системного админа |
| DELETE | `/sys-admins/{login}` | Отозвать роль               |

> Доступ: **ROOT / SYS_ADMIN**

### 5.2 Tenant Admins

**Base:** `/api/v1/{tenantCode}/admins/tnt-admins`

| Method | Path                  | Description              |
| ------ | --------------------- | ------------------------ |
| GET    | `/tnt-admins`         | Список админов тенанта   |
| POST   | `/tnt-admins`         | Назначить админа тенанта |
| DELETE | `/tnt-admins/{login}` | Отозвать роль            |

> Доступ: **SYS_ADMIN**

---

## 6. Client & Group Admins

**Naming decision:** используется **plural form** для коллекций ролей.

* `client-admins`
* `group-admins`

**Rationale:**

* единый REST‑паттерн (коллекция ресурсов)
* консистентность с `sys-admins`, `tnt-admins`

**Base:** `/api/v1/{tenantCode}/clients/{clientCode}/admins`

### 6.1 Client Admins

`/client-admins`

| Method | Path                     | Description            |
| ------ | ------------------------ | ---------------------- |
| GET    | `/client-admins`         | Список админов клиента |
| POST   | `/client-admins`         | Назначить админа       |
| DELETE | `/client-admins/{login}` | Отозвать роль          |

### 6.2 Group Admins

`/group-admins`

| Method | Path                    | Description             |
| ------ | ----------------------- | ----------------------- |
| GET    | `/group-admins`         | Список админов групп    |
| POST   | `/group-admins`         | Назначить админа группы |
| DELETE | `/group-admins/{login}` | Отозвать роль           |

> Доступ: **TNT_ADMIN**, **CLIENT_ADMIN**

---

## 7. Role Model (Summary)

| Role         | Scope       | Visibility                   |
| ------------ | ----------- | ---------------------------- |
| ROOT         | Вся система | Все тенанты                  |
| SYS_ADMIN    | Все тенанты | Все ресурсы                  |
| TNT_ADMIN    | Один тенант | Все клиенты и группы тенанта |
| CLIENT_ADMIN | Один клиент | Клиент + его группы          |
| GROUP_ADMIN  | Одна группа | Только своя группа           |

---

## 8. Scoping Rules

### Visibility

* **SYS_ADMIN** — видит все тенанты, клиентов, пользователей.
* **TNT_ADMIN** — видит всех клиентов, группы и логины внутри тенанта.
* **CLIENT_ADMIN** — видит:

  * только свой `clientCode`
  * все группы клиента
  * пользователей, привязанных к клиенту
* **GROUP_ADMIN** — видит:

  * только свою группу
  * пользователей группы

### Access Evaluation Order

1. Проверка `tenantCode` из URL == claim
2. Проверка роли
3. Проверка `clientCode / groupCode` (scope)

---

## 9. Auth Claims (JWT / OAuth2)

```json
{
  "sub": "user@login",
  "tenantCode": "tenant-a",
  "roles": ["TNT_ADMIN"],
  "clients": ["client-1", "client-2"],
  "groups": ["group-a", "group-b"]
}
```

**Rules:**

* `tenantCode` — всегда один
* `clients` — обязателен для CLIENT_ADMIN
* `groups` — обязателен для GROUP_ADMIN

---

## 10. Error Handling (403 vs 404)

| Scenario                   | Code | Reason             |
| -------------------------- | ---- | ------------------ |
| Нет роли                   | 403  | Forbidden          |
| Tenant mismatch            | 404  | Resource not found |
| Client outside scope       | 404  | Resource hidden    |
| Group outside scope        | 404  | Resource hidden    |
| Valid scope, invalid state | 409  | Conflict           |

> **Rule:** ресурсы вне scope маскируются как `404`.

---

## 11. Soft Delete Policy

### Applies to:

* tenants
* clients
* groups
* logins

### Model

* `deletedAt TIMESTAMP`
* `deletedBy`

### Behavior

* `GET` — не возвращает удалённые записи
* `POST` — не может создать ресурс с кодом soft‑deleted
* `DELETE` — soft delete
* `PUT` — запрещён для soft‑deleted (409)

### Restore (optional)

* `POST /{resource}/{code}/restore`

---

## 12. Notes

* Все операции выполняются в контексте `tenantCode`.
* Авторизация: JWT / OAuth2.
* Рекомендуется аудит изменений ролей и scope.
