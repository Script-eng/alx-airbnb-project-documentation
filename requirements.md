# Airbnb Clone Backend: Requirement Specifications

This document provides detailed technical and functional requirements for key features of the Airbnb Clone backend. Each section outlines the purpose of a feature and specifies the API endpoints, data formats, validation rules, and performance expectations.

---

## 1. User Authentication

**Feature Overview:** This feature handles user registration, login, and session management. It ensures that only authorized users can access protected resources, differentiating between roles like 'guest' and 'host'. Authentication is managed via JSON Web Tokens (JWT).

### 1.1. Endpoint: User Registration

- **Description:** Allows a new user to create an account as either a 'guest' or a 'host'.
- **Endpoint:** `POST /api/auth/register`
- **Request Body (Input):**
  ```json
  {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "password": "a-strong-password-123",
    "role": "guest"
  }
  ```
- **Success Response (Output):**
  - **Code:** `201 Created`
  - **Body:**
    ```json
    {
      "message": "User registered successfully",
      "user": {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "email": "john.doe@example.com",
        "role": "guest"
      },
      "token": "ey...[JWT_TOKEN]..."
    }
    ```
- **Error Responses:**
  - `400 Bad Request`: If required fields are missing or data is malformed.
  - `409 Conflict`: If a user with the provided email already exists.
- **Validation Rules:**
  - `firstName`, `lastName`, `email`, `password`, `role` are all required.
  - `email` must be a valid email format and must be unique in the `Users` table.
  - `password` must be at least 8 characters long.
  - `role` must be either `'guest'` or `'host'`.
- **Performance Criteria:** The registration process should complete in under 200ms under normal load.

### 1.2. Endpoint: User Login

- **Description:** Authenticates a user with their email and password and returns a JWT for session management.
- **Endpoint:** `POST /api/auth/login`
- **Request Body (Input):**
  ```json
  {
    "email": "john.doe@example.com",
    "password": "a-strong-password-123"
  }
  ```
- **Success Response (Output):**
  - **Code:** `200 OK`
  - **Body:**
    ```json
    {
      "message": "Login successful",
      "token": "ey...[JWT_TOKEN]...",
      "user": {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "role": "host"
      }
    }
    ```
- **Error Responses:**
  - `400 Bad Request`: If `email` or `password` fields are missing.
  - `401 Unauthorized`: If the credentials do not match any user record.
- **Validation Rules:**
  - `email` and `password` fields are required.
- **Performance Criteria:** Login must be highly performant, with a response time of under 150ms.

---

## 2. Property Listings Management

**Feature Overview:** This feature enables authenticated hosts to create, update, view, and delete their property listings. Guests and unauthenticated users can view these listings.

### 2.1. Endpoint: Create a New Property Listing

- **Description:** Allows an authenticated 'host' to add a new property listing.
- **Endpoint:** `POST /api/properties`
- **Headers:**
  - `Authorization: Bearer <JWT_TOKEN>`
- **Request Body (Input):**
  ```json
  {
    "title": "Cozy Beachfront Cottage",
    "description": "A beautiful cottage right on the beach, perfect for a weekend getaway.",
    "location": "Malibu, CA, USA",
    "price": 250.00,
    "maxGuests": 4,
    "amenities": ["Wi-Fi", "Kitchen", "Free Parking"]
  }
  ```
- **Success Response (Output):**
  - **Code:** `201 Created`
  - **Body:** Returns the full JSON object of the newly created property, including the server-generated `id`.
    ```json
    {
      "id": "prop_a4b5c6d7",
      "title": "Cozy Beachfront Cottage",
      "description": "A beautiful cottage right on the beach, perfect for a weekend getaway.",
      "location": "Malibu, CA, USA",
      "price": 250.00,
      "maxGuests": 4,
      "amenities": ["Wi-Fi", "Kitchen", "Free Parking"],
      "hostId": "123e4567-e89b-12d3-a456-426614174000"
    }
    ```
- **Error Responses:**
  - `400 Bad Request`: For validation errors (e.g., missing title, negative price).
  - `401 Unauthorized`: If the JWT is missing or invalid.
  - `403 Forbidden`: If the authenticated user's role is not 'host'.
- **Validation Rules:**
  - `title` and `description` cannot be empty.
  - `price` and `maxGuests` must be positive numbers.
  - The user must be authenticated and have the role of 'host'.
- **Performance Criteria:** Response time should be under 300ms.

---

## 3. Booking System

**Feature Overview:** This feature allows authenticated users to book properties for specific dates. It includes logic to prevent double-bookings and manages the lifecycle of a booking (e.g., pending, confirmed, cancelled).

### 3.1. Endpoint: Create a Booking

- **Description:** Allows an authenticated user (typically a 'guest') to book a property.
- **Endpoint:** `POST /api/bookings`
- **Headers:**
  - `Authorization: Bearer <JWT_TOKEN>`
- **Request Body (Input):**
  ```json
  {
    "propertyId": "prop_a4b5c6d7",
    "startDate": "2024-12-20",
    "endDate": "2024-12-25"
  }
  ```
- **Success Response (Output):**
  - **Code:** `201 Created`
  - **Body:** Returns the full booking object, initially with a `pending` or `confirmed` status, depending on the payment flow.
    ```json
    {
      "id": "booking_z9y8x7w6",
      "propertyId": "prop_a4b5c6d7",
      "guestId": "user_guest_abc123",
      "startDate": "2024-12-20",
      "endDate": "2024-12-25",
      "totalPrice": 1250.00,
      "status": "pending"
    }
    ```
- **Error Responses:**
  - `400 Bad Request`: If input is invalid (e.g., `endDate` is before `startDate`).
  - `401 Unauthorized`: If the user is not authenticated.
  - `404 Not Found`: If the `propertyId` does not exist.
  - `409 Conflict`: If the property is already booked for any of the requested dates.
- **Validation Rules:**
  - `propertyId` must correspond to an existing property.
  - `startDate` and `endDate` must be valid dates. `startDate` must be before `endDate`.
  - The system must check the `Bookings` table to ensure no other confirmed booking for the same property overlaps with the requested date range.
- **Performance Criteria:** As this involves multiple database lookups (property existence, booking conflicts), a response time of under 400ms is acceptable.
```