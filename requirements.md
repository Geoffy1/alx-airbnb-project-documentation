# Backend Requirement Specifications - Airbnb Clone

This document details the technical and functional requirements for key backend features of the Airbnb Clone project. These specifications serve as a blueprint for development, ensuring that each feature meets defined criteria for functionality, performance, and security.

## 1. User Authentication

### 1.1 User Registration (`POST /api/v1/auth/register`)

* **Objective:** Allow new users (Guests or Hosts) to create an account.
* **Method:** `POST`
* **Endpoint:** `/api/v1/auth/register`
* **Request Body (Input):**
    ```json
    {
      "first_name": "string",
      "last_name": "string",
      "email": "string (valid email format)",
      "password": "string (min 8 characters, strong password recommended)",
      "phone_number": "string (optional)",
      "role": "enum (guest, host)"
    }
    ```
* **Validation Rules:**
    * `first_name`, `last_name`, `email`, `password`, `role` are mandatory.
    * `email` must be unique and in a valid email format.
    * `password` must be at least 8 characters long and meet complexity requirements (e.g., contains uppercase, lowercase, number, special character).
    * `role` must be either 'guest' or 'host'.
* **Response (Success - 201 Created):**
    ```json
    {
      "message": "User registered successfully",
      "user_id": "UUID",
      "email": "string",
      "role": "string",
      "token": "string (JWT)"
    }
    ```
* **Response (Error - 400 Bad Request / 409 Conflict):**
    * `400 Bad Request`: Invalid input (e.g., missing fields, invalid email format, weak password).
    * `409 Conflict`: Email already registered.
* **Performance Criteria:** Response time < 200ms.

### 1.2 User Login (`POST /api/v1/auth/login`)

* **Objective:** Authenticate existing users and provide an access token.
* **Method:** `POST`
* **Endpoint:** `/api/v1/auth/login`
* **Request Body (Input):**
    ```json
    {
      "email": "string",
      "password": "string"
    }
    ```
* **Validation Rules:**
    * `email` and `password` are mandatory.
* **Response (Success - 200 OK):**
    ```json
    {
      "message": "Login successful",
      "user_id": "UUID",
      "email": "string",
      "role": "string",
      "token": "string (JWT)"
    }
    ```
* **Response (Error - 401 Unauthorized / 400 Bad Request):**
    * `401 Unauthorized`: Invalid email or password.
    * `400 Bad Request`: Missing credentials.
* **Performance Criteria:** Response time < 150ms.

## 2. Property Management (Host-specific)

### 2.1 Add New Property Listing (`POST /api/v1/listings`)

* **Objective:** Allow authenticated Hosts to create a new property listing.
* **Method:** `POST`
* **Endpoint:** `/api/v1/listings`
* **Authorization:** Requires valid Host JWT token.
* **Request Body (Input):**
    ```json
    {
      "name": "string (property title)",
      "description": "string",
      "location": "string (e.g., city, neighborhood)",
      "price_per_night": "decimal (e.g., 99.99)",
      "number_of_guests": "integer (max guests)",
      "amenities": ["string", "string", ...], // Array of amenity names
      "image_urls": ["string (URL)", "string (URL)", ...] // Array of image URLs
    }
    ```
* **Validation Rules:**
    * `name`, `description`, `location`, `price_per_night`, `number_of_guests` are mandatory.
    * `price_per_night` must be a positive decimal.
    * `number_of_guests` must be a positive integer.
    * User role must be 'host'.
* **Response (Success - 201 Created):**
    ```json
    {
      "message": "Property listing created successfully",
      "listing_id": "UUID",
      "name": "string",
      "host_id": "UUID"
    }
    ```
* **Response (Error - 400 Bad Request / 401 Unauthorized / 403 Forbidden):**
    * `400 Bad Request`: Invalid input (e.g., missing fields, invalid price).
    * `401 Unauthorized`: No token provided or invalid token.
    * `403 Forbidden`: User is not a 'host'.
* **Performance Criteria:** Response time < 300ms.

### 2.2 Retrieve Property Listing Details (`GET /api/v1/listings/{listing_id}`)

