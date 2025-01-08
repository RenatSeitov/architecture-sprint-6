Для создания эквивалентной схемы GraphQL, основанной на существующем контракте Swagger, следует создать схемы для сущностей Client, Document и Relative, а также запросы, которые позволят получить необходимую информацию. В GraphQL важно, чтобы клиент мог запрашивать только нужные данные, избегая дублирования информации и улучшая гибкость.

### Сущности:
1. *Client*
2. *Document*
3. *Relative*

### GraphQL Схема:

graphql
# Определение сущности клиента
type Client {
  id: ID!
  name: String!
  age: Int!
  documents: [Document!]!
  relatives: [Relative!]!
}

# Определение сущности документа
type Document {
  id: ID!
  type: String!
  number: String!
  issueDate: String!
  expiryDate: String!
}

# Определение сущности родственника
type Relative {
  id: ID!
  relationType: String!
  name: String!
  age: Int!
}

# Тип запроса для получения данных
type Query {
  # Получение информации о клиенте по ID
  client(id: ID!): Client

  # Получение списка документов клиента по ID
  clientDocuments(id: ID!): [Document!]!

  # Получение информации о родственниках клиента по ID
  clientRelatives(id: ID!): [Relative!]!
}


### Пояснение:
- *Client*: включает поля id, name, age, а также связи с документами и родственниками, которые являются массивами объектов Document и Relative.
- *Document*: представляет собой документ с полями id, type, number, issueDate и expiryDate.
- *Relative*: представляет собой родственника с полями id, relationType, name и age.

### Запросы:
- *client(id: ID!)*: Запрашивает информацию о клиенте по его уникальному идентификатору.
- *clientDocuments(id: ID!)*: Запрашивает все документы клиента по его ID.
- *clientRelatives(id: ID!)*: Запрашивает информацию о родственниках клиента по его ID.

### Примеры запросов:
1. *Запрос информации о клиенте*:
   graphql
   query {
     client(id: "12345") {
       id
       name
       age
     }
   }
   

2. *Запрос документов клиента*:
   graphql
   query {
     clientDocuments(id: "12345") {
       id
       type
       number
       issueDate
       expiryDate
     }
   }
   

3. *Запрос родственников клиента*:
   graphql
   query {
     clientRelatives(id: "12345") {
       id
       relationType
       name
       age
     }
   }
   

### Преимущества схемы GraphQL:
- *Гибкость запросов*: Пользователи могут запрашивать только необходимые поля для клиента, документов и родственников.
- *Избежание избыточных данных*: Запросы на основе GraphQL позволят получать только нужную информацию, что устраняет избыточность данных.
- *Единый источник правды*: С помощью GraphQL можно интегрировать различные ресурсы (клиенты, документы, родственники) в единую схему, предоставляя доступ к данным через единый API.