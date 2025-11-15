# Summary: Section 8 - Service Discovery & Service Registration

## Обзор секции

В восьмой секции курса изучались механизмы Service Discovery и Service Registration в микросервисной архитектуре. Был реализован Eureka Server для регистрации и обнаружения микросервисов, а также интеграция микросервисов как Eureka Clients.

## Что было изучено

### 1. Проблемы без Service Discovery
- Хардкод URL микросервисов
- Проблемы при масштабировании
- Необходимость обновления конфигураций при изменении адресов
- Отсутствие автоматического обнаружения новых инстансов

### 2. Service Discovery паттерн
- Автоматическое обнаружение доступных сервисов
- Динамическая маршрутизация запросов
- Балансировка нагрузки между инстансами
- Отказоустойчивость при сбоях сервисов

### 3. Netflix Eureka
- Архитектура Eureka:
  - Eureka Server - сервер регистрации
  - Eureka Client - клиенты, регистрирующие себя
- Регистрация сервисов при старте
- Периодическое обновление heartbeat
- Автоматическое удаление недоступных сервисов

### 4. Настройка Eureka Server
- Зависимость `spring-cloud-starter-netflix-eureka-server`
- Аннотация `@EnableEurekaServer`
- Конфигурация через `application.yml`:
  - Порт сервера (обычно 8070)
  - Настройки репликации (для HA)
  - Отключение self-registration

### 5. Настройка Eureka Client
- Зависимость `spring-cloud-starter-netflix-eureka-client`
- Автоматическая регистрация при старте
- Конфигурация:
  - URL Eureka Server
  - Имя приложения
  - Настройки heartbeat
  - Prefer IP address для контейнеров

### 6. Интеграция с OpenFeign
- Использование имен сервисов вместо URL
- Автоматическая балансировка нагрузки
- Ретри и circuit breaker через Feign
- Упрощение межсервисной коммуникации

### 7. Агрегация данных из нескольких сервисов
- Создание агрегирующих endpoints
- Использование Feign Clients для вызова других сервисов
- Пример: `CustomerController` в Accounts сервисе
- Получение данных из Accounts, Cards и Loans сервисов

### 8. Eureka Dashboard
- Web интерфейс для мониторинга зарегистрированных сервисов
- Просмотр статуса инстансов
- Информация о метаданных сервисов
- Мониторинг здоровья сервисов

## Что было реализовано

### Eureka Server
- **Порт**: 8070
- **Функциональность**:
  - Регистрация всех микросервисов
  - Хранение информации о доступных инстансах
  - Предоставление REST API для discovery
  - Web Dashboard для мониторинга

### Eureka Clients
- **Accounts**: регистрация как `ACCOUNTS`
- **Cards**: регистрация как `CARDS`
- **Loans**: регистрация как `LOANS`
- **Gateway Server**: регистрация для маршрутизации
- **Config Server**: регистрация для централизованных конфигураций

### Feign Clients
- `CardsFeignClient` - для вызова Cards сервиса
- `LoansFeignClient` - для вызова Loans сервиса
- Использование имен сервисов из Eureka
- Автоматическая балансировка нагрузки

### Агрегирующие endpoints
- `GET /api/customer` - получение полной информации о клиенте
- Агрегация данных из Accounts, Cards и Loans
- Единый ответ с данными из всех сервисов

## Технические детали

### Зависимости
- `spring-cloud-starter-netflix-eureka-server` - для сервера
- `spring-cloud-starter-netflix-eureka-client` - для клиентов
- `spring-cloud-starter-openfeign` - для межсервисных вызовов

### Конфигурация Eureka
```yaml
eureka:
  instance:
    preferIpAddress: true
  client:
    fetchRegistry: true
    registerWithEureka: true
    serviceUrl:
      defaultZone: http://localhost:8070/eureka/
```

### Feign Client пример
```java
@FeignClient("CARDS")
public interface CardsFeignClient {
    @GetMapping("/api/fetch")
    ResponseEntity<CardsDto> fetchCardDetails(@RequestParam String mobileNumber);
}
```

## Best Practices

1. **Использование имен сервисов** - вместо хардкода URL
2. **Health Checks** - интеграция с Actuator health endpoints
3. **Метаданные** - добавление полезной информации о сервисах
4. **Зоны доступности** - для географического распределения
5. **Self-preservation** - защита от временных сетевых проблем
6. **Мониторинг** - регулярная проверка состояния сервисов

## Результаты секции

После завершения восьмой секции студенты:
1. Понимают важность Service Discovery в микросервисах
2. Умеют настраивать Eureka Server
3. Знают, как регистрировать микросервисы как Eureka Clients
4. Умеют использовать OpenFeign для межсервисных вызовов
5. Понимают, как агрегировать данные из нескольких сервисов
6. Знают best practices для Service Discovery

## Следующие шаги

В следующих секциях курса будут изучены:
- API Gateway, маршрутизация и cross-cutting concerns (Section 9)
- Устойчивость микросервисов (Section 10)
- Мониторинг и observability (Section 11)
- Безопасность микросервисов (Section 12)

