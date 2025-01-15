
@startuml

title "InureTech_технологическая архитектура_to-be"

' Опционально можно настроить стиль оформления
skinparam rectangle {
  BackgroundColor #FFFFFF
  BorderColor #333333
  FontName Arial
}
skinparam actor {
  BackgroundColor #FFFFFF
  BorderColor #333333
  FontName Arial
}
skinparam arrow {
  Color #A9A9A9
  FontColor #A9A9A9
  FontName Arial
}
skinparam component {
  BackgroundColor #FFFFFF
  BorderColor #333333
  FontName Arial
}
skinparam note {
  BackgroundColor #FFFFEE
  BorderColor #B8860B
}

actor endUser as "Браузер пользователя"

rectangle "Yandex Cloud" as YCloud {
  
  ' Балансировщик (внешний точка входа)
  component "Load Balancer\n(Health Checks)" as LB
  
  ' Первая зона доступности
  rectangle "Зона доступности 1 (AZ1)" as AZ1 {
    component "Kubernetes Cluster\n(или Namespace InsureTech)" as K8s1
    component "PostgreSQL (Master)" as PGMaster
  }
  
  ' Вторая зона доступности
  rectangle "Зона доступности 2 (AZ2)" as AZ2 {
    component "Kubernetes Cluster\n(или Namespace InsureTech)" as K8s2
    component "PostgreSQL (Replica)" as PGReplica
  }
  
  ' Объектное хранилище для резервных копий
  artifact "Object Storage (Backups)" as Storage
}

' Взаимодействие актёра с балансировщиком
endUser --> LB : HTTP/HTTPS

' Балансировщик распределяет нагрузку по зонам
LB --> K8s1 : HTTP/HTTPS
LB --> K8s2 : HTTP/HTTPS

' Приложение в Kubernetes взаимодействует с БД
K8s1 --> PGMaster : TCP (записи/чтение)
K8s2 --> PGMaster : TCP (записи/чтение)

' Репликация БД
PGMaster --> PGReplica : "Синхр./Асинхр. Репликация"

' Резервное копирование
PGMaster --> Storage : "Backups (eg. WAL, basebackup)"
PGReplica --> Storage : "Backups (при необходимости)"

' Пример комментария о виде кластера K8s
note right of K8s1
  Здесь может быть комментарий:
  "Используем один растянутый кластер K8s в 2 AZ"
  или
  "Используем 2 независимых кластера для повышения отказоустойчивости."
end note

@enduml