# OAuth Flow #1 - Web Server Flow

## Use Case: User-Interactive Applications
**Best for:** Web applications where users log in through a browser

🔗 **[Official Salesforce Documentation](https://help.salesforce.com/s/articleView?id=xcloud.remoteaccess_oauth_web_server_flow.htm&type=5)**

---

## The Process (Authorization Code Flow)
```
1. App requests authorization code from Salesforce
2. User is redirected to Salesforce login page
3. User enters credentials + MFA
4. User approves app access (if not previously approved)
5. Salesforce returns authorization code (expires in 15 minutes)
6. App exchanges authorization code for access token
7. Salesforce returns access token + refresh token
8. App uses access token to access Salesforce data
```

## Key Features:
- ✅ Most secure for user-facing apps
- ✅ User explicitly grants permission
- ✅ **Provides refresh token** for long-term access
- ✅ User can revoke access anytime
- ✅ Recommended to use with **PKCE** (Proof Key for Code Exchange)

---

## When to Use This Flow

### ✅ Perfect For:
- Web applications with user interfaces
- Mobile applications (use with PKCE)
- Single Page Applications (SPAs)
- Any app where users authenticate themselves interactively

### ❌ Not Recommended For:
- Server-to-server integrations (use JWT or Client Credentials)
- Scheduled background jobs (use JWT)
- Automated data synchronization (use JWT)

---

## Implementation Steps

### Step 1: Request Authorization Code

Redirect user to Salesforce authorization endpoint:

```
https://login.salesforce.com/services/oauth2/authorize?
  response_type=code&
  client_id=YOUR_CLIENT_ID&
  redirect_uri=YOUR_REDIRECT_URI&
  state=RANDOM_STATE_STRING
```

**Key Parameters:**
- `response_type=code` - Required for web server flow
- `client_id` - Consumer key from Connected App
- `redirect_uri` - Must match Connected App callback URL (URL encoded)
- `state` - Random string to prevent CSRF attacks (recommended)

**Optional Parameters:**
- `scope` - Specific permissions (if omitted, all assigned scopes requested)
- `code_challenge` - For PKCE implementation (recommended for mobile)
- `prompt=login` - Force reauthentication

### Step 2: User Authenticates & Approves

User logs into Salesforce and approves app access (if not previously approved).

### Step 3: Receive Authorization Code

Salesforce redirects to your callback URL with authorization code:

```
https://www.yourapp.com/oauth2/callback?code=aPrx4sgoM2Nd1zWeFVlO...
```

**Important:** Authorization code expires in **15 minutes**.

### Step 4: Exchange Code for Access Token

**POST to token endpoint:**

```http
POST /services/oauth2/token HTTP/1.1
Host: login.salesforce.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTHORIZATION_CODE&
client_id=YOUR_CLIENT_ID&
client_secret=YOUR_CLIENT_SECRET&
redirect_uri=YOUR_REDIRECT_URI
```

**Security Note:** Always pass `client_secret` in POST body or header, **NEVER in URL query string**.

**Alternative - HTTP Basic Authentication:**
```http
POST /services/oauth2/token HTTP/1.1
Host: login.salesforce.com
Authorization: Basic Base64Encode(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTHORIZATION_CODE&
redirect_uri=YOUR_REDIRECT_URI
```

### Step 5: Receive Access Token Response

```json
{
  "access_token": "00D50000000IZ3Z!AQcAQH0dMHZ...",
  "refresh_token": "5Aep861TSESvWeug_xvFHRBTTbf...",
  "instance_url": "https://yourInstance.salesforce.com",
  "id": "https://login.salesforce.com/id/00D50000000IZ3Z/005...",
  "token_type": "Bearer",
  "issued_at": "1609459200000",
  "signature": "xyz123...",
  "scope": "web openid api"
}
```

**Response Parameters:**
- `access_token` - Use for API calls (short-lived)
- `refresh_token` - Use to get new access tokens (long-lived)
- `instance_url` - Your Salesforce instance URL
- `scope` - Granted permissions

---

## Using the Access Token

Include access token in Authorization header:

```http
GET /services/data/v58.0/sobjects/Account HTTP/1.1
Host: yourInstance.salesforce.com
Authorization: Bearer 00D50000000IZ3Z!AQcAQH0dMHZ...
```

---

## Refreshing Access Tokens

When access token expires, use refresh token to get a new one:

```http
POST /services/oauth2/token HTTP/1.1
Host: login.salesforce.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&
refresh_token=YOUR_REFRESH_TOKEN&
client_id=YOUR_CLIENT_ID&
client_secret=YOUR_CLIENT_SECRET
```

**Note:** Refresh tokens are long-lived and must be stored securely.

---

## Security Best Practices

### ✅ Do This:
- **Use HTTPS** for all redirect URIs
- **Implement PKCE** for mobile apps and public clients
- **Validate state parameter** to prevent CSRF attacks
- **Store tokens securely** (encrypted, never in client-side code)
- **Use refresh tokens** instead of storing user credentials
- **Pass sensitive data in POST body**, never in URL query strings

### ❌ Avoid This:
- Exposing client secrets in client-side code
- Using insecure (HTTP) redirect URIs
- Storing tokens in browser local storage without encryption
- Ignoring authorization code expiration (15 minutes)
- Not validating redirect URI matches exactly

---

## Common Errors & Solutions

| Error Code | Cause | Solution |
|------------|-------|----------|
| `invalid_client_id` | Client ID is incorrect | Verify Consumer Key from Connected App |
| `invalid_client` | Client secret is incorrect | Verify Consumer Secret from Connected App |
| `invalid_grant` | Authorization code expired or invalid | Code expires in 15 minutes - request new authorization |
| `redirect_uri_mismatch` | Redirect URI doesn't match | Ensure exact match with Connected App callback URL |
| `unauthorized_client` | Client not authorized for this grant | Enable OAuth settings in Connected App |

---

## PKCE (Proof Key for Code Exchange)

**Recommended for mobile apps and public clients** to prevent authorization code interception.

### Add to Authorization Request:
```
code_challenge=BASE64URL(SHA256(code_verifier))
```

### Add to Token Request:
```
code_verifier=RANDOM_STRING_WITH_HIGH_ENTROPY
```

Salesforce validates that the `code_challenge` matches the `code_verifier`.

---

## Key Takeaways

- ✅ **Most secure flow** for user-interactive applications
- ✅ **Provides refresh tokens** for long-term access without storing credentials
- ✅ **Authorization code expires in 15 minutes** - exchange it quickly
- ✅ **Use PKCE** for additional security in mobile/public clients
- ✅ **Never expose client secrets** in client-side code
- ✅ **Always use HTTPS** and validate state parameter
