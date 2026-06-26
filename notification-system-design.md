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

------

# Stage 2

This stage extends the Campus Notification Platform described in Stage 1. The existing layered architecture (React + Material UI frontend, Node.js + Express backend, external authentication, logging middleware, and Notification APIs) remains unchanged. Stage 2 focuses exclusively on the persistence layer, database design, query optimization, scalability, and implementation considerations.

---

# 2.1 Database Selection

## Selected Database: MongoDB (NoSQL)

MongoDB is selected because the notification platform primarily manages document-oriented data that evolves over time. Notification records may include optional fields such as attachments, links, metadata, or future notification attributes without requiring expensive schema migrations.

The application workload is dominated by:

* Read-heavy notification retrieval
* Filtering by category and priority
* Time-based sorting
* Pagination
* Full-text search
* Per-user read status

MongoDB naturally supports these access patterns while integrating seamlessly with the Node.js and Express ecosystem through Mongoose.

### Comparison with SQL

| MongoDB                     | Relational Database                     |
| --------------------------- | --------------------------------------- |
| Flexible schema             | Fixed schema                            |
| JSON documents              | Rows and columns                        |
| Fast document retrieval     | Strong relational consistency           |
| Easy horizontal scaling     | More complex scaling                    |
| Ideal for notification data | Better for highly transactional systems |

Since the platform does not perform complex joins or financial transactions, MongoDB provides a simpler and more maintainable solution while meeting all functional requirements.

---

# 2.2 Database Schema

Two primary collections are used.

## Notifications Collection

| Field       | Type     | Constraint  | Default      | Index        |
| ----------- | -------- | ----------- | ------------ | ------------ |
| _id         | ObjectId | Primary Key | Auto         | Yes          |
| title       | String   | Required    | —            | Text         |
| description | String   | Required    | —            | Text         |
| category    | String   | Enum        | placement    | Yes          |
| priority    | String   | Enum        | medium       | Yes          |
| source      | String   | Required    | —            | No           |
| attachments | Array    | Optional    | []           | No           |
| publishedAt | Date     | Required    | Current Date | Yes          |
| expiresAt   | Date     | Optional    | Null         | TTL (Future) |
| createdAt   | Date     | Auto        | Current Date | Yes          |
| updatedAt   | Date     | Auto        | Current Date | No           |

### Field Explanation

* **title** – Notification heading displayed in the list view.
* **description** – Detailed notification content.
* **category** – Placement, Result, or Event.
* **priority** – Determines display order.
* **source** – Department or authority issuing the notification.
* **attachments** – Supporting files or documents.
* **publishedAt** – Used for sorting latest notifications.
* **expiresAt** – Future archival support.
* **createdAt/updatedAt** – Audit timestamps managed automatically.

---

## UserNotifications Collection

Read status is stored separately to avoid duplicating notification documents.

| Field          | Type     | Constraint    | Index    |
| -------------- | -------- | ------------- | -------- |
| _id            | ObjectId | Primary Key   | Yes      |
| userId         | String   | Required      | Yes      |
| notificationId | ObjectId | Reference     | Compound |
| isRead         | Boolean  | Default False | Yes      |
| readAt         | Date     | Nullable      | No       |
| createdAt      | Date     | Auto          | No       |
| updatedAt      | Date     | Auto          | No       |

This design supports millions of users without modifying the original notification document.

---

# 2.3 Entity Relationships

The database follows a hybrid document model.

```
Notification
      │
      │ 1
      │
      ▼
UserNotification
      ▲
      │
      │ Many
      │
Student(User)
```

### Relationship Description

* One notification can belong to many students.
* Each student maintains an independent read status.
* Notifications remain immutable once published.
* User-specific metadata is stored in a referenced collection.

### Embedded vs Referenced Documents

Attachments are embedded because they are tightly coupled with notifications and always retrieved together.

Read status is referenced because it varies for every user and would otherwise duplicate notification data across documents.

---

# 2.4 Indexing Strategy

Efficient indexing is essential because notification retrieval is the most frequent operation.

| Index                    | Purpose              | Complexity            |
| ------------------------ | -------------------- | --------------------- |
| _id                      | Primary lookup       | O(log n)              |
| createdAt                | Latest notifications | O(log n)              |
| category                 | Category filtering   | O(log n)              |
| priority                 | Priority sorting     | O(log n)              |
| isRead                   | Unread queries       | O(log n)              |
| Text(title, description) | Search               | Optimized text search |

