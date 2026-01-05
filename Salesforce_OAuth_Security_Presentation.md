# Salesforce Security: OAuth Authentication & Authorization

---

## Slide 1: Title
**Salesforce Security Deep Dive**
*OAuth, Authentication & Authorization*

---

## Slide 2: Authentication vs Authorization - The Hotel Analogy 🏨

### **Authentication** = Checking into the Hotel
- **"Who are you?"**
- Show your ID at reception
- Prove your identity with your reservation
- Hotel verifies you are who you claim to be

### **Authorization** = What Your Room Key Can Access
- **"What are you allowed to do?"**
- Your key card opens YOUR room (not others)
- May access gym, pool, executive lounge based on room type
- Different guests have different access levels

---

## Slide 3: OAuth - The Complete Hotel Experience

**OAuth = Modern Hotel Security System**

```
Guest (User) → Check-in (Authentication) → Key Card (Token) → Access Resources (Authorization)
```

- **No sharing of master keys** (passwords)
- **Temporary access** (tokens expire like checkout time)
- **Specific permissions** (room, gym, pool - not housekeeping areas)
- **Traceable** (hotel knows who accessed what and when)

---

## Slide 4: General Security - Credential Types

### What You Know 🧠
- Username & Password
- Security Questions
- PIN Codes

**Hotel Analogy:** Your name and reservation number

### What You Have 📱
- **MFA - Multi-Factor Authentication**
- Phone/Authenticator App
- Security Token
- SMS/Email Code

**Hotel Analogy:** Your ID card + Credit card at check-in

---

## Slide 5: Why MFA Matters

### Single Factor (Password Only) ❌
```
Hacker steals password → Full Access
```

### Multi-Factor Authentication ✅
```
Hacker has password → Still needs your phone → Access DENIED
```

**Hotel Analogy:**
- Knowing the reservation number isn't enough
- You also need photo ID to prove it's YOUR reservation

---

## Slide 6: Access Tokens - Your Digital Key Card

### What is an Access Token?
A temporary credential that grants access to Salesforce resources without sharing your password.

**Hotel Analogy:** Your room key card
- Not permanent (expires at checkout)
- Limited access (only your room + amenities)
- Can be deactivated if lost
- Trackable (hotel knows when you used it)

---

## Slide 7: Token Components - What's Inside?

### 1. **Connected App**
Which application/integration requested access
- *Hotel: Which service is being used (mobile app, front desk, room service)*

### 2. **User Identity**
Which user was authenticated
- *Hotel: Which guest (name, room number)*

### 3. **Login Timestamp**
When was the token issued
- *Hotel: Check-in date and time*

### 4. **Resource Access (Scopes)**
What the token can access
- *Hotel: Room, gym, pool, breakfast - but not housekeeping closets*

### 5. **Expiration Time**
When the token becomes invalid
- *Hotel: Checkout time*

---

## Slide 8: Session Tracking - Security Accountability

### Critical Questions for Security:
1. **Which session belongs to which user?**
2. **Who is responsible for this session?**
3. **Is it a human or an integration?**
4. **Which team owns this integration?**

**Hotel Analogy:**
- Every key card swipe is logged
- Security knows: *Guest 405 accessed the pool at 3:47 PM*
- If something happens, there's an audit trail
- Hotel can contact the right person/department

---

## Slide 9: Why Session Tracking Matters

### Security Scenarios:
- **Suspicious Activity Detection**
  - *Unusual login location or time*
  - *Hotel: Guest key used at 3 AM in restricted area*

- **Breach Response**
  - *Quickly identify compromised sessions*
  - *Hotel: Deactivate lost key cards immediately*

- **Compliance & Audit**
  - *Who accessed sensitive data and when?*
  - *Hotel: Security logs for insurance claims*

- **Orphaned Integrations**
  - *Find sessions with no active owner*
  - *Hotel: Old guest key cards still active*

---

## Slide 10: OAuth Flow #1 - Web Server Flow

### Use Case: User-Interactive Applications
**Best for:** Web applications where users log in through a browser

### The Process:
```
1. User clicks "Login with Salesforce"
2. Redirected to Salesforce login page
3. User enters credentials + MFA
4. User approves application access
5. Salesforce returns authorization code
6. App exchanges code for access token
7. App uses token to access Salesforce data
```

**Hotel Analogy:**
- Guest arrives → Checks in at front desk → Shows ID & credit card → Receives key card → Accesses room

### Key Features:
- ✅ Most secure for user-facing apps
- ✅ User explicitly grants permission
- ✅ Refresh tokens for long-term access
- ✅ User can revoke access anytime

---

## Slide 11: Web Server Flow - Diagram

