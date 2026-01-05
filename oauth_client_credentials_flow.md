# OAuth Flow #3 - Client Credentials Flow

## Use Case: Machine-to-Machine Communication
**Best for:** Integrations where app acts on its own behalf (not a specific user)

## The Process:
```
1. App has Client ID + Client Secret
2. App sends credentials to Salesforce
3. Salesforce validates credentials
4. Salesforce returns access token
5. App uses token (acting as itself, not a user)
```

## Hotel Analogy:
- **Service Provider Access**
- Cleaning service has master access card
- Not tied to any specific guest
- Access during specific hours
- Can access multiple rooms but only for authorized tasks
- Company (not individual cleaner) is accountable

## Key Features:
- ✅ App authenticates as itself
- ✅ No user context needed
- ✅ Simpler than JWT (no certificates)
- ⚠️ Must protect client secret carefully
- ⚠️ Limited to app-level operations

---

## Client Credentials Flow - Diagram

```
┌──────────────────┐                          ┌────────────────┐
│  Integration     │                          │   Salesforce   │
│  (Client ID +    │                          │                │
│   Secret)        │                          │                │
└────────┬─────────┘                          └────────┬───────┘
         │                                             │
         │ 1. Send Client ID + Secret                 │
         ├────────────────────────────────────────────>│
         │                                             │
         │                                   2. Validate
         │                                      Credentials
         │                                             │
         │ 3. Access Token                            │
         │<────────────────────────────────────────────┤
         │                                             │
         │ 4. API Calls (as the app, not a user)      │
         ├────────────────────────────────────────────>│
         │                                             │
         │ 5. Return data                             │
         │<────────────────────────────────────────────┤
```

---

## When to Use This Flow

### Perfect For:
- Application-level operations
- System monitoring and health checks
- Metadata management
- Configuration management
- Anonymous access scenarios
- Microservices authentication

### Not Recommended For:
- Operations requiring user context
- Data operations tied to specific users
- When you need to track which user performed an action
- Operations requiring user-level permissions

---

## Setup Requirements

### 1. Create Connected App in Salesforce

1. Navigate to Setup → App Manager → New Connected App
2. Fill in basic information:
   - Connected App Name
   - API Name
   - Contact Email

3. Enable OAuth Settings:
   - Check "Enable OAuth Settings"
   - Add Callback URL (can be placeholder for this flow)
   - Select OAuth Scopes

4. Enable Client Credentials Flow:
   - Check "Enable Client Credentials Flow"
   - Select the "Run As" user (the user context for the app)

5. Save and retrieve:
   - Consumer Key (Client ID)
   - Consumer Secret (Client Secret)

### 2. Security Considerations During Setup

- Choose appropriate "Run As" user with minimal required permissions
- Limit OAuth scopes to only what's necessary
- Consider IP restrictions if applicable
- Set token expiration policies

---

## Implementation Example

### Step 1: Request Access Token

```http
POST /services/oauth2/token HTTP/1.1
Host: login.salesforce.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
client_id=YOUR_CLIENT_ID&
client_secret=YOUR_CLIENT_SECRET
```

### Step 2: Receive Access Token Response

```json
{
  "access_token": "00D50000000IZ3Z!AQcAQH0dMHZ...",
  "instance_url": "https://yourInstance.salesforce.com",
  "id": "https://login.salesforce.com/id/00D50000000IZ3Z/00550000000iY8Z",
  "token_type": "Bearer",
  "issued_at": "1609459200000",
  "signature": "xyz123..."
}
```

Note: Unlike Web Server Flow, there's no refresh token. When the access token expires, request a new one.

### Step 3: Use Access Token for API Calls

```http
GET /services/data/v58.0/sobjects/Account HTTP/1.1
Host: yourInstance.salesforce.com
Authorization: Bearer 00D50000000IZ3Z!AQcAQH0dMHZ...
Content-Type: application/json
```

---

## Code Examples

### Python Example

