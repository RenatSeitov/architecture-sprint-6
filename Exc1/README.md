
## Context Diagram


```
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

Person(user, "Пользователь", "Работает с приложением через браузер.")
System_Boundary(inuretech, "InureTech System") {
    System(frontend, "Frontend", "Веб-интерфейс для пользователей.")
    System(backend, "Backend", "Обработка запросов и бизнес-логика.")
    SystemDb(database, "PostgreSQL Database", "Хранение данных о пользователях и полисах.")
}

Rel(user, frontend, "Использует")
Rel(frontend, backend, "Отправляет запросы")
Rel(backend, database, "Читает и записывает данные")

@enduml

```

## Container Diagram


```
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

Person(user, "Пользователь", "Работает с приложением через браузер.")
System_Boundary(inuretech, "InureTech System") {
    Container(frontend, "Frontend", "React", "Отображение интерфейса и взаимодействие с пользователем.")
    Container(backend, "Backend", "Python (FastAPI)", "Обработка бизнес-логики и API.")
    ContainerDb(database, "PostgreSQL", "База данных", "Хранение пользовательских данных.")
    Container(api_gateway, "API Gateway", "NGINX", "Маршрутизация запросов и балансировка нагрузки.")
}

Rel(user, api_gateway, "Отправляет запросы через")
Rel(api_gateway, frontend, "Маршрутизирует запросы на")
Rel(api_gateway, backend, "Маршрутизирует API-запросы на")
Rel(backend, database, "Чтение и запись данных")

@enduml

```

## Component Diagram

```
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

Container(backend, "Backend", "Python (FastAPI)", "Обработка бизнес-логики и API.")
ContainerDb(database, "PostgreSQL", "База данных", "Хранение пользовательских данных.")

Component(auth_service, "Authentication Service", "Python", "Управляет регистрацией и авторизацией.")
Component(policy_mgmt, "Policy Management", "Python", "Управление страховыми полисами.")
Component(claim_proc, "Claim Processing", "Python", "Обработка заявок на страховые случаи.")

Rel(auth_service, database, "Запрашивает данные пользователей")
Rel(policy_mgmt, database, "Запрашивает данные полисов")
Rel(claim_proc, database, "Обновляет заявки")

Rel(backend, auth_service, "Вызывает")
Rel(backend, policy_mgmt, "Вызывает")
Rel(backend, claim_proc, "Вызывает")

@enduml

```


## Infrastracture

```
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Deployment.puml

Deployment_Node(region, "Yandex Cloud Region", "Облачная платформа") {
    Deployment_Node(zone1, "Зона доступности 1", "Кластер Kubernetes") {
        Deployment_Node(k8s_cluster, "Kubernetes Cluster", "Контейнеры") {
            Container(frontend, "Frontend", "React", "Веб-интерфейс.")
            Container(backend, "Backend", "Python (FastAPI)", "Бизнес-логика.")
        }
        Deployment_Node(database_vm, "PostgreSQL VM", "Virtual Machine") {
            ContainerDb(database, "PostgreSQL (master)", "Хранение данных.")
        }
    }
}

Deployment_Node(zone2, "Зона доступности 2", "Кластер Kubernetes") {
    Deployment_Node(k8s_cluster_2, "Kubernetes Cluster", "Контейнеры") {
        Container(frontend_2, "Frontend", "React", "Реплика интерфейса.")
        Container(backend_2, "Backend", "Python (FastAPI)", "Реплика логики.")
    }
    Deployment_Node(database_vm_2, "PostgreSQL Replica", "Virtual Machine") {
        ContainerDb(database_replica, "PostgreSQL (replica)", "Чтение данных.")
    }
}

Rel(frontend, backend, "Обращается к")
Rel(backend, database, "Записывает данные")
Rel(database, database_replica, "Реплицирует данные (asynchronous)")
Rel(frontend, frontend_2, "Балансировка нагрузки")
@enduml
```

### Ссылка на drow.io: https://drive.google.com/file/d/1ZSD3aPneajptgeHtRdBzPPbqDH-i9iY7/view?usp=sharing