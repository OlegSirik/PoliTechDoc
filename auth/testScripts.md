# Основные UC
## Добавить SYS_ADMIN
SYS_ADMIN может выполнять любые действия в настройкой прав доступа.

1. Добавляем 1 запись в БД в таблицу acc_tenants ( 0, 'ROOT'); Для ссылочной целостности.
2. Добавить запись ROOT в acc_accounts ( id = 0, parent_id = null, account_type = 'TENANT', tid = 0 )
3. Admin будет работать через какоето приложение. Поэтому нужно добавить строку в acc_clients
Например ( tid = 0, name = 'Adminka', client_id = 'ADMINKA' ( это client_id, которое потом придет из Кейклоки )
4. Добавить запись в ACC_LOGINS (tid = 0, id = sequence, user_login = admin_email )
5. Добавить запись в acc_account_logins( tid = 0, client_id = 1, account_id = 0 (экаунт рута), user_login = admin_email, user_role = 'SYS_ADMIN' )

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

## Новый Client
1. Пользователь с ролью SYS_ADMIN(tenant из header) или TENANT_ADMIN (того же тенанта)
2. Через приложение Clain_id=ADMINKA
3. Добавить запись в acc_client(tid=tenant_id, id=10, name='New Partner', Client_id = 'XXX')
4. Добавить запись в acc_accounts ( tid=tenant_id, cid=10, account_type=CLIENT )

### АПИ интеграция без создания учеток пользователей
5. Добавить запись в acc_account(tid=tenant_id, cid=10, account_type=ACCOUNT, parent_id=id из пункта 4)
6. Обновить запись из п4. установить default_account_id = id из пунка 5

### Интеграция с фронтом, будут создаваться учетки пользователей
5. Добавить учетка в Acc_login ( с ролью GROUP_ADMIN )
6. Добавить запись в acc_account_login ( логин из п 5, и cid созданного клиента )


<table>
  <tr><td>Действие</td><td>SYS_ADMIN</td><td>TNT_ADMIN</td><td>GRP_ADMIN</td></tr>
  <tr><td>Tenant (CRUD)</td><td>yes</td><td>no</td><td>no</td></tr>
  <tr><td>Client (CRUD)</td><td>yes</td><td>yes, только для своего tenant-id</td><td>no</td></tr>
  <tr><td>Group  (CRUD)</td><td>yes</td><td>yes, только для своего tenant-id</td><td>yes, только для группы к которой админ прикреплен</td></tr>
  <tr><td>Account, Token, Sub (CRUD)</td><td>yes</td><td>yes, только для своего tenant-id</td><td>yes, только для своей группы</td></tr>

</table>
