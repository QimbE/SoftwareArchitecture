# Лабораторная работа №4

Тема: Проектирование REST API
Цель работы: Получить опыт проектирования программного интерфейса.

## Выбранный сервис

**Order Service** — сервис управления заказами в системе заказа сэндвичей.

Данный сервис отвечает за:
- Создание и управление заказами клиентов
- Отслеживание статуса заказа
- Управление составом заказа
- Получение информации о доступном меню
- Расчет стоимости заказа

---

## Проектные решения при проектировании REST API

### 1. Версионирование API через URI

**Решение:** Версия API включается в путь запроса в формате `/api/v1/resource`.

**Обоснование:**
- Явное указание версии делает API предсказуемым и стабильным
- Позволяет поддерживать несколько версий API одновременно при миграции клиентов
- Упрощает обратную совместимость при внесении breaking changes
- Версионирование через URI проще для документирования и понимания клиентами

---

### 2. Использование правильных HTTP методов (REST principles)

**Решение:** Строгое соблюдение семантики HTTP методов:
- **GET** — только для чтения данных (безопасная операция, без side effects)
- **POST** — создание новых ресурсов
- **PUT** — полное обновление существующего ресурса (идемпотентная операция)
- **DELETE** — удаление ресурса (идемпотентная операция)

**Обоснование:**
- Следование REST conventions делает API интуитивно понятным
- Идемпотентность PUT и DELETE гарантирует, что повторный запрос даст тот же результат (важно при сетевых сбоях)
- Правильное использование GET позволяет кешировать ответы
- Соответствие стандартам упрощает интеграцию с HTTP-клиентами и прокси

---

### 3. Использование HTTP статус-кодов согласно стандарту RFC 7231

**Решение:** Возвращать правильные HTTP коды для каждого сценария:
- **200 OK** — успешная операция GET/PUT
- **201 Created** — ресурс успешно создан (POST)
- **204 No Content** — успешная операция без тела ответа (DELETE)
- **400 Bad Request** — ошибка валидации входных данных
- **401 Unauthorized** — отсутствует или невалидный токен аутентификации
- **403 Forbidden** — недостаточно прав (например, клиент пытается изменить чужой заказ)
- **404 Not Found** — запрашиваемый ресурс не существует
- **409 Conflict** — конфликт бизнес-логики (например, нельзя удалить заказ в статусе "В доставке")
- **500 Internal Server Error** — внутренняя ошибка сервера

**Обоснование:**
- Стандартизированные коды позволяют клиентам единообразно обрабатывать ответы
- Упрощается диагностика проблем (логирование, мониторинг)
- HTTP-клиенты и фреймворки имеют встроенную поддержку обработки стандартных кодов
- Соответствие стандартам делает API профессиональным и предсказуемым

---

### 4. Единый формат данных — JSON с camelCase

**Решение:**
- Все запросы и ответы используют `Content-Type: application/json`
- Именование полей в **camelCase** для соответствия JavaScript/TypeScript конвенциям
- UTF-8 кодировка для поддержки международных символов

**Обоснование:**
- JSON — стандарт де-факто для REST API (легковесный, читаемый, поддерживается всеми языками)
- camelCase — конвенция JavaScript/TypeScript, основных языков фронтенда
- Единообразие упрощает парсинг и сериализацию на клиенте и сервере
- UTF-8 необходим для поддержки мультиязычности (требование расширения за рубеж)

---

### 5. Аутентификация и авторизация через JWT Bearer Token

**Решение:**
- Все endpoints (кроме публичных, например GET /api/v1/menu) требуют JWT токен в заголовке
- Формат: `Authorization: Bearer <token>`
- Токен содержит `userId` и `role` для авторизации

**Обоснование:**
- JWT — stateless токены, не требуют хранения сессий на сервере (критично для масштабируемости)
- Bearer схема — стандарт OAuth 2.0 (RFC 6750)
- Токен содержит информацию о пользователе и роли, что позволяет сервису самостоятельно проверять права доступа
- Интеграция с User Service через общий секретный ключ для валидации JWT

**Правила авторизации:**
- Клиент может видеть/изменять только свои заказы
- Сотрудник магазина может видеть заказы своего магазина и обновлять их статус
- Водитель доставки видит заказы, назначенные ему

