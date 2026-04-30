## HTTP and REST APIs

Many modules need to communicate with external services — payment gateways, email providers, analytics platforms, AI services, social media APIs, and more. This chapter covers how to make HTTP requests, handle responses, store credentials securely, and — critically — how to disclose external connections to site administrators and users.

### Disclosing External Service Connections

> **Note:** **This is a requirement, not a suggestion.** If your module sends data to or receives data from any external server, you must clearly disclose this to the site administrator. This is essential for GDPR compliance, privacy regulations, and user trust.

When your module connects to an external service, you must:

1. **Document it in your module help.** In your `GetHelp()` output or `docs/help.inc` file, clearly list every external service your module connects to, including:
   - The service name and URL (e.g., "Stripe API at https://api.stripe.com").
   - What data is sent (e.g., "Customer email, order total, and payment card token").
   - When the connection is made (e.g., "On checkout form submission").
   - A link to the service's privacy policy and terms of service.
2. **Display a notice in the admin panel.** On your module's settings page, include a visible disclaimer:

```html
&lt;div class="pageoverflow"&gt;
  &lt;p class="note"&gt;
    &lt;strong&gt;External Service Disclaimer:&lt;/strong&gt; This module connects to the
    Stripe payment processing service (https://stripe.com) to process payments.
    Customer payment information is transmitted securely to Stripe's servers.
    By using this module, you agree to Stripe's
    &lt;a href="https://stripe.com/privacy" target="_blank"&gt;Privacy Policy&lt;/a&gt; and
    &lt;a href="https://stripe.com/legal" target="_blank"&gt;Terms of Service&lt;/a&gt;.
  &lt;/p&gt;
&lt;/div&gt;
```

3. **Never send data to external services without the admin's knowledge.** If your module has optional integrations (e.g., analytics tracking), make them opt-in, not opt-out.
4. **Document it in your README.** Include an "External Services" section in your module's README or Forge description.

#### What counts as an external connection?

- REST API calls to third-party services (payment, email, SMS, etc.).
- Loading scripts, fonts, or images from CDNs or external domains.
- Sending analytics or telemetry data.
- License verification or update checks to your own server.
- Embedding iframes from external services.

### The cms\_http\_request Class

CMSMS provides the `cms_http_request` class for making HTTP requests. It uses cURL when available and falls back to fsockopen.

#### GET request

```
$req = new cms_http_request();
$req->setTimeout(15);
$req->setTarget('https://api.example.com/holidays');
$req->setMethod('GET');
$req->addRequestHeader('Authorization: Bearer ' . $api_key);
$req->addRequestHeader('Accept: application/json');
$req->execute();

$status = $req->getStatus();
$body = $req->getResult();

if ($status == 200) {
    $data = json_decode($body, true);
    // Process $data...
} else {
    $error = $req->getError();
    // Handle error...
}
```

#### POST request with JSON body

```php
$req = new cms_http_request();
$req->setTimeout(30);
$req->setTarget('https://api.example.com/orders');
$req->setMethod('POST');
$req->addRequestHeader('Authorization: Bearer ' . $api_key);
$req->addRequestHeader('Content-Type: application/json');
$req->setRawPostData(json_encode([
    'customer_email' => $email,
    'amount' => $total,
    'currency' => 'usd',
]));
$req->execute();

$status = $req->getStatus();
$body = $req->getResult();

if ($status >= 200 && $status < 300) {
    $response = json_decode($body, true);
    // Success
} else {
    // API error
    audit('', $this->GetName(), 'API call failed: HTTP ' . $status);
}
```

#### POST request with form parameters

```
$req = new cms_http_request();
$req->setTarget('https://api.example.com/token');
$req->setMethod('POST');
$req->setParams([
    'grant_type' => 'client_credentials',
    'client_id' => $client_id,
    'client_secret' => $client_secret,
]);
$req->execute();
```

### Using PHP cURL Directly

For more control (PUT, DELETE, PATCH methods, file uploads, streaming), you can use PHP's cURL functions directly:

```
$ch = curl_init();
curl_setopt_array($ch, [
    CURLOPT_URL => 'https://api.example.com/items/' . $item_id,
    CURLOPT_CUSTOMREQUEST => 'DELETE',
    CURLOPT_HTTPHEADER => [
        'Authorization: Bearer ' . $api_key,
        'Accept: application/json',
    ],
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_TIMEOUT => 15,
]);

$result = curl_exec($ch);
$status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
$error = curl_error($ch);
curl_close($ch);

if ($status == 204) {
    // Successfully deleted
}
```

### Storing API Credentials Securely

Never hardcode API keys, secrets, or tokens in your source code. Store them as module preferences:

