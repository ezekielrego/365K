# Kurima API Documentation

## Table of Contents
1. [Getting Started](#getting-started)
2. [Authentication](#authentication)
3. [Email Verification](#email-verification)
4. [User Management](#user-management)
5. [Product Management](#product-management)
6. [Farm Management](#farm-management)
7. [Order Management](#order-management)
8. [Payment Management](#payment-management)
9. [Chat & Messaging](#chat--messaging)
10. [Product Negotiations](#product-negotiations)
11. [Notifications](#notifications)
12. [Vehicle Management](#vehicle-management)
13. [Job Management](#job-management)
14. [News Management](#news-management)
15. [Ads & Ratings](#ads--ratings)
16. [Error Handling](#error-handling)
17. [Testing & Examples](#testing--examples)

---

## Getting Started

### API Base URLs
- **Local Development**: `http://127.0.0.1:8000/api/`
- **Production**: `https://kurimaapi.bhiza.co.zw/api/`

### Authentication Methods
- **JWT Tokens**: For regular users (recommended)
- **API Tokens**: For superusers (via web interface)

### Authentication Header
```
Authorization: Bearer <your_jwt_token>
```

---

## Authentication

### Login
**POST** `/api/login/`
```json
{
    "email": "user@example.com",
    "password": "yourpassword"
}
```

**Success Response:**
```json
{
    "refresh": "<refresh_token>",
    "access": "<access_token>",
    "user": {
        "id": 1,
        "email": "user@example.com",
        "name": "John Doe",
        "role": "farmer",
        "email_verified_at": "2024-01-01T12:00:00Z"
    },
    "token_lifetime": {
        "access_token_days": 30,
        "refresh_token_days": 60
    }
}
```

**Error (email not verified):**
```json
{
    "error": "Email not verified. Please verify your email before logging in.",
    "email_verification_required": true
}
```

**Error (invalid credentials):**
```json
{
    "error": "Invalid credentials"
}
```

**Note:** Users must verify their email before they can log in successfully.

### Token Refresh
**POST** `/api/token/refresh/`
```json
{
    "refresh": "<refresh_token>"
}
```

**Success Response:**
```json
{
    "success": true,
    "access": "<new_access_token>",
    "refresh": "<new_refresh_token>",
    "token_lifetime": {
        "access_token_days": 30,
        "refresh_token_days": 60
    }
}
```

**Error Response:**
```json
{
    "success": false,
    "error": "Invalid refresh token"
}
```

### Password Reset

#### Request Password Reset
**POST** `/api/password-reset/request/`
```json
{
    "email": "user@example.com"
}
```

**Success Response:**
```json
{
    "success": true,
    "message": "Password reset email sent successfully"
}
```

**Error Response:**
```json
{
    "success": false,
    "error": "No user found with this email address"
}
```

#### Verify Reset Token
**POST** `/api/password-reset/verify/`
```json
{
    "token": "<reset_token>"
}
```

**Success Response:**
```json
{
    "success": true,
    "message": "Token is valid",
    "email": "user@example.com"
}
```

**Error Response:**
```json
{
    "success": false,
    "error": "Invalid or expired reset token"
}
```

#### Confirm Password Reset
**POST** `/api/password-reset/confirm/`
```json
{
    "token": "<reset_token>",
    "new_password": "newpassword123",
    "confirm_password": "newpassword123"
}
```

**Success Response:**
```json
{
    "success": true,
    "message": "Password reset successfully"
}
```

**Error Response:**
```json
{
    "success": false,
    "error": "Invalid or expired reset token"
}
```

### Authentication Features

#### Extended Token Lifetime
- **Access Token**: 30 days (instead of default 5 minutes)
- **Refresh Token**: 60 days
- **Automatic Refresh**: Tokens are automatically rotated for security

#### Password Reset Flow
1. **Request Reset**: User requests password reset via email
2. **Email Sent**: System sends reset link with 24-hour expiry
3. **Verify Token**: Frontend verifies token validity
4. **Reset Password**: User sets new password with token
5. **Login**: User can login with new password

#### Frontend Integration
```javascript
// Automatic token refresh
const refreshToken = async () => {
    try {
        const response = await fetch('/api/token/refresh/', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                refresh: localStorage.getItem('refresh_token')
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            localStorage.setItem('access_token', data.access);
            localStorage.setItem('refresh_token', data.refresh);
            return data.access;
        }
    } catch (error) {
        // Handle refresh failure
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        window.location.href = '/login';
    }
};

// Request password reset
const requestPasswordReset = async (email) => {
    const response = await fetch('/api/password-reset/request/', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email })
    });
    
    return response.json();
};
```

---

## Email Verification

### Send Verification Code
**POST** `/api/email-verification/`
```json
{
    "email": "user@example.com"
}
```

**Response:**
```json
{
    "message": "Verification code sent to email",
    "expires_at": "2024-01-01T12:00:00Z"
}
```

**Note:** The verification code is sent via email and is not returned in the API response for security reasons.

### Verify Email Code
**PUT** `/api/email-verification/`
```json
{
    "email": "user@example.com",
    "code": "123456"
}
```

**Response:**
```json
{
    "message": "Email verified successfully"
}
```

---

## User Management

### Register New User
**POST** `/api/register/`
```json
{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securepassword123",
    "role": "farmer"
}
```

**Response:**
```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "role": "farmer",
    "message": "User registered successfully. Verification email sent."
}
```

**Note:** After registration, a verification email is automatically sent to the user's email address.

### User Operations
- `GET /api/users/` — List users
- `POST /api/users/` — Create user (admin only)
- `GET /api/users/{id}/` — Retrieve user
- `PUT/PATCH /api/users/{id}/` — Update user
- `DELETE /api/users/{id}/` — Delete user

### User Fields
- `id`, `name`, `email`, `profile_photo`, `email_verified_at`, `kyc_verified`, `password`, `role`, `region`, `phone`, `location`, `created_at`, `updated_at`, `wallet_balance`, `kyc_verified_at`, `training_points`, `notifications_on`, `document`, `verified`

**User Roles:**
- `admin` - System administrator
- `farmer` - Agricultural producer
- `buyer` - Product purchaser
- `agent` - Sales agent
- `agro_dealer` - Agricultural equipment dealer

**Upload Document Example:**
```bash
curl -X PATCH https://kurimaapi.bhiza.co.zw/api/users/2/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -F "document=@/path/to/id_card.pdf"
```

---

## Product Management

### Product Categories
- `GET /api/categories/` — List categories
- `POST /api/categories/` — Create category
- `GET /api/categories/{id}/` — Retrieve category
- `PUT/PATCH /api/categories/{id}/` — Update category
- `DELETE /api/categories/{id}/` — Delete category

**Category Fields:** `id`, `name`, `description`, `created_at`, `updated_at`

### Products
- `GET /api/products/` — List products
- `POST /api/products/` — Create product
- `GET /api/products/{id}/` — Retrieve product
- `PUT/PATCH /api/products/{id}/` — Update product
- `DELETE /api/products/{id}/` — Delete product

**Product Fields:**
- `id`, `category`, `user`, `name`, `slug`, `description`, `price`, `quantity`, `unit`, `harvest_date`, `quantity_available`, `status`, `is_organic`, `is_featured`, `type`, `image`, `video`, `location`, `created_at`, `updated_at`, `is_opportunity`, `sold_count`
- **Additional Fields:**
  - `average_rating`: Average rating (float, null if no ratings)
  - `ratings_count`: Number of ratings for the product

**Example Product Response:**
```json
{
    "id": 1,
    "name": "Fresh Tomatoes",
    "price": "2.50",
    "quantity": 100,
    "unit": "kg",
    "average_rating": 4.5,
    "ratings_count": 2,
    "comments_count": 5,
    "comments": [
        {
            "id": 1,
            "user_name": "John Doe",
            "content": "Great quality tomatoes!",
            "created_at": "2024-01-01T10:00:00Z"
        }
    ]
}
```

### Product Comments
- `GET /api/product-comments/?product={product_id}` — Get product comments
- `POST /api/product-comments/` — Create comment
- `PUT /api/product-comments/{id}/` — Update comment
- `DELETE /api/product-comments/{id}/` — Delete comment

**Create Comment:**
```json
{
    "product": 1,
    "content": "Great product! Very fresh and good quality."
}
```

---

## Farm Management

### Farms
- `GET /api/farms/` — List farms
- `POST /api/farms/` — Create farm
- `GET /api/farms/{id}/` — Retrieve farm
- `PUT/PATCH /api/farms/{id}/` — Update farm
- `DELETE /api/farms/{id}/` — Delete farm

**Farm Fields:**
- `id`, `status`, `user`, `name`, `description`, `location`, `latitude`, `longitude`, `image`, `is_active`, `created_at`, `updated_at`

---

## Order Management

### Orders
- `GET /api/orders/` — List orders
- `POST /api/orders/` — Create order
- `GET /api/orders/{id}/` — Retrieve order
- `PUT/PATCH /api/orders/{id}/` — Update order
- `DELETE /api/orders/{id}/` — Delete order

**Order Fields:**
- `id`, `user`, `buyer`, `product`, `quantity`, `total_price`, `status`, `payment_status`, `created_at`, `updated_at`, `farmer`

---

## Payment Management

The payment system integrates with **Paynow Zimbabwe** using the official Python SDK. It supports both web-based payments and mobile money transactions (EcoCash, OneMoney).

### Create Payment (Web-based)
**POST** `/api/payments/create/`

**Request Body:**
```json
{
    "amount": "25.00",
    "product_id": 1,
    "quantity": 2,
    "payment_method": "web"
}
```

**Success Response:**
```json
{
    "success": true,
    "message": "Payment initiated successfully",
    "redirect_url": "https://paynow.co.zw/interface/initiatetransaction?hash=...",
    "has_redirect": true,
    "payment_id": 1,
    "paynow_reference": "PAY-123456789",
    "order_id": 1
}
```

### Create Mobile Payment
**POST** `/api/payments/create/`

**Request Body:**
```json
{
    "amount": "25.00",
    "product_id": 1,
    "quantity": 2,
    "payment_method": "ecocash",
    "phone_number": "0777123456"
}
```

**Success Response:**
```json
{
    "success": true,
    "message": "Mobile payment initiated successfully",
    "instructions": "Please check your phone for payment instructions...",
    "payment_id": 1,
    "paynow_reference": "PAY-123456789",
    "order_id": 1,
    "payment_method": "ecocash"
}
```

### Check Payment Status
**GET** `/api/payments/status/{payment_id}/`

**Response:**
```json
{
    "success": true,
    "status": "paid",
    "message": "Payment successful",
    "amount": 25.00,
    "reference": "Order-1-1",
    "paynow_reference": "PAY-123456789"
}
```

### Payment Records
- `GET /api/payments/` — List user's payments
- `GET /api/payments/{id}/` — Get payment details

**Payment Fields:**
- `id`, `order`, `user`, `amount`, `paynow_poll_url`, `paynow_payment_id`, `paynow_reference`, `status`, `created_at`, `updated_at`

### PayNow Callback URLs
- `GET /api/payments/return/` — User return URL (after payment completion)
- `POST /api/payments/result/` — Server callback URL (PayNow to server)

### Payment Flow

#### Web Payment Flow:
1. **Frontend** sends payment request to `/api/payments/create/`
2. **Backend** creates order and payment record
3. **Backend** initiates PayNow payment using official SDK
4. **Backend** returns `redirect_url` to frontend
5. **Frontend** redirects user to PayNow payment page
6. **User** completes payment on PayNow
7. **PayNow** redirects user back to return URL
8. **PayNow** sends status update to result URL
9. **Backend** updates payment and order status

#### Mobile Payment Flow:
1. **Frontend** sends mobile payment request with phone number
2. **Backend** creates order and payment record
3. **Backend** initiates mobile payment using official SDK
4. **Backend** returns payment instructions to frontend
5. **Frontend** displays instructions to user
6. **User** follows instructions on their phone
7. **PayNow** sends status update to result URL
8. **Backend** updates payment and order status

### Supported Payment Methods
- **Web**: Credit/Debit cards, bank transfers
- **Mobile**: EcoCash, OneMoney

### Payment Status Values
- `pending` - Payment initiated, waiting for completion
- `paid` - Payment completed successfully
- `failed` - Payment failed or was cancelled
- `cancelled` - Payment was cancelled by user

### Error Handling
```json
{
    "success": false,
    "error": "Payment creation failed"
}
```

**Common Errors:**
- `Product not found` - Invalid product ID
- `Payment creation failed` - PayNow API error
- `No poll URL available` - Payment record not properly created
- `Payment not found` - Invalid payment ID

---

## Chat & Messaging

### Chat Rooms
- `GET /api/chatrooms/` — List chat rooms
- `POST /api/chatrooms/` — Create chat room
- `GET /api/chatrooms/{id}/` — Retrieve chat room
- `PUT/PATCH /api/chatrooms/{id}/` — Update chat room
- `DELETE /api/chatrooms/{id}/` — Delete chat room

**ChatRoom Fields:**
- `id`, `name`, `users`, `created_at`, `updated_at`

### Messages
- `GET /api/messages/` — List messages
- `POST /api/messages/` — Send message
- `GET /api/messages/{id}/` — Retrieve message
- `PUT/PATCH /api/messages/{id}/` — Update message
- `DELETE /api/messages/{id}/` — Delete message

**Message Fields:**
- `id`, `room`, `sender`, `content`, `created_at`, `is_read`

---

## Product Negotiations

### Negotiations
- `GET /api/negotiations/` — List negotiations
- `POST /api/negotiations/` — Propose price
- `GET /api/negotiations/{id}/` — Retrieve negotiation
- `PUT/PATCH /api/negotiations/{id}/` — Update negotiation
- `DELETE /api/negotiations/{id}/` — Delete negotiation

**ProductNegotiation Fields:**
- `id`, `product`, `buyer`, `proposed_price`, `status` (pending, accepted, declined), `created_at`, `updated_at`

### Negotiation Actions
- `POST /api/negotiations/{id}/accept/` — Accept price offer
- `POST /api/negotiations/{id}/decline/` — Decline price offer

**Negotiation Flow:**
1. Buyer proposes a price for a product (creates negotiation, status=pending)
2. Product owner can view negotiations
3. Owner can accept or decline offers via the accept/decline endpoints
4. All negotiation history is saved

---

## Notifications

### Notification Management (Admin Only)
- `GET /api/notifications/` — List all notifications
- `POST /api/notifications/` — Create notification
- `GET /api/notifications/{id}/` — Retrieve notification
- `PUT/PATCH /api/notifications/{id}/` — Update notification
- `DELETE /api/notifications/{id}/` — Delete notification

**Create Notification:**
```json
{
    "type": "info",
    "message": "System maintenance at midnight.",
    "users": [2, 3]
}
```

### User Notifications
- `GET /api/notifications/pull/` — Pull active notifications for authenticated user
- `POST /api/notifications/push/` — Push notification to authenticated user

**Push Notification:**
```json
{
    "notification_id": 1
}
```

**Notification Fields:**
- `id`, `type` (info, warning, success, error), `message`, `created_at`, `users`, `is_active`

---

## Vehicle Management

### Vehicles (Admin Only)
- `GET /api/vehicles/` — List all vehicles
- `POST /api/vehicles/` — Create vehicle
- `GET /api/vehicles/{id}/` — Get vehicle details
- `PUT /api/vehicles/{id}/` — Update vehicle
- `DELETE /api/vehicles/{id}/` — Delete vehicle

**Create Vehicle:**
```json
{
    "name": "John Deere Tractor",
    "vehicle_type": "tractor",
    "description": "High-quality farming tractor",
    "capacity": "50 HP",
    "hourly_rate": "25.00",
    "daily_rate": "200.00",
    "location": "Harare",
    "destination": "Bulawayo",
    "image": "file_upload"
}
```

### Vehicle Hiring
- `GET /api/vehicle-hires/` — Get user's vehicle hires
- `POST /api/vehicle-hires/` — Hire a vehicle
- `PUT /api/vehicle-hires/{id}/` — Update hire status

**Hire Vehicle:**
```json
{
    "vehicle": 1,
    "start_date": "2024-01-01T08:00:00Z",
    "end_date": "2024-01-01T18:00:00Z",
    "total_cost": "200.00",
    "purpose": "Harvesting maize on my farm"
}
```

---

## Job Management

### Job Categories
- `GET /api/job-categories/` — List job categories
- `POST /api/job-categories/` — Create job category (admin only)

**Create Job Category:**
```json
{
    "name": "Farm Management",
    "description": "Jobs related to farm management"
}
```

### Jobs
- `GET /api/jobs/` — List all jobs
- `GET /api/jobs/featured/` — Get featured jobs
- `GET /api/jobs/by-category/?category={category_id}` — Get jobs by category
- `GET /api/jobs/{id}/` — Get job details
- `POST /api/jobs/` — Create job (admin only)
- `PUT /api/jobs/{id}/` — Update job (admin only)
- `DELETE /api/jobs/{id}/` — Delete job (admin only)

**Create Job:**
```json
{
    "title": "Farm Manager",
    "company": "Green Farms Ltd",
    "category": 1,
    "job_type": "full_time",
    "location": "Harare",
    "description": "Managing daily farm operations",
    "requirements": "5+ years experience in farming",
    "salary_min": "1000.00",
    "salary_max": "2000.00",
    "salary_type": "monthly",
    "contact_email": "hr@greenfarms.com",
    "contact_phone": "+263771234567",
    "application_deadline": "2024-02-01"
}
```

### Job Applications
- `GET /api/job-applications/` — Get user's applications
- `POST /api/job-applications/` — Apply for job
- `PUT /api/job-applications/{id}/` — Update application status (admin only)

**Apply for Job:**
```json
{
    "job": 1,
    "cover_letter": "I am interested in this position...",
    "resume": "file_upload"
}
```

---

## News Management

### News Categories
- `GET /api/news-categories/` — List news categories
- `POST /api/news-categories/` — Create news category (admin only)

**Create News Category:**
```json
{
    "name": "Technology",
    "description": "Farming technology news"
}
```

### News
- `GET /api/news/` — List all news
- `GET /api/news/trending/` — Get trending news
- `GET /api/news/by-category/?category={category_id}` — Get news by category
- `GET /api/news/{id}/` — Get news article
- `POST /api/news/` — Create news (admin only)
- `PUT /api/news/{id}/` — Update news (admin only)
- `DELETE /api/news/{id}/` — Delete news (admin only)

**Create News:**
```json
{
    "title": "New Farming Technology",
    "slug": "new-farming-technology",
    "category": 1,
    "content": "Article content here...",
    "summary": "Brief summary of the article",
    "image": "file_upload",
    "is_trending": true
}
```

---

## Ads & Ratings

### Ads (Public - No Authentication Required)
- `GET /api/ads/` — List active ads

**Response:**
```json
[
    {
        "type": "image",
        "title": "Get Special Offer",
        "subtitle": "Up to 40%",
        "description": "All Services Available | T&C Applied",
        "button": "Claim",
        "media": "https://kurimaapi.bhiza.co.zw/media/ads/ad-image.jpg",
        "link": "https://kurima365.com/special-offer"
    }
]
```

### Product Ratings

#### List Product Ratings
**GET** `/api/ratings/?product=<product_id>`

**Response:**
```json
[
    {
        "id": 1,
        "product": 123,
        "user": "john@example.com",
        "rating": 5,
        "review": "Excellent quality tomatoes!",
        "created_at": "2024-01-01T12:00:00Z"
    },
    {
        "id": 2,
        "product": 123,
        "user": "jane@example.com",
        "rating": 4,
        "review": "Good product, fast delivery",
        "created_at": "2024-01-02T10:30:00Z"
    }
]
```

#### Create Product Rating
**POST** `/api/ratings/`

**Request Body:**
```json
{
    "product": 123,
    "rating": 5,
    "review": "Great product!"
}
```

**Success Response:**
```json
{
    "id": 3,
    "product": 123,
    "user": "user@example.com",
    "rating": 5,
    "review": "Great product!",
    "created_at": "2024-01-03T15:45:00Z"
}
```

#### Update Rating (Owner Only)
**PUT/PATCH** `/api/ratings/{id}/`

**Request Body:**
```json
{
    "rating": 4,
    "review": "Updated review"
}
```

#### Delete Rating (Owner Only)
**DELETE** `/api/ratings/{id}/`

**Response:**
```json
{
    "message": "Rating deleted successfully"
}
```

#### Rating Fields
- `id` - Rating ID (integer)
- `product` - Product ID (integer, required)
- `user` - User email (string, read-only)
- `rating` - Rating value (integer, 1-5, required)
- `review` - Review text (string, optional)
- `created_at` - Creation timestamp (datetime)

#### Validation Rules
1. **Rating Range**: Must be between 1 and 5
2. **Unique Rating**: A user can only rate a product once
3. **Authentication**: User must be authenticated
4. **Product Existence**: Product must exist in the database

#### Error Responses

**Invalid Rating Value:**
```json
{
    "rating": ["Rating must be between 1 and 5"]
}
```

**Duplicate Rating:**
```json
{
    "non_field_errors": ["You have already rated this product"]
}
```

**Missing Product ID:**
```json
{
    "error": "Failed to create rating",
    "detail": "Column 'product_id' cannot be null"
}
```

**Unauthorized:**
```json
{
    "detail": "Authentication credentials were not provided."
}
```

**Product Not Found:**
```json
{
    "product": ["Invalid pk \"999\" - object does not exist."]
}
```

#### Notes
- Each user can rate a product only once
- Ratings are integers from 1 to 5
- Only the rating owner can update or delete their rating
- Product responses include `average_rating` and `ratings_count` fields
- The `average_rating` field is null if no ratings exist

---

## Error Handling

### Common Error Responses

**400 Bad Request**
```json
{
    "error": "Error message description"
}
```

**401 Unauthorized**
```json
{
    "detail": "Authentication credentials were not provided."
}
```

**403 Forbidden**
```json
{
    "detail": "You do not have permission to perform this action."
}
```

**404 Not Found**
```json
{
    "error": "Resource not found"
}
```

**500 Internal Server Error**
```json
{
    "error": "Failed to send verification email. Please try again."
}
```

### Field Validation Errors
```json
{
    "email": ["A user with this email already exists."],
    "password": ["This field is required."]
}
```

---

## Testing & Examples

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

### 2. Login and Get JWT Token
```bash
curl -X POST http://127.0.0.1:8000/api/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "yourpassword"
  }'
```

### 3. Refresh JWT Token
```bash
curl -X POST http://127.0.0.1:8000/api/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "<your_refresh_token>"
  }'
```

### 4. Request Password Reset
```bash
curl -X POST http://127.0.0.1:8000/api/password-reset/request/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com"
  }'
```

### 5. Verify Reset Token
```bash
curl -X POST http://127.0.0.1:8000/api/password-reset/verify/ \
  -H "Content-Type: application/json" \
  -d '{
    "token": "<reset_token>"
  }'
```

### 6. Confirm Password Reset
```bash
curl -X POST http://127.0.0.1:8000/api/password-reset/confirm/ \
  -H "Content-Type: application/json" \
  -d '{
    "token": "<reset_token>",
    "new_password": "newpassword123",
    "confirm_password": "newpassword123"
  }'
```

### 3. Send Email Verification
```bash
curl -X POST http://127.0.0.1:8000/api/email-verification/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com"
  }'
```

### 4. Verify Email Code
```bash
curl -X PUT http://127.0.0.1:8000/api/email-verification/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "code": "123456"
  }'
```

### 5. Make Authenticated Request
```bash
curl -H "Authorization: Bearer <your_jwt_token>" \
  http://127.0.0.1:8000/api/products/
```

### 6. Create a Product
```bash
curl -X POST http://127.0.0.1:8000/api/products/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Fresh Tomatoes",
    "category": 1,
    "price": "2.50",
    "quantity": 100,
    "unit": "kg",
    "description": "Fresh organic tomatoes"
  }'
```

### 7. Create Web Payment
```bash
curl -X POST http://127.0.0.1:8000/api/payments/create/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": "25.00",
    "product_id": 1,
    "quantity": 2,
    "payment_method": "web"
  }'
```

### 8. Create Mobile Payment
```bash
curl -X POST http://127.0.0.1:8000/api/payments/create/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": "25.00",
    "product_id": 1,
    "quantity": 2,
    "payment_method": "ecocash",
    "phone_number": "0777123456"
  }'
```

### 9. Check Payment Status
```bash
curl -H "Authorization: Bearer <your_jwt_token>" \
  http://127.0.0.1:8000/api/payments/status/1/
```

### 10. Create a Chat Room
```bash
curl -X POST http://127.0.0.1:8000/api/chatrooms/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Negotiation Room",
    "users": [2, 3]
  }'
```

### 11. Send a Message
```bash
curl -X POST http://127.0.0.1:8000/api/messages/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "room": 1,
    "content": "Hello, I am interested in your product."
  }'
```

### 12. Propose a Negotiation
```bash
curl -X POST http://127.0.0.1:8000/api/negotiations/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "product": 1,
    "proposed_price": "8.50"
  }'
```

### 13. Pull Notifications
```bash
curl -H "Authorization: Bearer <your_jwt_token>" \
  http://127.0.0.1:8000/api/notifications/pull/
```

### 14. Get Product Ratings
```bash
curl -H "Authorization: Bearer <your_jwt_token>" \
  http://127.0.0.1:8000/api/ratings/?product=1
```

### 15. Create Product Rating
```bash
curl -X POST http://127.0.0.1:8000/api/ratings/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "product": 1,
    "rating": 5,
    "review": "Excellent quality product!"
  }'
```

### 16. Update Product Rating
```bash
curl -X PUT http://127.0.0.1:8000/api/ratings/1/ \
  -H "Authorization: Bearer <your_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "rating": 4,
    "review": "Updated review"
  }'
```

### 17. Delete Product Rating
```bash
curl -X DELETE http://127.0.0.1:8000/api/ratings/1/ \
  -H "Authorization: Bearer <your_jwt_token>"
```

---

## Additional Information

### File Uploads
For endpoints that accept file uploads (images, resumes, documents, etc.), use `multipart/form-data` content type.

### Rate Limiting
- Email verification: 3 requests per minute per email
- Comments: 10 comments per minute per user
- Vehicle hires: 5 requests per minute per user
- Payments: 10 requests per minute per user

### Admin-Only Endpoints
The following endpoints require admin privileges:
- Vehicle creation, update, deletion
- Job creation, update, deletion
- News creation, update, deletion
- Job category management
- News category management
- Notification management

### Database Models
The API includes the following models:
- `User` - User accounts and profiles
- `EmailVerification` - Email verification codes
- `Region` - Geographic regions
- `ProductCategory` - Product categories
- `Product` - Agricultural products
- `ProductComment` - Product comments
- `ProductRating` - Product ratings
- `Farm` - Farm information
- `Order` - Purchase orders
- `Payment` - Payment records
- `Notification` - System notifications
- `ChatRoom` - Chat rooms
- `Message` - Chat messages
- `ProductNegotiation` - Price negotiations
- `Ad` - Advertisement content
- `Vehicle` - Vehicle information
- `VehicleHire` - Vehicle rental records
- `JobCategory` - Job categories
- `Job` - Job postings
- `JobApplication` - Job applications
- `NewsCategory` - News categories
- `News` - News articles

### Testing the API

#### 1. Start the server
```bash
python manage.py runserver
```

#### 2. Create a superuser
```bash
python manage.py createsuperuser
```

#### 3. Test endpoints using tools like:
- Postman
- curl
- Django REST Framework browsable API

### Notes
- All endpoints require a valid JWT token unless specified otherwise
- All endpoints support standard HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Foreign key fields use the referenced object's ID
- Only users with `notifications_on=true` will receive notifications
- Email verification is required before users can log in
- All endpoints return clear error messages for invalid input and authentication issues
- Payment integration uses the official Paynow Zimbabwe Python SDK
- Mobile payments support EcoCash and OneMoney methods
- Payment callbacks are processed automatically to update order status

---

## Support
For technical support or questions about the API, please contact the development team or refer to the project documentation. 