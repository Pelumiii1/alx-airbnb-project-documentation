# Backend Requirement Specifications

This document outlines the detailed technical and functional requirements for key backend features of the Airbnb Clone project.

## 1. User Authentication

This feature handles user registration, login, and profile management.

### 1.1. User Registration

*   **Description:** Allows a new user to create an account.
*   **API Endpoint:** `POST /api/v1/users/register`
*   **Input Specification:**
    ```json
    {
      "email": "user@example.com",
      "password": "securepassword123",
      "confirm_password": "securepassword123",
      "first_name": "John",
      "last_name": "Doe"
    }
    ```
*   **Output Specification (Success):**
    *   **Status Code:** `201 Created`
    *   **Body:**
        ```json
        {
          "user_id": "uuid-1234-abcd",
          "email": "user@example.com",
          "first_name": "John",
          "last_name": "Doe",
          "message": "User created successfully."
        }
        ```
*   **Output Specification (Error):**
    *   **Status Code:** `400 Bad Request`, `409 Conflict`
    *   **Body:**
        ```json
        {
          "error": "Description of the error (e.g., Passwords do not match, Email already exists)."
        }
        ```
*   **Validation Rules:**
    *   `email`: Must be a valid email format. Must be unique in the `Users` table.
    *   `password`: Minimum 8 characters, must contain at least one uppercase letter, one lowercase letter, one number, and one special character.
    *   `confirm_password`: Must match `password`.
    *   `first_name`, `last_name`: Required, string, max 50 characters.
*   **Performance Criteria:**
    *   The API response time should be less than 200ms under normal load.

### 1.2. User Login

*   **Description:** Authenticates a user and returns a JWT.
*   **API Endpoint:** `POST /api/v1/users/login`
*   **Input Specification:**
    ```json
    {
      "email": "user@example.com",
      "password": "securepassword123"
    }
    ```
*   **Output Specification (Success):**
    *   **Status Code:** `200 OK`
    *   **Body:**
        ```json
        {
          "access_token": "jwt.token.string",
          "token_type": "bearer"
        }
        ```
*   **Output Specification (Error):**
    *   **Status Code:** `401 Unauthorized`
    *   **Body:**
        ```json
        {
          "error": "Invalid credentials."
        }
        ```
*   **Validation Rules:**
    *   `email`: Must be a valid email format.
    *   `password`: Required.
*   **Performance Criteria:**
    *   The API response time should be less than 150ms under normal load.

---

## 2. Property Listings Management

This feature allows hosts to create, update, and manage their property listings.

### 2.1. Create Property Listing

*   **Description:** Allows an authenticated host to create a new property listing.
*   **API Endpoint:** `POST /api/v1/properties`
*   **Authentication:** Requires JWT Bearer token for a user with a 'host' role.
*   **Input Specification:**
    ```json
    {
      "title": "Cozy Downtown Apartment",
      "description": "A beautiful apartment in the heart of the city.",
      "location": "123 Main St, Anytown, USA",
      "price_per_night": 150.00,
      "amenities": ["Wi-Fi", "Pool", "Kitchen"],
      "max_guests": 4,
      "available_start_date": "2025-08-01",
      "available_end_date": "2025-12-31"
    }
    ```
*   **Output Specification (Success):**
    *   **Status Code:** `201 Created`
    *   **Body:**
        ```json
        {
          "property_id": "prop-uuid-5678",
          "host_id": "user-uuid-abcd",
          "title": "Cozy Downtown Apartment",
          "status": "active",
          "message": "Property listed successfully."
        }
        ```
*   **Output Specification (Error):**
    *   **Status Code:** `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`
    *   **Body:**
        ```json
        {
          "error": "Description of the validation or permission error."
        }
        ```
*   **Validation Rules:**
    *   `title`: Required, string, max 100 characters.
    *   `description`: Required, string.
    *   `location`: Required, string.
    *   `price_per_night`: Required, positive number.
    *   `max_guests`: Required, positive integer.
    *   `available_start_date`, `available_end_date`: Required, valid dates. `available_end_date` must be after `available_start_date`.
