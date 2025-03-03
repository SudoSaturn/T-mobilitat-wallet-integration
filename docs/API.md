# Cartera TMobilitat - API Documentation

## Overview

This document outlines the API endpoints and data structures for the Cartera TMobilitat Apple Wallet integration. These APIs facilitate the creation, distribution, and management of TMobilitat passes in Apple Wallet.

## API Overview
This document details the REST APIs required for the iOS NFC Travel Pass application. All APIs use HTTPS and authenticate using JWT tokens.

## Base URL
```
Production: https://api.your-domain.com/v1
Staging: https://api-staging.your-domain.com/v1
Development: https://api-dev.your-domain.com/v1
```

## Authentication
All API requests must include an Authorization header:
```http
Authorization: Bearer <jwt_token>
```

## API Endpoints

### 1. Pass Management

#### Create Pass
```http
POST /passes
Content-Type: application/json
Authorization: Bearer <token>

Request:
{
    "passengerName": "string",
    "validFrom": "2024-03-15T00:00:00Z",
    "validUntil": "2024-04-15T23:59:59Z",
    "passType": "STANDARD",
    "nfcData": {
        "encryptionKey": "string",
        "message": "string"
    }
}

Response: 200 OK
{
    "passData": {
        "serialNumber": "string",
        "passTypeIdentifier": "string",
        "pkpassUrl": "string"
    }
}
```

#### Register Device
```http
POST /passes/register
Content-Type: application/json
Authorization: Bearer <token>

Request:
{
    "deviceLibraryIdentifier": "string",
    "passTypeIdentifier": "string",
    "serialNumber": "string",
    "pushToken": "string"
}

Response: 201 Created
{
    "status": "registered",
    "lastUpdated": "2024-03-15T12:00:00Z"
}
```

#### Get Pass Updates
```http
GET /passes/{passTypeIdentifier}/{serialNumber}
Authorization: Bearer <token>

Response: 200 OK
{
    "serialNumber": "string",
    "lastUpdated": "2024-03-15T12:00:00Z",
    "status": "active",
    "pkpassUrl": "string"
}
```

### 2. NFC Operations

#### Validate Pass
```http
POST /passes/{serialNumber}/validate
Content-Type: application/json
Authorization: Bearer <token>

Request:
{
    "nfcData": "string",
    "timestamp": "2024-03-15T12:00:00Z",
    "location": {
        "latitude": number,
        "longitude": number
    }
}

Response: 200 OK
{
    "isValid": boolean,
    "validationId": "string",
    "timestamp": "2024-03-15T12:00:00Z"
}
```

#### Get Validation History
```http
GET /passes/{serialNumber}/validations
Authorization: Bearer <token>

Response: 200 OK
{
    "validations": [
        {
            "validationId": "string",
            "timestamp": "2024-03-15T12:00:00Z",
            "location": {
                "latitude": number,
                "longitude": number
            },
            "status": "string"
        }
    ]
}
```

### 3. User Management

#### Register User
```http
POST /users
Content-Type: application/json

Request:
{
    "email": "string",
    "password": "string",
    "firstName": "string",
    "lastName": "string"
}

Response: 201 Created
{
    "userId": "string",
    "email": "string",
    "token": "string"
}
```

#### Login
```http
POST /auth/login
Content-Type: application/json

Request:
{
    "email": "string",
    "password": "string"
}

Response: 200 OK
{
    "token": "string",
    "refreshToken": "string",
    "expiresIn": number
}
```

### 4. Error Responses

#### 400 Bad Request
```json
{
    "error": {
        "code": "INVALID_REQUEST",
        "message": "The request was invalid",
        "details": {
            "field": "description of the error"
        }
    }
}
```

#### 401 Unauthorized
```json
{
    "error": {
        "code": "UNAUTHORIZED",
        "message": "Authentication failed"
    }
}
```

#### 404 Not Found
```json
{
    "error": {
        "code": "NOT_FOUND",
        "message": "Resource not found"
    }
}
```

## Rate Limiting

- Rate limit: 1000 requests per minute per API key
- Headers included in response:
  ```http
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 999
  X-RateLimit-Reset: 1615880000
  ```

## Webhooks

### Pass Update Notification
```http
POST {webhook_url}
Content-Type: application/json

{
    "event": "pass.updated",
    "passTypeIdentifier": "string",
    "serialNumber": "string",
    "timestamp": "2024-03-15T12:00:00Z"
}
```

### Validation Notification
```http
POST {webhook_url}
Content-Type: application/json

{
    "event": "pass.validated",
    "serialNumber": "string",
    "validationId": "string",
    "timestamp": "2024-03-15T12:00:00Z"
}
```

## SDK Examples

### Swift Implementation
```swift
import NFCPassKit

// Initialize the client
let client = NFCPassClient(
    apiKey: "YOUR_API_KEY",
    environment: .production
)

// Create a pass
client.createPass(
    passengerName: "John Doe",
    validFrom: Date(),
    validUntil: Date().addingTimeInterval(30 * 24 * 60 * 60)
) { result in
    switch result {
    case .success(let pass):
        print("Pass created: \(pass.serialNumber)")
    case .failure(let error):
        print("Error: \(error)")
    }
}
```

## Best Practices

1. **API Versioning**
   - Include version in URL path
   - Support at least one previous version
   - Deprecate with 6 months notice

2. **Security**
   - Use HTTPS for all endpoints
   - Implement rate limiting
   - Validate all input
   - Use secure headers

3. **Error Handling**
   - Return appropriate HTTP status codes
   - Include detailed error messages
   - Log all errors

4. **Performance**
   - Cache responses where appropriate
   - Compress responses
   - Implement pagination for large datasets

## Support

For API support:
- Email: api-support@your-domain.com
- Documentation: https://docs.your-domain.com
- Status Page: https://status.your-domain.com 