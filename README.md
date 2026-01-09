# 🎓 Spring Boot Microservices Learning Platform

> **Learn by example:** Production-ready microservices with Spring Boot 3.x, Java 21, and Domain-Driven Design. Master modern architecture patterns through real-world code.

## 🚀 Quick Start

```bash
# Clone and start all services
docker-compose up

# Access services
- Gateway: http://localhost:8081
- User Service: http://localhost:8080/swagger-ui.html
- Bookstore Service: http://localhost:8082/swagger-ui.html
- Grafana: http://localhost:3000
```

## What You'll Master

**Architecture Patterns:**

- Microservices decomposition with clear boundaries
- Domain-Driven Design (value objects, aggregates, bounded contexts)
- API Gateway with reactive programming
- Database per service with multi-tenancy

**Modern Java & Spring:**

- Java 21 features (records, pattern matching, text blocks)
- Spring Boot 3.x with WebFlux reactive programming
- Type-safe configuration and validation

**Production Readiness:**

- JWT authentication with RSA256
- Distributed rate limiting and circuit breakers
- Observability (metrics, tracing, structured logging)
- Multi-tenant data isolation

## Architecture Overview

```
┌─────────────┐
│   Clients   │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────┐
│  🌐 Gateway (Port 8081)             │
│  • JWT Validation                   │
│  • Rate Limiting (Redis)            │
│  • Circuit Breaker                  │
│  • Request Routing                  │
└──────┬──────────────────┬───────────┘
       │                  │
       ▼                  ▼
┌──────────────┐   ┌──────────────────┐
│ 🔐 User      │   │ 📚 Bookstore     │
│ Service      │   │ Service          │
│ (Port 8080)  │   │ (Port 8082)      │
│              │   │                  │
│ • Auth/JWT   │   │ • DDD Patterns   │
│ • Multi-     │   │ • Value Objects  │
│   Tenancy    │   │ • Aggregates     │
└──────┬───────┘   └──────┬───────────┘
       │                  │
       ▼                  ▼
  PostgreSQL         PostgreSQL
  (Schema-per-       (Books &
   Tenant)            Inventory)
```

**Key Decisions:**

1. **Database Per Service** - Independent data ownership and scaling
2. **Reactive Gateway** - Non-blocking I/O for high throughput
3. **Schema-Per-Tenant** - Strong data isolation for multi-tenancy
4. **JWT Context Propagation** - Stateless authentication across services

## Platform Services

### 📚 Bookstore Service - Domain-Driven Design

**Learn:** Value objects, aggregates, domain services, bounded contexts

**DDD Patterns:**

```java
// Value Objects - Self-validating, immutable
var isbn = ISBN.of("978-0-13-468599-1");
var price = Money.usd(new BigDecimal("29.99"));

// Aggregate Root - Business logic in domain
var book = Book.create(title, author, isbn, price, description, category);
boolean canSell = book.canBeSold(); // Combines availability + stock

// Domain Service - Cross-aggregate operations
duplicateIsbnChecker.ensureUnique(isbn);
```

**Bounded Contexts:**

- `catalog` - Book management
- `inventory` - Stock tracking and reservations
- `shared` - Common value objects (ISBN, Money, BookId)

[Full Documentation →](iqscaffold-bookstore-service/README.md)

### 🔐 User Service - Authentication & Multi-Tenancy

**Learn:** JWT lifecycle, token blacklisting, schema-per-tenant isolation

**Security Patterns:**

- JWT with RSA256 (access: 15min, refresh: 7 days)
- JTI-based blacklisting with Redis TTL
- Account lockout (5 attempts → 15min)
- Email verification with rate limiting (3/hour)
- Schema-per-tenant data isolation

**Multi-Tenancy:**

```java
// Each tenant gets separate PostgreSQL schema
TenantContext.executeInTenantContext(tenantId, () ->
  userRepository.countByEnabledTrue()
);
```

[Full Documentation →](iqscaffold-user-service/README.md)

### 🌐 Gateway Service - Reactive API Gateway

