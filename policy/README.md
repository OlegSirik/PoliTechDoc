# Договор страхования

В данном контексте рассматривается только простой договор страхования, с одним объектом страхования. 
Объектом страхования может быть массив, но только если покрытие по договору одинаковое для всех элементов массива.

```
{
  "draftId": "string",
  "policyNumber": "string",
  "previousPolicyNumber": "string",
  "product": "string",
  "status": "NEW",
  "startDate": "string",
  "endDate": "string",
  "issueDate": "string",
  "waitingPeriod": "P16D",
  "policyTerm": "P100D",
  "createDate": "string",
  "premium": 1000,
  "placeOfIssue": "string",
  "policyHolder": {
    "person": {},
    "contacts": {},
    "identifiers": [{}],
    "addresses": [{}],
    "organization": {}
  },
  "packageCode": "BASIC",
  "coverage": [{
    "cover": {
      "code": "string"
    },
    "risk": "string",
    "startDate": "string",
    "endDate": "string",
    "sumInsured": 1000,
    "premium": 1000,
    "deductibleNr": 2,
    "deductible": "10% со вторго страхового случая"
  }],

  "insuredObjects": [
    {
      "ioType": "Person",
      "objectId": "string",
      "person": {},
    },
    {
      "ioType": "CE",
      "device": {
        "deviceName": "string",
        "deviceTypeCode": "string",
        "tradeMark": "string",
        "model": "string",
        "serialNr": "string",
        "licenseKey": "string",
        "imei": "string",
        "osName": "string",
        "osVersion": "string",
        "countryCode": "string",
        "devicePrice": 0
      }
    },
    {
      "ioType": "Property",
      "objectId": "string",
      "property": {
        "propertyType": {
          "code": "string"
        },
        "address": {
          "isPrimary": false,
          "typeCode": "REGISTRATION",
          "addressStr": "129515, г.Москва, ул.Академика Королева, д.2, к.3, кв.25",
        },
        "cadastrNr": "77:07:0018002:2590",
      }
    }
  ],
  "options": [
    {
      "code": "ROADSIDE_ASSISTANCE",
      "name": "Помощь на дороге",
      "category": "string",
      "premium": 0,
      "contractNr": "string"
    }
  ]
}
```

## Шапка договора
Шапка договора содержит номер, даты, общую премию, способ оплаты
### Номер
Номер договора, ИД котировки, номер предыдущего договора, если есть
### Страховая компания
Наименование страховой компании, выпустивший договор
### Статус
Статусы -
NEW - котировка
ISSUED - выпущенный договор, по которому еще не пришла оплата
PAID - оплаченный договор
### Даты
Дата выпуска договора
Дата начала и окончания действия
Дата оплаты первого взноса
Дата расторжения
### Премия
Общая премия по договору в рублях.
При необходимости можно указать сумму в валюте.
### Страхователь
Страхователь по договору может быть юр и физ лицом.
Если передается только раздел person, то считается что физ.лицо
Если добавить organization, то ИП
Если только organization, то это юр.лицо
### Покрытия по договору
Покрытия по договору хранятся в массива coverage. Каждый объект содержит код типа покрытия, даты начала-окончания покрытия, лимит и премию по покрытию, франшизу.
Если по продукту есть пакет обяхательных покрытий, то он передается в атрибуте packageCode
### Объект страхования
Объект страхования это массив. Количество объектов в массиве зависит от настройки продукта.
Тип объекта страхования содержитсяв ioType. Атрибутивный состав задается через настройку продукта. 
### Доп опции по договору
Это могут быть, например, не страховые услуги.
Эвакуатор, замена колес, помощь оценщика и т.д.
Опция содержит код, наименование, категорию (если большой список, по категории можно сгруппировать на UI) и стоимость.


# Хранение
uuid - уникальный ИД записи.    
policy_data_uuid ссылка на данные договора. Данные могут не менятся, только статус
draft_id - генерится при создании первой версии и далее не меняется.   
policy_nr - генерится до оплаты.    
version_nr - номер ревизии. ( version_nr & draft_id уникальны )   
business_version_nr - юридическая версия договора   
status ( WIP / ISSUED ) статус выерсии - в работе или выпущена   
top_version_wip ( bool ) - указатель на последнюю тех версию договора.   
top_version_issued ( bool ) - указатель на последнюю версию договора.   

## Текущая версия в проде
~~~
select * from policy_index where draft_id = :draft_id and top_version_issued = true
~~~
## Текущая версия в работе
~~~
select * from policy_index where draft_id = :draft_id and top_version_wip = true
~~~

## 1. Новый договор
new - in_payment - paid 
                   error 
~~~
draft_id = 123
policy_nr = 22TTT999
version_nr = 1
business_version_nr = 0
status = 'WIP'
top_version_wip = true
top_version_issued = false
policy_status = 'NEW'
~~~

## 2. Ожидает оплаты
Договор фактически выпущен
~~~
draft_id = 123
policy_nr = 22TTT999
version_nr = 2
business_version_nr = 0
status = 'ISSUED'
top_version_wip = false
top_version_issued = true
policy_status = 'IN_PAYMENT'
~~~

## 3.1. Оплачен
~~~
draft_id = 123
policy_nr = 22TTT999
version_nr = 3
business_version_nr = 0
status = 'ISSUED'
top_version_wip = false
top_version_issued = true
policy_status = 'PAID'
~~~
## 3.2. Ошибка
~~~
draft_id = 123
policy_nr = 22TTT999
version_nr = 3
business_version_nr = 0
status = 'WIP'
top_version_wip = true
top_version_issued = false
policy_status = 'ERROR'
~~~

## Создать допник
Получить тек. бизнес версию (select business_version_nr from policy_index where top_version_issued = true)
Проверить что уже не создана версия WIP (select * from policy_index where business_version_nr = :business_version_nr and top_version_wip = true)
Если нет, то создать новую, иначе ошибка.
