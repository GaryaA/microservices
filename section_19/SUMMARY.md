# Summary: Section 19 - Introduction to K8s Ingress, Service Mesh (Istio) & mTLS

## Обзор секции

В девятнадцатой секции курса изучались продвинутые сетевые концепции Kubernetes: Ingress для управления входящим трафиком, Service Mesh (Istio) для управления межсервисной коммуникацией и mTLS для обеспечения безопасности.

## Что было изучено

### 1. Kubernetes Ingress
- Управление входящим HTTP/HTTPS трафиком
- Единая точка входа для внешних запросов
- Маршрутизация на основе хоста и пути
- SSL/TLS termination

### 2. Ingress Controller
- Реализации Ingress контроллеров:
  - NGINX Ingress Controller
  - Traefik
  - Istio Gateway
- Установка и настройка
- Настройка правил маршрутизации

### 3. Ingress Rules
- Правила маршрутизации по хосту
- Правила маршрутизации по пути
- Backend services
- TLS сертификаты

### 4. Service Mesh концепция
- Управление межсервисной коммуникацией
- Observability на уровне сети
- Безопасность коммуникаций
- Управление трафиком

### 5. Istio
- Популярный Service Mesh
- Компоненты:
  - **Istiod** - control plane
  - **Envoy** - data plane (sidecar proxy)
  - **Pilot** - управление трафиком
  - **Citadel** - безопасность
  - **Galley** - конфигурация

### 6. Istio Features
- **Traffic Management** - маршрутизация, балансировка, retry
- **Security** - mTLS, авторизация
- **Observability** - метрики, логи, трассировка
- **Policy Enforcement** - rate limiting, quotas

### 7. mTLS (Mutual TLS)
- Двусторонняя аутентификация
- Шифрование трафика между сервисами
- Автоматическое управление сертификатами
- Защита от man-in-the-middle атак

### 8. Istio Configuration
- VirtualService - правила маршрутизации
- DestinationRule - политики для destinations
- Gateway - управление входящим трафиком
- PeerAuthentication - настройка mTLS

## Что было реализовано

### Ingress Configuration
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: eazybank-ingress
spec:
  rules:
  - host: api.eazybank.com
    http:
      paths:
      - path: /accounts
        pathType: Prefix
        backend:
          service:
            name: accounts
            port:
              number: 8080
```

### Istio Installation
- Установка Istio в кластер
- Настройка автоматической инъекции sidecar
- Конфигурация для микросервисов

### mTLS Configuration
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
```

### VirtualService Example
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: accounts
spec:
  hosts:
  - accounts
  http:
  - route:
    - destination:
        host: accounts
```

## Технические детали

### Ingress Controller
- NGINX как популярный выбор
- Установка через Helm или YAML
- Настройка для production

### Istio Components
- **Envoy Proxy** - sidecar в каждом поде
- **Istiod** - централизованное управление
- **Pilot** - service discovery и load balancing
- **Citadel** - управление сертификатами

### mTLS Modes
- **STRICT** - обязательный mTLS
- **PERMISSIVE** - поддержка plain text и mTLS
- **DISABLE** - отключение mTLS

## Best Practices

1. **Ingress для внешнего трафика** - единая точка входа
2. **Service Mesh для внутреннего трафика** - управление межсервисной коммуникацией
3. **mTLS везде** - шифрование всего трафика
4. **Gradual adoption** - постепенное внедрение Istio
5. **Monitoring** - использование observability возможностей
6. **Security policies** - настройка авторизации

## Результаты секции

После завершения девятнадцатой секции студенты:
1. Понимают Kubernetes Ingress
2. Знают концепцию Service Mesh
3. Умеют настраивать Istio
4. Понимают mTLS и его важность
5. Знают, как управлять трафиком через Istio
6. Знают best practices для сетевой безопасности

## Следующие шаги

В следующей секции курса:
- Заключение и итоги курса (Section 20)