```python
import requests

class SalesforceClientCredentials:
    def __init__(self, client_id, client_secret, is_sandbox=False):
        self.client_id = client_id
        self.client_secret = client_secret
        self.base_url = 'https://test.salesforce.com' if is_sandbox else 'https://login.salesforce.com'
        self.access_token = None
        self.instance_url = None
    
    def authenticate(self):
        """Get access token using client credentials"""
        token_url = f'{self.base_url}/services/oauth2/token'
        
        data = {
            'grant_type': 'client_credentials',
            'client_id': self.client_id,
            'client_secret': self.client_secret
        }
        
        headers = {
            'Content-Type': 'application/x-www-form-urlencoded'
        }
        
        response = requests.post(token_url, data=data, headers=headers)
        response.raise_for_status()
        
        result = response.json()
        self.access_token = result['access_token']
        self.instance_url = result['instance_url']
        
        return result
    
    def get_headers(self):
        """Return headers with authorization"""
        if not self.access_token:
            self.authenticate()
        
        return {
            'Authorization': f'Bearer {self.access_token}',
            'Content-Type': 'application/json'
        }
    
    def api_call(self, endpoint, method='GET', data=None):
        """Make an API call to Salesforce"""
        url = f'{self.instance_url}{endpoint}'
        headers = self.get_headers()
        
        if method == 'GET':
            response = requests.get(url, headers=headers)
        elif method == 'POST':
            response = requests.post(url, headers=headers, json=data)
        elif method == 'PATCH':
            response = requests.patch(url, headers=headers, json=data)
        elif method == 'DELETE':
            response = requests.delete(url, headers=headers)
        
        response.raise_for_status()
        return response.json() if response.text else None

# Usage
sf = SalesforceClientCredentials(
    client_id='3MVG9...',
    client_secret='ABC123...',
    is_sandbox=False
)

# Authenticate
sf.authenticate()

# Get list of available API versions
versions = sf.api_call('/services/data/')
print(versions)

# Query accounts
accounts = sf.api_call('/services/data/v58.0/query/?q=SELECT+Id,Name+FROM+Account+LIMIT+10')
print(accounts)
```

### Node.js Example

```javascript
const axios = require('axios');

class SalesforceClientCredentials {
  constructor(clientId, clientSecret, isSandbox = false) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.baseUrl = isSandbox 
      ? 'https://test.salesforce.com' 
      : 'https://login.salesforce.com';
    this.accessToken = null;
    this.instanceUrl = null;
  }

  async authenticate() {
    const tokenUrl = `${this.baseUrl}/services/oauth2/token`;
    
    const params = new URLSearchParams({
      grant_type: 'client_credentials',
      client_id: this.clientId,
      client_secret: this.clientSecret
    });

    try {
      const response = await axios.post(tokenUrl, params.toString(), {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      });

      this.accessToken = response.data.access_token;
      this.instanceUrl = response.data.instance_url;
      
      return response.data;
    } catch (error) {
      console.error('Authentication failed:', error.response?.data || error.message);
      throw error;
    }
  }

  getHeaders() {
    if (!this.accessToken) {
      throw new Error('Not authenticated. Call authenticate() first.');
    }

    return {
      'Authorization': `Bearer ${this.accessToken}`,
      'Content-Type': 'application/json'
    };
  }

  async apiCall(endpoint, method = 'GET', data = null) {
    const url = `${this.instanceUrl}${endpoint}`;
    const headers = this.getHeaders();

    try {
      const response = await axios({
        method,
        url,
        headers,
        data
      });

      return response.data;
    } catch (error) {
      console.error('API call failed:', error.response?.data || error.message);
      throw error;
    }
  }
}

// Usage
(async () => {
  const sf = new SalesforceClientCredentials(
    '3MVG9...',
    'ABC123...',
    false
  );

  // Authenticate
  await sf.authenticate();
  console.log('Authenticated successfully');

  // Get API versions
  const versions = await sf.apiCall('/services/data/');
  console.log('Available versions:', versions);

  // Query accounts
  const accounts = await sf.apiCall(
    '/services/data/v58.0/query/?q=SELECT+Id,Name+FROM+Account+LIMIT+10'
  );
  console.log('Accounts:', accounts);
})();
```

---

## Security Best Practices

### Client Secret Management:
- ✅ **NEVER** commit secrets to version control
- ✅ Use environment variables or secure vaults (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault)
- ✅ Rotate client secrets regularly
- ✅ Use different credentials for different environments
- ✅ Implement secret scanning in CI/CD pipelines
- ✅ Restrict secret access to only necessary personnel

### Token Management:
- ✅ Cache tokens to reduce authentication requests
- ✅ Implement token refresh before expiration
- ✅ Clear tokens from memory when done
- ✅ Never log tokens
- ✅ Use HTTPS exclusively