---

### 6. Идемпотентность PUT и DELETE операций

**Решение:**
- **PUT /api/v1/orders/{id}** — повторный запрос с теми же данными дает тот же результат (заказ обновлен)
- **DELETE /api/v1/orders/{id}** — повторный запрос возвращает 404 (заказ уже отменен), но это безопасно

**Обоснование:**
- Сетевые сбои могут привести к повторной отправке запроса клиентом
- Идемпотентность гарантирует, что повторный запрос не вызовет дублирования или ошибок
- Критично для операций изменения данных (отмена заказа не должна создавать несколько отмен)
- Упрощается реализация retry-логики на клиенте

---

### 7. Стандартизированный формат ошибок (Problem Details RFC 7807)

**Решение:** Все ошибки возвращаются в едином формате:
```json
{
  "error": "ErrorCode",
  "message": "Human-readable description",
  "details": { /* optional additional context */ }
}
```

**Обоснование:**
- Единый формат упрощает обработку ошибок на клиенте
- `error` — машиночитаемый код для программной обработки (например, показ локализованного сообщения)
- `message` — человекочитаемое описание для отладки
- `details` — дополнительный контекст (например, какие поля не прошли валидацию)
- Соответствие RFC 7807 (Problem Details for HTTP APIs) делает API стандартизированным

---

### 8. Использование ISO 8601 для дат и времени (UTC)

**Решение:** Все даты и время передаются в формате ISO 8601 с временной зоной UTC:
```
"createdAt": "2026-02-02T16:30:00Z"
```

**Обоснование:**
- ISO 8601 — международный стандарт, поддерживается всеми языками и библиотеками
- UTC исключает проблемы с временными зонами (клиент конвертирует в локальное время)
- Формат с `Z` явно указывает на UTC (избегаем неоднозначности)
- Критично для международной экспансии — разные страны, разные временные зоны

---

## Документация по API

### Endpoints

### 1. GET /api/v1/orders — Получить список заказов пользователя

**Описание:** Возвращает список заказов текущего аутентифицированного пользователя.

**Метод:** `GET`

**Аутентификация:** Обязательна (JWT Bearer Token)

**Заголовки запроса:**
```
GET /api/v1/orders
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

**Пример успешного ответа (200 OK):**
```json
[
  {
    "orderId": "550e8400-e29b-41d4-a716-446655440000",
    "customerId": "customer-123",
    "storeId": "store-456",
    "items": [
      {
        "menuItemId": "item-001",
        "name": "Сэндвич BLT",
        "quantity": 2,
        "price": 350.00,
        "subtotal": 700.00
      },
      {
        "menuItemId": "item-015",
        "name": "Кола 0.5л",
        "quantity": 1,
        "price": 120.00,
        "subtotal": 120.00
      }
    ],
    "totalAmount": 820.00,
    "status": "pending",
    "deliveryType": "pickup",
    "deliveryAddress": null,
    "createdAt": "2026-02-02T14:30:00Z",
    "estimatedReadyAt": "2026-02-02T14:50:00Z",
    "completedAt": null
  }
]
```

**Возможные коды ответа:**
- `200 OK` — список заказов успешно получен
- `401 Unauthorized` — отсутствует или невалидный токен

**Тестирование 200:**
![img.png](img.png)
![img_1.png](img_1.png)
![img_2.png](img_2.png)

---

### 2. GET /api/v1/orders/{id} — Получить конкретный заказ

**Описание:** Возвращает детальную информацию о конкретном заказе по ID.

**Метод:** `GET`

**Аутентификация:** Обязательна (JWT Bearer Token)

**Заголовки запроса:**
```
GET /api/v1/orders/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

