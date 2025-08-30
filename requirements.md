# Backend System Requirement Specification
This document outlines the technical specifications for the backend services on three key features: User Authentication, Property Management, and the Booking System. It defines API endpoints, data schemas, validation rules, and performance criteria.
## 1: User Authentication
### API Endpoints
| Method | Endpoint               | Description          | Auth Required |
| ------ | :--------:             | :-----------:        | :-----------: |
| POST   | /api/v1/auth/register  |  Register a new user |  No           |
| POST   |  /api/v1/auth/login    |  Authenticate a user | No            |
| GET    |  /api/v1/auth/me       |  Get current user profile | Yes      |

### Input/Output Specifications
#### POST /api/v1/properties
 - GOAL: A host creates a new property listing.
 - INPUT DATA : Request Body
   ```
    {
     "title": "Beautiful Beach House",
     "description": "A lovely house by the beach...",
     "location": "Miami, FL",
     "pricePerNight": 150.50,
     "bedroomCount": 2,
     "bathroomCount": 1,
     "maxGuests": 4,
     "amenities": ["wifi", "pool", "pet-friendly"]
    }
   ```
 - OUTPUT DATA :Success Response (201 Created)
   ```
    {
      "message": "Property created successfully",
      "propertyId": "prop_abc123def456"
    }
   ```
 - Error Responses:
   - 400 Bad Request: Validation errors
   - 403 Forbidden: User is not a host.
     
## 2: Property Management
### API Endpoints
| Method | Endpoint               | Description          | Auth Required | Role |
| ------ | :--------:             | :-----------:        | :-----------: | :-----: |
| POST   | /api/v1/properties  |  Get paginated list of properties |  Yes   | Host     |
| GET   |  /api/v1/properties    |  Authenticate a user | No           |              |
| GET    |  /api/v1/properties/:id |  Update a property listing | Yes     | Host (Owner) |
| DELETE |  /api/v1/properties/:id |  Delete a property listing | Yes    |   Host (Owner) |
### Input/Output Specifications
#### POST /api/v1/properties
 - GOAL: A host creates a new property listing.
 - Purpose: A host creates a new property listing.
 - INPUT DATA : Request Body
   ```
    {
     "title": "Beautiful Beach House",
     "description": "A lovely house by the beach...",
     "location": "Miami, FL",
     "pricePerNight": 150.50,
     "bedroomCount": 2,
     "bathroomCount": 1,
     "maxGuests": 4,
     "amenities": ["wifi", "pool", "pet-friendly"]
    }
   ```
 - OUTPUT DATA :Success Response (201 Created)
   ```
    {
    "message": "Property created successfully",
    "propertyId": "prop_abc123def456"
    }
   ```
 - Validation Rules
   - Title: Required, max length 100 characters.
   - Description: Required, max length 1000 characters.
   - Price: Required, positive number, with a maximum value (e.g., 10,000).
   - Numeric Fields (beds, baths, guests): Required, positive integers.
   - Authorization: Users can only update/delete their own properties.
- Performance Criteria
   - Latency: 99% of GET /properties requests should complete in < 200ms, even with pagination and filters.
   - Cache: Property listing search results should be cached in Redis with a 1-minute TTL to handle high read volume.
   - Throughput: The service should handle 5000 read requests per minute.

## 3: Booking System
### API Endpoints
| Method | Endpoint               | Description          | Auth Required | Role |
| ------ | :--------:             | :-----------:        | :-----------: | :----: |
| POST   | /api/v1/bookings  |  Create a new booking |  Yes          | Guest|
| POST   |  /api/v1/auth/login    |  Authenticate a user | No            |Guest/Host |
| GET    |  /api/v1/auth/me       |  Get current user profile | Yes      |Guest/Host (Owner) |
| PATCH    |  /api/v1/bookings/:id |  Cancel a booking | Yes      |Guest/Host (Owner)) |

### Input/Output Specifications
#### PATCH /api/v1/bookings/:id
 - GOAL: Cancel a booking. Updates status to cancelled.
 - INPUT DATA : Request Body
    ```
    {
     "status": "cancelled"
    }
    ```
 - OUTPUT DATA :Success Response (200 OK)
    ```
    {
     "message": "Booking cancelled successfully.",
     "refundAmount": 451.50 // Calculated based on policy
    }
    ```
 - Validation & Business Logic
   - Date Validation: checkInDate must be in the future. checkOutDate must be after checkInDate
   - Availability Check: Atomic operation is critical. Must check for existing bookings that conflict with the requested dates within a database transaction to prevent race conditions.
   - Authorization: Guests can only book and see their own bookings. Hosts can only see bookings for their properties.
   - Cancellation Policy: Logic must calculate refund amount based on the host's policy (e.g., "50% refund if cancelled 7 days before check-in").
 - Performance Criteria
   - Latency: 99% of POST /bookings requests (including availability check) complete in < 300ms.
   - Concurrency: The booking creation endpoint must handle concurrent requests for the same property/dates without creating double bookings (using pessimistic locking or optimistic concurrency control on the database level).
   - Data Consistency: Strong consistency is required for booking data. The availability check and booking creation must be wrapped in a database transaction.
