# Identity Cloud Service JavaScript SDK

Build Seamless Single Sign-On (SSO) and Cross-Application Access for Enterprise Users

**SDK Overview**Create seamless single sign-on (SSO) experiences across your enterprise applications. The ICS JavaScript SDK enables unified authentication, reducing friction for users while maintaining security. Features include federated user management, multi-method authentication, MFA, RBAC, and cross-application session management.

## What is the SDK?

The

**Identity Cloud Service (ICS) JavaScript SDK** enables seamless single sign-on (SSO) and unified identity management across your application ecosystem. Reduce login friction, improve user experience, and maintain enterprise-grade security. Key capabilities include: **Federated User Management:**Create, retrieve, and manage users across multiple applications **Seamless SSO:**Authenticate once, access multiple applications without re-login **Multi-Method Authentication:**Support password, federated identity (SAML, OIDC), social login **MFA:**Add multi-factor authentication with TOTP, SMS, and email for sensitive operations **Cross-Application Sessions:**Manage unified sessions and token refresh across application boundaries **RBAC & Attribute-Based Access:**Enforce fine-grained access control based on user attributes and roles

**Supported Environments**Modern browsers (Chrome, Firefox, Safari, Edge) and Node.js 14+.

## Getting Started

Before you begin, ensure you have the following:

- Active ICS account (sign up at console.identitycloud.io)

- API Key from ICS Management Console

- Familiarity with JavaScript/Node.js

**What You'll Learn**In this guide, you'll learn to set up the SDK, authenticate users securely, and implement production-grade identity workflows.

## Installation

Install the SDK via npm:

`npm install @identitycloud/sdk`

Or include in browser via CDN:

`<script src="https://cdn.identitycloud.io/sdk/v1.0.0/ics-sdk.js"></script>`

## Quick Start: Create and Authenticate a User

### Prerequisites Before You Begin

Ensure you have the following set up before running the Quick Start code:

- Active ICS account (sign up at console.identitycloud.io)

- API Key from ICS Management Console (Settings > API Keys)

- Node.js 14+ installed on your machine

- A terminal/command-line interface (CLI)

### Project Setup (One-Time)

Follow these steps to set up your development environment:

```
# 1. Create a new project directory
mkdir my-identity-app
cd my-identity-app

# 2. Initialize Node.js project
npm init -y

# 3. Install the ICS SDK and dotenv (for environment variables)
npm install @identitycloud/sdk dotenv

# 4. Create a .env file in your project root
echo "ICS_API_KEY=your_actual_api_key_here" > .env
echo "ICS_REGION=us-east-1" >> .env

# 5. Create your application file
touch app.js
```

**API Key Security**Never commit your .env file to version control. Add it to .gitignore:

`echo ".env" >> .gitignore`

### Step-by-Step Walkthrough

Here are the core steps broken down with explanations. See "Complete Working Example" below for the full runnable code.

#### Step 1: Initialize the Client

Before performing any operations, initialize the ICS client with your API credentials. This sets up the connection to the ICS service and configures the SDK for your application region.

```
require('dotenv').config();
const { IdentityCloudClient } = require('@identitycloud/sdk');

const client = new IdentityCloudClient({
  apiKey: process.env.ICS_API_KEY,
  region: process.env.ICS_REGION || 'us-east-1'
});
```

#### Step 2: Create a User

Register a new user in your ICS tenant. This user account becomes available across all applications integrated with ICS, enabling seamless SSO. The email serves as the primary identifier for federation across your application ecosystem.

```
const user = await client.users.registerUser({
  email: 'john@example.com',
  password: 'SecurePassw0rd!',
  firstName: 'John',
  lastName: 'Doe'
});

console.log('User created:', user.id);
```

#### Step 3: Authenticate the User

Authenticate the user and retrieve a session token. This token represents the user's authenticated session and can be used to access other applications in your ecosystem without requiring a new login. The access token is short-lived; session ID persists across application boundaries.

```
const session = await client.auth.login({
  email: 'john@example.com',
  password: 'SecurePassw0rd!'
});

console.log('Access Token:', session.accessToken);
console.log('Session ID:', session.sessionId);
```

### Complete Working Example

Copy the code below into your

`app.js` file. This combines all three steps into a complete, runnable application:

