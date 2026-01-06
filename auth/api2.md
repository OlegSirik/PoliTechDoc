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

> **Note:** Tenant-scoped endpoints MUST NOT accept tenant override headers (`X-Impersonate-Tenant`). TenantCode from URL is authoritative.

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

> **Impersonation:** System-scoped endpoints (`/sys-admins/**`, `/tnt-admins`) MAY accept `X-Impersonate-Tenant` header to query other tenants. Allowed **only for SYS_ADMIN**. Tenant-scoped endpoints MUST reject impersonation headers.

---

## 6. Client & Group Admins

**Naming decision:** используется **plural form** для коллекций ролей.

* `client-admins`
* `group-admins`

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

## 11. User Creation & Administration Model

### 11.1 User Account Basics

* Учетная запись (**login**) всегда привязана к **одному tenant**.
* Учетка может существовать **без ролей**.
* Учетка **без ролей не имеет доступа** к API и UI (inactive state).

**Rationale:**

* поддержка staged‑workflow (create → configure → assign roles)
* интеграции и external provisioning

---

### 11.2 User Lifecycle

1. **Create account**

   * создаётся login + tenantCode
   * роли не обязательны

2. **Assign roles**

   * назначение ролей активирует пользователя
   * роли определяют scope (tenant / client / group)

3. **Operate**

   * пользователь может выполнять действия в рамках scope

4. **Deactivate / Delete**

   * soft delete (см. ниже)

---

### 11.3 Role Assignment Rules

* **SYS_ADMIN**

  * может создавать пользователей в любом tenant
  * может назначать любые роли

* **TNT_ADMIN**

  * может создавать пользователей в своём tenant
  * может назначать tenant / client / group роли

* **CLIENT_ADMIN**

  * может создавать пользователей **только в своём clientCode**
  * может назначать роли: `USER`, `GROUP_ADMIN`

* **GROUP_ADMIN**

  * не может создавать пользователей

---

### 11.4 External / Third‑Party Administration

Поддерживаются **service accounts** и delegation‑модель.

#### Option A — Service Account (Recommended)

* OAuth2 Client / API Key
* Привязан к:

  * tenantCode
  * опционально clientCode

**Capabilities:**

* создание учеток
* назначение ограниченного набора ролей
* отсутствие interactive login

**Use cases:**

* HR‑системы
* CRM
* Identity provisioning

---

#### Option B — Client‑Bound Delegation

* внешнее приложение привязано к `clientCode`
* доступ только к:

  * `/api/v1/{tenantCode}/clients/{clientCode}/logins`

**Roles:**

* `CLIENT_ADMIN`
* `USER_MANAGER` (optional future role)

**Use cases:**

* партнерские порталы
* embedded admin UI

---

### 11.5 Recommended API Workflow

```text
POST   /api/v1/{tenantCode}/logins              -> create user (no roles)
POST   /api/v1/{tenantCode}/clients/{clientCode}/client-admins
POST   /api/v1/{tenantCode}/clients/{clientCode}/group-admins
```

---

## 12. Soft Delete Policy

## 12. Notes

* Все операции выполняются в контексте `tenantCode`.
* Авторизация: JWT / OAuth2.
* Рекомендуется аудит изменений ролей и scope.
* **Impersonation headers (`X-Impersonate-Tenant`)** разрешены **только на system-scoped endpoints** и **только для SYS_ADMIN**. Tenant-scoped endpoints должны их игнорировать или возвращать 400/403.
