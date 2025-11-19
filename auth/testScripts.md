# Основные UC
## Добавить SYS_ADMIN
SYS_ADMIN может выполнять любые действия в настройкой прав доступа.

1. Добавляем 1 запись в БД в таблицу acc_tenants ( 0, 'ROOT'); Для ссылочной целостности.
2. Добавить запись ROOT в acc_accounts ( id = 0, parent_id = null, account_type = 'TENANT', tid = 0 )
3. Admin будет работать через какоето приложение. Поэтому нужно добавить строку в acc_clients
Например ( tid = 0, name = 'Adminka', client_id = 'ADMINKA' ( это client_id, которое потом придет из Кейклоки )
4. Добавить запись в ACC_LOGINS (tid = 0, id = sequence, user_login = admin_email )
5. Добавить запись в acc_account_logins( tid = 0, client_id = 1, account_id = 0 (экаунт рута), user_login = admin_email, user_role = 'SYS_ADMIN' )

~~~
ACC_ACCOUNT (id=0, parent_id=null, name='ROOT', account_type='TENANT', tid = 0)         ------> TENANT (id=0,name='ROOT')       
  │
  └── ACC_ACCOUNT (id=1, parent_id=0, name='Adminka', account_type='CLIENT', tid = 0)   ------> CLIENT ( tid = 0, name = 'Adminka', client_id='ADMINKA')
      │
      └── ACCOUNT_LOGIN (tid = 0, client_id = 1, account_id = 0, user_login = admin_email, user_role = 'SYS_ADMIN') ---> LOGINS (tid=0, user_login=user_email)
~~~
```
acc_tenant (id=0, name='ROOT')
acc_account(tid=0, cid=null, id=0, account_type='TENANT', parent_id=null)
acc_client (tid=0, id=1, client_id = 'ADMINKA', name = 'Adminka')
acc_account(tid=0, cid=1, id=1, account_type='CLIENT', parent_id=0)
acc_logins( tid=0, user_login=admin_email )
acc_account_logins ( tid=0, cid=1, account_id=0, user_login=admin_email, user_role='SYS_ADMIN', is_default=true); 
```
Осталось добавить IdP, добавить в нем учетка админа и client_id.
После этого с этим токеном можно ходить а АПИ.
```
X-Tenant-Id=0
X-Account-Id=0
jwt.user_login=admin_email
jwt.clinet_id='ADMINKA'
```
Получим такие данные. 
1. Проверяем что в текущем тенанте есть такая учетка. 
```
select * from acc_logins where tid = X-Tenant-Id and user_login = jwt.user_login
```
2. Проверяем что есть такой клиент в этом тенанте и получаем id клиента. Таблица acc_clinets
``` 
select id from acc_clients where tid = X-Tenant-Id and client_id = jwt.client_id
```
3. Считаем что acc_clinet.id и его acc_account.id совпадают. Так проще администрировать. Этого можно добится при добавлении записи.
Проверяем роль по таблице acc_account_roles.
```
select user_role from acc_account_roles where tid = X-Tenant-Id and cid = clinet_id_из_пункта_3 
and ( account_id = X-Account-ID or ( X-Account-Id is null and is_defailt=true)) and user_login=jwt.user_login  
```
Запрос вернет 0 или 1 запись ( если что-то не накосячили в БД), это будет роль пользователя. Если она SYS_ADMIN то хорошо. Разрешаем все последующие действия )

## Добавить tenants
1. Добавляем 1 запись в БД в таблицу acc_tenants ( 1, 'VSK');
2. Добавить в таблицу accounts 1 строку ( parent_id = 0, name = VSK, tntid = 1 ('VSK') )
3. Добавляем админку этому арендатору. Запись в ACC_CLIENTS (tid=1,name='Adminke',Client_id='ADMINKA')
4. Добавить учетку админа, acc_account_logins(tid=1, user_login = admin_email )
5. Добавить запись в acc_account_logins(tid=1,cid=1,account_id=1 (экаунт тенанта вск),user_login=admin_login,user_role='TNT_ADMIN')
Это может сделать через админку sys_admin
```
X-Tenant-Id=1
X-Account-Id=10
jwt.user_login=admin_email_2
jwt.clinet_id='ADMINKA'
```
Получим такие данные. 
Проделав все проверки из (Добавить SYS_ADMIN) мы должны получить роль = 'TNT_ADMIN'
Обладая такой ролью, можно делать все что ниже -
~~~
ACC_ACCOUNT (id=10, parent_id=null, name='VSK', account_type='TENANT', tid = 0)         ------> TENANT (id=10,name='VSK')       
  │
  └── ACC_ACCOUNT (id=11, parent_id=10, name='Adminka', account_type='CLIENT', tid = 10)   ------> CLIENT ( tid = 10, name = 'Adminka', client_id='ADMINKA')
      │
      └── ACCOUNT_LOGIN (tid = 10, client_id = 11, account_id = 10, user_login = admin_email, user_role = 'TNT_ADMIN') ---> LOGINS (tid=10, user_login=user_email)