* **Objective:** Retrieve details of a specific property listing.
* **Method:** `GET`
* **Endpoint:** `/api/v1/listings/{listing_id}`
* **Authorization:** Optional (public access for viewing, but can be authenticated for specific actions).
* **Parameters:** `listing_id` (UUID) in path.
* **Validation Rules:**
    * `listing_id` must be a valid UUID.
* **Response (Success - 200 OK):**
    ```json
    {
      "listing_id": "UUID",
      "host_id": "UUID",
      "name": "string",
      "description": "string",
      "location": "string",
      "price_per_night": "decimal",
      "number_of_guests": "integer",
      "amenities": ["string", ...],
      "image_urls": ["string (URL)", ...],
      "average_rating": "decimal (optional)",
      "number_of_reviews": "integer (optional)",
      "created_at": "timestamp",
      "updated_at": "timestamp"
    }
    ```
* **Response (Error - 404 Not Found / 400 Bad Request):**
    * `404 Not Found`: Listing with `listing_id` does not exist.
    * `400 Bad Request`: Invalid `listing_id` format.
* **Performance Criteria:** Response time < 100ms.

## 3. Booking System

### 3.1 Create New Booking (`POST /api/v1/bookings`)

* **Objective:** Allow authenticated Guests to create a booking for an available property.
* **Method:** `POST`
* **Endpoint:** `/api/v1/bookings`
* **Authorization:** Requires valid Guest JWT token.
* **Request Body (Input):**
    ```json
    {
      "property_id": "UUID",
      "check_in_date": "string (YYYY-MM-DD)",
      "check_out_date": "string (YYYY-MM-DD)",
      "number_of_guests": "integer"
    }
    ```
* **Validation Rules:**
    * `property_id`, `check_in_date`, `check_out_date`, `number_of_guests` are mandatory.
    * `check_in_date` and `check_out_date` must be valid dates and `check_in_date` must be before `check_out_date`.
    * `check_in_date` must be in the future.
    * `number_of_guests` must be a positive integer and not exceed the property's maximum guest capacity.
    * The requested dates must not overlap with any existing confirmed bookings for that property.
    * User role must be 'guest'.
* **Response (Success - 201 Created):**
    ```json
    {
      "message": "Booking created successfully, awaiting payment",
      "booking_id": "UUID",
      "property_id": "UUID",
      "user_id": "UUID",
      "total_price": "decimal",
      "status": "pending"
    }
    ```
* **Response (Error - 400 Bad Request / 401 Unauthorized / 403 Forbidden / 409 Conflict):**
    * `400 Bad Request`: Invalid input (e.g., dates, guest count).
    * `401 Unauthorized`: No token provided or invalid token.
    * `403 Forbidden`: User is not a 'guest'.
    * `409 Conflict`: Dates are not available or exceed property capacity.
* **Performance Criteria:** Response time < 400ms (may involve multiple DB lookups).

### 3.2 Cancel Booking (`PUT /api/v1/bookings/{booking_id}/cancel`)

* **Objective:** Allow an authenticated Guest or Host to cancel a specific booking.
* **Method:** `PUT`
* **Endpoint:** `/api/v1/bookings/{booking_id}/cancel`
* **Authorization:** Requires valid JWT token (Guest who made booking or Host of property).
* **Parameters:** `booking_id` (UUID) in path.
* **Validation Rules:**
    * `booking_id` must be a valid UUID.
    * The booking must not already be in a 'completed' or 'canceled' state.
    * User must be either the `user_id` of the booking or the `host_id` of the associated property.
    * Cancellation policy checks (e.g., within X days of check-in) should be applied and reflected in response.
* **Response (Success - 200 OK):**
    ```json
    {
      "message": "Booking cancelled successfully",
      "booking_id": "UUID",
      "status": "canceled",
      "refund_amount": "decimal (if applicable)"
    }
    ```
* **Response (Error - 400 Bad Request / 401 Unauthorized / 403 Forbidden / 404 Not Found / 409 Conflict):**
    * `400 Bad Request`: Invalid `booking_id` format.
    * `401 Unauthorized`: No token or invalid token.
    * `403 Forbidden`: User is not authorized to cancel this booking.
    * `404 Not Found`: Booking with `booking_id` does not exist.
    * `409 Conflict`: Booking cannot be cancelled (e.g., already completed, past cancellation deadline).
* **Performance Criteria:** Response time < 250ms.

---

