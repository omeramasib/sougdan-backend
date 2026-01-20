# WhatsApp Business API Integration (AuthKey.io)

Complete implementation of WhatsApp Business API integration using AuthKey.io for your CodeIgniter 3 e-commerce platform.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Database Migration](#database-migration)
- [Admin Panel Setup](#admin-panel-setup)
- [API Endpoints](#api-endpoints)
- [Helper Functions](#helper-functions)
- [Usage Examples](#usage-examples)
- [Webhook Setup](#webhook-setup)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

---

## Features

✅ Send text messages via WhatsApp
✅ Send template-based messages
✅ Send media files (images, documents, videos, audio)
✅ Message status tracking (sent, delivered, read, failed)
✅ Conversation history management
✅ Webhook support for incoming messages
✅ Order notifications via WhatsApp
✅ OTP delivery via WhatsApp
✅ Admin panel for configuration
✅ RESTful API for mobile/web app integration
✅ Integration with existing notification system

---

## Installation

All required files have been created in your project:

### Created Files

```
application/
├── migrations/
│   └── 046_whatsapp_integration.php          # Database migration
├── libraries/
│   └── Whatsapp.php                           # WhatsApp API library
├── models/
│   └── Whatsapp_model.php                     # Database operations
├── controllers/
│   ├── admin/
│   │   └── Whatsapp_settings.php             # Admin settings controller
│   └── app/v1/
│       └── Whatsapp_api.php                  # Mobile API controller
├── views/
│   └── admin/pages/forms/
│       └── whatsapp-settings.php             # Admin settings view
├── helpers/
│   └── function_helper.php                   # Updated with WhatsApp functions
└── config/
    └── eshop.php                             # Updated with permissions
```

---

## Configuration

### Step 1: Get AuthKey.io Credentials

1. Sign up at [https://console.authkey.io/](https://console.authkey.io/)
2. Navigate to WhatsApp API section
3. Obtain the following credentials:
   - **API Key** (Bearer token)
   - **Phone Number ID** (Your WhatsApp Business phone number ID)
   - **Business Account ID** (Your WhatsApp Business account ID)
   - **Base URL** (Default: `https://api.authkey.io/v1/whatsapp/`)

### Step 2: Database Migration

Run the database migration to create required tables:

```bash
# Access your database and run the migration
# OR use CodeIgniter's migration system
```

The migration creates 3 tables:
- `whatsapp_messages` - Stores all sent/received messages
- `whatsapp_message_templates` - Stores message templates
- `whatsapp_webhook_events` - Stores webhook events

### Step 3: Set Permissions

Add admin user permissions for WhatsApp settings in your database:

```sql
-- The whatsapp-settings permission has been added to config/eshop.php
-- Ensure your admin user has access to this module
```

---

## Admin Panel Setup

### Accessing WhatsApp Settings

1. Log in to your admin panel
2. Navigate to: **Admin Dashboard > WhatsApp Settings**
3. Configure the following settings:

#### Required Settings

| Field | Description | Example |
|-------|-------------|---------|
| Enable Integration | Toggle to enable/disable WhatsApp | ✓ Enabled |
| Base URL | AuthKey.io API endpoint | `https://api.authkey.io/v1/whatsapp/` |
| API Key | Your AuthKey.io API key | `your-api-key-here` |
| Phone Number ID | WhatsApp Business phone ID | `1234567890` |
| Business Account ID | WhatsApp Business account ID | `9876543210` |
| Default Country Code | Default code for phone numbers | `91` (for India) |
| Webhook Verify Token | Token for webhook verification | `your-secure-token` |

### Testing Configuration

1. **Test Connection**: Click "Test Connection" button to verify API credentials
2. **Send Test Message**: Use the "Send Test Message" form to send a test WhatsApp message

---

## Database Migration

### Running the Migration

Execute the migration file:

```bash
# Using PHP CLI
php index.php migrate

# OR manually import SQL into your database
```

### Database Tables Structure

#### whatsapp_messages
```sql
- id (INT, PRIMARY KEY)
- phone_number (VARCHAR 20)
- message_type (ENUM: text, image, document, video, audio, template)
- content (LONGTEXT)
- template_name (VARCHAR 100)
- template_params (TEXT)
- wamid (VARCHAR 100) - WhatsApp message ID
- status (ENUM: pending, sent, delivered, read, failed)
- error_message (TEXT)
- related_entity_type (VARCHAR 50) - e.g., 'order', 'user'
- related_entity_id (INT)
- direction (ENUM: inbound, outbound)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)
```

#### whatsapp_message_templates
```sql
- id (INT, PRIMARY KEY)
- name (VARCHAR 100, UNIQUE)
- category (VARCHAR 50)
- language (VARCHAR 10)
- body_text (LONGTEXT)
- header_text (VARCHAR 255)
- footer_text (VARCHAR 255)
- button_configs (TEXT)
- approval_status (VARCHAR 50)
- is_active (TINYINT 1)
- created_at (TIMESTAMP)
```

#### whatsapp_webhook_events
```sql
- id (INT, PRIMARY KEY)
- event_type (VARCHAR 50)
- webhook_data (LONGTEXT)
- processed (TINYINT 1)
- created_at (TIMESTAMP)
```

---

## API Endpoints

Base URL: `https://yourdomain.com/app/v1/Whatsapp_api/`

All API endpoints require JWT authentication (except webhook).

### 1. Send Message

**Endpoint:** `POST /send_message`

**Request Body:**
```json
{
  "phone_number": "919876543210",
  "message": "Hello! Your order has been confirmed.",
  "related_entity_type": "order",
  "related_entity_id": 12345
}
```

**Response:**
```json
{
  "error": false,
  "message": "Message sent successfully",
  "data": {
    "success": true,
    "message_id": "wamid.xxxxx"
  }
}
```

### 2. Send Template Message

**Endpoint:** `POST /send_template_message`

**Request Body:**
```json
{
  "phone_number": "919876543210",
  "template_name": "order_confirmation",
  "template_params": {
    "1": "John Doe",
    "2": "ORD12345",
    "3": "$99.99"
  }
}
```

### 3. Send Media

**Endpoint:** `POST /send_media`

**Request Body:**
```json
{
  "phone_number": "919876543210",
  "media_type": "image",
  "media_url": "https://yourdomain.com/uploads/invoice.jpg",
  "caption": "Your order invoice"
}
```

### 4. Get Conversations

**Endpoint:** `GET /get_conversations?limit=20&offset=0`

**Response:**
```json
{
  "error": false,
  "message": "Conversations retrieved successfully",
  "total": 15,
  "data": [
    {
      "phone_number": "919876543210",
      "last_message_time": "2025-01-15 10:30:00",
      "message_count": 25,
      "unread_count": 3
    }
  ]
}
```

### 5. Load Conversation

**Endpoint:** `GET /load_conversation?phone_number=919876543210&limit=50&offset=0`

**Response:**
```json
{
  "error": false,
  "message": "Messages retrieved successfully",
  "total": 25,
  "data": [
    {
      "id": 1,
      "phone_number": "919876543210",
      "message_type": "text",
      "content": "Hello!",
      "status": "delivered",
      "direction": "outbound",
      "created_at": "2025-01-15 10:30:00"
    }
  ]
}
```

### 6. Get Message Status

**Endpoint:** `GET /get_message_status?message_id=wamid.xxxxx`

### 7. Mark as Read

**Endpoint:** `POST /mark_as_read`

**Request Body:**
```json
{
  "phone_number": "919876543210"
}
```

### 8. Get Unread Count

**Endpoint:** `GET /get_unread_count?phone_number=919876543210`

### 9. Get Templates

**Endpoint:** `GET /get_templates?active_only=1`

### 10. Get Messages by Entity

**Endpoint:** `GET /get_messages_by_entity?entity_type=order&entity_id=12345`

---

## Helper Functions

Use these helper functions anywhere in your CodeIgniter application:

### 1. send_whatsapp_message()

```php
$result = send_whatsapp_message(
    '919876543210',           // Phone number
    'Hello from our store!',  // Message
    null,                     // Template name (optional)
    []                        // Template params (optional)
);

if (!$result['error']) {
    echo "Message sent successfully!";
}
```

### 2. send_whatsapp_template()

```php
$result = send_whatsapp_template(
    '919876543210',
    'order_confirmation',
    ['John Doe', 'ORD12345', '$99.99']
);
```

### 3. send_whatsapp_media()

```php
$result = send_whatsapp_media(
    '919876543210',
    'image',
    'https://yourdomain.com/uploads/product.jpg',
    'Check out this product!'
);
```

### 4. send_whatsapp_order_notification()

```php
$result = send_whatsapp_order_notification(
    '919876543210',       // Customer phone
    'ORD12345',          // Order ID
    'order_placed',      // Notification type
    $order_data          // Order details (optional)
);
```

**Supported notification types:**
- `order_placed`
- `order_confirmed`
- `order_shipped`
- `order_delivered`
- `order_cancelled`
- `return_request`
- `refund_processed`

### 5. send_whatsapp_otp()

```php
$otp = '123456';
$result = send_whatsapp_otp('919876543210', $otp);
```

### 6. is_whatsapp_enabled()

```php
if (is_whatsapp_enabled()) {
    // WhatsApp integration is active
    send_whatsapp_message($phone, $message);
}
```

### 7. get_whatsapp_conversation()

```php
$messages = get_whatsapp_conversation('919876543210', 50);
```

---

## Usage Examples

### Example 1: Send Order Confirmation

```php
// In your order controller
$this->load->helper('function');

$order_id = 'ORD12345';
$customer_phone = '919876543210';

// Send WhatsApp notification
$result = send_whatsapp_order_notification(
    $customer_phone,
    $order_id,
    'order_placed',
    $order_details
);

if (!$result['error']) {
    log_message('info', 'WhatsApp order notification sent for order: ' . $order_id);
}
```

### Example 2: Send OTP During Registration

```php
// In your authentication controller
$this->load->helper('function');

$phone_number = $this->input->post('phone');
$otp = rand(100000, 999999);

// Store OTP in session/database
$this->session->set_userdata('registration_otp', $otp);

// Send OTP via WhatsApp
$result = send_whatsapp_otp($phone_number, $otp);

if (!$result['error']) {
    // OTP sent successfully
    echo json_encode(['success' => true, 'message' => 'OTP sent to WhatsApp']);
} else {
    // Fallback to SMS or email
    echo json_encode(['success' => false, 'message' => $result['message']]);
}
```

### Example 3: Send Invoice Document

```php
$this->load->helper('function');

$customer_phone = '919876543210';
$invoice_url = base_url('uploads/invoices/INV12345.pdf');

$result = send_whatsapp_media(
    $customer_phone,
    'document',
    $invoice_url,
    'Your order invoice - INV12345'
);
```

### Example 4: Integration with Order Status Update

```php
// In your order model or controller
public function update_order_status($order_id, $new_status) {
    // Update order status in database
    $this->db->where('id', $order_id);
    $this->db->update('orders', ['status' => $new_status]);

    // Get customer details
    $order = $this->get_order_details($order_id);
    $customer_phone = $order['customer_phone'];

    // Map status to notification type
    $notification_map = [
        'confirmed' => 'order_confirmed',
        'shipped' => 'order_shipped',
        'delivered' => 'order_delivered',
        'cancelled' => 'order_cancelled'
    ];

    // Send WhatsApp notification
    if (isset($notification_map[$new_status])) {
        send_whatsapp_order_notification(
            $customer_phone,
            $order_id,
            $notification_map[$new_status]
        );
    }
}
```

---

## Webhook Setup

### Configure Webhook in AuthKey.io Dashboard

1. Go to AuthKey.io dashboard
2. Navigate to WhatsApp API settings
3. Set webhook URL: `https://yourdomain.com/admin/whatsapp_settings/webhook`
4. Set verify token: (Same as configured in admin panel)
5. Save webhook configuration

### Webhook Endpoint

**URL:** `https://yourdomain.com/admin/whatsapp_settings/webhook`

**Methods:**
- **GET** - For webhook verification (automatic)
- **POST** - For receiving incoming messages and status updates

### Webhook Event Processing

The webhook automatically:
- ✅ Stores incoming messages in database
- ✅ Updates message delivery status
- ✅ Logs all webhook events
- ✅ Can trigger custom logic (extend as needed)

### Testing Webhook

```bash
# Test webhook verification (GET request)
curl "https://yourdomain.com/admin/whatsapp_settings/webhook?hub_verify_token=your-token&hub_challenge=test123"

# Should return: test123
```

---

## Testing

### 1. Test API Connection

```bash
# From admin panel
Click "Test Connection" button in WhatsApp Settings
```

### 2. Send Test Message

```bash
# Using cURL
curl -X POST "https://yourdomain.com/app/v1/Whatsapp_api/send_message" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "phone_number": "919876543210",
    "message": "Test message from API"
  }'
```

### 3. Test Helper Function

```php
// Create a test controller
<?php
class Whatsapp_test extends CI_Controller {
    public function send_test() {
        $this->load->helper('function');

        $result = send_whatsapp_message(
            '919876543210',
            'This is a test message!'
        );

        echo '<pre>';
        print_r($result);
        echo '</pre>';
    }
}
```

Access: `https://yourdomain.com/whatsapp_test/send_test`

---

## Troubleshooting

### Common Issues

#### 1. Message Not Sending

**Problem:** Messages fail to send

**Solutions:**
- ✅ Verify API key is correct
- ✅ Check phone number format (include country code, no spaces or special characters)
- ✅ Ensure WhatsApp integration is enabled in admin panel
- ✅ Check AuthKey.io account balance/credits
- ✅ Review error logs: `application/logs/`

#### 2. Webhook Not Working

**Problem:** Incoming messages not received

**Solutions:**
- ✅ Verify webhook URL is publicly accessible (no localhost)
- ✅ Check verify token matches in both admin panel and AuthKey.io
- ✅ Ensure SSL certificate is valid (HTTPS required)
- ✅ Check webhook logs in database: `whatsapp_webhook_events` table

#### 3. Phone Number Format Issues

**Problem:** Invalid phone number errors

**Solutions:**
- ✅ Use international format: `919876543210` (country code + number)
- ✅ Remove spaces, dashes, parentheses: `+91 98765-43210` → `919876543210`
- ✅ Set default country code in admin settings

#### 4. Message Status Not Updating

**Problem:** Status stuck on 'sent'

**Solutions:**
- ✅ Ensure webhook is properly configured
- ✅ Check webhook URL is receiving POST requests
- ✅ Verify AuthKey.io is sending status updates to your webhook

### Debug Mode

Enable CodeIgniter debug mode to see detailed errors:

```php
// In index.php
define('ENVIRONMENT', 'development');
```

Check logs:
```bash
tail -f application/logs/log-2025-01-*.php
```

### Support

For issues specific to:
- **AuthKey.io API**: Contact AuthKey.io support
- **This Integration**: Check implementation files and error logs
- **CodeIgniter**: Refer to CodeIgniter 3 documentation

---

## API Rate Limits

Be aware of AuthKey.io rate limits:
- Check your plan's message limits
- Implement queuing for bulk messages
- Use message templates for high-volume scenarios

---

## Security Considerations

1. **API Key Protection**
   - Store API keys securely in database (encrypted if possible)
   - Never expose keys in frontend code
   - Use environment variables in production

2. **Webhook Security**
   - Always verify webhook signature/token
   - Use HTTPS for webhook endpoint
   - Validate incoming webhook data

3. **Phone Number Validation**
   - Validate phone numbers before sending
   - Implement rate limiting to prevent abuse
   - Log all message attempts for audit

4. **Permission Control**
   - Restrict WhatsApp settings access to admin users only
   - Implement proper role-based access control
   - Monitor API usage for suspicious activity

---

## Advanced Integration

### Integrate with Existing Notification System

Update your notification sending logic:

```php
// In your notification function
function send_notification_to_customer($phone, $message, $type) {
    $notification_settings = get_settings('send_notification_settings', true);

    // Send via SMS
    if ($notification_settings['notification_via_sms'] == 1) {
        send_sms($phone, $message);
    }

    // Send via Email
    if ($notification_settings['notification_via_mail'] == 1) {
        send_email($email, $message);
    }

    // Send via WhatsApp
    if ($notification_settings['notification_via_whatsapp'] == 1 && is_whatsapp_enabled()) {
        send_whatsapp_message($phone, $message);
    }
}
```

### Queue System for Bulk Messages

For sending bulk messages, implement a queue:

```php
// Create a queue table and cron job to process messages
// Example queue processor
public function process_whatsapp_queue() {
    $this->load->model('Whatsapp_model');

    $pending_messages = $this->db->get_where('whatsapp_queue', ['status' => 'pending'], 10);

    foreach ($pending_messages->result_array() as $msg) {
        $result = send_whatsapp_message($msg['phone'], $msg['message']);

        // Update queue status
        $this->db->where('id', $msg['id']);
        $this->db->update('whatsapp_queue', [
            'status' => $result['error'] ? 'failed' : 'sent',
            'processed_at' => date('Y-m-d H:i:s')
        ]);
    }
}
```

---

## File Structure Reference

```
/Users/pc/Desktop/Code v3.0.0/
├── application/
│   ├── config/
│   │   └── eshop.php (updated with permissions)
│   ├── controllers/
│   │   ├── admin/
│   │   │   └── Whatsapp_settings.php
│   │   └── app/v1/
│   │       └── Whatsapp_api.php
│   ├── models/
│   │   └── Whatsapp_model.php
│   ├── views/
│   │   └── admin/pages/forms/
│   │       └── whatsapp-settings.php
│   ├── libraries/
│   │   └── Whatsapp.php
│   ├── helpers/
│   │   └── function_helper.php (updated)
│   └── migrations/
│       └── 046_whatsapp_integration.php
└── WHATSAPP_INTEGRATION_README.md (this file)
```

---

## Version History

- **v1.0.0** (2025-01-15) - Initial implementation
  - WhatsApp Business API integration with AuthKey.io
  - Admin panel configuration
  - RESTful API endpoints
  - Helper functions
  - Webhook support
  - Message tracking and status updates

---

## License & Credits

This integration is built for CodeIgniter 3 framework and uses AuthKey.io WhatsApp Business API.

**Important:** Ensure you comply with WhatsApp Business API policies and guidelines.

---

## Next Steps

1. ✅ Run database migration
2. ✅ Configure settings in admin panel
3. ✅ Test connection
4. ✅ Configure webhook in AuthKey.io
5. ✅ Send test message
6. ✅ Integrate with your order/notification flow
7. ✅ Monitor message logs and status

---

**Need Help?** Review the files in your project or contact your development team.