**Learn:** Reactive programming, distributed rate limiting, circuit breakers

**Reactive Patterns:**

```java
// Non-blocking filter chain
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
  return validateToken(exchange)
    .flatMap((userContext) -> enrichHeaders(exchange, userContext))
    .then(chain.filter(exchange));
}
```

**Features:**

- Sliding window rate limiting (Redis ZSET)
- Circuit breaker per service (Resilience4j)
- JWT validation with `ReactiveSecurityContextHolder`
- Correlation ID generation and propagation

[Full Documentation →](iqscaffold-gateway-service/README.md)

## Technology Stack

| Layer             | Technology                                                |
| ----------------- | --------------------------------------------------------- |
| **Runtime**       | Java 21 (records, pattern matching, text blocks)          |
| **Framework**     | Spring Boot 3.5.6, Spring Cloud 2025.0.0                  |
| **Database**      | PostgreSQL 15+ with Liquibase migrations                  |
| **Caching**       | Redis (distributed cache, rate limiting, token blacklist) |
| **Security**      | JWT RSA256, Spring Security OAuth2 Resource Server        |
| **Resilience**    | Resilience4j (circuit breaker, rate limiter)              |
| **Observability** | OpenTelemetry, Prometheus, Grafana, Loki                  |
| **API Docs**      | SpringDoc OpenAPI 3 with Swagger UI                       |
| **Testing**       | JUnit 5, Testcontainers, ArchUnit, Reactor Test           |
| **Containers**    | Docker with multi-stage builds, Docker Compose            |

## Core Patterns Demonstrated

### Domain-Driven Design

**Value Objects:**

```java
@Embeddable
public class ISBN {

  private String value;

  public static ISBN of(String value) {
    // Validation logic
    return new ISBN(normalize(value));
  }
}
```

**Aggregate Roots:**

```java
@Entity
public class Book extends AggregateRoot<Long> {

  private ISBN isbn;
  private Money price;

  public boolean canBeSold() {
    return this.available && isInStock();
  }
}
```

### Microservices Architecture

**Database Per Service:**

- User Service → PostgreSQL (users, roles, tenants)
- Bookstore Service → PostgreSQL (books, inventory)
- No shared databases or cross-service joins

**JWT Context Propagation:**

```
Authorization: Bearer <jwt-token>
X-User-ID: 123
X-Username: john.doe
X-User-Roles: USER,ADMIN
X-Tenant-ID: tenant-123
X-Correlation-ID: abc-123
```

**Circuit Breaker:**

```yaml
resilience4j:
  circuitbreaker:
    instances:
      bookstore-service:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
```

### Security Patterns

**JWT Authentication:**

- RSA256 signing with JWK Set endpoint
- Token validation at gateway and services
- Comprehensive claims (userId, roles, permissions, tenantId)

**Token Blacklisting:**

```java
// Redis-backed logout with TTL
redisTemplate.opsForValue().set(
  "blacklist:" + jti,
  "true",
  Duration.ofMinutes(15)
);
```

**Role-Based Access Control:**

```java
@PreAuthorize("hasAnyRole('ADMIN', 'SUPER_ADMIN')")
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
  bookService.deleteBook(id);
  return ResponseEntity.noContent().build();
}
```

### Reactive Programming

**Non-Blocking I/O:**

```java
// WebFlux with Mono/Flux
public Mono<ServerResponse> handleRequest(ServerRequest request) {
  return userService
    .findById(id)
    .flatMap((user) -> ServerResponse.ok().bodyValue(user))
    .switchIfEmpty(ServerResponse.notFound().build());
}
```

**Reactive Rate Limiting:**

```java
// Sliding window with Redis ZSET
return redisTemplate.opsForZSet()
  .removeRangeByScore(key, 0, cutoffTime)
  .then(redisTemplate.opsForZSet().add(key, requestId, currentTime))
  .flatMap(added -> redisTemplate.opsForZSet().count(key, windowStart, currentTime))
  .map(count -> count <= limit);
```

