Обновленный фрагмент кода PlantUML, который отражает  изменения и дополнения. В схеме добавлены osago-aggregator, подходы к API, средства интеграции, а также паттерны отказоустойчивости.



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
    component "OSAGO Aggregator" as osagoAggregator <<Container: Kotlin, SpringBoot>>
    database "OSAGO DB" as osagoDB <<PostgreSQL>>
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
webApp --> coreApp: "GraphQL Queries/Mutations"
coreApp --> productAggregator: "Get Product Tariffs"
coreApp -> kafka: "Publishes Events (Policy Creation, Settlement Updates)"
coreApp -> outboxTable: "Writes Events"
outboxTable -> kafka: "Publishes Events"
coreApp --> coreDB: "Reads/Writes"
coreApp --> osagoAggregator: "Get OSAGO Tariffs [gRPC]"
osagoAggregator --> osagoDB: "Reads/Writes"
osagoAggregator --> insuranceSystems: "Fetch Tariffs [REST/SOAP]"

productAggregator -> tariffTopic: "Subscribes to Tariff Updates"
productAggregator --> insuranceSystems: "Requests Tariffs [REST/SOAP/GraphQL]"

settlementService -> settlementTopic: "Subscribes to Settlement Events"
settlementService --> settlementDB: "Reads/Writes"
settlementService --> insuranceSystems: "Processes Settlements [REST]"

paymentService --> coreApp: "Processes Payments [REST]"
partnerSystem --> coreApp: "Partners Register Policies [REST]"

note right of coreApp
  - Rate Limiting for external API calls
  - Circuit Breaker for osagoAggregator
  - Retry with exponential backoff
  - Timeout for responses
end note

@enduml

```


## Добавлен osago-aggregator:

Предоставляет API на базе gRPC для взаимодействия с core-app, что позволяет минимизировать задержки и повысить эффективность.
Использует своё хранилище данных osago_db для хранения тарифов ОСАГО и сопутствующих данных.
Паттерны отказоустойчивости:

 - Circuit Breaker для взаимодействия с osago-aggregator, чтобы предотвратить каскадные отказы.
 - Rate Limiter для ограничения нагрузки на ins-product-aggregator.
 - Retry + Timeout для обработки временных сбоев при вызовах client-info.
 - Средство интеграции между веб-приложением и core-app:

## Используется HTTP для взаимодействия с веб-приложением.
Учёт развертывания в нескольких экземплярах:

 - Добавлены паттерны для поддержки отказоустойчивости при масштабировании сервисов.
 - Эта схема поможет более эффективно масштабировать приложение и обеспечить его устойчивость при росте нагрузки.