*   **Performance Criteria:**
    *   The API response time should be less than 300ms.
    *   Image uploads (if included) should be handled asynchronously.

### 2.2. Get Property Details

*   **Description:** Retrieves details for a specific property.
*   **API Endpoint:** `GET /api/v1/properties/{property_id}`
*   **Authentication:** Not required.
*   **Input Specification:** URL parameter `property_id`.
*   **Output Specification (Success):**
    *   **Status Code:** `200 OK`
    *   **Body:** (Full property details including host info and reviews)
        ```json
        {
          "property_id": "prop-uuid-5678",
          "title": "Cozy Downtown Apartment",
          "description": "A beautiful apartment in the heart of the city.",
          "location": "123 Main St, Anytown, USA",
          "price_per_night": 150.00,
          "amenities": ["Wi-Fi", "Pool", "Kitchen"],
          "max_guests": 4,
          "host": {
            "user_id": "user-uuid-abcd",
            "first_name": "John"
          },
          "reviews": [
            {
              "review_id": "rev-uuid-111",
              "rating": 5,
              "comment": "Amazing place!"
            }
          ]
        }
        ```
*   **Output Specification (Error):**
    *   **Status Code:** `404 Not Found`
*   **Performance Criteria:**
    *   Response time should be under 250ms, utilizing caching for frequently accessed properties.

---

## 3. Booking Management

This feature handles the creation and management of property bookings.

### 3.1. Create Booking

*   **Description:** Allows an authenticated guest to book a property for specific dates.
*   **API Endpoint:** `POST /api/v1/bookings`
*   **Authentication:** Requires JWT Bearer token for a user with a 'guest' role.
*   **Input Specification:**
    ```json
    {
      "property_id": "prop-uuid-5678",
      "start_date": "2025-09-10",
      "end_date": "2025-09-15",
      "number_of_guests": 2
    }
    ```
*   **Output Specification (Success):**
    *   **Status Code:** `201 Created`
    *   **Body:**
        ```json
        {
          "booking_id": "book-uuid-9101",
          "property_id": "prop-uuid-5678",
          "guest_id": "user-uuid-efgh",
          "status": "pending_payment",
          "total_price": 750.00,
          "message": "Booking created successfully. Please proceed to payment."
        }
        ```
*   **Output Specification (Error):**
    *   **Status Code:** `400 Bad Request` (e.g., dates not available), `401 Unauthorized`, `403 Forbidden`, `404 Not Found` (property not found).
    *   **Body:**
        ```json
        {
          "error": "Property is not available for the selected dates."
        }
        ```
*   **Validation Rules:**
    *   `property_id`: Must exist in the `Properties` table.
    *   `start_date`, `end_date`: Must be valid dates. `end_date` must be after `start_date`.
    *   The selected date range must be within the property's availability and must not overlap with existing bookings for that property.
    *   `number_of_guests`: Must be a positive integer and not exceed the property's `max_guests`.
*   **Performance Criteria:**
    *   The booking creation process, including date validation checks, should complete within 400ms.

### 3.2. Cancel Booking

*   **Description:** Allows a guest or host to cancel a booking.
*   **API Endpoint:** `PUT /api/v1/bookings/{booking_id}/cancel`
*   **Authentication:** Requires JWT Bearer token. The user must be the guest who made the booking or the host of the property.
*   **Input Specification:** URL parameter `booking_id`.
*   **Output Specification (Success):**
    *   **Status Code:** `200 OK`
    *   **Body:**
        ```json
        {
          "booking_id": "book-uuid-9101",
          "status": "canceled",
          "message": "Booking has been canceled successfully."
        }
        ```
*   **Output Specification (Error):**
    *   **Status Code:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.
*   **Validation Rules:**
    *   The booking must exist.
    *   The user must have permission to cancel (guest or host).
    *   Cancellation policies might apply (e.g., cannot cancel 24 hours before check-in). This logic should be checked.
*   **Performance Criteria:**
    *   Response time should be under 200ms.