```
┌─────────┐                ┌──────────────┐                ┌────────────┐
│  User   │                │  Your App    │                │ Salesforce │
└────┬────┘                └──────┬───────┘                └─────┬──────┘
     │                             │                              │
     │ 1. Click "Login"            │                              │
     ├────────────────────────────>│                              │
     │                             │                              │
     │                             │ 2. Redirect to SF Login      │
     │                             ├─────────────────────────────>│
     │                                                             │
     │ 3. Enter Credentials + MFA                                 │
     ├────────────────────────────────────────────────────────────>│
     │                                                             │
     │ 4. Authorization Code                                      │
     │<────────────────────────────────────────────────────────────┤
     │                             │                              │
     │ 5. Pass Code to App         │                              │
     ├────────────────────────────>│                              │
     │                             │                              │
     │                             │ 6. Exchange Code for Token   │
     │                             ├─────────────────────────────>│
     │                             │                              │
     │                             │ 7. Access Token + Refresh    │
     │                             │<─────────────────────────────┤
     │                             │                              │
     │ 8. Access Granted           │                              │
     │<────────────────────────────┤                              │
```

---

## Slide 12: OAuth Flow #2 - JWT (JSON Web Token) Flow

### Use Case: Server-to-Server Integration
**Best for:** Trusted backend integrations, no user interaction needed

### The Process:
```
1. Pre-authorized certificate stored securely
2. App creates JWT (signed with certificate)
3. App sends JWT to Salesforce
4. Salesforce validates signature & certificate
5. Salesforce returns access token
6. App uses token to access data
```

**Hotel Analogy:**
- **VIP Guest with Pre-Authorization**
- Hotel already has your profile & preferences
- Security recognizes your credentials instantly
- Automatic check-in → Key card issued immediately
- No front desk interaction needed

### Key Features:
- ✅ No user interaction required
- ✅ Certificate-based security (very secure)
- ✅ Perfect for automated jobs
- ✅ Pre-authorized access
- ⚠️ Requires certificate management

---

## Slide 13: JWT Flow - Diagram

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

## Slide 14: OAuth Flow #3 - Client Credentials Flow

### Use Case: Machine-to-Machine Communication
**Best for:** Integrations where app acts on its own behalf (not a specific user)

### The Process:
```
1. App has Client ID + Client Secret
2. App sends credentials to Salesforce
3. Salesforce validates credentials
4. Salesforce returns access token
5. App uses token (acting as itself, not a user)
```

**Hotel Analogy:**
- **Service Provider Access**
- Cleaning service has master access card
- Not tied to any specific guest
- Access during specific hours
- Can access multiple rooms but only for authorized tasks
- Company (not individual cleaner) is accountable

### Key Features:
- ✅ App authenticates as itself
- ✅ No user context needed
- ✅ Simpler than JWT (no certificates)
- ⚠️ Must protect client secret carefully
- ⚠️ Limited to app-level operations

---

## Slide 15: Client Credentials Flow - Diagram

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

## Slide 16: Comparing the Three Flows

| Feature | Web Server Flow | JWT Flow | Client Credentials |
|---------|----------------|----------|-------------------|
| **User Interaction** | Required | Not Required | Not Required |
| **Use Case** | Web/Mobile Apps | Server-to-Server | Machine-to-Machine |
| **Security Method** | User login + OAuth | Certificate | Client Secret |
| **User Context** | Yes (acts as user) | Yes (pre-authorized user) | No (acts as app) |
| **Complexity** | Medium | High | Low |
| **Refresh Token** | Yes | No (get new token) | No (get new token) |
| **Best For** | Interactive apps | Scheduled jobs | App-level operations |

**Hotel Analogy:**
- **Web Server:** Guest checking in personally
- **JWT:** VIP with pre-authorization
- **Client Credentials:** Service company access

---

## Slide 17: Security Best Practices

### 1. **Token Management**
- ✅ Store tokens securely (encrypted)
- ✅ Use HTTPS always
- ✅ Set appropriate expiration times
- ✅ Rotate refresh tokens regularly

### 2. **Monitoring & Auditing**
- ✅ Log all authentication attempts
- ✅ Monitor for suspicious patterns
- ✅ Regular access reviews
- ✅ Alert on anomalies

### 3. **Principle of Least Privilege**
- ✅ Grant minimum required permissions
- ✅ Limit token scope
- ✅ Use different Connected Apps for different purposes

**Hotel Analogy:**
- Don't give all guests the master key
- Monitor who accesses restricted areas
- Deactivate cards immediately when guests check out

---

## Slide 18: Session Governance - Who's Responsible?

### Tracking Session Ownership:

```
Session → User → Integration → Team → Business Owner
```

### Implementation:
1. **Naming Conventions**
   - Connected App: `[Team]-[Purpose]-[Environment]`
   - Example: `Sales-DataSync-Production`

