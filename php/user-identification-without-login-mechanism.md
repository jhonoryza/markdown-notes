# User Identification without Login Mechanism

## Cookie-Based Approach

### How It Works with Cookies in Frontend and Backend

The backend sets a cookie with specific information, such as a token or flag,
when the user claims a reward. This cookie is stored in the user's browser.

The frontend does not need to explicitly manage the cookie, as the browser
automatically includes cookies in requests to the same domain (if configured
correctly).

### Pros and Cons of the Cookie-Based Approach

**Pros:**

- Simple to implement.
- The browser automatically handles cookies.

**Cons:**

- Cookies can be deleted by users.
- Less effective in incognito mode or across different devices.

### Example Using `fetch`

```javascript
const response = await fetch("https://backend-domain.com/api/claim-reward", {
    method: "POST",
    credentials: "include", // Enables credential (cookie) sharing
    headers: {
        "Content-Type": "application/json",
    },
});
```

### Example Using `axios`

```javascript
const response = await axios.post(
    "https://backend-domain.com/api/claim-reward",
    {},
    { withCredentials: true }, // Enables credential sharing
);
```

### Differences Between `fetch` and `axios` for Credential Management

| Feature                  | Fetch                                | Axios                                |
| ------------------------ | ------------------------------------ | ------------------------------------ |
| Credential Configuration | Uses `credentials` (e.g., `include`) | Uses `withCredentials: true`         |
| Default Behavior         | `same-origin`                        | Does not send credentials            |
| Global Configuration     | Requires wrapping `fetch`            | Can be set globally (via `defaults`) |
| Browser Support          | Native to modern browsers            | Requires an external library         |

### Key Considerations

1. **HTTPS and SameSite:**
   - If using `SameSite=None` for cookies, the backend must operate over HTTPS.
     Modern browsers require this for security.

2. **CORS:**
   - The backend must be configured to support CORS with credentials
     (`supports_credentials: true`).

3. **Incognito Mode:**
   - Some browsers may limit or delete cookies in incognito mode, affecting
     cookie-based mechanisms.

4. **Alternatives:**
   - To avoid manual credential handling, libraries like `axios` may be more
     convenient than `fetch`.

### PHP Example for Reward Claim

```php
$cookieName = 'reward_claimed';

// Check if the cookie already exists
if ($request->cookie($cookieName)) {
    return response()->json([
        'message' => 'You have already claimed the reward!'
    ], 403);
}

$response = response()->json([
    'message' => 'Reward claimed successfully!'
]);

return $response->cookie(
    $cookieName, 
    true, 
    60 * 24 * 30, // 30 days
    null, 
    null, 
    true, // Secure
    true, // HttpOnly
    false, // Raw
    'None' // SameSite
);
```

## CORS Configuration

```php
return [
    'paths' => ['api/*', '/csrf-token'],

    'allowed_methods' => ['*'],

    'allowed_origins' => ['http://localhost:3000'], // Frontend domain

    'allowed_origins_patterns' => [],

    'allowed_headers' => ['*'],

    'exposed_headers' => [],

    'max_age' => 0,

    'supports_credentials' => true, // Important to allow credentials
];
```

## Combining Cookies and Backend Session ID

### Step 1: Frontend Accesses `/csrf-token` (Initial Request)

- When the frontend is first opened, it sends a GET request to the `/csrf-token`
  endpoint to obtain a CSRF token.
- The backend starts a session if it has not already and includes a
  `backend_session` cookie in the response.
- The backend returns the CSRF token in a JSON response.

### Step 2: Frontend Stores the CSRF Token

- After receiving the response, the frontend stores the CSRF token for
  subsequent requests.

### Step 3: Frontend Sends Reward Claim Request

- When the user submits the reward claim form, the frontend sends a POST request
  to the `/claim-reward` API.
- The previously received CSRF token is included in the `X-CSRF-TOKEN` header.
- The `backend_session` cookie containing the user's session ID is automatically
  sent with the request because it was set by the backend and
  `withCredentials: true` is used in the frontend.

### Session Configuration in Laravel

`config/session.php`

```php
return [
    'driver' => 'cookie',
    'lifetime' => 10080, // Session duration in minutes (7 days)
    'expire_on_close' => false, // Delete session cookie on browser close
    'cookie' => 'backend_session',
    'secure' => env('SESSION_SECURE_COOKIE', false), // Use `true` if the app runs over HTTPS
    'same_site' => 'none', // Prevent cross-site CSRF
    'http_only' => true, // Cookie accessible only via HTTP
];
```

### Reward Claim Endpoint in Laravel

`web/routes.php`

```php
Route::post('/claim-reward', function () {
    $sessionId = session()->getId();

    $period = Period::where('name', 'current')->first(); 
    if (!$period) {
        return response()->json(['message' => 'Period not found.'], 404);
    }

    $endAt = Carbon::parse($period->end_at);
    $duration = $endAt->diffInMinutes(Carbon::now());

    if (session()->has('claimed')) {
        return response()->json(['message' => 'You have already claimed.'], 403);
    }

    session(['claimed' => true]);

    $cookieLifetime = max($duration, 1); // Ensure duration is not negative, set a minimum of 1 minute
    $cookie = cookie('backend_session', session()->getId(), $cookieLifetime);

    return response()->json(['message' => 'Claim successful!'])->withCookie($cookie);
});
```
