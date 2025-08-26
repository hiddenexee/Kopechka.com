# Kopechka API Documentation

Kopechka API is a RESTful API that allows you to programmatically manage temporary email addresses. With this API, you can create orders, retrieve email contents, and perform account operations.

## Base URL

```
https://api.kopechka.com/api/v1
```

## Authentication

For all API requests, you need to send your API key in the `X-API-Key` header.

```http
X-API-Key: YOUR_API_KEY
```

### Security Warning
- Keep your API key secure and never use it in client-side code
- You can create your API key from the dashboard

## Rate Limiting

API requests are subject to rate limiting. When the limit is exceeded, you will receive a `429 Too Many Requests` error.

## Response Format

All API responses are in JSON format:

```json
{
  "success": true,
  "data": {},
  "message": "Success message"
}
```

## Error Codes

| HTTP Status | Description |
|-------------|-------------|
| 200 | Success |
| 400 | Bad request |
| 401 | Unauthorized - Invalid or missing API key |
| 404 | Resource not found |
| 429 | Rate limit exceeded |
| 500 | Server error |

## API Endpoints

### User Operations

#### Profile Information
```http
GET /profile
```

Retrieves the user's profile information.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "user_id",
    "email": "user@example.com",
    "username": "username",
    "balance": 150.75,
    "role": "user",
    "emailVerified": true,
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Balance Inquiry
```http
GET /balance
```

Queries the user's current balance.

**Response:**
```json
{
  "success": true,
  "data": {
    "balance": 150.75
  }
}
```

### Domain Operations

#### Domain List
```http
GET /domains
```

Retrieves the list of currently active domains.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "domain_id",
      "name": "example.com",
      "price": 5.00,
      "category": "popular",
      "description": "Popular domain for general use"
    }
  ]
}
```

### Order Operations

#### Order List
```http
GET /orders?page=1&limit=10&status=active&domain=example.com
```

Lists all orders for the user.

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Records per page (default: 10)
- `status` (optional): Order status filter
- `domain` (optional): Domain filter

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "order_id",
      "orderId": "API-1234567890",
      "email": "temp@example.com",
      "domain": "example.com",
      "site": "instagram.com",
      "status": "active",
      "price": 5.00,
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25,
    "pages": 3
  }
}
```

#### Specific Order Information
```http
GET /orders/{orderId}
```

Retrieves details of a specific order.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "order_id",
    "orderId": "API-1234567890",
    "email": "temp@example.com",
    "domain": "example.com",
    "site": "instagram.com",
    "status": "active",
    "price": 5.00,
    "duration": 10,
    "createdAt": "2024-01-01T00:00:00Z",
    "endTime": "2024-01-01T00:10:00Z"
  }
}
```

#### Create Order
```http
POST /orders
```

Creates a new email order.

**Request Body:**
```json
{
  "domain": "example.com",
  "site": "instagram.com"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Order created successfully",
  "data": {
    "orderId": "API-1234567890",
    "email": "temp@example.com",
    "domain": "example.com",
    "site": "instagram.com",
    "price": 5.00,
    "status": "pending",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Cancel Order
```http
DELETE /orders/{orderId}
```

Cancels an existing order. Full refund is processed if no email has been received.

**Response:**
```json
{
  "success": true,
  "message": "Order cancelled and refund processed",
  "refundAmount": 5.00
}
```

#### Get Order Messages
```http
GET /orders/{orderId}/messages
```

Retrieves email messages received by a specific order.

**Response:**
```json
{
  "success": true,
  "messages": [
    {
      "id": "msg_123",
      "from": "noreply@example.com",
      "subject": "Verification Code",
      "content": "Your verification code is: 123456",
      "receivedAt": "2024-01-01T12:00:00Z"
    }
  ],
  "order": {
    "id": "order_id",
    "email": "temp@example.com",
    "domain": "example.com",
    "status": "active",
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```

#### Start Email Polling
```http
POST /orders/{orderId}/start-polling
```

Starts email polling process for an order. This endpoint is required to fetch new emails.

**Note:** This endpoint can only be used for active orders that have received at least one email.

**Response:**
```json
{
  "success": true,
  "message": "Email polling started",
  "data": {
    "orderId": "API-1234567890",
    "pollingStatus": "active"
  }
}
```

## Code Examples

### JavaScript (Node.js)

```javascript
const axios = require('axios');

const api = axios.create({
  baseURL: 'https://api.kopechka.com/api/v1',
  headers: {
    'X-API-Key': 'YOUR_API_KEY',
    'Content-Type': 'application/json'
  }
});

// Balance check
async function getBalance() {
  try {
    const response = await api.get('/balance');
    console.log('Balance:', response.data.data.balance);
  } catch (error) {
    console.error('Error:', error.response.data);
  }
}

// Create order
async function createOrder() {
  try {
    const response = await api.post('/orders', {
      domain: 'example.com',
      site: 'instagram.com'
    });
    console.log('Order created:', response.data.data);
  } catch (error) {
    console.error('Error:', error.response.data);
  }
}

// Fetch messages
async function getMessages(orderId) {
  try {
    const response = await api.get(`/orders/${orderId}/messages`);
    console.log('Messages:', response.data.messages);
  } catch (error) {
    console.error('Error:', error.response.data);
  }
}
```

### Python

```python
import requests

class KopeechkaAPI:
    def __init__(self, api_key):
        self.base_url = 'https://api.kopechka.com/api/v1'
        self.headers = {
            'X-API-Key': api_key,
            'Content-Type': 'application/json'
        }
    
    def get_balance(self):
        response = requests.get(f'{self.base_url}/balance', headers=self.headers)
        return response.json()
    
    def create_order(self, domain, site):
        data = {'domain': domain, 'site': site}
        response = requests.post(f'{self.base_url}/orders', json=data, headers=self.headers)
        return response.json()
    
    def get_messages(self, order_id):
        response = requests.get(f'{self.base_url}/orders/{order_id}/messages', headers=self.headers)
        return response.json()
    
    def cancel_order(self, order_id):
        response = requests.delete(f'{self.base_url}/orders/{order_id}', headers=self.headers)
        return response.json()

# Usage
api = KopeechkaAPI('YOUR_API_KEY')

# Balance check
balance = api.get_balance()
print(f"Balance: {balance['data']['balance']}")

# Create order
order = api.create_order('example.com', 'instagram.com')
print(f"Order ID: {order['data']['orderId']}")
```

### cURL

```bash
# Balance check
curl -X GET "https://api.kopechka.com/api/v1/balance" \
  -H "X-API-Key: YOUR_API_KEY"

# Create order
curl -X POST "https://api.kopechka.com/api/v1/orders" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "domain": "example.com",
    "site": "site.com"
  }'

# Fetch messages
curl -X GET "https://api.kopechka.com/api/v1/orders/API-1234567890/messages" \
  -H "X-API-Key: YOUR_API_KEY"

# Cancel order
curl -X DELETE "https://api.kopechka.com/api/v1/orders/API-1234567890" \
  -H "X-API-Key: YOUR_API_KEY"
```

## Quick Start

1. **Get API Key**: Create your API key from the dashboard
2. **Send Your First Request**: Send requests to the API with the X-API-Key header
3. **Process Responses**: Use JSON formatted responses in your application

## Support

For questions about the API:
- Email: support@kopechka.com
- Website: https://kopechka.com

## License

This API documentation is provided by Kopeechka.
