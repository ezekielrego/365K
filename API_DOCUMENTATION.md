# Kurima API Documentation

## Getting Started

### 1. Authentication
- **Superusers can generate API tokens via the web interface.**
- **Regular users can register and log in via the API using email and password.**
- Log in at `/home/` (superuser) to view and copy your API token, or use the `/api/login/` endpoint (see below).
- Use the token in the `Authorization` header for all API requests:
  ```
  Authorization: Bearer <your_jwt_token>
  ```

### 2. API Base URL
- Local: `http://127.0.0.1:8000/api/`
- Production: `https://kurimaapi.bhiza.co.zw/api/`

---

## Endpoints Overview
All endpoints require authentication via API token or JWT unless otherwise noted.

### Registration
- `POST /api/register/` — Register a new user
  - **Request Body:**
    ```json
    {
      "name": "Test User",
      "email": "testuser@example.com",
      "password": "testpassword123",
      "role": "farmer"
    }
    ```
  - **Response:**
    ```json
    {
      "id": 2,
      "name": "Test User",
      "email": "testuser@example.com",
      "role": "farmer",
      "notifications_on": true,
      ...
    }
    ```
  - **Error (duplicate email):**
    ```json
    { "email": ["A user with this email already exists."] }
    ```
  - **Error (missing field):**
    ```json
    { "password": ["This field is required."] }
    ```

### Login (JWT)
- `POST /api/login/` — Obtain JWT access and refresh tokens
  - **Request Body:**
    ```json
    {
      "email": "user@example.com",
      "password": "yourpassword"
    }
    ```
  - **Response:**
    ```json
    {
      "refresh": "<refresh_token>",
      "access": "<access_token>"
    }
    ```
  - **Error (invalid credentials):**
    ```json
    { "error": "Invalid credentials" }
    ```
  - **Error (no user found):**
    ```json
    { "error": "No user found with this email" }
    ```

### Notifications
- `GET /api/notifications/` — List all notifications (admin only)
  - **Response:**
    ```json
    [
      {
        "id": 1,
        "type": "info",
        "message": "Welcome to Kurima!",
        "created_at": "2024-06-01T12:00:00Z",
        "users": [2],
        "is_active": true
      }
    ]
    ```
- `POST /api/notifications/` — Create a notification (admin only)
  - **Request Body:**
    ```json
    {
      "type": "info",
      "message": "System maintenance at midnight.",
      "users": [2, 3]
    }
    ```
  - **Response:**
    ```json
    {
      "id": 2,
      "type": "info",
      "message": "System maintenance at midnight.",
      "users": [2, 3],
      "is_active": true
    }
    ```
- `GET /notifications/pull/` — Pull (fetch) active notifications for the authenticated user
  - **Response:**
    ```json
    [
      {
        "id": 1,
        "type": "info",
        "message": "Welcome to Kurima!",
        "created_at": "2024-06-01T12:00:00Z",
        "is_active": true
      }
    ]
    ```
- `POST /notifications/push/` — Push a notification to the authenticated user
  - **Request Body:**
    ```json
    { "notification_id": 1 }
    ```
  - **Response:**
    ```json
    { "status": "Notification pushed to user" }
    ```
  - **Error (not found):**
    ```json
    { "error": "Notification not found" }
    ```

**Notification Fields:**
- `id`, `type` (info, warning, success, error), `message`, `created_at`, `users`, `is_active`

### Users
- `GET /api/users/` — List users
- `POST /api/users/` — Create user (admin only)
- `GET /api/users/{id}/` — Retrieve user
- `PUT/PATCH /api/users/{id}/` — Update user
- `DELETE /api/users/{id}/` — Delete user

**User Fields:**
- `id`, `name`, `email`, `profile_photo`, `email_verified_at`, `kyc_verified`, `password`, `role`, `region`, `phone`, `location`, `created_at`, `updated_at`, `wallet_balance`, `kyc_verified_at`, `training_points`, `notifications_on`
- **Foreign Key:**
  - `region`: ID of a Region object. See Region model.

### Product Categories
- `GET /api/categories/` — List categories
- `POST /api/categories/` — Create category
- ...

**Fields:** `id`, `name`, `description`, `created_at`, `updated_at`

### Products
- `GET /api/products/` — List products
- `POST /api/products/` — Create product
- ...

**Fields:** `id`, `category`, `user`, `name`, `slug`, `description`, `price`, `quantity`, `unit`, `harvest_date`, `quantity_available`, `status`, `is_organic`, `is_featured`, `type`, `image`, `video`, `location`, `created_at`, `updated_at`, `is_opportunity`, `sold_count`
- **Foreign Keys:**
  - `category`: ID of a ProductCategory object. See ProductCategory model.
  - `user`: ID of a User object (the product owner).

