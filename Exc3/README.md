# Анализ текущей архитектуры

## Текущие проблемы и риски:

### Узкие места в производительности:
  - Синхронные REST-взаимодействия между сервисами увеличивают задержки при большом количестве запросов.
  - Отсутствие Event-Streaming может стать узким местом для обработки больших объемов данных.
### Отказоустойчивость:
  - Нет явного механизма обеспечения обработки ошибок при передаче данных между сервисами.
  - Центральная зависимость от базы данных (например, ins-comp-settlement-db) создает риск потери данных при сбое.
### Масштабирование:
  - Отсутствие асинхронной коммуникации между ключевыми сервисами ограничивает горизонтальное масштабирование.
  - Базы данных не настроены для шардирования или репликации.
### Транзакционность:
  - Нет реализации паттерна Transactional Outbox, что может привести к неконсистентности данных при сбое.

## Пример решения:

### Event-Streaming:
  - Внедрение Kafka для асинхронного обмена данными между core-app, client-info и ins-product-aggregator.
### Transactional Outbox:
  - Добавить механизм для безопасного сохранения событий в базу данных с последующей публикацией в стриминг-систему.
### Репликация и отказоустойчивость баз данных:
  - Настроить master-slave репликацию для core-db и ins-comp-settlement-db.


# Обновление диаграммы контейнеров

Внесите изменения в диаграмму контейнеров InsureTech для внедрения Event-Streaming и Transactional Outbox.

Основные изменения:

Добавьте Kafka или другой стриминг-сервис:
    Используйте его для передачи данных между:
      - core-app и ins-product-aggregator
      - core-app и client-info
      - Это снизит задержки и улучшит масштабируемость.
    Внедрите паттерн Transactional Outbox:
     -  На уровне core-db и ins-comp-settlement-db добавьте таблицу для хранения событий, которые публикуются в стриминг-сервис.
    Масштабирование:
      - Добавьте репликацию баз данных.
      - Укажите механизмы обработки отказов и распределения нагрузки.

```
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

Person(user, "Клиент", "Пользователь системы InsureTech")
System_Boundary(InsureTech, "InsureTech System") {
    Container(core_app, "Core App", "Kotlin + SpringBoot", "Основная бизнес-логика")
    Container(client_info, "Client Info", "Kotlin + SpringBoot", "Сервис учета клиентских данных")
    Container(ins_product_aggregator, "Ins Product Aggregator", "Kotlin + SpringBoot", "Агрегация продуктов")
    Container(ins_comp_settlement, "Ins Comp Settlement", "Kotlin + SpringBoot", "Сервис оформления страховок")
    ContainerDb(core_db, "Core DB", "PostgreSQL", "Тарифы и продукты")
    ContainerDb(ins_comp_settlement_db, "Ins Comp Settlement DB", "PostgreSQL", "Информация о клиентах и полисах")
    Container(kafka, "Kafka", "Event-Streaming", "Механизм асинхронной обработки событий")
}

Rel(user, core_app, "HTTP")
Rel(core_app, client_info, "Асинхронный вызов через Kafka")
Rel(core_app, ins_product_aggregator, "Асинхронный вызов через Kafka")
Rel(core_app, ins_comp_settlement, "HTTP")
Rel(core_app, core_db, "Запись/чтение данных")
Rel(ins_comp_settlement, ins_comp_settlement_db, "Запись/чтение данных")

@enduml
```