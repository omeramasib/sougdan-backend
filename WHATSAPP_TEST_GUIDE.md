# WhatsApp Integration Test Guide - AuthKey.io

## Your Configuration

**WhatsApp ID:** 84319
**Test Mobile Number:** 966510017016 (Saudi Arabia)
**Country Code:** 966
**Mobile:** 510017016

---

## ⚠️ REQUIRED INFORMATION

Before testing, I need the following from your AuthKey.io dashboard:

### 1. **Authkey** (Required)
- Go to: https://console.authkey.io/dashboard
- Find your API authentication key
- It should be a long alphanumeric string

### 2. **Template ID (wid)** (Required)
- Go to: https://console.authkey.io/dashboard/whatsapp-templates
- Create a template or find an existing one
- Note the template ID number (e.g., 101, 102, etc.)
- Example template: "Hello {#name#}, your OTP is {#otp#}"

### 3. **Template Variables** (If applicable)
- List the variable names in your template (e.g., name, otp, order_id)

---

## 🧪 Test Curl Requests

### Test 1: Check Account Balance

This verifies your authkey is correct:

```bash
curl "https://console.authkey.io/restapi/getbalance.php?authkey=YOUR_AUTHKEY_HERE"
```

**Example Response:**
```
Balance: 500 Credits
```

---

### Test 2: Send Simple Template Message (GET Method)

**Without Template Variables:**

```bash
curl "https://console.authkey.io/request?authkey=YOUR_AUTHKEY_HERE&mobile=510017016&country_code=966&wid=YOUR_TEMPLATE_ID"
```

**With Template Variables (e.g., name and OTP):**

```bash
curl "https://console.authkey.io/request?authkey=YOUR_AUTHKEY_HERE&mobile=510017016&country_code=966&wid=YOUR_TEMPLATE_ID&name=John&otp=123456"
```

**Example Response:**
```json
{
  "status": "success",
  "message": "Message sent successfully",
  "message_id": "wamid.xxxxx"
}
```

---

### Test 3: Send Template Message (POST Method - JSON)

```bash
curl -X POST "https://console.authkey.io/restapi/requestjson.php" \
  -H "Authorization: YOUR_AUTHKEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "country_code": "966",
    "mobile": "510017016",
    "wid": "YOUR_TEMPLATE_ID",
    "type": "text",
    "bodyValues": {
      "var1": "John Doe",
      "var2": "123456"
    }
  }'
```

---

### Test 4: Send Media Message (Image/Document/PDF)

**GET Method:**

```bash
curl "https://console.authkey.io/restapi/request.php?authkey=YOUR_AUTHKEY_HERE&mobile=510017016&country_code=966&wid=YOUR_MEDIA_TEMPLATE_ID&template_type=media&HeaderData=https://example.com/image.jpg&headerFileName=invoice.jpg"
```

**POST Method:**

```bash
curl -X POST "https://console.authkey.io/restapi/requestjson.php" \
  -H "Authorization: YOUR_AUTHKEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "country_code": "966",
    "mobile": "510017016",
    "wid": "YOUR_MEDIA_TEMPLATE_ID",
    "type": "media",
    "headerValues": {
      "headerData": "https://example.com/invoice.pdf",
      "headerFileName": "invoice.pdf"
    }
  }'
```

---

## 🔧 PHP Test Script

Create a test file: `/Users/pc/Desktop/Code v3.0.0/test_whatsapp.php`

```php
<?php
// Configuration
$authkey = "YOUR_AUTHKEY_HERE";
$mobile = "510017016";
$country_code = "966";
$template_id = "YOUR_TEMPLATE_ID"; // e.g., 101

// Test 1: Check Balance
echo "=== Test 1: Check Balance ===\n";
$balance_url = "https://console.authkey.io/restapi/getbalance.php?authkey=" . $authkey;
$balance = file_get_contents($balance_url);
echo "Balance: " . $balance . "\n\n";

// Test 2: Send Template Message (GET)
echo "=== Test 2: Send Template Message (GET) ===\n";
$message_url = "https://console.authkey.io/request?" . http_build_query([
    'authkey' => $authkey,
    'mobile' => $mobile,
    'country_code' => $country_code,
    'wid' => $template_id,
    // Add your template variables here
    // 'name' => 'John',
    // 'otp' => '123456'
]);
$response = file_get_contents($message_url);
echo "Response: " . $response . "\n\n";

// Test 3: Send Template Message (POST JSON)
echo "=== Test 3: Send Template Message (POST JSON) ===\n";
$post_url = "https://console.authkey.io/restapi/requestjson.php";
$post_data = [
    'country_code' => $country_code,
    'mobile' => $mobile,
    'wid' => $template_id,
    'type' => 'text',
    'bodyValues' => [
        'var1' => 'Test User',
        'var2' => '123456'
    ]
];

$ch = curl_init($post_url);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($post_data));
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Authorization: ' . $authkey,
    'Content-Type: application/json'
]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$post_response = curl_exec($ch);
curl_close($ch);

echo "Response: " . $post_response . "\n\n";
?>
```

