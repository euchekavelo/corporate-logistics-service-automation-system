# Система автоматизации корпоративной логистики

## Описание

Указанная система отражает работу некоторого перечня распространенных бизнес-процессов, 
которые могут происходить при оформлении любого онлайн-заказа, начиная от его создания до самой доставки конечному потребителю.


## Используемый стек

* Spring Boot 2.7.9
* Java 17
* Prometheus
* Apache Kafka
* PostgreSQL
* Docker
* Spring Data JPA
* Spring Security
* Spring Cloud
* Apache Maven
* Swagger OpenAPI
* Lombok


## Состав системы

Проект содержит следующие микросервисы:
1. Gateway-service (Шлюз)
2. Discovery-service (Сервис обнаружения, который объединён с Config-service)
3. Auth-service (Сервис аутентификации)
4. Order-service (Сервис заказов)
5. Payment-service (Сервис платежей)
6. Inventory-service (Сервис инвентаря)
7. Delivery-service (Сервис доставки)


## Окружение

Для запуска необходимой инфраструктуры (PostgreSQL, Kafka, Prometheus) необходимо выполнить следующую команду в корневой директории проекта:
```bash
docker-compose up -d
```

Для визуального мониторинга сообщений в партициях доступен удобный [веб-интерфейс для Apache Kafka](https://github.com/provectus/kafka-ui):

http://localhost:9999/

Теперь вы можете запустить сервисы приложения из вашей IDE в следующем порядке
- Discovery
- Auth-service
- Order-service
- Payment-service
- Inventory-service
- Delivery-service
- Gateway-service

После этого на шлюзе (Gateway) вы найдёте объединённый Swagger UI по адресу:

http://localhost:9090/swagger-ui.html


## Базовые возможности

Вы можете использовать приведённые ниже curl-команды или выполнить все действия через Swagger UI.

### Сервис аутентификации

Для создания пользователя отправьте следующий запрос к сервису аутентификации:
```bash
curl --location --request POST 'http://localhost:9090/auth-service/user/signup' \
--header 'Content-Type: application/json' \
--data '{
    "name": "user1",
    "password": "user1"
}'
```

После этого вы можете получить токен:
```bash
curl --location --request POST 'http://localhost:9090/auth-service/token/generate' \
--header 'Content-Type: application/json' \
--data '{
    "name": "user1",
    "password": "user1"
}'
```

Теперь вы можете использовать этот токен для аутентификации запросов к другим сервисам.

### Сервис платежей

Для создания баланса отправьте POST-запрос:

```bash
curl --location --request POST 'http://localhost:9090/payment-service/balance' \
--header 'Authorization: Bearer <вставьте токен сюда>'
```

Для пополнения баланса пользователя отправьте PATCH-запрос:

```bash
curl --location --request PATCH 'http://localhost:9090/payment-service/balance/1' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <вставьте токен сюда>' \
--data '{
    "sum": 9999999
}'
```

### Сервис инвентаря

Для создания записи инвентаря отправьте POST-запрос:

```bash
curl --location --request POST 'http://localhost:9090/inventory-service/inventory' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <вставьте токен сюда>' \
--data '{
    "title": "Test inventory",
    "count": 22,
    "costPerPiece": 10
}'
```

Для пополнения инвентаря отправьте PATCH-запрос:

```bash
curl --location --request PATCH 'http://localhost:9090/inventory-service/inventory/1' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <вставьте токен сюда>' \
--data '{
    "count": 20
}'
```

### Сервис заказов

Для создания заказа отправьте POST-запрос:

```bash
curl --location --request POST 'http://localhost:9090/order-service/order' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <вставьте токен сюда>' \
--data '{
  "orderDtoList": [
    {
      "productId": 1,
      "count": 2
    }
  ],
  "cost": 20,
  "destinationAddress": "test"
}'
```

Вы можете изменить статус заказа с помощью PATCH-запроса:

```bash
curl --location --request PATCH 'http://localhost:9090/order-service/order/1' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <вставьте токен сюда>' \
--data '{
    "status": "REGISTERED",
    "serviceName": "ORDER_SERVICE",
    "comment": "Some comment to status"
}'
```

### Сервис доставки

Для удаления доставки отправьте DELETE-запрос:

```bash
curl --location --request DELETE 'http://localhost:9090/delivery-service/delivery/1' \
--header 'Authorization: Bearer <put token here>'
```


## Мониторинг

Дополнительно с каждого микросервиса системы производится сбор метрик.
Вы также можете просматривать различные метрики приложения в рамках мониторинга и их последующей визуализации:

http://localhost:9998/ 
