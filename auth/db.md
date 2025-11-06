# Схема данных

## Список разделов (tenant). 
Хранит список разделов. В большинстве случаев, будет 1 запись.

~~~
CREATE TABLE IF NOT EXISTS acc_tenants (
    id BIGINT PRIMARY KEY DEFAULT nextval('account_seq'),
    name VARCHAR(250),
    is_deleted BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
~~~

## Список клиентов, партнеров.
Справочник подключенных партнеров, с аттрибутами настройки подключения.
Партнерское подключение создается в рамках tenant, и видно только в нем.
client_id - из JWT токена.
default_account_id - id портфеля, в который будут попадать договоры, если не указан целевой портфель. JWT не содержит user.

~~~
CREATE TABLE IF NOT EXISTS acc_clients (
    tid BIGINT NOT NULL REFERENCES acc_tenants(id),
    id BIGINT PRIMARY KEY DEFAULT nextval('account_seq'),
    client_id varchar(255) NOT NULL,
    default_account_id BIGINT,
    name VARCHAR(250),
    is_deleted BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
~~~

## Структура доступа к данным ( портфели )

~~~
CREATE TABLE IF NOT EXISTS acc_accounts (
    id BIGINT PRIMARY KEY,
    tid BIGINT NOT NULL REFERENCES acc_tenants(id),
    client_id BIGINT REFERENCES acc_clients(id),
    parent_id BIGINT REFERENCES acc_accounts(id),
    node_type VARCHAR(10) NOT NULL,
    name VARCHAR(250),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
~~~

## Настройка доступов к продуктам
Содержит привязку портфеля к продукту.
~~~
-- Product roles table
CREATE TABLE IF NOT EXISTS acc_product_roles (
    id BIGINT PRIMARY KEY ,
    tid BIGINT NOT NULL REFERENCES acc_tenants(id),
    account_id BIGINT NOT NULL REFERENCES acc_accounts(id),
    role_product_id BIGINT NOT NULL,
    role_account_id BIGINT NOT NULL REFERENCES acc_accounts(id),
    is_deleted BOOLEAN DEFAULT FALSE,
    can_read BOOLEAN DEFAULT FALSE,
    can_quote BOOLEAN DEFAULT FALSE,
    can_policy BOOLEAN DEFAULT FALSE,
    can_addendum BOOLEAN DEFAULT FALSE,
    can_cancel BOOLEAN DEFAULT FALSE,
    can_prolongate BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    properties JSONB DEFAULT '{}'
);
~~~


-- Account logins table
CREATE TABLE IF NOT EXISTS acc_logins (
    id BIGINT PRIMARY KEY ,
    tid BIGINT NOT NULL REFERENCES acc_tenants(id),
    user_login VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    properties JSONB DEFAULT '{}',
    UNIQUE (user_login, tid)
);

-- Account logins table
CREATE TABLE IF NOT EXISTS acc_account_logins (
    id BIGINT PRIMARY KEY ,
    tid BIGINT NOT NULL REFERENCES acc_tenants(id),
    user_login VARCHAR(255) NOT NULL,
    client_id BIGINT NOT NULL REFERENCES acc_clients(id),
    is_default BOOLEAN DEFAULT FALSE,
    account_id BIGINT NOT NULL REFERENCES acc_accounts(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (tid, user_login) REFERENCES acc_logins(tid, user_login),
    UNIQUE (user_login, account_id)
);


-- Account tokens table
CREATE TABLE IF NOT EXISTS acc_account_tokens (
    id BIGINT PRIMARY KEY,
    tid BIGINT NOT NULL REFERENCES acc_tenants(id),
    token VARCHAR(255) NOT NULL,
    client_id BIGINT NOT NULL REFERENCES acc_clients(id),
    account_id BIGINT NOT NULL REFERENCES acc_accounts(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (token, client_id)
);



