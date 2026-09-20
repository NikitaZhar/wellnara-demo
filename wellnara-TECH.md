# Wellnara — Technical Overview

> Engineering companion to the product overview. This document is for technical
> reviewers (CTOs, tech leads, platform recruiters) and describes the architecture,
> stack, and implementation behind the application. For the product/feature
> description, see the client-facing overview.

**Status:** a complete, self-contained web application — full request-to-database
stack with authentication, server-rendered UI, persistence, migrations, and
transactional email.

---

## What it is (one line)

A private-practice management platform: specialists set up services, invite clients,
manage bookings against a per-client wallet, and sync sessions to their calendar —
"services, booking, and payment in one calm space."

## Architecture

A classic, well-layered Spring Boot MVC monolith with server-side rendering — the
right shape for a focused product: one deployable unit, one database, no
inter-service complexity to operate.

```
  Browser
     │  HTTP (forms, pages)
     ▼
  ┌────────────────────────────────────────────┐
  │            Spring Boot (MVC)                │
  │  Controllers ─▶ Services ─▶ Repositories    │
  │       │            │             │          │
  │  Thymeleaf     Spring        Spring Data     │
  │  templates     Security        JPA           │
  │                   │             │            │
  │              auth / roles   Flyway migrations │
  │                   │             │            │
  │              Spring Mail   ┌──────────┐      │
  │              (email)       │PostgreSQL│      │
  └───────────────────────────┴──────────┴──────┘
```

Request flow: Thymeleaf-rendered pages and forms hit Spring MVC controllers, which
delegate to a service layer holding the booking/wallet logic, which persists through
Spring Data JPA repositories. Spring Security guards routes and roles
(specialist vs. client); Spring Mail sends confirmations, invitations, and password
recovery; Flyway keeps the schema versioned.

## Tech stack

| Area           | Technology                                              |
|----------------|---------------------------------------------------------|
| Language       | Java 17                                                  |
| Framework      | Spring Boot 3.3.2                                        |
| Web / UI       | Spring MVC + Thymeleaf (server-side rendering)          |
| Security       | Spring Security (authentication, role-based access)     |
| Persistence    | Spring Data JPA + PostgreSQL                             |
| Migrations     | Flyway (incl. PostgreSQL module)                        |
| Email          | Spring Mail (Spring Boot mail starter)                  |
| Validation     | Jakarta Bean Validation                                 |
| Build          | Maven                                                    |
| Testing        | JUnit 5, Mockito, Spring Security Test, H2 (test scope) |
| Dev            | Spring Boot DevTools                                     |

## Notable capabilities (and how they're built)

- **Authentication & two user roles** — specialist and client dashboards, gated by
  Spring Security.
- **Booking against a per-client wallet** — external payment is recorded as balance;
  booking a session deducts its cost on specialist confirmation. Money movement is
  handled inside the service/transaction layer.
- **Calendar integration** — sessions export to Google, Outlook, and Apple via ICS.
- **Transactional email** — confirmations, cancellations, time-limited invitation
  links, and password recovery via Spring Mail.
- **Internationalization** — bilingual (RU/EN) interface with time-zone-aware display.
- **Validated forms** — Bean Validation on inbound data before it reaches the domain.

## Data & migrations

PostgreSQL with Flyway-managed schema: every change ships as a versioned migration,
so local, test, and production schemas converge deterministically. Tests run against
H2 for speed in the unit/slice layer.

## Testing

JUnit 5 with Mockito for service-layer logic and Spring Security Test for
authentication/authorization paths; `@DataJpaTest`-style slices exercise the
persistence layer.

## Running locally

```bash
# Prerequisites: JDK 17, PostgreSQL (local or Docker)

# configure DB + mail in application.yml / env vars
#   spring.datasource.*, spring.mail.*

./mvnw clean verify        # build + tests
./mvnw spring-boot:run     # start the app  ->  http://localhost:8080
```

<!-- TODO (Nik): add a docker-compose for PostgreSQL, a live demo URL with a
     read-only test login, and 2–3 screenshots of the specialist/client dashboards. -->

## What this project demonstrates

The ability to ship a real, complete product end to end: secure multi-role auth,
non-trivial domain logic (bookings + wallet), third-party integration (calendars,
email), i18n, database migrations, and a tested Spring Boot codebase — exactly the
shape of work that small and mid-size clients hire for.
