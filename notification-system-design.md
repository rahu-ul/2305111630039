# Software Design Document (SDD)

# Campus Notification System

**Technology Stack:** React + Material UI, Node.js, Express.js, MongoDB, External Notification API



### 1.1 Overview

The Campus Notification System is a web-based application that provides students with a centralized platform to receive and manage important institutional notifications. Instead of visiting multiple portals for placement drives, examination results, or campus events, students access a single interface where notifications are retrieved, categorized, filtered, and displayed in real time.

Authentication is assumed to be handled by an external Identity Provider. The application therefore focuses on authorization, notification retrieval, presentation, logging, and communication with an external Notification API.

The primary objectives are:

* Deliver notifications reliably.
* Reduce notification access time.
* Provide a responsive user experience.
* Maintain complete observability through centralized logging.
* Support future scalability without major architectural changes.

---

### 1.2 Architectural Goals

The architecture emphasizes five core principles:

**Scalability** – Stateless backend services allow horizontal scaling through load balancers or container orchestration platforms.

**Maintainability** – Clear separation of concerns ensures that UI, business logic, data access, and integrations remain independent.

**Reliability** – Standardized error handling, retries, and structured logging ensure predictable system behavior.

**Performance** – Server-side pagination, optimized database indexes, and efficient API design reduce latency.

**Extensibility** – New notification categories, user roles, or delivery channels can be added without redesigning the system.

---

### 1.3 Why a Layered Architecture?

A layered architecture was selected because it balances simplicity with scalability. Each layer has a single responsibility:

* React handles presentation.
* Express handles request processing.
* Services implement business rules.
* MongoDB stores notification data.
* External APIs provide institutional notifications.

This modular structure simplifies testing, debugging, and future enhancements while avoiding the operational complexity of microservices.

---

# 2. Functional Requirements

The system provides the following core functionality.

## 2.1 View Notifications

Students can view notifications after successful authentication. Notifications display:

* Title
* Description
* Category
* Priority
* Publish Date
* Read Status

Notifications are sorted by priority and publication time so that urgent information is always displayed first.

---

## 2.2 Notification Categories

Three notification categories are supported:

* Placement
* Result
* Event

The design allows future categories to be added without changing the frontend because categories are retrieved dynamically through the API.

---

## 2.3 Search and Filtering

Users can locate notifications using:

* Category
* Priority
* Date Range
* Keyword Search
* Read/Unread Status

Filtering is performed on the backend to reduce network traffic and leverage database indexing.

---

## 2.4 Pagination

Notifications are returned in pages rather than loading the complete dataset.

Example:

```http
GET /notifications?page=1&limit=20
```

Benefits include:

* Faster page loading
* Reduced memory usage
* Lower network bandwidth
* Better scalability

---

## 2.5 Notification Details

Selecting a notification opens a detailed view containing:

* Full description
* Attachments
* Source
* Publication date
* Related links

Only detailed data is loaded when required, reducing unnecessary API traffic.

---

## 2.6 Loading and Empty States

To improve user experience, the application displays:

* Skeleton loaders during requests.
* Empty-state illustrations when no notifications are available.
* Retry buttons after failures.

These states provide clear feedback and prevent the interface from appearing unresponsive.

---

## 2.7 Responsive User Interface

Material UI provides responsive layouts using predefined breakpoints.

Supported devices include:

* Mobile phones
* Tablets
* Laptops
* Desktop systems

Components automatically adjust spacing, typography, and layout based on screen size.

---

## 2.8 Notification Badge

The application header displays unread notification counts retrieved through a dedicated lightweight API endpoint.

This avoids downloading the complete notification list merely to update the badge.

---

# 3. Non-Functional Requirements

## Performance

Target response time:

* API Response: <250 ms
* Database Query: <50 ms
* Search: <100 ms

Performance is achieved through indexing, pagination, caching, and optimized queries.

---

## Availability

Target uptime:

99.9%

Availability is supported through:

* Stateless servers
* Health monitoring
* Automatic restarts
* Load balancing

---

## Reliability

Reliability is achieved using:

* Retry mechanisms
* Timeout handling
* Graceful degradation
* Centralized exception handling
* Structured logging

---

## Security

Security controls include:

* JWT Bearer authentication
* HTTPS
* CORS
* Rate limiting
* Input validation
* Secure environment variables

---

## Scalability

The backend stores no session state, allowing additional Express instances to be deployed without modifying application logic.

Future deployments may use Docker and Kubernetes for automatic scaling.

