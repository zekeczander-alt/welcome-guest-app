# API Specification Document

**Document Version:** 1.0  
**Date Created:** September 15, 2025  
**Author(s):** AI-Generated  
**Project Name:** Guest Link - A WhatsApp-based Hospitality App

---

## 1. Introduction
This API Specification Document details the backend endpoints for Guest Link, a WhatsApp-integrated hospitality management platform. It adopts a RESTful design pattern and provides comprehensive documentation for both internal API routes and external integrations, particularly with the WhatsApp Business API.

The API is structured to support the core functionalities identified in the Requirements Specification:
- Welcome page generation and management
- WhatsApp communication integration
- Service request handling
- Guest and stay management
- Analytics and reporting

Base URL: `/api/v1` (relative to app domain)
Primary Tech Stack:
- Next.js 13+ (App Router)
- Node.js for microservices
- PostgreSQL for data persistence
- Redis for caching and real-time features

---

## 2. API Overview

### Base Configuration
- **Base URL:** https://guest-link.app/api/v1
- **Authentication:** JWT Bearer token in Authorization header
- **Request Format:** JSON (Content-Type: application/json)
- **Response Format:** JSON (Accept: application/json)

### Common Headers
```json
{
  "Authorization": "Bearer <token>",
  "Content-Type": "application/json",
  "Accept": "application/json",
  "X-Property-Id": "<property_id>"
}
```

### Common Response Schemas

**ErrorResponse:**
```json
{
  "type": "object",
  "properties": {
    "error": { 
      "type": "string",
      "description": "Error message"
    },
    "code": { 
      "type": "integer",
      "description": "HTTP status code"
    },
    "details": {
      "type": "object",
      "description": "Additional error context"
    }
  },
  "required": ["error", "code"]
}
```

**PaginationMeta:**
```json
{
  "type": "object",
  "properties": {
    "total": { "type": "integer" },
    "page": { "type": "integer" },
    "limit": { "type": "integer" },
    "hasMore": { "type": "boolean" }
  },
  "required": ["total", "page", "limit", "hasMore"]
}
```

---

## 3. Endpoints

### 3.1 Authentication Endpoints

#### POST /auth/login
- **Description:** Authenticates a host/staff user and returns JWT token
- **Parameters:**
  **Body:**
  | Field    | Type   | Required | Description        |
  |----------|--------|----------|--------------------|
  | email    | string | Yes      | User email        |
  | password | string | Yes      | User password     |

- **Response (200 OK):**
  ```json
  {
    "type": "object",
    "properties": {
      "token": { "type": "string" },
      "user": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "email": { "type": "string" },
          "role": { "type": "string", "enum": ["host", "staff"] },
          "propertyIds": { "type": "array", "items": { "type": "string" } }
        }
      }
    }
  }
  ```

### 3.2 Welcome Page Endpoints

#### POST /welcome-pages
- **Description:** Creates a new welcome page for a stay
- **Parameters:**
  **Body:**
  | Field       | Type   | Required | Description              |
  |-------------|--------|----------|--------------------------|
  | stayId      | string | Yes      | Stay identifier         |
  | content     | object | Yes      | Page content            |
  | branding    | object | Yes      | Property branding info  |

- **Response (201 Created):**
  ```json
  {
    "type": "object",
    "properties": {
      "id": { "type": "string" },
      "url": { "type": "string" },
      "content": { "type": "object" },
      "createdAt": { "type": "string", "format": "date-time" }
    }
  }
  ```

#### GET /welcome-pages/{pageId}
- **Description:** Retrieves a welcome page by ID
- **Parameters:**
  **Path:**
  | Parameter | Type   | Description        |
  |-----------|--------|--------------------|
  | pageId    | string | Welcome page ID   |

- **Response (200 OK):**
  ```json
  {
    "type": "object",
    "properties": {
      "id": { "type": "string" },
      "content": { "type": "object" },
      "branding": { "type": "object" },
      "stayInfo": {
        "type": "object",
        "properties": {
          "checkIn": { "type": "string", "format": "date-time" },
          "checkOut": { "type": "string", "format": "date-time" },
          "guestName": { "type": "string" }
        }
      }
    }
  }
  ```

### 3.3 WhatsApp Integration Endpoints

#### POST /whatsapp/webhook
- **Description:** Receives WhatsApp webhook events
- **Parameters:**
  **Body:**
  | Field     | Type   | Required | Description        |
  |-----------|--------|----------|--------------------|
  | event     | string | Yes      | Event type        |
  | payload   | object | Yes      | Event data        |

- **Response (200 OK):**
  ```json
  {
    "type": "object",
    "properties": {
      "received": { "type": "boolean" }
    }
  }
  ```

#### POST /whatsapp/send
- **Description:** Sends a WhatsApp message to a guest
- **Parameters:**
  **Body:**
  | Field       | Type   | Required | Description              |
  |-------------|--------|----------|--------------------------|
  | stayId      | string | Yes      | Stay identifier         |
  | template    | string | Yes      | Message template ID     |
  | params      | object | No       | Template parameters     |

- **Response (202 Accepted):**
  ```json
  {
    "type": "object",
    "properties": {
      "messageId": { "type": "string" },
      "status": { "type": "string" }
    }
  }
  ```