**Run the test:**
```bash
php test_whatsapp.php
```

---

## 🎯 CodeIgniter Integration Test

Create a test controller: `/Users/pc/Desktop/Code v3.0.0/application/controllers/Test_whatsapp.php`

```php
<?php
defined('BASEPATH') or exit('No direct script access allowed');

class Test_whatsapp extends CI_Controller
{
    public function __construct()
    {
        parent::__construct();
        $this->load->library('Whatsapp');
        $this->load->helper('function');
    }

    /**
     * Test 1: Check if WhatsApp is enabled
     * URL: http://yourdomain.com/test_whatsapp/check_status
     */
    public function check_status()
    {
        echo "<h2>WhatsApp Integration Status</h2>";

        if ($this->whatsapp->is_enabled()) {
            echo "<p style='color:green'>✓ WhatsApp integration is ENABLED</p>";

            $credentials = $this->whatsapp->get_credentials();
            echo "<pre>";
            print_r($credentials);
            echo "</pre>";
        } else {
            echo "<p style='color:red'>✗ WhatsApp integration is DISABLED</p>";
            echo "<p>Please configure settings in admin panel first.</p>";
        }
    }

    /**
     * Test 2: Check Balance
     * URL: http://yourdomain.com/test_whatsapp/check_balance
     */
    public function check_balance()
    {
        echo "<h2>Check AuthKey.io Balance</h2>";

        $result = $this->whatsapp->check_balance();

        echo "<pre>";
        print_r($result);
        echo "</pre>";
    }

    /**
     * Test 3: Send Template Message
     * URL: http://yourdomain.com/test_whatsapp/send_test
     *
     * IMPORTANT: Replace YOUR_TEMPLATE_ID with actual template ID
     */
    public function send_test()
    {
        echo "<h2>Send Test WhatsApp Message</h2>";

        $phone = "966510017016"; // Your test number
        $template_id = "YOUR_TEMPLATE_ID"; // Replace with actual template ID from AuthKey.io

        // Template parameters (customize based on your template)
        $params = [
            'name' => 'Test User',
            'otp' => '123456'
            // Add more parameters as needed based on your template
        ];

        $result = $this->whatsapp->send_template_message($phone, $template_id, $params);

        echo "<h3>Result:</h3>";
        echo "<pre>";
        print_r($result);
        echo "</pre>";

        if (!isset($result['error']) || !$result['error']) {
            echo "<p style='color:green'>✓ Message sent successfully!</p>";
        } else {
            echo "<p style='color:red'>✗ Failed to send message</p>";
            echo "<p>Error: " . $result['message'] . "</p>";
        }
    }

    /**
     * Test 4: Send using helper function
     * URL: http://yourdomain.com/test_whatsapp/test_helper
     */
    public function test_helper()
    {
        echo "<h2>Test Helper Function</h2>";

        if (!is_whatsapp_enabled()) {
            echo "<p style='color:red'>WhatsApp is not enabled</p>";
            return;
        }

        $phone = "966510017016";
        $template_id = "YOUR_TEMPLATE_ID"; // Replace with actual template ID
        $params = ['name' => 'John', 'otp' => '999888'];

        $result = send_whatsapp_template($phone, $template_id, $params);

        echo "<pre>";
        print_r($result);
        echo "</pre>";
    }

    /**
     * Test 5: View message history
     * URL: http://yourdomain.com/test_whatsapp/message_history
     */
    public function message_history()
    {
        $this->load->model('Whatsapp_model');

        echo "<h2>WhatsApp Message History</h2>";

        $messages = $this->Whatsapp_model->get_conversation("966510017016", 50);

        if (empty($messages)) {
            echo "<p>No messages found</p>";
        } else {
            echo "<table border='1' cellpadding='10'>";
            echo "<tr><th>ID</th><th>Type</th><th>Content</th><th>Status</th><th>Direction</th><th>Created</th></tr>";

            foreach ($messages as $msg) {
                echo "<tr>";
                echo "<td>" . $msg['id'] . "</td>";
                echo "<td>" . $msg['message_type'] . "</td>";
                echo "<td>" . htmlspecialchars(substr($msg['content'], 0, 50)) . "...</td>";
                echo "<td>" . $msg['status'] . "</td>";
                echo "<td>" . $msg['direction'] . "</td>";
                echo "<td>" . $msg['created_at'] . "</td>";
                echo "</tr>";
            }

            echo "</table>";
        }
    }
}
```