2. **Custom Fields**
   - Integration Owner
   - Team Contact
   - Business Purpose

3. **Regular Audits**
   - Quarterly session reviews
   - Deactivate unused integrations
   - Update ownership information

**Hotel Analogy:**
- Hotel knows which company booked the conference room
- Can contact the right department if issues arise
- Can bill the correct organization

---

## Slide 19: Common Security Pitfalls ⚠️

### ❌ Don't Do This:
1. **Hard-coding credentials in code**
   - *Like leaving your room key on the counter*

2. **Sharing integration users**
   - *Multiple people using the same hotel key*

3. **No expiration on tokens**
   - *Hotel key that works forever, even after checkout*

4. **Overly permissive scopes**
   - *Key that opens every room in the hotel*

5. **No monitoring**
   - *Hotel with no security cameras or logs*

---

## Slide 20: The Complete Security Picture

```
┌─────────────────────────────────────────────────────┐
│                  USER / APPLICATION                  │
└────────────────────┬────────────────────────────────┘
                     │
         ┌───────────▼──────────┐
         │   AUTHENTICATION     │ → Who are you?
         │  (Credentials + MFA) │
         └───────────┬──────────┘
                     │
         ┌───────────▼──────────┐
         │    OAuth Flow        │ → Choose the right method
         │ (Web/JWT/Client)     │
         └───────────┬──────────┘
                     │
         ┌───────────▼──────────┐
         │   ACCESS TOKEN       │ → Temporary credential
         │  (With Metadata)     │
         └───────────┬──────────┘
                     │
         ┌───────────▼──────────┐
         │   AUTHORIZATION      │ → What can you do?
         │  (Scopes & Perms)    │
         └───────────┬──────────┘
                     │
         ┌───────────▼──────────┐
         │  SESSION TRACKING    │ → Who's responsible?
         │  (Audit & Monitor)   │
         └──────────────────────┘
```

---

## Slide 21: Additional Security Considerations

### IP Restrictions
- Limit access by IP address
- *Hotel: Only allow check-in from registered locations*

### Session Timeout Policies
- Auto-logout after inactivity
- *Hotel: Key card expires if not used*

### Login Hours Restrictions
- Restrict access to business hours
- *Hotel: Pool only open 6 AM - 10 PM*

### Device Trust
- Recognize and trust known devices
- *Hotel: Returning guest recognition program*

---

## Slide 22: Real-World Scenario - Data Breach Prevention

### Scenario: Suspicious Login Detected

**Without Proper OAuth & Tracking:**
- Unknown session accessing data
- Can't identify the user or integration
- Can't determine the scope of breach
- No way to revoke access quickly

**With OAuth & Session Tracking:**
- ✅ Identify exact user/integration
- ✅ See all accessed resources
- ✅ Contact responsible team immediately
- ✅ Revoke token in seconds
- ✅ Full audit trail for investigation

**Hotel Analogy:**
- Security detects unauthorized access
- Video shows which key card was used
- Contact guest immediately
- Deactivate card
- Review all areas accessed

---

## Slide 23: Implementation Checklist

### For Your Salesforce Org:

- [ ] Enable MFA for all users
- [ ] Create separate Connected Apps per integration
- [ ] Use naming conventions for traceability
- [ ] Implement token rotation policies
- [ ] Set up session monitoring & alerts
- [ ] Document integration ownership
- [ ] Regular access reviews (quarterly)
- [ ] Use appropriate OAuth flow per use case
- [ ] Encrypt stored credentials
- [ ] Enable IP restrictions where possible
- [ ] Set session timeout policies
- [ ] Create incident response plan

---

## Slide 24: Key Takeaways

### 🔑 Remember:

1. **Authentication** = Proving who you are (check-in)
2. **Authorization** = What you're allowed to do (room key access)
3. **OAuth** = Modern, secure way to grant access without sharing passwords
4. **Tokens** = Temporary, traceable, limited-scope credentials
5. **Choose the right flow** = Web Server, JWT, or Client Credentials
6. **Track everything** = Know who's responsible for each session
7. **Monitor constantly** = Detect and respond to threats quickly

**Hotel Analogy Final Thought:**
*Just like a well-run hotel, good security is about knowing who's in your building, what they can access, and having the ability to respond quickly when something seems wrong.*

---

## Slide 25: Questions & Discussion

**Thank you!**

### Additional Resources:
- Salesforce OAuth 2.0 Documentation
- Security Implementation Guide
- Integration Best Practices
- Session Management Guide

---

## Appendix: Technical Details

### Token Structure Example:
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

### Connected App Configuration:
- Callback URL
- OAuth Scopes
- Certificate (for JWT)
- Session Policies
- IP Restrictions
