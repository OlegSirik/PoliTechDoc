## Создание новой версии продукта

~~~
POST /{tid}/products/{id}/versions/cmd/new-version
~~~

### Preconditions
1. Нет текущей версии в статусе DEV ( в процессе разработки )
~~~
select prod_version_no, dev_version_no from pt_products where id = :id 
~~~
Если dev_version_no, вернуть ошибку, что уже есть версия в разработке. 

### Process
1. Взять product из pt_product_version where product_id = :id and version_no = prod_version_no
2. Добавить след. версию в pt_product_version. version_no =+ 1
3. Обновить номер dev версии в pt_products.dev_version_no
4. Скопировать калькуляторы, если есть, в новую версию
~~~
select * from pr_calculators where product_id = :id and version_no = version_no
~~~
добавить записи с version_no =+ 1