~~~

## Новый Client
1. Пользователь с ролью SYS_ADMIN(tenant из header) или TENANT_ADMIN (того же тенанта)
2. Через приложение Clain_id=ADMINKA
3. Добавить запись в acc_client(tid=tenant_id, id=10, name='New Partner', Client_id = 'XXX')
4. Добавить запись в acc_accounts ( tid=tenant_id, cid=10, account_type=CLIENT )

### АПИ интеграция без создания учеток пользователей
На этом все

~~~
ACC_ACCOUNT (id=10, parent_id=null, name='VSK', account_type='TENANT', tid = 0)         ------> TENANT (id=10,name='VSK')       
  │
  └── ACC_ACCOUNT (id=12, parent_id=10, name='SRAVNI-RU', account_type='CLIENT', tid = 10)   ------> CLIENT ( tid = 10, name = 'SRAVNI-RU', client_id='Sravni.RU', default_account_id=13)
      │
      └── ACC_ACCOUNT (id=13, parent_id=12, name='SRAVNI-RU Account', account_type='ACCOUNT', tid = 10)
~~~

### Интеграция с фронтом, будут создаваться учетки пользователей
5. Добавить запись в acc_account(tid=tenant_id, cid=10, account_type=ACCOUNT, parent_id=id из пункта 4)
6. Добавить учетка в Acc_login ( с ролью GROUP_ADMIN )
7. Добавить запись в acc_account_login ( логин из п 5, и cid созданного клиента )

~~~
ACC_ACCOUNT (id=10, parent_id=null, name='VSK', account_type='TENANT', tid = 0)         ------> TENANT (id=10,name='VSK')       
  │
  └── ACC_ACCOUNT (id=22, parent_id=10, name='SRAVNI-RU-2', account_type='CLIENT', tid = 10)   ------> CLIENT ( tid = 10, name = 'SRAVNI-RU-2', client_id='Sravni.RU.Ru')
      │
      └── ACC_ACCOUNT (id=23, parent_id=22, name='Страхование животных', account_type='ACCOUNT', tid = 10)
            │
            └── ACCOUNT_LOGIN (tid = 10, client_id = 22, account_id = 23, user_login = sale_email1, user_role = 'SALE') ---> LOGINS (tid=10, user_login=sale_email1)
            │
            └── ACCOUNT_LOGIN (tid = 10, client_id = 22, account_id = 23, user_login = sale_email2, user_role = 'SALE') ---> LOGINS (tid=10, user_login=sale_email2)
~~~

<table>
  <tr><td>Действие</td><td>SYS_ADMIN</td><td>TNT_ADMIN</td><td>GRP_ADMIN</td></tr>
  <tr><td>Tenant (CRUD)</td><td>yes</td><td>no</td><td>no</td></tr>
  <tr><td>Client (CRUD)</td><td>yes</td><td>yes, только для своего tenant-id</td><td>no</td></tr>
  <tr><td>Group  (CRUD)</td><td>yes</td><td>yes, только для своего tenant-id</td><td>yes, только для группы к которой админ прикреплен</td></tr>
  <tr><td>Account, Token, Sub (CRUD)</td><td>yes</td><td>yes, только для своего tenant-id</td><td>yes, только для своей группы</td></tr>

</table>

## Примеры 
Для примера есть 2 tenants - VSK (страховые продукты) и MSG (сервисные услуги)
Варианты передачи
1. header x-tenant-id
2. url {base_url}/tnt_code/API - ww.base.ru/pt/v2/__VSK__/policies или ww.base.ru/pt/v2/**MSG**/policies
   
### Запрос через нашу админку
```
jwt.user_login=admin_email_2
jwt.clinet_id='ADMINKA'
X-Account-Id или isPrimary из acc_user_accounts ( если вернет больше 1 строки то ошибка. нужно указать явно X-Account-Id )
X-Tenant-Id из acc_user_accounts
```
### Запрос через АПИ без user (только продажа полисов)
```
jwt.clinet_id='SRAVNI_RU'
X-Tenant-Id - из url 
X-Account-Id из таблицы acc_accounts по tenant_id и client_id
select id from acc_accounts where tid = X-Tenant-Id and cid = jwt.clinet_id
При такой модели должна быть только 1 запись в acc_accounts
```
### Запрос через АПИ
```
jwt.user_login=user_email
jwt.clinet_id='SRAVNI_RU'
X-Tenant-Id - из header

X-Account-Id из header или таблицы acc_account_logins по tenant_id и client_id и user_login
```