---

## Accessibility

The frontend follows WCAG guidelines through:

* Semantic HTML
* Keyboard navigation
* Screen-reader compatibility
* Color contrast compliance

---

## Observability

Operational visibility is provided through:

* Structured logs
* Correlation IDs
* Health endpoints
* Performance metrics

This enables production debugging without reproducing issues locally.

---

# 4. System Architecture

## High-Level Architecture

```text
+------------------------+
| React + Material UI    |
+-----------+------------+
            |
            | HTTPS
            v
+------------------------+
| API Client (Axios)     |
+-----------+------------+
            |
            v
+------------------------+
| Express Backend        |
+-----------+------------+
            |
     +------+------+
     |             |
     v             v
Notification   Logging
 Service       Middleware
     |             |
     v             v
External API   Log Storage
```

---

## Layer Responsibilities

### Frontend

The React application manages:

* User interface
* API requests
* Routing
* State management
* Loading states
* Error handling

Business logic remains on the server.

---

### API Layer

The API client centralizes:

* Base URLs
* Authentication headers
* Request interceptors
* Response interceptors
* Retry logic

Keeping HTTP logic in one location avoids duplication.

---

### Backend

Express acts as the orchestration layer responsible for:

* Authentication
* Validation
* Business logic
* Communication with external services
* Response formatting
* Logging

Controllers remain thin, delegating processing to service classes.

---

### Notification Service

This service encapsulates all communication with the external Notification API.

Responsibilities include:

* Building outbound requests
* Retrying transient failures
* Normalizing third-party responses
* Applying business rules

This abstraction isolates the rest of the application from changes in external providers.

---

### Logging Middleware

Logging is implemented as middleware so that every request is captured consistently.

Each log entry records:

* Request ID
* User ID
* Endpoint
* Execution time
* Status code
* Timestamp

Structured logging enables centralized monitoring and troubleshooting.

---

# 5. Component Architecture

## Frontend Components

Major React components include:

* Layout
* Header
* NotificationList
* NotificationCard
* NotificationDetails
* SearchBar
* FilterPanel
* Pagination
* LoadingSkeleton
* ErrorState
* EmptyState

Each component performs a single responsibility, improving reuse and maintainability.

---

## Custom Hooks

Business logic is encapsulated in reusable hooks:

* useNotifications()
* usePagination()
* useSearch()
* useFilters()
* useAuth()

Separating UI from data-fetching logic improves readability and simplifies testing.

---

## Backend Components

The backend is organized into:

* Controllers
* Services
* Repositories
* Models
* Middleware
* Validators
* Utilities
* Configuration

Each layer communicates only with the layer directly beneath it, reducing coupling.

---

## Component Interaction

```text
React Component
      │
      ▼
Custom Hook
      │
      ▼
API Client
      │
      ▼
Express Controller
      │
      ▼
Service Layer
      │
      ▼
Repository / External API
      │
      ▼
Database
```

This interaction pattern keeps presentation logic separate from business logic and data access.

---

# 6. API Design (Condensed)

The system exposes RESTful APIs under:

```text
/api/v1
```

### Core Endpoints

| Method | Endpoint                    | Purpose                   |
| ------ | --------------------------- | ------------------------- |
| GET    | /notifications              | Retrieve notifications    |
| GET    | /notifications/:id          | Notification details      |
| PATCH  | /notifications/:id/read     | Mark notification as read |
| GET    | /notifications/unread-count | Retrieve unread count     |
| GET    | /notifications/categories   | List categories           |
| POST   | /notifications/refresh      | Synchronize notifications |
| GET    | /health                     | Service health            |

---

## Request Headers

```http
Authorization: Bearer <JWT>

Content-Type: application/json
```

---

## Standard Success Response

```json
{
  "success": true,
  "message": "Notifications retrieved successfully.",
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "totalPages": 5,
    "totalRecords": 95
  }
}
```

---

## Standard Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid category."
  },
  "requestId": "REQ-91AFD2"
}
```

---

## API Design Principles

The API follows these engineering practices:

* RESTful resource naming.
* Stateless communication.
* JSON payloads.
* Consistent response structure.
* Server-side pagination.
* Query parameter filtering.
* Standard HTTP status codes.
* URI versioning.
* Centralized validation.
* Structured logging.


## 7. Authentication Flow

### 7.1 Overview

Authentication is handled by an external Identity Provider, while the Campus Notification System is responsible for validating the received JWT Bearer Token and authorizing access to protected resources. This separation keeps the application stateless, improves scalability, and allows integration with existing campus authentication systems such as OAuth 2.0 or OpenID Connect.

Every API request must include a valid Bearer Token in the `Authorization` header. Requests without a valid token are rejected before reaching business logic.

```http
Authorization: Bearer <JWT_TOKEN>
```

### Authentication Lifecycle

```text
Student
    │
    ▼