## Learning Path

### 1. Start with Bookstore Service (DDD Fundamentals)

- Study value objects (`ISBN`, `Money`)
- Understand aggregates (`Book`, `Inventory`)
- Explore bounded contexts (`catalog`, `inventory`)
- Learn factory methods and domain services

**Why first:** DDD provides the foundation for maintainable domain logic.

### 2. Move to User Service (Security & Multi-Tenancy)

- Understand JWT generation and validation
- Study schema-per-tenant isolation
- Explore token blacklisting and account protection
- Learn audit logging patterns

**Why second:** Security is critical and builds on domain concepts.

### 3. Study Gateway Service (Reactive & Cross-Cutting Concerns)

- Understand reactive programming with WebFlux
- Study distributed rate limiting with Redis
- Explore circuit breaker patterns
- Learn request transformation and routing

**Why third:** Gateway ties everything together with advanced patterns.

### 4. Explore Integration (End-to-End Flows)

- Trace requests: client → gateway → services
- Understand correlation ID propagation
- Study JWT context flow across services
- Learn distributed tracing with OpenTelemetry

**Why last:** Integration shows how all patterns work together.

## Hands-On Exercises

**Exercise 1: Add a New Value Object**

- Create `Publisher` value object in Bookstore Service
- Add validation (name, country, founded year)
- Update `Book` aggregate to include publisher
- Write unit tests for validation logic

**Exercise 2: Implement a New Endpoint**

- Add "Get Books by Publisher" endpoint
- Implement repository query method
- Add Redis caching
- Write integration tests with Testcontainers

**Exercise 3: Add Custom JWT Claims**

- Extend User Service to include department in JWT
- Update Gateway to propagate department header
- Modify Bookstore Service to log department context
- Test end-to-end claim propagation

**Exercise 4: Create a New Microservice**

- Build Order Service following established patterns
- Implement order placement with book reservation
- Add JWT validation and user context extraction
- Register routes in Gateway Service

## Adapting for Your Domain

### Replace the Bookstore Domain

```java
// Current: Book aggregate
Book book = Book.create(title, author, isbn, price, description, category);

// Adapt to: Product catalog
Product product = Product.create(name, sku, price, description, category);

// Adapt to: Asset management
Asset asset = Asset.create(name, serialNumber, cost, location, department);
```

**Maintain these principles:**

- Value objects for domain concepts
- Aggregate roots with business logic
- Factory methods for creation
- Bounded contexts for organization

### Real-World Examples

**E-Commerce:**

- User Service → Customer authentication
- Bookstore → Product catalog
- Add: Order Service, Shipping Service

**Healthcare:**

- User Service → Provider/patient auth
- Bookstore → Medical records
- Add: Appointment Service, Billing Service

**Financial:**

- User Service → Customer identity/KYC
- Bookstore → Financial products
- Add: Transaction Service, Reporting Service

## API Documentation

Once services are running:

- **User Service:** http://localhost:8080/swagger-ui.html
- **Gateway Service:** http://localhost:8081/swagger-ui.html
- **Bookstore Service:** http://localhost:8082/swagger-ui.html

**Monitoring:**

- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000
- Health Checks: http://localhost:808x/actuator/health

## Resources

**Official Documentation:**

- [Spring Boot 3.x](https://spring.io/projects/spring-boot)
- [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)
- [Spring Security](https://spring.io/projects/spring-security)
- [Project Reactor](https://projectreactor.io/)

**Recommended Books:**

- "Domain-Driven Design" by Eric Evans
- "Building Microservices" by Sam Newman
- "Reactive Spring" by Josh Long
- "Spring Boot: Up and Running" by Mark Heckler

**Community:**

- [Spring Community](https://spring.io/community)
- [DDD Community](https://github.com/ddd-crew)
- [Microservices Patterns](https://microservices.io/patterns/)

---

**This platform is your blueprint** for building production-ready microservices. The code demonstrates real-world patterns you can learn from and adapt to your own projects.
