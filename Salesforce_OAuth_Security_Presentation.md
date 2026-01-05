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

### What is a Refresh Token?
A long-lived credential used to obtain new access tokens without re-authentication.

**Hotel Analogy:** Your loyalty membership card
- Allows you to get new room keys without checking in again
- Long-lasting (doesn't expire at checkout like room key)
- More powerful (can create new access tokens)
- Must be stored very securely (like keeping your passport safe)
- Can be revoked if compromised

**Key Differences:**
- **Access Token:** Short-lived (hours), used for API calls, less sensitive if leaked (expires quickly)
- **Refresh Token:** Long-lived (days/months), used to get new access tokens, very sensitive (must be encrypted)

**Not all OAuth flows provide refresh tokens:**
- ✅ Web Server Flow: Yes (for long-term user sessions)
- ❌ JWT Flow: No (generate new token with certificate)
- ❌ Client Credentials Flow: No (generate new token with client secret)

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

**Hotel Analogy:** Guest arrives → Checks in at front desk → Shows ID & credit card → Receives key card → Accesses room

### Key Features:
- ✅ Most secure for user-facing apps
- ✅ User explicitly grants permission
- ✅ **Provides refresh tokens** for long-term access
- ✅ User can revoke access anytime
- ✅ Full user context and permissions

📄 **[Full documentation and implementation guide →](oauth_web_server_flow.md)**

🔗 **[Official Salesforce Documentation →](https://help.salesforce.com/s/articleView?id=xcloud.remoteaccess_oauth_web_server_flow.htm&type=5)**

---

## Slide 11: OAuth Flow #2 - JWT (JSON Web Token) Flow

### Use Case: Server-to-Server Integration
**Best for:** When you need **higher security** or **multiple users** in automated integrations

**Hotel Analogy:** VIP Guest with Pre-Authorization - Hotel already has your profile, security recognizes your credentials instantly, automatic check-in

### Key Features:
- ✅ No user interaction required
- ✅ **Certificate-based security (most secure)**
- ✅ **Can be configured for different users** (pre-authorized)
- ✅ Perfect for automated jobs with user context
- ❌ **Does NOT provide refresh tokens** (per Salesforce: "This flow never issues a refresh token")
- ⚠️ Requires certificate management (more complex setup)


**When to choose JWT:**
- Need **stronger security** than client credentials
- Need **multiple user contexts** (different pre-authorized users)
- Scheduled jobs that require specific user permissions
- Enterprise-grade integrations

📄 **[Full documentation and implementation guide →](oauth_jwt_flow.md)**

🔗 **[Official Salesforce Documentation →](https://help.salesforce.com/s/articleView?id=xcloud.remoteaccess_oauth_jwt_flow.htm&type=5)**

---

## Slide 12: OAuth Flow #3 - Client Credentials Flow

### Use Case: Machine-to-Machine Communication
**Best for:** **Simple integrations** with a **single user context** (defined on Salesforce side)

**Hotel Analogy:** Service Provider Access - Cleaning service has master access card, not tied to any specific guest

### Key Features:
- ✅ **Simplest setup** (no certificates needed)
- ✅ **Always runs as ONE specific user** (configured in Connected App)
- ❌ **No refresh token** (get new access token with credentials)
- ✅ No user interaction required
- ✅ Quick to implement
- ⚠️ Must protect client secret carefully
- ⚠️ Limited to single user context
- ⚠️ Lower security than JWT

**When to choose Client Credentials:**
- Need **simple, fast setup**
- Only need **one user context** (always same user)
- App-level operations
- Don't need certificate management complexity

📄 **[Full documentation and implementation guide →](oauth_client_credentials_flow.md)**

🔗 **[Official Salesforce Documentation →](https://help.salesforce.com/s/articleView?id=xcloud.remoteaccess_oauth_client_credentials_flow.htm&type=5)**

---

## Slide 13: Comparing the Three Flows - Summary Table

| Feature | Web Server Flow | JWT Flow | Client Credentials |
|---------|----------------|----------|-------------------|
| **User Interaction** | ✅ Required | ❌ Not Required | ❌ Not Required |
| **User Context** | Multiple users (live login) | Multiple users (pre-authorized) | **Single user** (defined in SF) |
| **Security Level** | High | **Very High** | Medium |
| **Security Method** | User login + OAuth | **Certificate** | Client Secret |
| **Refresh Token** | ✅ Yes | ❌ No | ❌ No |
| **Token Renewal** | Use refresh token | Create new JWT | Use client credentials |
| **Best For** | Interactive apps | **High-security jobs, multiple users** | **Simple integrations, one user** |
| **Use Case** | Web/Mobile Apps | Scheduled jobs needing security/users | App-level operations |

**Why JWT doesn't need refresh tokens:** The certificate itself acts as the refresh mechanism - you can always generate a new access token by creating and signing a new JWT with your certificate.

### **Decision Guide:**

**Choose JWT when:**
- ✅ Need **higher security** (certificate-based)
- ✅ Need **multiple user contexts** in automation
- ✅ Enterprise-grade integration requirements

**Choose Client Credentials when:**
- ✅ Need **simple, quick setup**
- ✅ **One user context** is sufficient
- ✅ Don't want certificate management overhead

**Choose Web Server when:**
- ✅ Users need to **authenticate themselves**
- ✅ Interactive application

**Hotel Analogy:**
- **Web Server:** Guest checking in personally (user present)
- **JWT:** VIP with pre-authorization (secure, can be different VIPs)
- **Client Credentials:** Service company access (always same company, simple card)

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


## Slide 23: Key Takeaways

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

**Thank you!**
