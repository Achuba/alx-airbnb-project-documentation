# Airbnb Clone Backend — Technical and Functional Requirements

## Objective
This document defines the technical and functional requirements for the Airbnb Clone backend system.  
It focuses on three core features: **User Authentication**, **Property Management**, and **Booking System**.

---

## 1️⃣ User Authentication

### **Functional Requirements**
- Allow new users to register using first name, last name, email, and password.
- Allow existing users to log in and receive an authentication token (JWT).
- Securely store passwords using hashing (e.g., bcrypt).
- Enable password recovery via email verification.
- Restrict access to authenticated users for certain API endpoints.

### **API Endpoints**
| Method | Endpoint | Description |
|---------|-----------|-------------|
| `POST` | `/api/v1/auth/register` | Register a new user |
| `POST` | `/api/v1/auth/login` | Authenticate a user and return a JWT token |
| `POST` | `/api/v1/auth/logout` | Invalidate current user session |
| `POST` | `/api/v1/auth/forgot-password` | Request password reset |
| `POST` | `/api/v1/auth/reset-password` | Reset password using token |

### **Input/Output Specifications**
**Register Request:**
```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "StrongPass123!"
}

## Validation Rules
  ~ email must be unique and valid.
  ~ password must be ≥ 8 characters, include letters and numbers.
  ~ first_name and last_name cannot be empty.

## Performance Criteria
  ~ Registration and login should complete within 300ms.
  ~ Authentication tokens expire after 24 hours.
  ~ Rate limit login attempts to 5 per minute per IP.


2️⃣ Property Management Requirements

## Functional Requirements
The Property Management module should allow:
1. **Hosts** to:
   - Create, edit, and delete property listings.
   - Upload and manage property photos.
   - Set availability, price per night, and amenities.
2. **Guests** to:
   - Search for properties based on filters (location, price range, date range, amenities).
   - View detailed property information.
3. **Admins** to:
   - Review, verify, or deactivate property listings if needed.

---

## API Endpoints

| Method | Endpoint | Access | Description |
|---------|-----------|--------|-------------|
| `POST` | `/api/v1/properties` | Host | Create a new property listing |
| `GET` | `/api/v1/properties` | Public | Retrieve all available properties |
| `GET` | `/api/v1/properties/:id` | Public | Get detailed information for a single property |
| `PUT` | `/api/v1/properties/:id` | Host | Update an existing property |
| `DELETE` | `/api/v1/properties/:id` | Host/Admin | Remove or deactivate a property listing |
| `GET` | `/api/v1/properties/search` | Public | Search properties by filters (location, price, amenities) |

---

## Input/Output Specifications

### **Create Property Request**
```json
{
  "name": "Cozy Apartment in Berlin",
  "description": "A beautiful modern apartment near city center.",
  "location": "Berlin, Germany",
  "price_per_night": 120.50,
  "amenities": ["WiFi", "Kitchen", "Parking"],
  "images": ["image1.jpg", "image2.jpg"]
}

Create Property Response:
{
  "message": "Property created successfully",
  "property_id": "uuid-67890",
  "host_id": "uuid-12345",
  "created_at": "2025-10-25T12:00:00Z"
}

## Validation Rules
  ~ name, description, and location are required.
  ~ price_per_night must be a positive number.
  ~ amenities must be an array of strings.
  ~ images must be valid URLs or file paths.
  ~ Only users with the role host can create or modify properties.

## Performance Criteria
  ~ Property listings retrieval should complete within 500ms for ≤ 1000 records.
  ~ Use database indexing on location, price_per_night, and host_id for optimized queries.
  ~ Image uploads limited to 5 MB per file and stored using cloud storage (e.g., AWS S3).

## Security Considerations
  ~ Only authenticated hosts can modify or delete their listings.
  ~ Admins can flag or remove fraudulent or inappropriate listings.
  ~ Validate and sanitize all text inputs to prevent XSS or SQL injection.
  ~ Rate-limit property creation to prevent spam (e.g., max 10 per minute).



3️⃣ Booking System Requirements


---

## Functional Requirements
The Booking System must:
1. Allow guests to:
   - Create new bookings for available properties.
   - View, modify, or cancel their bookings.
2. Allow hosts to:
   - View bookings related to their properties.
   - Approve or manage booking statuses if applicable.
3. Prevent double-booking for the same property and date range.
4. Calculate the total cost based on property price per night and stay duration.
5. Record booking timestamps and maintain booking history.

---

## API Endpoints

| Method | Endpoint | Access | Description |
|---------|-----------|--------|-------------|
| `POST` | `/api/v1/bookings` | Guest | Create a new booking |
| `GET` | `/api/v1/bookings` | Authenticated User | Retrieve all bookings for the user |
| `GET` | `/api/v1/bookings/:id` | Authenticated User | Get details of a single booking |
| `PUT` | `/api/v1/bookings/:id` | Guest | Update booking details (e.g., change dates) |
| `DELETE` | `/api/v1/bookings/:id` | Guest | Cancel an existing booking |
| `GET` | `/api/v1/host/bookings` | Host | View all bookings for the host’s properties |

---

## Input/Output Specifications

### **Create Booking Request**
```json
{
  "property_id": "uuid-67890",
  "start_date": "2025-07-10",
  "end_date": "2025-07-15"
}

Create Booking Response
{
  "message": "Booking created successfully",
  "booking_id": "uuid-abc123",
  "property_id": "uuid-67890",
  "user_id": "uuid-12345",
  "total_price": 602.50,
  "status": "confirmed",
  "created_at": "2025-10-25T12:00:00Z"
}

## Business Logic
1. Booking Creation:
    ~ System verifies property availability for the date range.
    ~ Calculates total_price = price_per_night × number_of_nights.
    ~ Stores booking record and returns confirmation.

2. Booking Update:
  ~ Guests can modify dates if the new range is available.
  ~ System recalculates total price.

3. Booking Cancellation:
  ~ Guests can cancel up to a defined cutoff time (e.g., 24 hours before check-in).
  ~ If canceled, update status to "canceled".

4. Host Access:
  ~ Hosts can view bookings related to properties they own.

## Validation Rules
  ~ start_date must be before end_date.
  ~ Booking duration must be at least 1 day.
  ~ Property must exist and be available during the requested dates.
  ~ user_id and property_id must be valid UUIDs.
  ~ Prevent overlapping bookings for the same property.

## Performance Criteria
    ~ Booking creation should complete within 400ms.
    ~ Concurrent bookings for the same property must maintain data consistency (use database transactions or locks).
    ~ Index property_id, start_date, and end_date for faster availability checks.

## Security Considerations
    ~ Only authenticated guests can create bookings.
    ~ Guests can only view or modify their own bookings.
    ~ Hosts can only access bookings linked to their own properties.
    ~ Input data must be validated and sanitized to prevent SQL injection and logic tampering.