```php
// Save from settings form
$this->SetPreference('api_key', trim($params['api_key']));
$this->SetPreference('api_secret', trim($params['api_secret']));

// Retrieve when needed
$api_key = $this->GetPreference('api_key', '');
$api_secret = $this->GetPreference('api_secret', '');
```
> **Note:** Module preferences are stored in the database, which is more secure than hardcoding in files. However, they are not encrypted. For highly sensitive credentials, consider using environment variables or a dedicated secrets manager if your hosting supports it.

### Error Handling and Timeouts

- **Always set a timeout.** External services can be slow or unresponsive. A 15–30 second timeout prevents your page from hanging indefinitely.
- **Check the HTTP status code.** Don't assume success — check for 2xx status codes.
- **Log failures.** Use `audit()` to log API errors for debugging.
- **Fail gracefully.** If an API call fails, show a user-friendly message rather than a PHP error. Don't let a third-party outage break your site.
- **Retry with backoff.** For background jobs, implement retry logic with exponential backoff rather than hammering a failing service.

```php
$req = new cms_http_request();
$req->setTimeout(15);
$req->setTarget($api_url);
$req->execute();

$status = $req->getStatus();
if ($status == 0) {
    // Connection failed entirely (timeout, DNS failure, etc.)
    audit('', $this->GetName(), 'API connection failed: ' . $req->getError());
    $this->SetError($this->Lang('error_api_unavailable'));
    $this->RedirectToAdminTab();
} elseif ($status >= 400) {
    // API returned an error
    $error_body = json_decode($req->getResult(), true);
    $message = $error_body['message'] ?? 'Unknown API error (HTTP ' . $status . ')';
    audit('', $this->GetName(), 'API error: ' . $message);
}
```

### Caching API Responses

If your module displays data from an external API on the frontend, cache the response to avoid making an API call on every page load:

```php
$cache_key = 'api_holidays_' . md5($api_url);
$cached = $this->GetPreference($cache_key, '');
$cache_time = (int) $this->GetPreference($cache_key . '_time', 0);

// Cache for 1 hour
if ($cached && (time() - $cache_time) < 3600) {
    $data = json_decode($cached, true);
} else {
    // Make the API call
    $req = new cms_http_request();
    $req->setTimeout(10);
    $req->setTarget($api_url);
    $req->execute();

    if ($req->getStatus() == 200) {
        $data = json_decode($req->getResult(), true);
        $this->SetPreference($cache_key, $req->getResult());
        $this->SetPreference($cache_key . '_time', time());
    } else {
        $data = $cached ? json_decode($cached, true) : []; // Use stale cache as fallback
    }
}
```

### Receiving Webhooks

Some APIs send data to your site via webhooks (e.g., Stripe payment confirmations, GitHub push events). Handle these with a module action that suppresses admin output:

```
// action.webhook.php
if (!defined('CMS_VERSION')) exit;

// Read the raw POST body
$payload = file_get_contents('php://input');
$data = json_decode($payload, true);

// Verify the webhook signature (service-specific)
$signature = $_SERVER['HTTP_X_WEBHOOK_SIGNATURE'] ?? '';
$expected = hash_hmac('sha256', $payload, $webhook_secret);

if (!hash_equals($expected, $signature)) {
    http_response_code(403);
    echo json_encode(['error' => 'Invalid signature']);
    exit;
}

// Process the webhook
try {
    // Handle the event...
    http_response_code(200);
    echo json_encode(['status' => 'ok']);
} catch (\Exception $e) {
    http_response_code(500);
    echo json_encode(['error' => $e->getMessage()]);
}
exit;
```

### cms\_http\_request API Reference

| Method | Description |
| --- | --- |
| `setTarget($url)` | Set the target URL |
| `setMethod($method)` | Set HTTP method ('GET' or 'POST') |
| `setTimeout($seconds)` | Set timeout in seconds |
| `setParams($array)` | Set request parameters (for form-encoded POST or GET query string) |
| `addParam($name, $value)` | Add a single request parameter |
| `setRawPostData($string)` | Set raw POST body (for JSON, XML, etc.) |
| `addRequestHeader($string)` | Add a custom HTTP header |
| `setAuth($username, $password)` | Set HTTP Basic Authentication credentials |
| `setUseragent($agent)` | Set the User-Agent string |
| `followRedirects($bool)` | Whether to follow HTTP redirects (default: true) |
| `execute()` | Execute the request. Returns the response body. |
| `getResult()` | Get the response body |
| `getStatus()` | Get the HTTP status code (200, 404, 500, etc.) |
| `getHeaders()` | Get the response headers |
| `getError()` | Get the last error message |
| `clear()` | Reset all properties for a new request |