### Application Security:
- ✅ Implement IP restrictions in Connected App
- ✅ Use minimal required scopes
- ✅ Monitor and log all authentication attempts
- ✅ Set up alerts for failed authentications
- ✅ Regular security audits

---

## Environment Variable Setup

### .env file (Never commit this!)
```bash
# Salesforce Client Credentials
SF_CLIENT_ID=3MVG9...
SF_CLIENT_SECRET=ABC123...
SF_IS_SANDBOX=false
```

### Using in Python
```python
import os
from dotenv import load_dotenv

load_dotenv()

sf = SalesforceClientCredentials(
    client_id=os.getenv('SF_CLIENT_ID'),
    client_secret=os.getenv('SF_CLIENT_SECRET'),
    is_sandbox=os.getenv('SF_IS_SANDBOX', 'false').lower() == 'true'
)
```

### Using in Node.js
```javascript
require('dotenv').config();

const sf = new SalesforceClientCredentials(
  process.env.SF_CLIENT_ID,
  process.env.SF_CLIENT_SECRET,
  process.env.SF_IS_SANDBOX === 'true'
);
```

---

## Troubleshooting

### Common Errors

#### Error: "invalid_client_id"
**Cause:** Client ID is incorrect or Connected App doesn't exist  
**Solution:** 
- Verify Client ID from Connected App
- Ensure Connected App is active
- Check for typos

#### Error: "invalid_client"
**Cause:** Client secret is incorrect  
**Solution:**
- Verify Client Secret
- Regenerate secret if needed
- Check for whitespace or encoding issues

#### Error: "unsupported_grant_type"
**Cause:** Client Credentials Flow not enabled  
**Solution:**
- Enable "Client Credentials Flow" in Connected App settings
- Verify OAuth is enabled

#### Error: "invalid_grant"
**Cause:** Various reasons including disabled app or user  
**Solutions:**
- Ensure Connected App is active
- Check "Run As" user is active
- Verify "Run As" user has necessary permissions

---

## Comparison with Other OAuth Flows

| Feature | Client Credentials | Web Server Flow | JWT Flow |
|---------|-------------------|-----------------|----------|
| **Complexity** | Low | Medium | High |
| **User Context** | No (app context) | Yes | Yes (pre-authorized) |
| **Setup Time** | Minutes | Medium | Hours (certificates) |
| **User Interaction** | None | Required | None |
| **Certificate Required** | No | No | Yes |
| **Refresh Token** | No | Yes | No |
| **Best For** | App-level tasks | Interactive apps | Scheduled jobs |
| **Security Level** | Medium | High | Very High |

---

## When to Choose Client Credentials Flow

### ✅ Use Client Credentials When:
- You need app-level operations (not user-specific)
- User context is not required
- Simplicity is preferred over certificate management
- Quick setup is needed
- Operating in microservices architecture
- Performing system-level health checks
- Managing metadata or configuration

### ❌ Don't Use Client Credentials When:
- You need to track which user performed an action
- User-specific permissions are required
- Compliance requires user attribution
- Data access needs user context
- You need interactive user authentication

---

## Rate Limiting Considerations

Client Credentials Flow tokens count against API limits:

- **API Request Limits:** Shared with other API calls
- **Concurrent Request Limits:** Typically lower than user sessions
- **Token Request Limits:** Avoid excessive token requests

### Best Practices:
- Cache tokens until expiration
- Implement exponential backoff for retries
- Monitor API usage
- Use bulk APIs when possible

---

## Monitoring and Logging

### What to Log:
- ✅ Authentication attempts (success/failure)
- ✅ Token issuance timestamps
- ✅ API call volume and patterns
- ✅ Error rates and types
- ❌ Never log secrets or tokens

### Metrics to Track:
- Authentication success rate
- Token lifetime utilization
- API call patterns
- Error frequency
- Response times

### Example Logging Implementation:

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class SalesforceClientCredentials:
    # ... (previous code)
    
    def authenticate(self):
        logger.info(f"Attempting authentication for client: {self.client_id[:8]}...")
        
        try:
            # ... authentication logic
            logger.info("Authentication successful")
            return result
        except Exception as e:
            logger.error(f"Authentication failed: {str(e)}")
            raise
    
    def api_call(self, endpoint, method='GET', data=None):
        logger.info(f"{method} request to {endpoint}")
        
        try:
            # ... api call logic
            logger.info(f"{method} request successful")
            return response
        except Exception as e:
            logger.error(f"API call failed: {str(e)}")
            raise
```