### Compound Index

```
category + priority + publishedAt
```

Supports queries such as:

* High priority placement notifications.
* Latest event notifications.
* Category filtering with chronological sorting.

Without this index, MongoDB would perform expensive collection scans as the dataset grows.

---

# 2.5 Query Design

The following queries directly support the REST APIs defined in Stage 1.

### Fetch Notifications

```javascript
db.notifications.find({})
.sort({publishedAt:-1})
.skip((page-1)*limit)
.limit(limit)
```

Returns paginated notifications ordered by publication date.

---

### Filter by Category

```javascript
db.notifications.find({
category:"placement"
})
```

Uses the category index for efficient retrieval.

---

### Filter by Priority

```javascript
db.notifications.find({
priority:"high"
})
```

Ensures urgent notifications appear first.

---

### Latest Notifications

```javascript
db.notifications.find({})
.sort({publishedAt:-1})
.limit(10)
```

Returns the ten most recent notifications.

---

### Full Text Search

```javascript
db.notifications.find({
$text:{
$search:"Google"
}
})
```

Uses MongoDB text indexes on title and description.

---

### Unread Notifications

```javascript
db.userNotifications.find({
userId:"STU10245",
isRead:false
})
```

Supports unread filters and notification badges.

---

### Mark Notification as Read

```javascript
db.userNotifications.updateOne(
{
userId:"STU10245",
notificationId:ObjectId(id)
},
{
$set:{
isRead:true,
readAt:new Date()
}
}
)
```

Updates only the user's read state without modifying the notification.

---

### Count Unread Notifications

```javascript
db.userNotifications.countDocuments({
userId:"STU10245",
isRead:false
})
```

Supports the unread badge displayed in the application header.

---

# 2.6 Scalability Problems

As adoption grows, several database challenges may emerge.

### Large Collections

Thousands of notifications accumulated over multiple semesters increase storage requirements and query costs.

### Slow Pagination

Using large `skip()` values becomes inefficient for deep pages because MongoDB scans skipped documents.

### Frequent Reads

Students repeatedly request the same notification list during placement seasons, increasing read load.

### High Write Volume

Publishing multiple notifications within short intervals can temporarily increase write latency.

### Index Growth

Additional indexes improve query performance but consume memory and storage.

### Duplicate Notifications

Repeated synchronization with the external notification API may introduce duplicate records if uniqueness is not enforced.

### Storage Growth

Attachments and historical notifications continuously increase disk utilization.

### Memory Pressure

Large working sets may exceed available RAM, forcing disk-based reads.

### Network Latency

External Notification API calls increase end-to-end response time.

---

# 2.7 Scalability Solutions

The proposed solutions remain achievable within the project scope.

| Problem                 | Solution                                                  |
| ----------------------- | --------------------------------------------------------- |
| Large Collections       | Archive expired notifications                             |
| Slow Pagination         | Use indexed sorting and cursor-based pagination in future |
| Frequent Reads          | Future Redis cache for unread counts and categories       |
| High Writes             | Batch synchronization jobs                                |
| Duplicate Notifications | Unique external notification ID validation                |
| Storage Growth          | Archive historical records                                |
| Memory Pressure         | Proper indexing and query projection                      |
| Network Latency         | Connection reuse and request timeouts                     |

Connection pooling provided by the MongoDB driver reduces repeated connection establishment overhead.

Caching is intentionally listed as a future enhancement rather than a current dependency to keep the implementation lightweight.

---

# 2.8 Database Security

Database security follows defense-in-depth principles.

### Input Validation

All request parameters are validated before reaching the repository layer.

### Injection Prevention

Queries are constructed using Mongoose models instead of dynamically concatenated strings, preventing NoSQL injection attacks.

### Data Integrity

* Required fields
* Enum validation
* Automatic timestamps
* Unique constraints where applicable

### Access Control

Application users never communicate directly with MongoDB.

Only the backend service has database access.

### Least Privilege

The database account is granted only the permissions required for notification operations.

Administrative privileges are reserved for deployment and maintenance.

