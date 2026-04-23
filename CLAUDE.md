# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Requirements

- Java 17
- Maven 3.x or Gradle 8.x

## Commands

```bash
# Build
./mvnw clean package

# Run
./mvnw spring-boot:run

# Run tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=OwnerControllerTests

# Run with MySQL profile
./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql

# Compile SCSS to CSS (needed after Bootstrap changes)
./mvnw package -P css

# Build container image (requires Docker)
./mvnw spring-boot:build-image
```

Gradle equivalents use `./gradlew bootRun`, `./gradlew test`, etc.

## Architecture

The app is a Spring MVC + Thymeleaf CRUD application with no explicit service layer — controllers call Spring Data JPA repositories directly.

**Package layout** under `org.springframework.samples.petclinic`:

- `model/` — shared base classes (`BaseEntity`, `Person`, `NamedEntity`) extended by domain classes
- `owner/` — all Owner/Pet/Visit domain classes, controllers, and repositories in one package
- `vet/` — Vet/Specialty domain, controller (serves both HTML and JSON via `@ResponseBody`), and repository
- `system/` — infrastructure: cache config, web config, crash/welcome controllers

**Data access**: Spring Data JPA repositories with query-method naming conventions. No service layer exists — business logic lives in controllers and domain objects.

**Database profiles**: H2 in-memory by default; activate `mysql` or `postgres` Spring profiles for those databases. SQL init scripts live in `src/main/resources/db/{h2,mysql,postgres}/`.

**Caching**: JCache API backed by Caffeine; the vet list is the only cached data (`CacheConfiguration`).

**Validation**: Jakarta Validation on domain objects (`@NotBlank`, `@Pattern`), plus a custom `PetValidator` for pet-specific rules.

**Testing approach**: Controllers are tested with `@WebMvcTest` + Mockito (`@MockitoBean`). Integration tests use `@SpringBootTest` with H2 (default), Testcontainers MySQL, or Docker Compose PostgreSQL. Test classes mirror the main package structure under `src/test/`.

## Code Style

Checkstyle and Spring Java Format are enforced at build time. Run `./mvnw validate` to check formatting without a full build. The project bans plain `http://` URLs (enforced by NoHTTP Checkstyle rule).
