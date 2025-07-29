# Payment Integration Guide

## Overview

This guide explains how to integrate payments with the Bhiza API using the new direct payment system. The system allows frontend applications to initiate payments and handle returns from PayNow.

## Fixed Issues

### 1. Reset Password URL (404 Error)
- **Problem**: The URL `reset-password` was not defined in Django URL patterns
- **Solution**: Added `ResetPasswordView` and URL pattern `/reset-password/`
- **Usage**: Users can now access reset password links sent via email

## Payment Flow

### 1. Frontend Payment Initiation

#### Method 1: Direct URL Redirect
```javascript
// Redirect user directly to payment URL
const paymentUrl = `https://kurimaapi.bhiza.co.zw/payment/?user_id=${userId}&item_name=${itemName}&item_number=${itemNumber}&amount=${amount}`;
window.location.href = paymentUrl;
```

#### Method 2: API Call (Recommended)
```javascript
// Make API call to create payment
const response = await fetch('https://kurimaapi.bhiza.co.zw/api/payment/create/', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        user_id: userId,
        item_name: itemName,
        item_number: itemNumber,
        amount: amount
    })
});

const result = await response.json();
if (result.success) {
    // Redirect to PayNow
    window.location.href = result.payment_url;
}
```

### 2. Payment Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | Integer | Yes | User ID from the database |
| `item_name` | String | Yes | Name of the item being purchased |
| `item_number` | String | Yes | Item number or identifier |
| `amount` | Float | Yes | Payment amount in ZWL |

### 3. Payment Return Handling

After payment completion, users are redirected to:
```
https://kurimaapi.bhiza.co.zw/api/payments/return/?paymentId=xxx&status=Paid&paynowreference=xxx
```

The return page provides:
- Payment status (success/failed)
- Payment details
- User-friendly interface
- Automatic status updates

## API Endpoints

### 1. Create Payment
```
POST /api/payment/create/
```

**Request Body:**
```json
{
    "user_id": 123,
    "item_name": "Product Name",
    "item_number": "PROD001",
    "amount": 100.00
}
```

**Response:**
```json
{
    "success": true,
    "payment_url": "https://paynow.co.zw/payment/...",
    "payment_id": 456,
    "paynow_reference": "PAYNOW123456",
    "message": "Payment created successfully"
}
```

### 2. Check Payment Status
```
GET /api/payments/status/{payment_id}/
```

**Response:**
```json
{
    "success": true,
    "status": "paid",
    "message": "Payment successful",
    "amount": 100.00,
    "reference": "Order-123-456",
    "paynow_reference": "PAYNOW123456"
}
```

### 3. Reset Password
```
GET /reset-password/?token=xxx
POST /reset-password/
```

## Frontend Integration Examples

### React Example
```javascript
import React, { useState } from 'react';

const PaymentButton = ({ userId, itemName, itemNumber, amount }) => {
    const [loading, setLoading] = useState(false);

    const handlePayment = async () => {
        setLoading(true);
        try {
            const response = await fetch('https://kurimaapi.bhiza.co.zw/api/payment/create/', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    user_id: userId,
                    item_name: itemName,
                    item_number: itemNumber,
                    amount: amount
                })
            });

            const result = await response.json();
            if (result.success) {
                window.location.href = result.payment_url;
            } else {
                alert('Payment creation failed: ' + result.error);
            }
        } catch (error) {
            alert('Error creating payment: ' + error.message);
        } finally {
            setLoading(false);
        }
    };

    return (
        <button 
            onClick={handlePayment} 
            disabled={loading}
            className="payment-btn"
        >
            {loading ? 'Processing...' : 'Pay Now'}
        </button>
    );
};
```

### Vanilla JavaScript Example
```javascript
function initiatePayment(userId, itemName, itemNumber, amount) {
    // Method 1: Direct redirect
    const paymentUrl = `https://kurimaapi.bhiza.co.zw/payment/?user_id=${userId}&item_name=${encodeURIComponent(itemName)}&item_number=${encodeURIComponent(itemNumber)}&amount=${amount}`;
    window.location.href = paymentUrl;
    
    // Method 2: API call
    fetch('https://kurimaapi.bhiza.co.zw/api/payment/create/', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            user_id: userId,
            item_name: itemName,
            item_number: itemNumber,
            amount: amount
        })
    })
    .then(response => response.json())
    .then(result => {
        if (result.success) {
            window.location.href = result.payment_url;
        } else {
            alert('Payment creation failed: ' + result.error);
        }
    })
    .catch(error => {
        alert('Error creating payment: ' + error.message);
    });
}
```

## Error Handling

### Common Errors
1. **Missing Parameters**: Ensure all required parameters are provided
2. **User Not Found**: Verify the user_id exists in the database
3. **Invalid Amount**: Amount must be a positive number
4. **PayNow Error**: Network or PayNow service issues

### Error Response Format
```json
{
    "success": false,
    "error": "Error description"
}
```

## Security Considerations

1. **User Authentication**: Ensure users are authenticated before creating payments
2. **Input Validation**: Validate all input parameters on both frontend and backend
3. **HTTPS**: Always use HTTPS for payment operations
4. **Token Security**: Protect reset password tokens

## Testing

### Test Payment Flow
1. Create a test user in the database
2. Use the test user ID to create payments
3. Test both successful and failed payment scenarios
4. Verify return URL handling

### Test URLs
```
# Direct payment
https://kurimaapi.bhiza.co.zw/payment/?user_id=1&item_name=Test&item_number=TEST001&amount=10.00

# Reset password
https://kurimaapi.bhiza.co.zw/reset-password/?token=your_token_here
```

## Support

For technical support or questions about the payment integration, please contact the development team or refer to the API documentation. 