### Farms
- `GET /api/farms/` — List farms
- `POST /api/farms/` — Create farm
- ...

**Fields:** `id`, `status`, `user`, `name`, `description`, `location`, `latitude`, `longitude`, `image`, `is_active`, `created_at`, `updated_at`
- **Foreign Key:**
  - `user`: ID of a User object (the farm owner).

### Orders
- `GET /api/orders/` — List orders
- `POST /api/orders/` — Create order
- ...

**Fields:** `id`, `user`, `buyer`, `product`, `quantity`, `total_price`, `status`, `payment_status`, `created_at`, `updated_at`, `farmer`
- **Foreign Keys:**
  - `user`: ID of a User object (the user placing the order; can be null).
  - `buyer`: ID of a User object (the buyer).
  - `product`: ID of a Product object.
  - `farmer`: ID of a User object (the farmer; can be null).

### Chat & Messaging
- `GET /api/chatrooms/` — List chat rooms
- `POST /api/chatrooms/` — Create a chat room
- `GET /api/chatrooms/{id}/` — Retrieve a chat room
- `GET /api/messages/` — List messages
- `POST /api/messages/` — Send a message
- `GET /api/messages/{id}/` — Retrieve a message

**ChatRoom Fields:**
- `id`, `name`, `users`, `created_at`, `updated_at`

**Message Fields:**
- `id`, `room`, `sender`, `content`, `created_at`, `is_read`

### Product Negotiations
- `GET /api/negotiations/` — List negotiations
- `POST /api/negotiations/` — Propose a price for a product
- `GET /api/negotiations/{id}/` — Retrieve negotiation details
- `POST /negotiations/{id}/accept/` — Accept a price offer
- `POST /negotiations/{id}/decline/` — Decline a price offer

**ProductNegotiation Fields:**
- `id`, `product`, `buyer`, `proposed_price`, `status` (pending, accepted, declined), `created_at`, `updated_at`

**Negotiation Flow:**
- Buyer proposes a price for a product (creates a negotiation, status=pending)
- Product owner can view negotiations
- Buyer can accept or decline offers via the accept/decline endpoints
- All negotiation history is saved

---

## Example Usage

### 1. Register a New User
```bash
curl -X POST http://127.0.0.1:8000/api/register/ \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "testuser@example.com",
    "password": "testpassword123",
    "role": "farmer"
  }'
```

### 2. Obtain JWT Token (Login)
```bash
curl -X POST http://127.0.0.1:8000/api/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "yourpassword"
  }'
```

### 3. Make an Authenticated Request
```bash
curl -H "Authorization: Bearer <your_jwt_token>" http://127.0.0.1:8000/api/products/
```

### 4. Pull Notifications
```bash
curl -H "Authorization: Bearer <your_jwt_token>" http://127.0.0.1:8000/notifications/pull/
```

### 5. Push Notification to User
```bash
curl -X POST http://127.0.0.1:8000/notifications/push/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{ "notification_id": 1 }'
```

### 6. Create a Chat Room
```bash
curl -X POST http://127.0.0.1:8000/api/chatrooms/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Negotiation Room",
    "users": [2, 3]
  }'
```

### 7. Send a Message
```bash
curl -X POST http://127.0.0.1:8000/api/messages/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "room": 1,
    "sender": 2,
    "content": "Hello, I'm interested in your product."
  }'
```

### 8. Propose a Negotiation
```bash
curl -X POST http://127.0.0.1:8000/api/negotiations/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "product": 1,
    "buyer": 2,
    "proposed_price": "8.50"
  }'
```

### 9. Accept a Negotiation
```bash
curl -X POST http://127.0.0.1:8000/negotiations/1/accept/ \
  -H "Authorization: Bearer <your_jwt_token>"
```

### 10. Decline a Negotiation
```bash
curl -X POST http://127.0.0.1:8000/negotiations/1/decline/ \
  -H "Authorization: Bearer <your_jwt_token>"
```

---

## Notes
- All endpoints require a valid API token (superuser) or JWT (regular user), unless otherwise noted.
- All endpoints support standard HTTP methods (GET, POST, PUT, PATCH, DELETE).
- For file uploads, use `multipart/form-data`.
- Pagination, filtering, and search can be added as needed.
- **Foreign Key fields**: When providing a foreign key field, use the referenced object's ID. See above for which model each field references.
- **Notifications**: Only users with `notifications_on=true` will receive notifications.
- **Error Handling**: All endpoints return clear error messages for invalid input, duplicate emails, and authentication issues.

---

## Download
- This documentation is available as `API_DOCUMENTATION.md` in your project folder.
- You can also download it from the home page. 