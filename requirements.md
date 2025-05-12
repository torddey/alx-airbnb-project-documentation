# Airbnb Clone Project - Backend Feature Requirements


## Overview

This document outlines the detailed backend requirements for three core features of the Airbnb Clone Project. It includes API endpoint definitions, input/output specs, validation rules, and performance criteria. These features are implemented using Django, Django REST Framework, PostgreSQL, Celery, and Redis.

---

## 1. User Authentication

### Features

* User registration
* Secure login/logout with token-based authentication
* OTP or email verification (optional)
* Profile retrieval and update

### API Endpoints

#### `POST /api/v1/auth/register/`

**Description**: Registers a new user
**Input**:

```json
{
  "email": "user@example.com",
  "password": "StrongP@ssword123",
  "full_name": "John Doe"
}
```

**Output**:

```json
{
  "message": "Registration successful. Please verify your email."
}
```

**Validation**:

* Email must be unique and valid
* Password must be ≥ 8 characters, contain a number and a special character
* Full name is required

#### `POST /api/v1/auth/login/`

**Description**: Authenticates the user and returns JWT tokens
**Input**:

```json
{
  "email": "user@example.com",
  "password": "StrongP@ssword123"
}
```

**Output**:

```json
{
  "access_token": "jwt_access_token",
  "refresh_token": "jwt_refresh_token"
}
```

**Validation**:

* Valid email/password combination
* Rate limiting: 5 attempts per 15 minutes per IP

#### `GET /api/v1/auth/profile/`

**Description**: Retrieves authenticated user profile
**Authorization**: Bearer token required
**Output**:

```json
{
  "id": 1,
  "email": "user@example.com",
  "full_name": "John Doe",
  "created_at": "2025-05-01T12:00:00Z"
}
```

---

## 2. Property Management

### Features

* Create, update, retrieve, and delete property listings
* Upload multiple images per property
* Each property linked to a user (host)

### API Endpoints

#### `POST /api/v1/properties/`

**Description**: Creates a new property listing
**Input**:

```json
{
  "title": "Cozy Cabin in the Woods",
  "description": "A peaceful cabin for 2 guests.",
  "price_per_night": 120.00,
  "address": "123 Forest Lane, Asheville, NC",
  "amenities": ["wifi", "parking", "fireplace"]
}
```

**Output**:

```json
{
  "id": 101,
  "message": "Property created successfully"
}
```

**Validation**:

* Title, price, and address required
* Price must be a positive decimal
* Max 5 images per property

#### `GET /api/v1/properties/<id>/`

**Description**: Retrieves property details
**Output**:

```json
{
  "id": 101,
  "title": "Cozy Cabin in the Woods",
  "description": "A peaceful cabin for 2 guests.",
  "host": {
    "id": 1,
    "full_name": "John Doe"
  },
  "price_per_night": 120.00,
  "reviews_count": 12,
  "average_rating": 4.8
}
```

#### `PATCH /api/v1/properties/<id>/`

**Description**: Updates a property listing
**Authorization**: Must be property owner
**Input**:

```json
{
  "price_per_night": 150.00
}
```

**Output**:

```json
{
  "message": "Property updated successfully"
}
```

---

## 3. Booking System

### Features

* Book a property for specific dates
* Validate availability before booking
* Cancel bookings

### API Endpoints

#### `POST /api/v1/bookings/`

**Description**: Creates a booking for a property
**Input**:

```json
{
  "property_id": 101,
  "check_in": "2025-06-01",
  "check_out": "2025-06-05"
}
```

**Output**:

```json
{
  "booking_id": 501,
  "status": "confirmed",
  "total_price": 480.00
}
```

**Validation**:

* Dates must not overlap with existing bookings
* Check-out must be after check-in
* Minimum stay rules (e.g., 2 nights) can be configured

#### `GET /api/v1/bookings/<id>/`

**Description**: Retrieves booking details
**Authorization**: Must be the user who made the booking
**Output**:

```json
{
  "id": 501,
  "property": {
    "id": 101,
    "title": "Cozy Cabin"
  },
  "user": {
    "id": 2,
    "email": "guest@example.com"
  },
  "check_in": "2025-06-01",
  "check_out": "2025-06-05",
  "status": "confirmed",
  "total_price": 480.00
}
```

#### `DELETE /api/v1/bookings/<id>/`

**Description**: Cancels a booking
**Output**:

```json
{
  "message": "Booking cancelled successfully"
}
```

---

## Performance Criteria

### General

* All endpoints must respond within **300ms** under typical load
* Pagination and filtering on list endpoints
* Caching popular properties and profiles with **Redis**
* Asynchronous task queue (Celery) for non-blocking operations like sending emails

### User Auth

* Use **Redis** to manage session tokens and blacklist invalid refresh tokens

### Bookings

* Use **PostgreSQL locks or transaction isolation** to prevent race conditions on booking overlaps

---

## Security & Compliance

* JWT Authentication for all protected endpoints
* Rate limiting, input sanitization, and validation enforced
* GDPR-compliant user data handling
* HTTPS enforced in production

---