### Environment Variables

Sensitive values are stored outside source code.

Examples:

* `MONGODB_URI`
* `DB_USERNAME`
* `DB_PASSWORD`

### Secrets Management

Production environments should use managed secret stores such as AWS Secrets Manager or Azure Key Vault rather than hard-coded credentials.

---

# 2.9 Performance Optimization

Several optimization techniques are incorporated into the database layer.

### Pagination

Server-side pagination limits result sets to reduce memory usage and response time.

### Projection

Only required fields are returned for list views.

Example:

```javascript
db.notifications.find(
{},
{
title:1,
category:1,
priority:1,
publishedAt:1
}
)
```

Reducing document size improves throughput.

### Index Usage

Frequently queried fields are indexed to minimize collection scans.

### Sorting

Sorting uses indexed fields (`publishedAt`, `priority`) to avoid in-memory sorting.

### Limited Response Fields

Detailed notification content is retrieved only when the user opens an individual notification.

### Connection Reuse

MongoDB connection pooling allows multiple requests to reuse existing database connections, reducing latency and improving throughput.

Under normal campus workloads, these optimizations are expected to keep notification retrieval below the performance targets established in Stage 1.

---

# 2.10 API Mapping

| REST API                        | Controller             | Service             | Database Operation          | Response             |
| ------------------------------- | ---------------------- | ------------------- | --------------------------- | -------------------- |
| GET /notifications              | NotificationController | NotificationService | `find()`                    | Notification List    |
| GET /notifications/:id          | NotificationController | NotificationService | `findById()`                | Notification Details |
| GET /notifications/categories   | NotificationController | NotificationService | `distinct()`                | Categories           |
| PATCH /notifications/:id/read   | NotificationController | NotificationService | `updateOne()`               | Updated Status       |
| GET /notifications/unread-count | NotificationController | NotificationService | `countDocuments()`          | Count                |
| POST /notifications/refresh     | NotificationController | NotificationService | `insertMany()/updateMany()` | Sync Result          |
| GET /health                     | HealthController       | HealthService       | Database Ping               | Health Status        |

The controller validates requests, the service applies business rules, and the repository executes optimized MongoDB operations before returning a standardized response.

---

# 2.11 Future Improvements

The current database design intentionally avoids unnecessary complexity while leaving room for practical enhancements.

Future improvements include:

* **Redis Cache** for frequently accessed notification lists and unread counts.
* **MongoDB Replica Sets** to improve availability and fault tolerance.
* **Sharding** for horizontal scaling if notification volume grows significantly.
* **TTL Indexes** to automatically remove expired notifications after archival.
* **Analytics Database** for reporting without impacting operational queries.
* **Data Warehouse Integration** for institutional reporting and long-term analytics.

These enhancements are identified as future work and are not required for the current implementation, ensuring the solution remains achievable within the scope of the Affordmed evaluation while supporting future growth.



-------

# Stage 3

## Database Performance Review

This stage extends the existing Software Design Document by reviewing the performance characteristics of the notification database after significant data growth. The application now serves approximately **50,000 students** and stores **5,000,000 notification records**. At this scale, query execution plans, indexing strategy, and data access patterns become critical to maintaining acceptable response times.

---

# 3.1 Query Analysis

The current production query is:

```sql
SELECT *
FROM notifications
WHERE studentID = 1042
AND isRead = false
ORDER BY createdAt ASC;
```

## Objective

The query retrieves all unread notifications for a particular student and displays them in chronological order, with the oldest unread notification appearing first.

This behavior matches the functional requirements defined in Stage 1:

* Retrieve notifications for an authenticated student.
* Filter unread notifications.
* Display notifications in a deterministic order.

## Logical Correctness

From a business perspective, the query is correct. It applies two filtering conditions:

* `studentID = 1042`
* `isRead = false`

and then sorts the matching rows by `createdAt`.

The issue is not correctness but efficiency. As the notification table grows into millions of rows, executing this query without appropriate indexing results in unnecessary work for the database engine.

## Inefficiencies

Several characteristics contribute to poor performance:

* `SELECT *` retrieves every column, even when only a subset is required.
* Filtering on two columns without a composite index may require scanning a large number of rows.
* Sorting by `createdAt` may require an explicit sort operation if no suitable index exists.
* Returning every unread notification without pagination increases memory consumption and response time.