React Application
    │
Authorization Header
    ▼
Express Middleware
    │
Validate JWT
    │
Verify Expiration
    │
Attach User Context
    ▼
Controller
    ▼
Business Service
```

### Validation Steps

The authentication middleware performs:

* Verify Authorization header.
* Extract JWT.
* Validate token signature.
* Verify expiration (`exp` claim).
* Validate issuer (`iss`).
* Validate audience (`aud`).
* Attach authenticated user information to `req.user`.

Example user context:

```javascript
req.user = {
  userId: "STU10245",
  role: "STUDENT",
  department: "CSE"
};
```

### Authentication Responses

| Status | Description                        |
| ------ | ---------------------------------- |
| 200    | Authenticated                      |
| 401    | Missing, invalid, or expired token |
| 403    | User lacks permission              |

The backend never stores sessions, allowing any server instance to process authenticated requests, which simplifies horizontal scaling.

---

# 8. Notification Flow

### Overview

The notification flow describes how information travels from the external notification provider to the student's browser. The backend acts as an orchestration layer, isolating the frontend from third-party integrations and ensuring consistent data formatting.

### End-to-End Flow

```text
Student
   │
   ▼
React UI
   │
Axios API Client
   │
Express Backend
   │
Notification Service
   │
External Notification API
   │
Normalize Response
   │
Logging Middleware
   │
React UI
```

### Processing Steps

1. Student opens the notification page.
2. React sends a request with the Bearer Token.
3. Middleware authenticates and validates the request.
4. Controller delegates processing to the Notification Service.
5. Notification Service calls the external API.
6. Response is normalized into the application's internal model.
7. Structured logs are generated.
8. The formatted response is returned to the frontend.
9. React updates the UI and unread badge.

### Failure Handling

If the external service is unavailable:

* Retry transient failures.
* Return cached data if available.
* Display a user-friendly message.
* Record detailed logs for diagnosis.

---

# 9. Logging Architecture

### Purpose

Logging is mandatory because it provides operational visibility, supports auditing, and enables rapid production troubleshooting. Every request and response is recorded using structured JSON logs rather than simple console statements.

### Logging Objectives

* Track every HTTP request.
* Measure response time.
* Capture authentication failures.
* Record external API latency.
* Monitor database performance.
* Simplify debugging.
* Support centralized monitoring.

### Logging Flow

```text
Client Request
      │
      ▼
Logging Middleware
      │
Authentication
      │
Controller
      │
Service
      │
Database / External API
      │
Response
      │
