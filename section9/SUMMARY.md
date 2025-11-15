# Summary: Section 9 - Gateway, Routing & Cross Cutting Concerns

## Обзор секции

В девятой секции курса изучались API Gateway, маршрутизация запросов и cross-cutting concerns (сквозные задачи) в микросервисной архитектуре. Был реализован Spring Cloud Gateway для централизованной маршрутизации и обработки общих задач.

## Что было изучено

### 1. Проблемы без API Gateway
- Прямое обращение клиентов к каждому микросервису
- Дублирование логики аутентификации/авторизации
- Отсутствие централизованной маршрутизации
- Сложность управления CORS, rate limiting и т.д.

### 2. API Gateway паттерн
- Единая точка входа для всех клиентов
- Централизованная маршрутизация
- Обработка cross-cutting concerns
- Упрощение клиентских приложений

### 3. Spring Cloud Gateway
- Reactive gateway на основе Spring WebFlux
- Интеграция с Eureka для service discovery
- Динамическая маршрутизация
- Высокая производительность

### 4. Настройка Gateway
- Зависимость `spring-cloud-starter-gateway`
- Конфигурация маршрутов через Java DSL или YAML
- Интеграция с Eureka через `lb://` префикс
- Настройка фильтров

### 5. Маршрутизация запросов
- Определение путей для каждого микросервиса
- Rewrite path для изменения URL
- Load balancing через Eureka
- Примеры маршрутов:
  - `/eazybank/accounts/**` → `lb://ACCOUNTS`
  - `/eazybank/cards/**` → `lb://CARDS`
  - `/eazybank/loans/**` → `lb://LOANS`

### 6. Gateway Filters
- **Rewrite Path Filter** - изменение пути запроса
- **Add Response Header** - добавление заголовков
- **Request/Response Tracing** - отслеживание запросов
- Кастомные фильтры для специфичной логики

### 7. Request/Response Tracing
- Генерация уникальных trace ID
- Добавление trace ID в заголовки
- Логирование для отслеживания запросов
- Интеграция с системами мониторинга

### 8. Cross-Cutting Concerns
- Централизованная обработка:
  - Аутентификация и авторизация
  - Rate limiting
  - CORS
  - Логирование
  - Мониторинг
- Упрощение микросервисов

## Что было реализовано

### Spring Cloud Gateway Server
- **Порт**: 8072
- **Маршруты**:
  - Accounts: `/eazybank/accounts/**` → `lb://ACCOUNTS`
  - Cards: `/eazybank/cards/**` → `lb://CARDS`
  - Loans: `/eazybank/loans/**` → `lb://LOANS`
- **Фильтры**:
  - Rewrite path для удаления префикса
  - Добавление response headers
  - Tracing фильтры

### Tracing Filters
- `RequestTraceFilter` - добавление trace ID к входящим запросам
- `ResponseTraceFilter` - добавление trace ID к ответам
- `FilterUtility` - утилиты для работы с заголовками

### Конфигурация через Java DSL
```java
@Bean
public RouteLocator eazyBankRouteConfig(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p.path("/eazybank/accounts/**")
            .filters(f -> f.rewritePath("/eazybank/accounts/(?<segment>.*)", "/${segment}")
                .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
            .uri("lb://ACCOUNTS"))
        .build();
}
```

## Технические детали

### Зависимости
- `spring-cloud-starter-gateway` - основной gateway
- `spring-cloud-starter-netflix-eureka-client` - для service discovery
- `spring-boot-starter-actuator` - для мониторинга

### Gateway Endpoints
- `/actuator/gateway/routes` - список всех маршрутов
- `/actuator/gateway/refresh` - обновление маршрутов
- Health check endpoints

### Интеграция с Eureka
- Использование `lb://` для load balancing
- Автоматическое обнаружение инстансов
- Балансировка нагрузки между репликами

## Best Practices

1. **Единая точка входа** - все запросы через Gateway
2. **Tracing** - добавление trace ID для отслеживания
3. **Фильтры** - централизованная обработка общих задач
4. **Мониторинг** - отслеживание производительности Gateway
5. **Безопасность** - централизованная аутентификация
6. **Версионирование API** - поддержка разных версий через маршруты

## Результаты секции

После завершения девятой секции студенты:
1. Понимают роль API Gateway в микросервисах
2. Умеют настраивать Spring Cloud Gateway
3. Знают, как создавать маршруты и фильтры
4. Умеют реализовывать request/response tracing
5. Понимают концепцию cross-cutting concerns
6. Знают best practices для API Gateway

## Следующие шаги

В следующих секциях курса будут изучены:
- Устойчивость микросервисов (Section 10)
- Мониторинг и observability (Section 11)
- Безопасность микросервисов (Section 12)
- Event-driven архитектура (Sections 13-14)