```
// Load environment variables from .env file
require('dotenv').config();

// Import the SDK
const { IdentityCloudClient } = require('@identitycloud/sdk');

// Initialize the ICS client with your credentials
const client = new IdentityCloudClient({
  apiKey: process.env.ICS_API_KEY,
  region: process.env.ICS_REGION || 'us-east-1'
});

// Main function to demonstrate the full flow
async function main() {
  try {
    console.log('Starting ICS Quick Start...\n');

    // Step 1: Create a new user
    console.log('Step 1: Creating a new user...');
    const user = await client.users.registerUser({
      email: 'john@example.com',
      password: 'SecurePassw0rd!',
      firstName: 'John',
      lastName: 'Doe'
    });

    console.log('✓ User created successfully');
    console.log('  User ID:', user.id);
    console.log('  Email:', user.email, '\n');

    // Step 2: Authenticate the user
    console.log('Step 2: Authenticating the user...');
    const session = await client.auth.login({
      email: 'john@example.com',
      password: 'SecurePassw0rd!'
    });

    console.log('✓ Authentication successful');
    console.log('  Access Token:', session.accessToken.substring(0, 20) + '...');
    console.log('  Session ID:', session.sessionId);
    console.log('  Expires in:', session.expiresIn, 'seconds\n');

    // Step 3: Get current user info
    console.log('Step 3: Retrieving current user info...');
    const currentUser = await client.auth.getSession();

    console.log('✓ Current session retrieved');
    console.log('  Session is valid for user:', currentUser.userId, '\n');

    console.log('Quick Start complete! You can now use this session across your applications.');
  } catch (error) {
    console.error('Error during Quick Start:', error.message);
    if (error.details) {
      console.error('Details:', error.details);
    }
  }
}

// Run the main function
main();
```

### Run Your First Program

`node app.js`

**Expected output:**

```
Starting ICS Quick Start...

Step 1: Creating a new user...

✓ User created successfully

User ID: usr_1234567890abcdef

Email: john@example.com

Step 2: Authenticating the user...

✓ Authentication successful

Access Token: eyJhbGciOiJIUzI1NiIs...

Session ID: sess_0987654321fedcba

Expires in: 3600 seconds

Step 3: Retrieving current user info...

✓ Current session retrieved

Session is valid for user: usr_1234567890abcdef

Quick Start complete! You can now use this session across your applications.
```

### What Just Happened?

**Step 1 – User Registration:**Created a new user account in your ICS tenant. This user is now available across all your applications for seamless SSO. **Step 2 – Authentication:**Logged in with the user's credentials and received session tokens. The access token is short-lived (1 hour by default); session ID persists across applications. **Step 3 – Session Validation:**Verified the current session is valid. In production, your apps check this before granting access.

**Next Steps**Now that you've authenticated, you can integrate this session into your web app. Use the session token to protect API routes and enable seamless SSO across your application ecosystem. See Authentication and Best Practices sections for production patterns.

## Authentication

The SDK supports multiple authentication methods:

**API Key Auth:**Server-to-server requests **OAuth 2.0:**Third-party integrations **MFA:**Multi-factor authentication flows

**Security Alert**Never expose your API key in client-side JavaScript. Always use environment variables and server-side authentication for sensitive operations.

## Working with Users

- Retrieve user by ID or current user
- Update user info and custom attributes
- Enable MFA and check authentication status
- Delete users (with proper authorization)
- Batch operations for multiple users

## Error Handling

The SDK throws specific error types for different scenarios. Each error includes a code, message, and optional details to help you diagnose and fix issues:

| Error Type           | Cause                        | How to Fix |
| -------------------- | --------------------------- | ---------- |
| `AuthenticationError`| Bad credentials or expired token | Check password spelling, verify user exists, or redirect to login if session expired. Implement automatic token refresh via `client.auth.refreshToken()`. |
| `ValidationError`    | Invalid input data          | Check error.details for which field failed validation. Ensure emails match RFC 5322, passwords meet requirements (8+ chars, mixed case, special char), and required fields are provided. |
| `NotFoundError`      | Resource does not exist     | Verify the user ID or session ID is correct. If user was recently created, ensure replication lag (typically <1 second) has passed. Check user permissions if accessing another user's data. |
| `RateLimitError`     | Too many requests in short time | Implement exponential backoff retry (see code example below). Cache credentials locally when possible. Contact support if limits conflict with legitimate traffic patterns. |
| `ServerError`        | ICS internal error          | Check status.identitycloud.io for service incidents. Implement retry with backoff. Log error.requestId for support ticket reference. |

### Error Handling Example with Recovery

