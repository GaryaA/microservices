# Summary: Section 12 - Microservices Security

## Обзор секции

В двенадцатой секции курса изучалась безопасность микросервисов. Были реализованы аутентификация и авторизация с использованием Keycloak, OAuth2, JWT токенов и Spring Security для защиты API.

## Что было изучено

### 1. Проблемы безопасности в микросервисах
- Множественные точки входа
- Управление сессиями в распределенной системе
- Валидация токенов в каждом сервисе
- Сложность централизованного управления доступом

### 2. OAuth2 и JWT
- OAuth2 как стандарт авторизации
- JWT (JSON Web Tokens) для передачи информации о пользователе
- Stateless аутентификация
- Микросервисы не хранят сессии

### 3. Keycloak
- Open Source Identity and Access Management
- Поддержка OAuth2, OpenID Connect, SAML
- Управление пользователями и ролями
- Выдача JWT токенов

### 4. Spring Security OAuth2 Resource Server
- Защита API endpoints
- Валидация JWT токенов
- Извлечение claims из токенов
- Настройка через `spring.security.oauth2.resourceserver.jwt.jwk-set-uri`

### 5. Роли и авторизация
- Определение ролей в Keycloak
- Маппинг ролей в Spring Security
- Защита endpoints на основе ролей
- `@PreAuthorize` и метод-level security

### 6. Gateway Security
- Централизованная аутентификация в Gateway
- Валидация токенов на уровне Gateway
- Передача информации о пользователе в микросервисы
- Фильтрация неавторизованных запросов

### 7. Keycloak Role Converter
- Извлечение ролей из JWT токенов
- Преобразование формата ролей Keycloak в Spring Security
- Маппинг `realm_access.roles` в `ROLE_*` authorities

### 8. Конфигурация безопасности
- Настройка JWK Set URI
- Публичные endpoints (GET запросы)
- Защищенные endpoints по ролям
- Отключение CSRF для stateless API

## Что было реализовано

### Keycloak Server
- **Порт**: 7080
- **Функциональность**:
  - Управление пользователями
  - Создание ролей (ACCOUNTS, CARDS, LOANS)
  - Выдача JWT токенов
  - OAuth2 endpoints

### Gateway Security Configuration
```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    http.authorizeExchange(exchanges -> exchanges
        .pathMatchers(HttpMethod.GET).permitAll()
        .pathMatchers("/eazybank/accounts/**").hasRole("ACCOUNTS")
        .pathMatchers("/eazybank/cards/**").hasRole("CARDS")
        .pathMatchers("/eazybank/loans/**").hasRole("LOANS"))
        .oauth2ResourceServer(oAuth2 -> oAuth2
            .jwt(jwt -> jwt.jwtAuthenticationConverter(grantedAuthoritiesExtractor())));
    return http.build();
}
```

### Keycloak Role Converter
- Извлечение ролей из `realm_access.roles`
- Преобразование в формат `ROLE_*`
- Интеграция с Spring Security

### Конфигурация
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: "http://localhost:7080/realms/master/protocol/openid-connect/certs"
```

## Технические детали

### Зависимости
- `spring-boot-starter-security` - базовая безопасность
- `spring-security-oauth2-resource-server` - OAuth2 Resource Server
- `spring-security-oauth2-jose` - JWT поддержка

### JWT Token Structure
- Header: алгоритм подписи
- Payload: claims (роли, пользователь, срок действия)
- Signature: подпись для валидации

### Роли в Keycloak
- Realm roles: ACCOUNTS, CARDS, LOANS
- Client roles (опционально)
- Маппинг в Spring Security authorities

## Best Practices

1. **Централизованная аутентификация** - Gateway как единая точка входа
2. **Stateless токены** - использование JWT вместо сессий
3. **Валидация токенов** - проверка подписи и срока действия
4. **Принцип наименьших привилегий** - минимальные необходимые роли
5. **HTTPS** - шифрование трафика в production
6. **Token refresh** - обновление токенов для долгих сессий
7. **Логирование безопасности** - отслеживание попыток доступа

## Результаты секции

После завершения двенадцатой секции студенты:
1. Понимают важность безопасности в микросервисах
2. Знают OAuth2 и JWT основы
3. Умеют настраивать Keycloak
4. Знают, как защищать API с Spring Security
5. Умеют реализовывать role-based access control
6. Понимают, как интегрировать безопасность в Gateway
7. Знают best practices для безопасности микросервисов

## Следующие шаги

В следующих секциях курса будут изучены:
- Event-driven архитектура с RabbitMQ (Section 13)
- Event-driven архитектура с Kafka (Section 14)
- Kubernetes оркестрация (Section 15)
- Helm для управления Kubernetes (Section 16)