---

# 3.2 Why Is the Query Slow

## Full Table Scan

Without a usable index, the database examines every row in the notifications table to identify records matching the requested student and unread status.

With 5 million records, this means millions of comparisons for a single request.

---

## Large Dataset

The application stores notifications for approximately 50,000 students.

As the table grows:

* more disk pages must be accessed,
* cache efficiency decreases,
* response time increases.

Query latency grows because more data must be inspected before the matching rows are located.

---

## Sorting Cost

The query requests:

```sql
ORDER BY createdAt ASC
```

If the filtered result is not already ordered by an index, the database performs an additional sorting step.

This may involve:

* allocating temporary memory,
* sorting thousands of rows,
* spilling to disk if memory limits are exceeded.

Sorting therefore becomes an additional performance bottleneck beyond filtering.

---

## Missing Indexes

If indexes do not exist on the filtering columns, the optimizer has no efficient access path.

Instead of performing indexed lookups, it performs sequential scans followed by sorting.

---

## Disk I/O

Rows not present in memory must be read from storage.

Large sequential scans increase:

* disk reads,
* storage latency,
* operating system cache pressure.

Reducing the number of pages read has a direct impact on response time.

---

## Memory Usage

Sorting operations consume memory.

Large result sets may exceed the database's configured sort buffer, forcing temporary data to disk.

Disk-based sorting is significantly slower than in-memory sorting.

---

## Execution Plan

Without suitable indexes, a typical execution plan is:

```text
Table Scan
      ↓
Apply studentID Filter
      ↓
Apply isRead Filter
      ↓
Sort by createdAt
      ↓
Return Results
```

The expensive steps are the table scan and explicit sort.

---

## Temporary Tables

Some database engines create temporary tables when sorting or processing large intermediate results.

Temporary tables increase:

* memory allocation,
* disk usage,
* execution time.

Avoiding them is one of the primary goals of index optimization.

---

# 3.3 Time Complexity

Theoretical complexity provides a useful approximation of query behavior.

| Strategy        | Estimated Complexity |
| --------------- | -------------------- |
| No Index        | O(n) + O(k log k)    |
| Single Index    | O(log n + k)         |
| Composite Index | O(log n + k)         |

Where:

* **n** = total rows in the table.
* **k** = matching rows for a student.

### Without Index

Every row is examined.

Complexity:

```text
O(n)
```

followed by sorting:

```text
O(k log k)
```

---

### With Single Index

Indexing only `studentID` reduces lookup time but still requires filtering unread records and possibly sorting.

Performance improves substantially but is not optimal.

---

### With Composite Index

A composite index matching the filter and sort order allows the optimizer to:

* locate matching rows,
* preserve ordering,
* avoid an explicit sort.

This is the preferred production solution.

---

# 3.4 Index Design

## Primary Key

```sql
PRIMARY KEY (notificationID)
```

Supports direct notification lookup.

---

## Student Index

```sql
CREATE INDEX idx_student
ON notifications(studentID);
```

Accelerates student-specific queries.

---

## Read Status Index

```sql
CREATE INDEX idx_isread
ON notifications(isRead);
```

Useful when unread notifications represent only a small percentage of records.

---

## Created Time Index

```sql
CREATE INDEX idx_created
ON notifications(createdAt);
```

Supports chronological queries.

---

## Composite Index (Recommended)

```sql
CREATE INDEX idx_student_read_created
ON notifications(studentID, isRead, createdAt);
```

### Why This Order?

The optimizer evaluates predicates from left to right.

* `studentID` has high selectivity.
* `isRead` further narrows the result set.
* `createdAt` satisfies the required ordering.

Because the index already stores rows in `createdAt` order for each `(studentID, isRead)` combination, the database can often avoid an additional sort operation.

---

## Covering Index

If list views require only summary information, a covering index can reduce table lookups.

Example:

```sql
(studentID, isRead, createdAt, notificationType, title)
```

The database can satisfy many requests directly from the index without accessing the table pages.

---

# 3.5 Optimized Query

Instead of retrieving every column:

```sql
SELECT notificationID,
       title,
       notificationType,
       createdAt
FROM notifications
WHERE studentID = 1042
  AND isRead = FALSE
ORDER BY createdAt ASC
LIMIT 50;
```