---

## 📋 Step-by-Step Testing Process

### Step 1: Configure Admin Panel

1. Access admin panel: `http://yourdomain.com/admin/whatsapp_settings`
2. Fill in the settings:
   - **Enable Integration**: ✓ Checked
   - **Base URL**: `https://console.authkey.io/`
   - **Authkey**: [YOUR_AUTHKEY_FROM_DASHBOARD]
   - **WhatsApp ID**: `84319`
   - **Default Country Code**: `966`
3. Click "Save Settings"

### Step 2: Test Connection

1. In admin panel, click "Test Connection" button
2. Should return your account balance

### Step 3: Create a Template (if not exists)

1. Go to: https://console.authkey.io/dashboard/whatsapp-templates
2. Create a new template, for example:
   ```
   Template Name: test_otp
   Template Content: Hello {#name#}, your OTP is {#otp#}. Valid for 10 minutes.
   ```
3. Wait for template approval (Meta/WhatsApp approval)
4. Note the template ID (wid)

### Step 4: Run PHP Tests

Access these URLs in your browser:

1. **Check Status**
   `http://yourdomain.com/test_whatsapp/check_status`

2. **Check Balance**
   `http://yourdomain.com/test_whatsapp/check_balance`

3. **Send Test Message** (Update template ID first!)
   `http://yourdomain.com/test_whatsapp/send_test`

4. **Test Helper Function**
   `http://yourdomain.com/test_whatsapp/test_helper`

5. **View Message History**
   `http://yourdomain.com/test_whatsapp/message_history`

---

## 🔍 Troubleshooting

### Issue 1: "Authkey is required" error
**Solution:** Make sure you've entered the authkey in admin panel and enabled integration

### Issue 2: "Template not found" error
**Solution:** Verify template ID (wid) exists in your AuthKey.io dashboard

### Issue 3: "Template not approved" error
**Solution:** Wait for WhatsApp to approve your template (can take 24-48 hours)

### Issue 4: Message not delivered
**Solution:**
- Verify phone number has opted in to receive WhatsApp messages from your business
- Check if phone number has WhatsApp installed
- Verify template is approved

### Issue 5: No response from API
**Solution:**
- Check if authkey is correct
- Verify account has credits/balance
- Check server can make outbound HTTPS requests

---

## 📞 Next Steps After Testing

1. ✅ Verify all tests pass
2. ✅ Integrate into your order notification system
3. ✅ Set up webhook for incoming messages (if needed)
4. ✅ Monitor message delivery in database
5. ✅ Configure notification preferences

---

## 📝 Required Information Checklist

Before you can test, please provide:

- [ ] **Authkey** - Your AuthKey.io authentication key
- [ ] **Template ID (wid)** - At least one approved template ID
- [ ] **Template Variables** - List of variables in your template
- [ ] **Media Template ID** (Optional) - If you want to test media messages

---

## 💡 Quick Start Command

Replace placeholders and run:

```bash
# Check balance
curl "https://console.authkey.io/restapi/getbalance.php?authkey=YOUR_AUTHKEY"

# Send test message
curl "https://console.authkey.io/request?authkey=YOUR_AUTHKEY&mobile=510017016&country_code=966&wid=YOUR_TEMPLATE_ID"
```

---

**Once you provide the authkey and template ID, I can give you the exact curl commands to test immediately!**