**Пример успешного ответа (200 OK):**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "customerId": "customer-123",
  "storeId": "store-456",
  "storeName": "Subway на Ленина",
  "storeAddress": "ул. Ленина, 15",
  "items": [
    {
      "menuItemId": "item-001",
      "name": "Сэндвич BLT",
      "quantity": 2,
      "price": 350.00,
      "subtotal": 700.00
    }
  ],
  "totalAmount": 700.00,
  "status": "preparing",
  "deliveryType": "pickup",
  "deliveryAddress": null,
  "paymentMethod": "card",
  "paymentStatus": "paid",
  "createdAt": "2026-02-02T14:30:00Z",
  "estimatedReadyAt": "2026-02-02T14:50:00Z",
  "completedAt": null,
  "statusHistory": [
    {
      "status": "pending",
      "timestamp": "2026-02-02T14:30:00Z"
    },
    {
      "status": "preparing",
      "timestamp": "2026-02-02T14:35:00Z"
    }
  ]
}
```

**Возможные коды ответа:**
- `200 OK` — заказ успешно получен
- `401 Unauthorized` — отсутствует или невалидный токен
- `403 Forbidden` — пользователь не имеет прав на просмотр этого заказа
- `404 Not Found` — заказ с указанным ID не существует

**Тестирование 200:**

![img_6.png](img_6.png)
![img_7.png](img_7.png)
![img_8.png](img_8.png)

---

### 3. POST /api/v1/orders — Создать новый заказ

**Описание:** Создает новый заказ для текущего пользователя.

**Метод:** `POST`

**Аутентификация:** Обязательна (JWT Bearer Token)

**Заголовки запроса:**
```
POST /api/v1/orders
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

**Тело запроса (Request Body):**
```json
{
  "storeId": "store-456",
  "items": [
    {
      "menuItemId": "item-001",
      "quantity": 2
    },
    {
      "menuItemId": "item-015",
      "quantity": 1
    }
  ],
  "deliveryType": "pickup",
  "deliveryAddress": null,
  "paymentMethod": "card"
}
```

**Пример успешного ответа (201 Created):**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "customerId": "customer-123",
  "storeId": "store-456",
  "items": [
    {
      "menuItemId": "item-001",
      "name": "Сэндвич BLT",
      "quantity": 2,
      "price": 350.00,
      "subtotal": 700.00
    },
    {
      "menuItemId": "item-015",
      "name": "Кола 0.5л",
      "quantity": 1,
      "price": 120.00,
      "subtotal": 120.00
    }
  ],
  "totalAmount": 820.00,
  "status": "pending",
  "deliveryType": "pickup",
  "deliveryAddress": null,
  "paymentMethod": "card",
  "paymentStatus": "pending",
  "createdAt": "2026-02-02T14:30:00Z",
  "estimatedReadyAt": "2026-02-02T14:50:00Z"
}
```

**Заголовки ответа:**
```
HTTP/1.1 201 Created
Location: /api/v1/orders/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
```

**Возможные коды ответа:**
- `201 Created` — заказ успешно создан
- `400 Bad Request` — ошибка валидации (пустой список items, невалидный storeId, и т.д.)
- `401 Unauthorized` — отсутствует или невалидный токен
- `404 Not Found` — один из menuItemId не существует в меню

**Пример ошибки валидации (400 Bad Request):**
```json
{
  "error": "ValidationError",
  "message": "Invalid order data",
  "details": {
    "items": "At least one item is required",
    "deliveryAddress": "Delivery address is required when deliveryType is 'delivery'"
  }
}
```

**Тестирование 200:**

![img_9.png](img_9.png)
![img_10.png](img_10.png)
![img_11.png](img_11.png)

---

### 4. PUT /api/v1/orders/{id} — Обновить заказ

**Описание:** Обновляет состав заказа (доступно только для заказов в статусе `pending`).

**Метод:** `PUT`

**Аутентификация:** Обязательна (JWT Bearer Token)

**Заголовки запроса:**
```
PUT /api/v1/orders/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

**Тело запроса (Request Body):**
```json
{
  "items": [
    {
      "menuItemId": "item-001",
      "quantity": 3
    },
    {
      "menuItemId": "item-020",
      "quantity": 1
    }
  ],
  "deliveryType": "delivery",
  "deliveryAddress": "ул. Пушкина, 10, кв. 5"
}
```

