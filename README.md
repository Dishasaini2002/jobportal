[README.md](https://github.com/user-attachments/files/35286898/README.md)

**Project Overview**
JobPortal API is a production-oriented backend platform for job marketplaces, designed to handle the full lifecycle of recruitment: employer onboarding, company management, job posting, candidate applications, and resume management, backed by role-based access control and centralized observability.

The system is built for teams that need a reference implementation of enterprise Spring Boot architecture — not a tutorial project. It targets three audiences:

Backend engineers who need a well-structured, layered Spring Boot service to extend or study.
Platform teams evaluating patterns for authentication, caching, and monitoring in a Java microservice.
Recruiting and HR technology teams building or integrating job-marketplace functionality.
Architectural Goals
Separation of concerns across controller, service, and repository layers, with DTOs isolating the persistence model from the public API contract.
Stateless authentication using JWT, enabling horizontal scaling without sticky sessions.
Defense in depth, combining Spring Security filters, role-based authorization, password hashing, and input validation.
Operational visibility by default — every instance exposes health, metrics, and readiness endpoints suitable for orchestration platforms.
Predictable performance through Caffeine-backed caching for high-read, low-mutation data such as job listings and company profiles.
Container-first deployment, with a multi-stage Dockerfile producing a minimal, reproducible runtime image.
The result is a service that is straightforward to run locally, but structured the way it would need to be structured to survive a production rollout: instrumented, secured, documented, and testable.

Screenshots Preview
Swagger UI

docs/screenshots/swagger-ui.png

Grafana Dashboard
Prometheus Targets
Docker Desktop
Sample API Response
MySQL Database Schema
Architecture Diagram
Authentication Flow
Note

Screenshot assets are referenced from docs/screenshots/. Replace with actual captures before publishing.

Table of Contents
Features
Tech Stack
Architecture
Authentication Flow
Monitoring Flow
Project Structure
Modules
Database
Caching
Security
Monitoring
Docker
API Documentation
Installation Guide
Running the Project
Configuration
REST APIs
Sample Requests
Sample Responses
Monitoring URLs
Performance Optimizations
Exception Handling
Future Improvements
Contributing
License
Author
Features
Authentication
JWT-based stateless login and registration
Refresh-token issuance and rotation
Password reset workflow with time-limited tokens
Authorization
Role-based access control (ADMIN, EMPLOYER, JOB_SEEKER)
Method-level security via @PreAuthorize
Endpoint-level route protection through Spring Security filter chains
User Management
Profile creation, update, and soft deletion
Account activation and email verification hooks
Audit fields (createdAt, updatedAt, createdBy) on all core entities
Employer Features
Company registration and verification workflow
Job posting, editing, and archival
Applicant tracking per job posting
Job Seeker Features
Resume upload and versioning
Job search with filters (location, salary range, employment type)
Application submission and status tracking
Company Management
Company profile and branding metadata
Employer-to-company association
Company-level job listing aggregation
Caching
Caffeine in-memory cache for job listings and company profiles
Configurable TTL and maximum entry size
Cache eviction on write operations
Monitoring
Spring Boot Actuator health and info endpoints
Micrometer metrics exported to Prometheus
Pre-built Grafana dashboard definitions
Logging
Structured JSON logging via Logback
Correlation IDs propagated through MDC
Environment-specific log levels
API Documentation
OpenAPI 3.0 specification
Interactive Swagger UI
Grouped endpoints by domain
Docker
Multi-stage Dockerfile for minimal image size
Docker Compose stack for local development (API, MySQL, Prometheus, Grafana)
Validation
Bean Validation (jakarta.validation) on all request DTOs
Centralized validation error formatting
Security
BCrypt password hashing
CORS configuration
CSRF disabled for stateless API, mitigated via JWT + SameSite cookies where applicable
Error Handling
Global exception handler (@ControllerAdvice)
Consistent error response schema across all endpoints
Performance
Lazy loading and batch fetching for JPA associations
Pagination on all list endpoints
Connection pooling via HikariCP
Tech Stack
Technology	Purpose	Version
Java	Core language runtime	17 (LTS)
Spring Boot	Application framework	3.2.x
Spring Security	Authentication & authorization	6.2.x
JWT (jjwt)	Stateless token-based auth	0.12.x
Spring Data JPA	Data access abstraction	3.2.x
Hibernate	ORM implementation	6.4.x
MySQL	Relational data store	8.0.x
Swagger / springdoc-openapi	API documentation UI	2.4.x
OpenAPI	API specification standard	3.0
Spring Boot Actuator	Health checks & operational endpoints	3.2.x
Prometheus	Metrics collection & storage	2.51.x
Grafana	Metrics visualization	10.x
Micrometer	Metrics facade	1.12.x
Docker	Containerization	24.x
Maven	Build & dependency management	3.9.x
Lombok	Boilerplate reduction	1.18.x
Caffeine	In-memory caching	3.1.x
Jakarta Bean Validation	Request payload validation	3.0
Spring AOP	Cross-cutting concerns (logging, auditing)	6.1.x
🏗️ Architecture

🔐 Authentication Flow

📊 Monitoring Flow

Project Structure
jobportal-api/
├── src/
│   ├── main/
│   │   ├── java/com/jobportal/
│   │   │   ├── config/            # Security, cache, OpenAPI, and CORS configuration
│   │   │   ├── controller/        # REST controllers grouped by domain
│   │   │   ├── dto/               # Request/response payload objects
│   │   │   ├── entity/            # JPA entity definitions
│   │   │   ├── exception/         # Custom exceptions and global handler
│   │   │   ├── repository/        # Spring Data JPA repositories
│   │   │   ├── security/          # JWT provider, filters, and user details service
│   │   │   ├── service/           # Business logic layer
│   │   │   ├── util/              # Shared utility classes
│   │   │   └── JobPortalApplication.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/migration/      # Flyway versioned migrations
│   └── test/
│       └── java/com/jobportal/    # Unit and integration tests
├── docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── prometheus/prometheus.yml
│   └── grafana/dashboards/
├── pom.xml
└── README.md
Folder responsibilities:

Folder	Responsibility
config/	Wires security rules, cache managers, and OpenAPI metadata
controller/	Exposes REST endpoints; delegates to services, performs no business logic
dto/	Decouples the public API contract from persistence entities
entity/	Defines JPA-mapped domain models and relationships
exception/	Centralizes custom exceptions and the global @ControllerAdvice handler
repository/	Spring Data JPA interfaces for persistence access
security/	JWT issuance, validation, filters, and UserDetailsService implementation
service/	Encapsulates business rules, transactional boundaries, and cache interactions
util/	Shared helpers (mappers, constants, date utilities)
Modules
Authentication
Security
Jobs
Companies
Users
Resume
Contact
Monitoring
Caching
Configuration
Exception Handling
Utilities
Database
The schema follows third normal form (3NF) with explicit foreign key constraints and indexed lookup columns.

Core entities and relationships:

Entity	Relationship
User	One-to-one with Employer or JobSeeker profile
Company	One-to-many with Job
Job	Many-to-one with Company; one-to-many with Application
Application	Many-to-one with Job and JobSeeker
Resume	One-to-many with JobSeeker
Indexing strategy:

Composite index on (company_id, status) for job listing queries
Index on email (unique) for authentication lookups
Index on job_id, applicant_id for application uniqueness enforcement
Performance considerations:

Foreign keys are indexed by default to avoid full table scans on join queries
Soft deletes (deleted_at) avoid cascading destructive deletes on historical application data
Read-heavy tables (job, company) are the primary targets of the caching layer
Caching
The application uses Caffeine for in-memory caching of frequently read, infrequently mutated data such as job listings and company profiles.

Setting	Value	Rationale
TTL (job listings)	5 minutes	Balances freshness against read load
TTL (company profiles)	15 minutes	Company data changes infrequently
Maximum size	10,000 entries	Bounds memory footprint per instance
Eviction policy	Size + time-based (LRU)	Prevents unbounded growth
Caching reduces average job-search response time and offloads read pressure from MySQL during traffic spikes, at the cost of eventual consistency bounded by the configured TTL.

Security
JWT: Access tokens are short-lived (15 minutes); refresh tokens (7 days) are used to obtain new access tokens without re-authentication.
Role-Based Access Control: Enforced both at the filter-chain level (HttpSecurity matchers) and the method level (@PreAuthorize).
Password Encryption: BCrypt with a configurable strength factor.
CORS: Explicit allow-list of origins configured per environment.
CSRF: Disabled, as the API is stateless and does not rely on cookie-based session authentication.
Security Filters: Custom JwtAuthenticationFilter inserted before UsernamePasswordAuthenticationFilter in the chain.
Authentication Flow: See Authentication Flow.
Authorization Flow: Role claims embedded in the JWT are mapped to Spring Security GrantedAuthority objects on each request.
Monitoring
Spring Boot Actuator: Exposes /actuator/health, /actuator/info, and /actuator/metrics.
Micrometer: Bridges application metrics to the Prometheus exposition format.
Prometheus: Scrapes /actuator/prometheus on a configurable interval.
Grafana: Pre-built dashboards for JVM memory, HTTP request latency, and cache hit ratio.
Health Checks: Composite health indicators for database connectivity and disk space.
Metrics: Request counts, latency percentiles, error rates, and custom business metrics.
Dashboards: JSON dashboard definitions stored under docker/grafana/dashboards/ for provisioning.
Docker
Docker is used to guarantee a consistent runtime across local, staging, and production environments, and to simplify onboarding to a single docker compose up.

Benefits:

Reproducible builds via a multi-stage Dockerfile (build stage uses Maven, runtime stage uses a slim JRE)
Isolated local stack including MySQL, Prometheus, and Grafana
Immutable, versioned images suitable for CI/CD promotion across environments
# Build the image
docker build -t jobportal-api:latest -f docker/Dockerfile .

# Run the container
docker run -p 8080:8080 --env-file .env jobportal-api:latest
Dockerfile stages:

Stage	Base Image	Purpose
build	maven:3.9-eclipse-temurin-17	Compiles and packages the application
runtime	eclipse-temurin:17-jre-alpine	Runs the packaged JAR with a minimal footprint
API Documentation
Swagger UI is generated from the OpenAPI 3.0 specification via springdoc-openapi and is available once the application is running.

Resource	URL
Swagger UI	http://localhost:8080/swagger-ui/index.html
OpenAPI JSON	http://localhost:8080/v3/api-docs
Endpoints are grouped by domain tag (Authentication, Jobs, Companies, Users, Admin) for navigability within the Swagger interface.

Installation Guide
Prerequisites
Tool	Minimum Version
Java (JDK)	17
Maven	3.9
MySQL	8.0
Docker Desktop	24.x
Git	2.x
Steps
# 1. Clone the repository
git clone https://github.com/your-org/jobportal-api.git
cd jobportal-api

# 2. Install dependencies
mvn clean install

# 3. Configure the database
mysql -u root -p -e "CREATE DATABASE jobportal;"

# 4. Set required environment variables
export DB_URL=jdbc:mysql://localhost:3306/jobportal
export DB_USERNAME=root
export DB_PASSWORD=your_password
export JWT_SECRET=your_jwt_secret

# 5. Run the project
mvn spring-boot:run

# 6. Verify installation
curl http://localhost:8080/actuator/health
Running the Project
Local
mvn spring-boot:run
Docker
docker compose -f docker/docker-compose.yml up --build
Production
java -jar target/jobportal-api-1.0.0.jar --spring.profiles.active=prod
Important

Production deployments should source secrets (JWT signing key, database credentials) from a secrets manager rather than plain environment files.

Configuration
Configuration is layered across application.yml, application-dev.yml, and application-prod.yml, activated via the spring.profiles.active property.

Property	Description	Example
spring.datasource.url	Database connection URL	jdbc:mysql://localhost:3306/jobportal
spring.datasource.username	Database username	root
spring.datasource.password	Database password	********
jwt.secret	HMAC signing key for JWT	********
jwt.access-token-expiration	Access token TTL (ms)	900000
jwt.refresh-token-expiration	Refresh token TTL (ms)	604800000
server.port	Application HTTP port	8080
management.endpoints.web.exposure.include	Exposed Actuator endpoints	health,info,prometheus,metrics
All sensitive values are expected to be supplied via environment variables and are never committed to source control.

REST APIs
Authentication
Method	Endpoint	Description
POST	/api/v1/auth/register	Register a new user
POST	/api/v1/auth/login	Authenticate and receive tokens
POST	/api/v1/auth/refresh	Exchange a refresh token for a new access token
POST	/api/v1/auth/logout	Invalidate the current refresh token
Jobs
Method	Endpoint	Description
GET	/api/v1/jobs	List jobs (paginated, filterable)
GET	/api/v1/jobs/{id}	Retrieve a single job
POST	/api/v1/jobs	Create a job posting (employer only)
PUT	/api/v1/jobs/{id}	Update a job posting
DELETE	/api/v1/jobs/{id}	Archive a job posting
POST	/api/v1/jobs/{id}/apply	Submit an application
Companies
Method	Endpoint	Description
GET	/api/v1/companies	List companies
GET	/api/v1/companies/{id}	Retrieve company details
POST	/api/v1/companies	Register a company
PUT	/api/v1/companies/{id}	Update company details
Users
Method	Endpoint	Description
GET	/api/v1/users/me	Retrieve the current user's profile
PUT	/api/v1/users/me	Update the current user's profile
Admin
Method	Endpoint	Description
GET	/api/v1/admin/users	List all users
PUT	/api/v1/admin/users/{id}/status	Enable or disable a user account
Contacts
Method	Endpoint	Description
POST	/api/v1/contact	Submit a support inquiry
Health
Method	Endpoint	Description
GET	/actuator/health	Application health status
Sample Requests
Registration
Login
Create Company
Create Job
Search Job
Apply to Job
Sample Responses
{
  "id": 101,
  "title": "Backend Engineer",
  "company": {
    "id": 1,
    "name": "Acme Corporation"
  },
  "location": "Remote",
  "employmentType": "FULL_TIME",
  "minSalary": 90000,
  "maxSalary": 130000,
  "status": "ACTIVE",
  "createdAt": "2026-07-01T09:30:00Z"
}
{
  "timestamp": "2026-07-07T10:15:00Z",
  "status": 400,
  "error": "VALIDATION_ERROR",
  "message": "Request contains invalid fields",
  "details": [
    {
      "field": "email",
      "issue": "must be a well-formed email address"
    }
  ]
}
Monitoring URLs
Resource	URL
Swagger UI	http://localhost:8080/swagger-ui/index.html
Actuator Root	http://localhost:8080/actuator
Health	http://localhost:8080/actuator/health
Metrics	http://localhost:8080/actuator/metrics
Prometheus Scrape Endpoint	http://localhost:8080/actuator/prometheus
Prometheus UI	http://localhost:9090
Grafana	http://localhost:3000
Performance Optimizations
Caching: Caffeine reduces repeated database hits for job and company reads.
Batch Fetching: Hibernate @BatchSize reduces N+1 query patterns on collection associations.
Lazy Loading: Associations default to FetchType.LAZY, with explicit fetch joins where eager loading is required.
Logging: Asynchronous appenders prevent I/O from blocking request threads under load.
Exception Handling
Global Exception Handler: A single @ControllerAdvice class translates all exceptions into a consistent JSON error schema.
Validation Errors: MethodArgumentNotValidException is mapped to a 400 response with a field-level breakdown.
Authentication Errors: Invalid or expired credentials return 401 with a generic message to avoid leaking account existence.
Authorization Errors: Insufficient role or permission returns 403 with no sensitive detail exposed.
Future Improvements
Migrate refresh-token storage to Redis for distributed session invalidation
Add rate limiting at the API gateway layer
Introduce event-driven notifications (Kafka) for application status changes
Add full-text search via Elasticsearch for job listings
Support OAuth2 login (Google, LinkedIn)
Add multi-tenancy support for enterprise recruiting clients
Introduce GraphQL as an alternative query interface
Add automated resume parsing and keyword extraction
Implement soft-delete recovery workflows for admins
Add distributed tracing via OpenTelemetry
Introduce blue-green deployment pipeline in CI/CD
Add configurable email notification templates
Support file storage via S3-compatible object storage for resumes
Add integration test coverage for all controller endpoints
Introduce API versioning strategy for backward compatibility
Contributing
Contributions are welcome. Please follow the process below:

Fork the repository and create a feature branch (git checkout -b feature/your-feature).
Write tests for any new functionality.
Ensure mvn verify passes locally before opening a pull request.
Follow the existing code style (Lombok conventions, layered architecture).
Submit a pull request with a clear description of the change and its motivation.
Tip

For substantial changes, open an issue first to discuss the proposed approach.

Author
Gayathri Chauhan

GitHub LinkedIn
