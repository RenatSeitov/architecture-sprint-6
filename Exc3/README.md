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

actor Client as client

package "InsureTech Web" {
    component "Web App" as webApp <<Container: JavaScript, React>>
}

package "InsureTech Prod" {
    component "Core App" as coreApp <<Container: Kotlin, SpringBoot>>
    component "Core DB" as coreDB <<Container: PostgreSQL>>
    component "Client Info" as clientInfo <<Container: Kotlin, SpringBoot>>
    component "Product Aggregator" as productAggregator <<Container: Kotlin, SpringBoot>>
    component "Settlement Service" as settlementService <<Container: Kotlin, SpringBoot>>
    component "Settlement DB" as settlementDB <<Container: PostgreSQL>>
}

package "Payment Service" {
    component "External Payment" as paymentService <<Software System>>
}

package "Partner Systems" {
    component "Partner System" as partnerSystem <<Software System>>
}

package "Insurance Companies" {
    component "Insurance Systems" as insuranceSystems <<Software System>>
}

package "Event Streaming" {
    component "Kafka" as kafka <<Event Broker>>
    queue "tariff-update-topic" as tariffTopic
    queue "policy-creation-topic" as policyTopic
    queue "settlement-update-topic" as settlementTopic
}

package "Transactional Outbox" {
    database "Outbox Table" as outboxTable
}

client --> webApp: "Interacts"
webApp --> coreApp: "Get Tariffs / Create Policies [REST]"
coreApp --> productAggregator: "Get Product Tariffs"
coreApp -> kafka: "Publishes Events (Policy Creation, Settlement Updates)"
coreApp -> outboxTable: "Writes Events"
outboxTable -> kafka: "Publishes Events"
coreApp --> coreDB: "Reads/Writes"

productAggregator -> tariffTopic: "Subscribes to Tariff Updates"
productAggregator --> insuranceSystems: "Requests Tariffs [REST/SOAP/GraphQL]"

settlementService -> settlementTopic: "Subscribes to Settlement Events"
settlementService --> settlementDB: "Reads/Writes"
settlementService --> insuranceSystems: "Processes Settlements [REST]"

paymentService --> coreApp: "Processes Payments [REST]"
partnerSystem --> coreApp: "Partners Register Policies [REST]"

@enduml
```