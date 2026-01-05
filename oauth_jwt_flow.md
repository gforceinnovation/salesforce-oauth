# OAuth Flow #2 - JWT (JSON Web Token) Flow

## Use Case: Server-to-Server Integration
**Best for:** Trusted backend integrations, no user interaction needed

## The Process:
```
1. Pre-authorized certificate stored securely
2. App creates JWT (signed with certificate)
3. App sends JWT to Salesforce
4. Salesforce validates signature & certificate
5. Salesforce returns access token
6. App uses token to access data
```

## Hotel Analogy:
- **VIP Guest with Pre-Authorization**
- Hotel already has your profile & preferences
- Security recognizes your credentials instantly
- Automatic check-in → Key card issued immediately
- No front desk interaction needed

## Key Features:
- ✅ No user interaction required
- ✅ Certificate-based security (very secure)
- ✅ Perfect for automated jobs
- ✅ Pre-authorized access
- ⚠️ Requires certificate management

---

## JWT Flow - Diagram

```
┌──────────────────┐                          ┌────────────────┐
│  Backend System  │                          │   Salesforce   │
│  (with Cert)     │                          │                │
└────────┬─────────┘                          └────────┬───────┘
         │                                             │
         │ 1. Create JWT (signed with cert)           │
         │                                             │
         │ 2. Send JWT to Salesforce                  │
         ├────────────────────────────────────────────>│
         │                                             │
         │                                   3. Validate JWT
         │                                      & Certificate
         │                                             │
         │ 4. Access Token                            │
         │<────────────────────────────────────────────┤
         │                                             │
         │ 5. Make API calls with token               │
         ├────────────────────────────────────────────>│
         │                                             │
         │ 6. Return data                             │
         │<────────────────────────────────────────────┤
```

**No user login required - fully automated!**

---

## When to Use This Flow

### Perfect For:
- Scheduled batch jobs
- Data synchronization processes
- Backend system integrations
- CI/CD pipelines
- Automated deployments
- ETL processes

### Not Recommended For:
- User-facing applications
- Mobile applications
- Applications where user context is required
- Simple integrations that don't need certificate management

---

## Setup Requirements

### 1. Generate Certificate and Private Key

```bash
# Generate private key
openssl genrsa -out server.key 2048

# Generate certificate signing request
openssl req -new -key server.key -out server.csr

# Generate self-signed certificate (valid for 1 year)
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

### 2. Configure Connected App in Salesforce

1. Create a new Connected App
2. Enable OAuth Settings
3. Upload the certificate (server.crt)
4. Select OAuth Scopes
5. Enable "Use digital signatures"
6. Save and note the Consumer Key (Client ID)

### 3. Pre-Authorize Users

In the Connected App settings:
- Click "Manage"
- Under "OAuth Policies", edit "Permitted Users"
- Select "Admin approved users are pre-authorized"
- Assign users via Permission Sets or Profiles

---

## JWT Structure

A JWT consists of three parts separated by dots:

```
header.payload.signature
```

### Header
```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

### Payload (Claims)
```json
{
  "iss": "3MVG9...",           // Client ID from Connected App
  "sub": "user@example.com",  // Salesforce username
  "aud": "https://login.salesforce.com",  // or https://test.salesforce.com for sandbox
  "exp": 1609462800           // Expiration time (Unix timestamp, typically 5 minutes)
}
```

### Signature
```
RSASHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  privateKey
)
```

---

## Implementation Example

### Step 1: Create JWT

```python
import jwt
import time

# Load private key
with open('server.key', 'r') as key_file:
    private_key = key_file.read()

# Create claims
claims = {
    'iss': '3MVG9...',  # Your Client ID
    'sub': 'user@example.com',  # Salesforce user
    'aud': 'https://login.salesforce.com',
    'exp': int(time.time()) + 300  # 5 minutes from now
}

# Sign JWT
jwt_token = jwt.encode(claims, private_key, algorithm='RS256')
```

### Step 2: Exchange JWT for Access Token

```http
POST /services/oauth2/token HTTP/1.1
Host: login.salesforce.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer&
assertion=YOUR_JWT_TOKEN
```

### Step 3: Receive Access Token