Final Log Entry
```

### Information Captured

Each log includes:

* Timestamp
* Request ID
* Correlation ID
* User ID
* HTTP Method
* Endpoint
* Status Code
* Execution Time
* Error Details (if applicable)

Example:

```json
{
  "timestamp":"2026-06-26T11:15:10Z",
  "requestId":"REQ-1023",
  "userId":"STU10245",
  "method":"GET",
  "endpoint":"/api/v1/notifications",
  "statusCode":200,
  "duration":"82ms"
}
```

### Log Levels

| Level | Usage                   |
| ----- | ----------------------- |
| INFO  | Successful operations   |
| DEBUG | Development diagnostics |
| WARN  | Recoverable issues      |
| ERROR | Application failures    |
| FATAL | Critical system errors  |

### Frontend Logging

The frontend logs:

* Route changes
* API failures
* Rendering errors
* Retry attempts
* Performance metrics

Sensitive data such as passwords and JWT tokens are never logged.

### Backend Logging

Backend logging records:

* Incoming requests
* Authentication results
* Validation failures
* Database queries
* External API interactions
* Exceptions

Using centralized middleware ensures consistent logging without duplicating code across controllers.

---

# 10. Error Handling Strategy

The system uses centralized exception handling to ensure all failures produce predictable responses.

### Standard Error Response

```json
{
  "success": false,
  "error": {
    "code":"VALIDATION_ERROR",
    "message":"Invalid category."
  }
}
```

### HTTP Status Codes

| Code | Meaning                      |
| ---- | ---------------------------- |
| 400  | Invalid request              |
| 401  | Authentication required      |
| 403  | Permission denied            |
| 404  | Resource not found           |
| 429  | Too many requests            |
| 500  | Internal server error        |
| 503  | External service unavailable |

### Retry Strategy

Retry is applied only to transient failures such as:

* Network timeout
* External API timeout
* HTTP 503

Exponential backoff:

* Immediate
* 1 second
* 2 seconds

Permanent errors (400, 401, 403) are never retried.

### Fallback Strategy

When the external Notification API is unavailable:

* Return cached notifications when possible.
* Display an informative message.
* Continue serving unaffected features.
* Log the incident for operational review.

---

# 11. Database Design

MongoDB stores notification documents because its flexible schema accommodates evolving notification structures.

### Notification Collection

```javascript
{
  _id,
  title,
  description,
  category,
  priority,
  source,
  attachments,
  publishedAt,
  createdAt,
  updatedAt
}
```

A separate collection maintains per-user read status, preventing duplication of notification records.

### Indexing

Recommended indexes:

* publishedAt
* category
* priority
* title (text)
* description (text)

Compound Index:

```
category + priority + publishedAt
```

This supports common filtering and sorting operations efficiently.

### Pagination

Server-side pagination:

```http
GET /notifications?page=1&limit=20
```

Benefits:

* Lower memory usage.
* Faster responses.
* Improved scalability.

### Archiving

Expired notifications are archived rather than deleted to support reporting and audit requirements.

Future deployments can implement scheduled archival jobs.

---

# 12. Security Considerations

Security is integrated into every application layer.

### Authentication

* JWT Bearer Tokens
* Signature verification
* Token expiration validation

### Authorization

Current role:

* Student

Future roles:

* Admin
* Placement Officer
* Faculty

### Input Validation

Every request is validated for:

* Required fields
* Data types
* Length
* Enum values
* Sanitization

### HTTPS

All communication occurs over HTTPS to protect confidentiality and integrity.

### CORS

Only approved frontend domains may access backend APIs.

### Environment Variables

Sensitive configuration is stored outside source code.

Examples:

```
JWT_SECRET
MONGODB_URI
API_KEY
```

Secrets should be managed using dedicated secret-management services in production.

---

# 13. Performance Optimizations

Performance improvements include:

* Server-side pagination.
* Lazy loading notification details.
* Debounced search requests.
* Gzip/Brotli compression.
* Connection pooling.
* Query optimization.
* Optional Redis caching.

These optimizations reduce latency while improving scalability.

---

# 14. Production Folder Structure

### Frontend

```text
src/
 ├── api/
 ├── components/
 ├── hooks/
 ├── pages/
 ├── layouts/
 ├── services/
 ├── context/
 ├── utils/
 ├── styles/
 └── App.jsx
```

Responsibilities:

* **api** – Axios clients.
* **components** – Reusable UI.
* **hooks** – Shared business logic.
* **pages** – Route components.
* **services** – API interaction.
* **utils** – Helper functions.
* **context** – Global state.

### Backend

```text
src/
 ├── config/
 ├── controllers/
 ├── middleware/
 ├── models/
 ├── repositories/
 ├── routes/
 ├── services/
 ├── validators/
 ├── utils/
 ├── app.js
 └── server.js
```

Controllers manage HTTP requests, services implement business rules, repositories abstract MongoDB operations, middleware provides authentication, validation, and logging, while configuration centralizes environment-specific settings.

---

# 15. Future Improvements

The architecture is intentionally extensible and supports future enhancements without major redesign.

Planned improvements include:

* Push notifications using Firebase Cloud Messaging.
* WebSocket support for real-time updates.
* Role-Based Access Control (RBAC).
* Administrative dashboard.
* Notification scheduling.
* Analytics and reporting.
* Progressive Web App (PWA) offline support.
* Email and SMS notification channels.
* Redis caching.
* OpenTelemetry, Prometheus, and Grafana integration for advanced observability.

These enhancements can be introduced incrementally because the layered architecture isolates presentation, business logic, data access, and external integrations.

---

# Conclusion

The Campus Notification System adopts a layered architecture that emphasizes modularity, scalability, and operational reliability. Stateless authentication, standardized REST APIs, structured logging, centralized error handling, optimized MongoDB design, and enterprise security practices provide a strong foundation for production deployment. By separating concerns across frontend, backend, service, and data layers, the system remains maintainable and adaptable to future requirements such as real-time notifications, administrative workflows, and advanced analytics without significant architectural changes.





