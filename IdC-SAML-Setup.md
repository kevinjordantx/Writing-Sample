# AWS Cognito + Identity Center SAML Setup

Create a test application using Amazon Cognito with AWS Identity Center as a SAML identity provider.

**What You'll Build**  
This guide walks through creating a simple Node.js application that authenticates users via AWS Identity Center and receives SAML attributes through Amazon Cognito. You'll learn how to configure both services, set up SAML federation, and view the attributes passed during authentication.

## Step 1: Set up AWS Identity Center

### 1. Enable AWS Identity Center

`aws sso-admin create-instance --name "MyIdentityCenter" --tags Key=Purpose,Value=Testing`

### 2. Add Users and Groups

- Go to the AWS Identity Center console
- Navigate to Users and add a test user with various attributes
- Create a group and add your user to it

## Step 2: Create an Amazon Cognito User Pool

### 1. Create a User Pool

```bash
aws cognito-idp create-user-pool \
--pool-name "TestSAMLAttributes" \
--auto-verified-attributes email \
--schema Name=email,Required=true Name=name,Required=true Name=given_name,Required=false Name=family_name,Required=false
```

### 2. Note the User Pool ID

`USER_POOL_ID="us-east-1_example" # Replace with your actual pool ID`

### 3. Create a User Pool Client

```bash
aws cognito-idp create-user-pool-client \
--user-pool-id $USER_POOL_ID \
--client-name "TestSAMLApp" \
--no-generate-secret \
--allowed-o-auth-flows code implicit \
--allowed-o-auth-scopes "phone" "email" "openid" "profile" \
--callback-urls '["https://localhost:3000/callback"]' \
--logout-urls '["https://localhost:3000/logout"]'
```

### 4. Note the Client ID

Save the Client ID from the response for later use.

## Step 3: Configure AWS Identity Center as SAML Provider

### 1. Access AWS Identity Center Console

Navigate to AWS Identity Center in the AWS Management Console.

### 2. Add a New Application

- In the left navigation pane, choose "Applications"
- Click "Add application"

### 3. Setup Preference

- Select "I have an application I want to set up"
- Click "Next"

### 4. Choose Application Type

- Select "Customer managed application"
- Click "Next"

### 5. Configure Application Settings

- Enter "Cognito Test App" for Display name
- (Optional) Add a description
- Click "Next"

### 6. Configure SAML 2.0

Enter the following values:

**Application ACS URL:**

`https://us-east-1.auth.amazoncognito.com/saml2/idpresponse`

**Application SAML audience:**

`urn:amazon:cognito:sp:us-east-1_2iwDNqI7W`

Click "Next"

**Note** Replace the region and User Pool ID in the URLs above with your actual values.

### 7. Configure Attribute Mappings

Map the following Identity Center attributes to SAML attributes:

| SAML Attribute | Identity Center Value   | Format     |
| -------------- | ----------------------- | ---------- |
| Subject        | `${user:subject}`       | persistent |
| `${user:email}` | -                      | -          |
| name           | `${user:name}`          | -          |
| given_name     | `${user:givenName}`     | -          |
| family_name    | `${user:familyName}`    | -          |
| groups         | `${user:groups}`        | -          |

Click "Next"

### 8. Review and Add Application

- Review all settings
- Click "Add application"

### 9. Download SAML Metadata

- After creation, go to application details
- Find the "IAM Identity Center metadata" section
- Click "Download metadata file" and save locally

### 10. Assign Users or Groups

- Go to the "Assigned users" tab
- Click "Assign users"
- Select users or groups for access
- Click "Assign users"

### 11. Upload SAML Metadata to Cognito

```bash
aws cognito-idp create-identity-provider \
--user-pool-id $USER_POOL_ID \
--provider-name "IdentityCenter" \
--provider-type SAML \
--provider-details MetadataFile="/path/to/metadata.xml" \
--attribute-mapping email=http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
```

## Step 4: Create Test Application

### 1. Initialize Node.js Project

```bash
mkdir cognito-saml-test
cd cognito-saml-test
npm init -y
npm install express amazon-cognito-identity-js
```

### 2. Create Basic Application (index.js)

```javascript
const express = require('express');
const app = express();
const port = 3000;

// Configuration
const config = {
  userPoolId: 'YOUR_USER_POOL_ID',
  clientId: 'YOUR_CLIENT_ID',
  domain: 'YOUR_COGNITO_DOMAIN',
  region: 'YOUR_REGION'
};

// Routes
app.get('/', (req, res) => {
  res.send(`
