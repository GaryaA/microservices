# Summary: Section 10 - Making Microservices Resilient

## Обзор секции

В десятой секции курса изучались паттерны и механизмы для обеспечения устойчивости (resilience) микросервисов. Были реализованы Circuit Breaker, Retry, Rate Limiter и другие паттерны для обработки сбоев и обеспечения отказоустойчивости системы.

## Что было изучено

### 1. Проблемы без устойчивости
- Каскадные сбои при падении одного сервиса
- Отсутствие обработки временных сбоев
- Перегрузка сервисов из-за большого количества запросов
- Долгое ожидание ответа от недоступных сервисов

### 2. Resilience4j библиотека
- Замена Netflix Hystrix
- Легковесная библиотека для Java
- Интеграция с Spring Boot
- Поддержка различных паттернов устойчивости

### 3. Circuit Breaker паттерн
- Защита от каскадных сбоев
- Три состояния:
  - **Closed** - нормальная работа
  - **Open** - сервис недоступен, запросы блокируются
  - **Half-Open** - тестирование восстановления
- Настройки:
  - `slidingWindowSize` - размер окна для анализа
  - `failureRateThreshold` - порог ошибок
  - `waitDurationInOpenState` - время ожидания перед переходом в Half-Open

### 4. Retry паттерн
- Автоматические повторные попытки при временных сбоях
- Настройки:
  - `maxAttempts` - максимальное количество попыток
  - `waitDuration` - задержка между попытками
  - `exponentialBackoff` - экспоненциальная задержка
  - `retryExceptions` - исключения для retry

### 5. Rate Limiter паттерн
- Ограничение количества запросов
- Защита от перегрузки сервисов
- Настройки:
  - `limitForPeriod` - лимит запросов за период
  - `limitRefreshPeriod` - период обновления
  - `timeoutDuration` - таймаут ожидания

### 6. Time Limiter
- Ограничение времени выполнения операций
- Предотвращение долгого ожидания
- Настройка таймаутов для операций

### 7. Bulkhead паттерн
- Изоляция ресурсов
- Предотвращение влияния сбоев одного сервиса на другие
- Разделение thread pools

### 8. Интеграция с Spring Cloud Gateway
- Circuit Breaker в маршрутах Gateway
- Retry для определенных маршрутов
- Rate Limiter с использованием Redis
- Fallback URI для обработки сбоев

### 9. Интеграция с OpenFeign
- Circuit Breaker в Feign Clients
- Автоматическая обработка сбоев
- Fallback методы для обработки ошибок

## Что было реализовано

### Circuit Breaker в Gateway
- Настройка для маршрута Accounts
- Fallback URI при сбое: `/contactSupport`
- Конфигурация через Resilience4J

### Retry в Gateway
- Настройка для маршрута Loans
- 3 попытки с экспоненциальной задержкой
- Только для GET запросов

### Rate Limiter в Gateway
- Настройка для маршрута Cards
- Интеграция с Redis
- Лимит: 1 запрос в секунду

### Resilience4j конфигурация
```yaml
resilience4j.circuitbreaker:
  configs:
    default:
      slidingWindowSize: 10
      failureRateThreshold: 50
      waitDurationInOpenState: 10000

resilience4j.retry:
  configs:
    default:
      maxAttempts: 3
      waitDuration: 500
      enableExponentialBackoff: true
```

### Gateway Route с Circuit Breaker
```java
.route(p -> p.path("/eazybank/accounts/**")
    .filters(f -> f.circuitBreaker(config -> 
        config.setName("accountsCircuitBreaker")
            .setFallbackUri("forward:/contactSupport")))
    .uri("lb://ACCOUNTS"))
```

## Технические детали

### Зависимости
- `spring-cloud-starter-circuitbreaker-reactor-resilience4j` - для Gateway
- `spring-cloud-starter-circuitbreaker-resilience4j` - для Feign
- `spring-boot-starter-data-redis-reactive` - для Rate Limiter

### Мониторинг
- Actuator endpoints для Circuit Breaker
- Метрики через Micrometer
- Интеграция с Prometheus

### Конфигурация
- Глобальные настройки в `application.yml`
- Специфичные настройки для каждого сервиса
- Программная конфигурация через Java Beans

## Best Practices

1. **Настройка порогов** - баланс между защитой и доступностью
2. **Мониторинг** - отслеживание состояния Circuit Breakers
3. **Fallback стратегии** - обработка сбоев с полезными ответами
4. **Тестирование** - проверка поведения при сбоях
5. **Документация** - документирование стратегий устойчивости
6. **Постепенное внедрение** - применение паттернов постепенно

## Результаты секции

После завершения десятой секции студенты:
1. Понимают важность устойчивости в микросервисах
2. Знают различные паттерны устойчивости (Circuit Breaker, Retry, Rate Limiter)
3. Умеют настраивать Resilience4j в Spring Boot
4. Знают, как интегрировать паттерны в Gateway и Feign
5. Понимают, как настраивать fallback стратегии
6. Знают best practices для обеспечения устойчивости

## Следующие шаги

В следующих секциях курса будут изучены:
- Мониторинг и observability (Section 11)
- Безопасность микросервисов (Section 12)
- Event-driven архитектура с RabbitMQ (Section 13)
- Event-driven архитектура с Kafka (Section 14)