```
try {
  const user = await client.users.getUser(userId);
} catch (error) {
  if (error instanceof client.errors.AuthenticationError) {
    // Session expired - redirect to login
    window.location.href = '/login?redirect=' + encodeURIComponent(window.location.href);
  } else if (error instanceof client.errors.ValidationError) {
    // Show user-friendly validation error
    console.log('Invalid input:', error.details);
    displayFormError(error.details[0].field, error.details[0].message);
  } else if (error instanceof client.errors.NotFoundError) {
    // Resource missing - possibly timing issue, retry once
    console.log('User not found, retrying...');
    await new Promise(resolve => setTimeout(resolve, 1000));
    return client.users.getUser(userId);
  } else if (error instanceof client.errors.RateLimitError) {
    // Too many requests - implement backoff
    const retryAfter = error.retryAfter || 2000;
    console.log('Rate limited. Retrying in', retryAfter, 'ms');
    await new Promise(resolve => setTimeout(resolve, retryAfter));
    return client.users.getUser(userId);
  } else if (error instanceof client.errors.ServerError) {
    // Server issue - log for diagnostics
    console.error('Server error. Request ID:', error.requestId);
    throw new Error('Service temporarily unavailable. Please try again.');
  } else {
    console.error('Unexpected error:', error);
    throw error;
  }
}
```

## Best Practices

### Security

- Store API keys in environment variables only
- Use HTTPS for all API calls
- Validate all user input before processing
- Use httpOnly cookies for session tokens
- Implement proper access control checks

### Performance

- Reuse client instances across requests
- Cache non-sensitive user data when appropriate
- Use pagination for large user lists (default: 50 per page)
- Implement request batching for bulk operations

## Advanced Topics

**Custom User Attributes:**Store custom metadata on user objects **Role-Based Access Control (RBAC):**Define and enforce roles and permissions **Token Auto-Refresh:**Implement automatic token refresh logic **Webhooks:**Subscribe to identity events (user.created, auth.failed, etc.) **Retry Logic:**Implement exponential backoff for rate-limited requests **Real-Time Events:**Use client.webhooks for event streaming

## Troubleshooting & FAQ

### Q: Why am I getting CORS errors?

**A:** CORS errors occur when your client-side application domain is not registered with ICS. **Solution:** (1) Log into ICS Management Console, (2) Navigate to Settings > Allowed Origins, (3) Add your application domain (e.g., `https://myapp.example.com`), (4) Restart your application. If using localhost for development, add `http://localhost:3000` to the allowed list.

### Q: Login fails after enabling MFA

**A:** MFA authentication requires time-sensitive codes. **Solution:** (1) Verify user's device clock is synchronized with NTP (time may drift), (2) Check TOTP secret was correctly configured in the user's authenticator app, (3) Ensure the code is entered within 30 seconds of generation (codes expire quickly for security), (4) If device time is incorrect, resync and retry. For support, bypass MFA temporarily via the ICS Console's "User Actions" panel.

### Q: How do I handle Error 429 (Rate Limit)?

**A:** Rate limiting protects the service from abuse. **Solution:** (1) Implement exponential backoff retry logic (see code example below), (2) Cache authentication results when possible to reduce redundant calls, (3) If limits are too restrictive for legitimate traffic, contact support to request quota increase. **Code example:**

```
async function loginWithRetry(email, password, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await client.auth.login({ email, password });
    } catch (error) {
      if (error.statusCode === 429 && i < maxRetries - 1) {
        const backoffMs = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${backoffMs}ms...`);
        await new Promise(resolve => setTimeout(resolve, backoffMs));
      } else {
        throw error;
      }
    }
  }
}
```

### Q: Users are not seeing SSO when accessing a second application

**A:** SSO requires proper session cookie configuration. **Solution:** (1) Verify both applications share the same root domain (e.g., both under `*.example.com`), (2) Check that session cookies are marked as `SameSite=Lax` or `SameSite=None; Secure`, (3) Ensure the second application checks for existing ICS session via `client.auth.getSession()` before showing login form, (4) Test in incognito mode to rule out browser extensions.

**Known Issues**View all known issues and report bugs on GitHub Issues.

## Go-Live Checklist

- API keys are securely stored in environment variables
- HTTPS is enabled for all endpoints
- All OAuth and MFA flows are tested in production environment
- SDK is updated to the latest stable version
- Admin credentials and debug mode are removed from production code
- Error handling and logging are properly configured
- Rate limiting and retry logic are implemented
- User feedback and support channels are active

## Support & Feedback

Need help? We're here for you:

**Help Us Improve**Found an issue or have a suggestion? Use the "Report an Issue" link on any documentation page or file an issue on GitHub.