```json
{
  "access_token": "00D50000000IZ3Z!AQcAQH0dMHZ...",
  "scope": "id api refresh_token",
  "instance_url": "https://yourInstance.salesforce.com",
  "id": "https://login.salesforce.com/id/00D50000000IZ3Z/00550000000iY8Z",
  "token_type": "Bearer"
}
```

---

## Security Best Practices

### Certificate Management:
- ✅ Store private keys securely (encrypted, not in version control)
- ✅ Use environment variables or secure vault services
- ✅ Rotate certificates regularly (annually recommended)
- ✅ Use strong encryption algorithms (RSA 2048-bit minimum)
- ✅ Restrict file permissions on private key files
- ✅ Different certificates for different environments

### JWT Best Practices:
- ✅ Keep expiration time short (3-5 minutes)
- ✅ Validate all JWT claims on the receiving end
- ✅ Use secure random number generation
- ✅ Never expose JWTs in URLs or logs
- ✅ Implement token caching to reduce token requests

### Monitoring:
- ✅ Log all token requests
- ✅ Monitor for unusual patterns
- ✅ Set up alerts for failed authentications
- ✅ Regular audit of pre-authorized users
- ✅ Track certificate expiration dates

---

## Common Issues and Solutions

### Issue: "invalid_grant: user hasn't approved this consumer"
**Solution:** Ensure the user is pre-authorized via Permission Set or Profile

### Issue: "invalid_grant: invalid assertion"
**Solutions:**
- Check JWT expiration time (not too far in future, not expired)
- Verify the audience (aud) matches login URL
- Confirm issuer (iss) is the correct Client ID
- Ensure subject (sub) is valid Salesforce username

### Issue: "invalid_client_id"
**Solution:** Verify the Client ID in the JWT matches the Connected App

### Issue: "Certificate validation failed"
**Solutions:**
- Confirm certificate uploaded to Connected App matches private key
- Check certificate hasn't expired
- Verify certificate format (PEM)

---

## Comparison with Other Flows

| Feature | JWT Flow | Web Server Flow | Client Credentials |
|---------|----------|-----------------|-------------------|
| User Interaction | None | Required | None |
| User Context | Yes (pre-authorized) | Yes (live user) | No |
| Security | Certificate | User credentials | Client secret |
| Complexity | High | Medium | Low |
| Setup Effort | High (certificates) | Medium | Low |
| Best For | Scheduled jobs | Interactive apps | App-level tasks |

---

## Complete Code Example (Python)

```python
import jwt
import time
import requests

class SalesforceJWTAuth:
    def __init__(self, client_id, username, private_key_path, is_sandbox=False):
        self.client_id = client_id
        self.username = username
        self.private_key_path = private_key_path
        self.base_url = 'https://test.salesforce.com' if is_sandbox else 'https://login.salesforce.com'
        
    def get_access_token(self):
        # Load private key
        with open(self.private_key_path, 'r') as key_file:
            private_key = key_file.read()
        
        # Create JWT claims
        claims = {
            'iss': self.client_id,
            'sub': self.username,
            'aud': self.base_url,
            'exp': int(time.time()) + 300
        }
        
        # Sign JWT
        jwt_token = jwt.encode(claims, private_key, algorithm='RS256')
        
        # Request access token
        token_url = f'{self.base_url}/services/oauth2/token'
        headers = {'Content-Type': 'application/x-www-form-urlencoded'}
        data = {
            'grant_type': 'urn:ietf:params:oauth:grant-type:jwt-bearer',
            'assertion': jwt_token
        }
        
        response = requests.post(token_url, headers=headers, data=data)
        response.raise_for_status()
        
        return response.json()

# Usage
auth = SalesforceJWTAuth(
    client_id='3MVG9...',
    username='user@example.com',
    private_key_path='server.key',
    is_sandbox=False
)

token_response = auth.get_access_token()
access_token = token_response['access_token']
instance_url = token_response['instance_url']

# Use the token for API calls
headers = {
    'Authorization': f'Bearer {access_token}',
    'Content-Type': 'application/json'
}

api_response = requests.get(
    f'{instance_url}/services/data/v58.0/sobjects/Account',
    headers=headers
)
```