<h1>SAML Attribute Test</h1>
<a href="https://${config.domain}.auth.${config.region}.amazoncognito.com/login?response_type=code&client_id=${config.clientId}&redirect_uri=http://localhost:${port}/callback">
Login with Identity Center
</a>
`);
});

app.get('/callback', async (req, res) => {
  const code = req.query.code;
  res.send(`
<h1>Authentication Successful</h1>
<p>Authorization Code: ${code}</p>
<p>In a real application, this code would be exchanged for tokens that contain the SAML attributes.</p>
`);
});

app.listen(port, () => {
  console.log(`Test app listening at http://localhost:${port}`);
});
```

### 3. Start the Application

`node index.js`

## Step 5: Test the SAML Flow

- Open your browser and navigate to  
  `http://localhost:3000`
- Click the "Login with Identity Center" link
- You'll be redirected to the AWS Identity Center login page
- After logging in, you'll be redirected back to your application

## Step 6: View and Verify Attributes

### Enhanced Application with Token Exchange

To view all SAML attributes, enhance the callback handler to exchange the authorization code for tokens:

```javascript
// Add to your application
const https = require('https');
const querystring = require('querystring');

app.get('/callback', async (req, res) => {
  const code = req.query.code;

  // Exchange code for tokens
  const tokenParams = querystring.stringify({
    grant_type: 'authorization_code',
    client_id: config.clientId,
    redirect_uri: `http://localhost:${port}/callback`,
    code: code
  });

  const tokenOptions = {
    hostname: `${config.domain}.auth.${config.region}.amazoncognito.com`,
    port: 443,
    path: '/oauth2/token',
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      'Content-Length': tokenParams.length
    }
  };

  const tokenReq = https.request(tokenOptions, (tokenRes) => {
    let data = '';
    tokenRes.on('data', (chunk) => {
      data += chunk;
    });
    tokenRes.on('end', () => {
      const tokens = JSON.parse(data);

      // Decode the ID token (it's a JWT)
      const idToken = tokens.id_token;
      const [header, payload, signature] = idToken.split('.');
      const decodedPayload = Buffer.from(payload, 'base64').toString('utf8');
      const attributes = JSON.parse(decodedPayload);

      res.send(`
<h1>Authentication Successful</h1>
<h2>SAML Attributes Received:</h2>
<pre>${JSON.stringify(attributes, null, 2)}</pre>
`);
    });
  });

  tokenReq.on('error', (error) => {
    res.status(500).send(`Error: ${error.message}`);
  });

  tokenReq.write(tokenParams);
  tokenReq.end();
});
```

## Available SAML Attributes

### Standard User Attributes

| Attribute          | Description                        |
| ------------------ | ---------------------------------- |
| `Subject` (NameID) | The unique identifier for the user |
| `email`            | User's email address               |
| `name`             | User's full name                   |
| `givenName`        | User's first name                  |
| `familyName`       | User's last name                   |
| `displayName`      | User's display name                |

### Group Membership Attributes

| Attribute | Description                         |
| --------- | ----------------------------------- |
| `groups`  | List of groups the user belongs to  |
| `roles`   | Roles assigned to the user          |

### Additional Configurable Attributes

| Attribute           | Description                                  |
| ------------------- | -------------------------------------------- |
| `userId`            | The unique ID of the user in Identity Center |
| `username`          | The username in Identity Center              |
| `phoneNumber`       | User's phone number                          |
| `createdDate`       | When the user was created                    |
| `modifiedDate`      | When the user was last modified              |
| `addressPostalCode` | User's postal code                           |
| `addressStreet`     | User's street address                        |
| `addressCountry`    | User's country                               |
| `addressRegion`     | User's region/state                          |
| `addressLocality`   | User's city/locality                         |

### Microsoft AD Specific Attributes (if using AD)

| Attribute           | Description                          |
| ------------------- | ------------------------------------ |
| `distinguishedName` | The full DN of the user in AD        |
| `userPrincipalName` | The UPN format (user@domain)         |
| `sAMAccountName`    | The legacy Windows username          |
| `objectSid`         | Security identifier for the user     |
| `memberOf`          | Group memberships                    |
| `department`        | User's department                    |
| `company`           | User's company                       |
| `title`             | User's job title                     |
| `manager`           | User's manager (DN format)           |