**Пример успешного ответа (200 OK):**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "customerId": "customer-123",
  "storeId": "store-456",
  "items": [
    {
      "menuItemId": "item-001",
      "name": "Сэндвич BLT",
      "quantity": 3,
      "price": 350.00,
      "subtotal": 1050.00
    },
    {
      "menuItemId": "item-020",
      "name": "Картофель фри",
      "quantity": 1,
      "price": 150.00,
      "subtotal": 150.00
    }
  ],
  "totalAmount": 1200.00,
  "status": "pending",
  "deliveryType": "delivery",
  "deliveryAddress": "ул. Пушкина, 10, кв. 5",
  "updatedAt": "2026-02-02T14:35:00Z"
}
```

**Возможные коды ответа:**
- `200 OK` — заказ успешно обновлен
- `400 Bad Request` — ошибка валидации
- `401 Unauthorized` — отсутствует или невалидный токен
- `403 Forbidden` — пользователь не владелец заказа
- `404 Not Found` — заказ не существует
- `409 Conflict` — заказ нельзя изменить (статус не `pending`)

**Пример ошибки конфликта (409 Conflict):**
```json
{
  "error": "OrderCannotBeModified",
  "message": "Cannot modify order in status 'preparing'",
  "details": {
    "orderId": "550e8400-e29b-41d4-a716-446655440000",
    "currentStatus": "preparing"
  }
}
```

**Тестирование 200:**

![img_12.png](img_12.png)
![img_13.png](img_13.png)
![img_14.png](img_14.png)

---

### 5. DELETE /api/v1/orders/{id} — Отменить заказ

**Описание:** Отменяет заказ (доступно только для заказов в статусах `pending` или `preparing`).

**Метод:** `DELETE`

**Аутентификация:** Обязательна (JWT Bearer Token)

**Заголовки запроса:**
```
DELETE /api/v1/orders/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Пример успешного ответа (204 No Content):**
```
HTTP/1.1 204 No Content
```

(Тело ответа отсутствует при успешной отмене)

**Возможные коды ответа:**
- `204 No Content` — заказ успешно отменен
- `401 Unauthorized` — отсутствует или невалидный токен
- `403 Forbidden` — пользователь не владелец заказа
- `404 Not Found` — заказ не существует
- `409 Conflict` — заказ нельзя отменить (статус `in_delivery`, `completed`)

**Пример ошибки конфликта (409 Conflict):**
```json
{
  "error": "OrderCannotBeCancelled",
  "message": "Cannot cancel order in status 'in_delivery'",
  "details": {
    "orderId": "550e8400-e29b-41d4-a716-446655440000",
    "currentStatus": "in_delivery"
  }
}
```

**Идемпотентность:**
Повторный вызов `DELETE` для уже отмененного заказа вернет `404 Not Found` (заказ не существует или уже отменен), что является ожидаемым поведением.


**Тестирование 204:**

![img_16.png](img_16.png)
![img_17.png](img_17.png)
![img_18.png](img_18.png)

---

### 6. GET /api/v1/menu — Получить меню

**Описание:** Возвращает список всех доступных позиций меню.

**Метод:** `GET`

**Аутентификация:** Не требуется (публичный endpoint)

**Заголовки запроса:**
```
GET /api/v1/menu?category=sandwich
Content-Type: application/json
```

**Пример успешного ответа (200 OK):**
```json
{
  "items": [
    {
      "menuItemId": "item-001",
      "name": "Сэндвич BLT",
      "description": "Бекон, салат, помидоры на итальянском хлебе",
      "category": "sandwich",
      "price": 350.00,
      "isAvailable": true,
      "imageUrl": "https://cdn.example.com/menu/blt.jpg"
    },
    {
      "menuItemId": "item-002",
      "name": "Сэндвич с курицей терияки",
      "description": "Куриная грудка терияки с овощами",
      "category": "sandwich",
      "price": 380.00,
      "isAvailable": true,
      "imageUrl": "https://cdn.example.com/menu/teriyaki.jpg"
    },
    {
      "menuItemId": "item-015",
      "name": "Кола 0.5л",
      "description": "Coca-Cola 0.5 литра",
      "category": "drink",
      "price": 120.00,
      "isAvailable": true,
      "imageUrl": "https://cdn.example.com/menu/cola.jpg"
    }
  ]
}
```

**Возможные коды ответа:**
- `200 OK` — меню успешно получено
- `400 Bad Request` — невалидные параметры фильтрации


**Тестирование 200:**
![img_20.png](img_20.png)
![img_21.png](img_21.png)
![img_22.png](img_22.png)