### 3.4 Service Management Endpoints

#### POST /services/requests
- **Description:** Creates a new service request
- **Parameters:**
  **Body:**
  | Field       | Type   | Required | Description              |
  |-------------|--------|----------|--------------------------|
  | stayId      | string | Yes      | Stay identifier         |
  | serviceId   | string | Yes      | Service identifier      |
  | requestedFor| string | Yes      | Requested datetime      |
  | notes       | string | No       | Additional notes        |

- **Response (201 Created):**
  ```json
  {
    "type": "object",
    "properties": {
      "id": { "type": "string" },
      "status": { "type": "string" },
      "serviceDetails": { "type": "object" },
      "requestedFor": { "type": "string", "format": "date-time" },
      "createdAt": { "type": "string", "format": "date-time" }
    }
  }
  ```

#### GET /services/requests
- **Description:** Lists service requests for a property
- **Parameters:**
  **Query:**
  | Parameter  | Type    | Required | Description              |
  |------------|---------|----------|--------------------------|
  | propertyId | string  | Yes      | Property identifier     |
  | status     | string  | No       | Filter by status        |
  | date       | string  | No       | Filter by date          |
  | page       | integer | No       | Page number (default: 1) |
  | limit      | integer | No       | Items per page (default: 20) |

- **Response (200 OK):**
  ```json
  {
    "type": "object",
    "properties": {
      "requests": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "id": { "type": "string" },
            "service": { "type": "object" },
            "stay": { "type": "object" },
            "status": { "type": "string" },
            "requestedFor": { "type": "string", "format": "date-time" }
          }
        }
      },
      "meta": { "$ref": "#/components/schemas/PaginationMeta" }
    }
  }
  ```

### 3.5 Analytics Endpoints

#### GET /analytics/service-usage
- **Description:** Retrieves service usage analytics
- **Parameters:**
  **Query:**
  | Parameter   | Type   | Required | Description              |
  |-------------|--------|----------|--------------------------|
  | propertyId  | string | Yes      | Property identifier     |
  | startDate   | string | Yes      | Start date (ISO 8601)   |
  | endDate     | string | Yes      | End date (ISO 8601)     |
  | serviceId   | string | No       | Filter by service       |

- **Response (200 OK):**
  ```json
  {
    "type": "object",
    "properties": {
      "totalRequests": { "type": "integer" },
      "byService": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "serviceId": { "type": "string" },
            "serviceName": { "type": "string" },
            "requestCount": { "type": "integer" },
            "completionRate": { "type": "number" }
          }
        }
      },
      "timeline": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "date": { "type": "string", "format": "date" },
            "requestCount": { "type": "integer" }
          }
        }
      }
    }
  }
  ```

---

## 4. External APIs

### WhatsApp Business API
- **Base URL:** https://graph.facebook.com/v17.0
- **Authentication:** Access token in Authorization header
- **Rate Limits:** Varies by business tier
- **Key Endpoints:**

#### POST /{phone-number-id}/messages
- **Description:** Sends a message to a WhatsApp user
- **Parameters:**
  ```json
  {
    "to": "string",
    "template": {
      "name": "string",
      "language": {
        "code": "string"
      },
      "components": [
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "string"
            }
          ]
        }
      ]
    }
  }
  ```
- **Rate Limiting:** 80 messages per second
- **Error Handling:** Implements exponential backoff

---

## 5. Security and Best Practices

### Authentication & Authorization
- JWT tokens with 1-hour expiry
- Refresh tokens with 7-day expiry
- Role-based access control (RBAC)
- Property-level authorization checks

### Rate Limiting
- 100 requests per minute per IP
- 1000 requests per minute per authenticated user
- Implemented using Redis

### Data Protection
- End-to-end encryption for sensitive data
- GDPR and CCPA compliance
- Regular security audits
- Data retention policies enforced

### Best Practices
- Input validation using Zod
- Request logging with correlation IDs
- Error tracking with Sentry
- Performance monitoring with New Relic

### CORS Policy
```typescript
{
  origin: [
    'https://guest-link.app',
    /https:\/\/.*\.guest-link\.app$/
  ],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
}
```

---

## 6. Risks and Assumptions

### Assumptions
- WhatsApp Business API remains stable and accessible
- Users maintain stable internet connectivity
- Message delivery times under 30 seconds
- Cloud infrastructure provides 99.9% uptime

### Risks
1. **WhatsApp API Limitations**
   - Mitigation: Message queuing and rate limiting
   - Fallback to SMS or email for critical communications

2. **Data Privacy Compliance**
   - Mitigation: Regular audits and encrypted storage
   - Clear data handling documentation

3. **System Scalability**
   - Mitigation: Horizontal scaling and caching
   - Load testing and performance monitoring

4. **Real-time Communication**
   - Mitigation: Websocket connection pooling
   - Fallback to polling for degraded conditions

---

## 7. Revision History
- September 15, 2025 - v1.0 - Initial API specification document
- Future updates will be logged here

---

**Document Purpose Recap:** This API Specification Document provides a comprehensive blueprint for Guest Link's backend services, ensuring consistent implementation of features while maintaining security, scalability, and reliability. It serves as a crucial reference for development teams and integrating systems.