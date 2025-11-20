# Процесс быстрого расчете
При работе через UI пользователю нужно выбрать продукт, который он собирается продать страхователю.

Чтобы быстрый расчёт (Quick Quote) работал без выбора продукта заранее и при этом подстраивался под разные продукты страхования дома, вопросник должен быть основан не на продукте, а на характеристиках объекта страхования и квалификационных фильтрах.

Для одного объекта страхования таких продуктов может быть много, и чтобы не перегружать агента или страхователя выбором, процесс продажи должен начинаться не с выбора продукта, а с заполнения вопросника.
Выбирается линия бизнеса, например страхование имущества. Далее систем формирует набор вопросов, получив ответы на которые, система может предложить списов подходящих продуктов.
Через API список вопросов можно получить через GET /lobs/{lob_id}/questionary
POST /lob/{lob_id}/ body = заполненный вопросник, в ответ придет список продуктов, с описанием, которые подходят под условия.

Главная идея

Сначала собираем ключевые характеристики объекта, чтобы определить список подходящих программ/продуктов.
Потом — дополнительные вопросы для выбранного продукта.

То есть:
Questions → Eligibility + Classification → Product candidates → Product questions → Quote

Пример:

Q: objectType = "House"
Q: constructionType = "Concrete"
Q: area = 160 m2
Q: yearBuilt = 2015

Система вычисляет:

Eligible Products = [HomeBasic, HomeExtended]
Not Eligible = [ApartmentBasic, FlatBasic]

Правила выбора продукта

Каждый продукт имеет правила применимости (eligibility):

{
  "productId": "HomeExtended",
  "rules": [
    "objectType == 'House'",
    "constructionType IN ['Concrete','Brick']",
    "area <= 500"
  ]
}

⚙ Как работает процесс
Flow
flowchart TD
A[Enter object details] --> B[Evaluate eligibility rules]
B --> C[Show list of eligible products]
C --> D[Ask product-specific questions]
D --> E[Final quote]

Best Practices (Guidewire, DuckCreek)

Вопросник = Attribute-based underwriting

Продукт определяется автоматически из Attributes → Coverage Pattern

Выбор продукта после сбора информации, а не до