### Improvements

* Avoids `SELECT *`.
* Returns only required fields.
* Uses pagination through `LIMIT`.
* Enables index-assisted ordering.
* Reduces network traffic and memory usage.

When the composite index exists, `ORDER BY createdAt` can be satisfied directly from the index, eliminating the expensive sort phase.

---

# 3.6 Should Every Column Be Indexed?

No.

Indexing every column is generally poor engineering practice.

## Storage Overhead

Each index consumes additional disk space.

More indexes mean larger databases and increased backup sizes.

---

## Insert Cost

Every insert must update every affected index.

Additional indexes therefore increase write latency.

---

## Update Cost

Updating indexed columns requires rebuilding index entries.

Frequently updated fields become more expensive to maintain.

---

## Delete Cost

Deleting rows also requires removing entries from every related index.

Large numbers of indexes slow deletion operations.

---

## Optimizer Complexity

More indexes increase the number of execution plans the optimizer must evaluate.

Excessive indexing can actually result in poorer plan selection.

---

## Maintenance Cost

Indexes require:

* monitoring,
* rebuilding,
* statistics updates,
* storage management.

Indexes should therefore exist only when justified by real query patterns.

The recommended approach is to index frequently filtered, joined, or sorted columns rather than indexing every attribute.

---

# 3.7 Placement Query

The following query returns every student who received a **Placement** notification during the previous seven days.

```sql
SELECT DISTINCT studentID,
       notificationID,
       title,
       createdAt
FROM notifications
WHERE notificationType = 'Placement'
  AND createdAt >= CURRENT_DATE - INTERVAL 7 DAY
ORDER BY createdAt DESC;
```

### Explanation

* Filters only placement notifications.
* Restricts results to the last seven days.
* Returns unique student records.
* Orders the newest notifications first.

For PostgreSQL, the interval syntax becomes:

```sql
CURRENT_DATE - INTERVAL '7 days'
```

---

# 3.8 Additional Performance Improvements

## Pagination

Always retrieve notifications in pages rather than loading the complete result set.

```sql
LIMIT 50 OFFSET 0;
```

---

## Projection

Avoid:

```sql
SELECT *
```

Return only fields required by the client.

Smaller result sets reduce network transfer and memory usage.

---

## Index Usage

Verify execution plans regularly using `EXPLAIN` to ensure indexes are actually being used.

Unused indexes increase maintenance cost without improving performance.

---

## Sorting

Prefer sorting on indexed columns.

This avoids expensive in-memory or disk-based sorting operations.

---

## Connection Pooling

Reuse existing database connections through the Node.js connection pool.

Benefits include:

* lower connection overhead,
* improved throughput,
* better resource utilization.

---

## Batch Reads

Where appropriate, retrieve multiple notifications in a single query instead of issuing repeated database requests.

This reduces network round trips.

---

## Future Enhancement: Query Cache

A cache layer may later store frequently requested notification summaries or unread counts.

This is intentionally future work rather than part of the current implementation.

---

## Future Enhancement: Partitioning

Partitioning by creation date can improve maintenance and archival of historical notifications.

This should be considered only if table growth significantly exceeds current projections.

---

## Future Enhancement: Materialized Views

Frequently requested reporting queries may later use materialized views.

Operational APIs should continue querying the base notification table to maintain data freshness.

---

# 3.9 Engineering Recommendation

The current query is functionally correct but does not scale efficiently against a table containing millions of notification records.

The primary performance issues are:

* full table scans,
* unnecessary sorting,
* retrieving all columns,
* absence of an optimal composite index.

The recommended improvements are:

1. Create a composite index on `(studentID, isRead, createdAt)`.
2. Replace `SELECT *` with explicit column projection.
3. Introduce pagination using `LIMIT`.
4. Verify execution plans with `EXPLAIN`.
5. Continue monitoring index usage as data volume grows.

These changes significantly reduce disk I/O, minimize sorting overhead, improve cache utilization, and provide substantially faster response times while remaining straightforward to implement within the current Node.js, Express, and SQL-based architecture. Future enhancements such as query caching, partitioning, and materialized views can be introduced incrementally if application growth demands additional optimization.





