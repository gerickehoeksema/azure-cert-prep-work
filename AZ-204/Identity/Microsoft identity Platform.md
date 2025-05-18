## 🔑 OAuth Scopes

### What Are Scopes?

Scopes (OAuth 2.0 permissions) represent the fine-grained permissions that an application can request to access protected resources. They define the specific actions an application can perform, following the principle of least privilege.

### Types of Permission Scopes

- **Delegated permissions**: Used when an app acts on behalf of a signed-in user
    - The effective permissions are the intersection of user permissions and app permissions
    - Example: An app requests `Mail.Read` to read a user's email on their behalf
- **Application permissions**: Used when an app acts as itself without a user present
    - Often requires admin consent and grants the application direct access
    - Example: A background service that needs to read all users' calendars

### Scope Format

- **Microsoft Graph scopes**: `https://graph.microsoft.com/{scope}`
    - Examples: `https://graph.microsoft.com/User.Read`, `https://graph.microsoft.com/Mail.Send`
- **Custom API scopes**: `api://{appId}/{scope}` or `{appUri}/{scope}`
    - Example: `api://11111111-1111-1111-1111-111111111111/orders.read`

### Defining Scopes for Your API

To expose an API with custom scopes:

1. Go to Azure portal > App registrations > Your API app
2. Select "Expose an API"
3. Set an Application ID URI (if not already set)
4. Click "Add a scope"
5. Define scope details:
    - Scope name (e.g., `user.read`)
    - Admin consent display name
    - Admin consent description
    - User consent display name
    - User consent description
    - State (Enabled/Disabled)

```csharp
// C# Example: Defining a scope in an app registration using Microsoft Graph SDK
var appRegistration = new Application
{
    // Other properties
    Api = new ApiApplication
    {
        OAuth2PermissionScopes = new List<PermissionScope>()
        {
            new PermissionScope
            {
                Id = Guid.NewGuid(),
                AdminConsentDescription = "Allows the app to read user data on behalf of the signed-in user",
                AdminConsentDisplayName = "Read user data",
                IsEnabled = true,
                Type = "User",
                Value = "user.read",
                UserConsentDescription = "Allows this app to read your user data",
                UserConsentDisplayName = "Read your user data"
            }
        }
    }
};
```

### Requesting Scopes in Client Applications

```csharp
// C# Example: Requesting scopes using MSAL
// For Microsoft Graph
string[] graphScopes = new string[] { "https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Mail.Read" };

// For custom API
string[] apiScopes = new string[] { "api://myapi.example.com/user.read", "api://myapi.example.com/data.write" };

// For interactive authentication
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();

// For client credentials flow (app permissions)
var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

### Configuring Client App API Permissions

To allow your client app to access an API with scopes:

1. Go to Azure portal > App registrations > Your client app
2. Select "API permissions"
3. Click "Add a permission"
4. Select the API (Microsoft Graph, your custom API, etc.)
5. Choose between delegated permissions or application permissions
6. Select the required scopes
7. Click "Add permissions"
8. For application permissions or admin-restricted delegated permissions, click "Grant admin consent"

### Validating Scopes in APIs

```csharp
// C# Example: Validating scopes in an API controller with Microsoft.Identity.Web
[Authorize]
[RequiredScope(RequiredScopesConfigurationKey = "AzureAd:Scopes:Required")]
public IActionResult GetUserData()
{
    // Only accessible if the token contains the required scope
    return Ok(new { message = "Data accessed successfully" });
}

// More granular manual scope validation
[Authorize]
public IActionResult UpdateData()
{
    // Manual scope validation
    if (!User.HasScope("data.write"))
    {
        return Forbid();
    }
    
    // Process the update
    return Ok();
}

// Extension method for scope validation
public static class ClaimsPrincipalExtensions
{
    public static bool HasScope(this ClaimsPrincipal principal, string scope)
    {
        var scopes = principal.FindFirst("http://schemas.microsoft.com/identity/claims/scope")?.Value.Split(' ');
        return scopes != null && scopes.Contains(scope);
    }
}
```

### Consent Experience

#### Types of Consent

- **Static consent**: All permissions requested upfront
- **Incremental consent**: Permissions requested as needed throughout the app experience
- **User consent**: End user can consent to delegated permissions
- **Admin consent**: Organization administrator must consent (for sensitive permissions or application permissions)

#### Consent Prompts

- The user/admin will see a consent prompt showing:
    - App requesting access
    - Permissions being requested
    - Resources the app will access
    - Option to consent or decline

#### Implementing Incremental Consent

```csharp
// Initial login with basic scopes
string[] initialScopes = new string[] { "User.Read" };
var result = await app.AcquireTokenInteractive(initialScopes).ExecuteAsync();

// Later, when more permissions are needed
string[] additionalScopes = new string[] { "User.Read", "Mail.Read" };
try {
    // Try to get token silently first
    var result = await app.AcquireTokenSilent(additionalScopes, account).ExecuteAsync();
} catch (MsalUiRequiredException) {
    // If silent acquisition fails, fallback to interactive with incremental consent
    var result = await app.AcquireTokenInteractive(additionalScopes).ExecuteAsync();
}
```

> 💡 **Exam Tip**: Understand when admin consent is required versus user consent. Admin consent is required for all application permissions and for certain high-privilege delegated permissions.

> ⚠️ **Gotcha**: When testing scope validation, remember that delegated permissions in access tokens will be limited by both the application's permissions AND the user's permissions. A user cannot access something they don't have permission for, even if the app requests that scope.

### App Roles vs. Scopes

|Feature|App Roles|Scopes|
|---|---|---|
|Purpose|Role-based access control|Permission-based access control|
|Applied to|Users, groups, and applications|Applications|
|In tokens|`roles` claim|`scp` claim (delegated) or `roles` claim (application)|
|Use case|"Who" can access resources|"What" an app can do|

> 💡 **Exam Tip**: For user-based authorization, use roles. For application-based authorization, use scopes. Know when to use each.# 🔐 AZ-204 Study Guide: Azure Identity Platform

## 📋 Table of Contents

- [Introduction](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#introduction)
- [Microsoft Identity Platform](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#microsoft-identity-platform)
- [App Registrations](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#app-registrations)
- [Authentication Flows](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#authentication-flows)
- [Microsoft Authentication Library (MSAL)](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#microsoft-authentication-library-msal)
- [Managed Identities](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#managed-identities)
- [OAuth Scopes](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#oauth-scopes)
- [Azure AD B2C](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#azure-ad-b2c)
- [Secure Access to Resources](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#secure-access-to-resources)
- [Exam Tips & Gotchas](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#exam-tips--gotchas)
- [Practice Questions](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#practice-questions)
- [Further Resources](https://claude.ai/chat/03e40ca5-4c20-4340-99c3-b49114fa16ce#further-resources)

## 🌟 Introduction

The Identity section of the AZ-204 exam covers approximately 15-20% of the total exam content. This section focuses on how to implement authentication and authorization using the Microsoft identity platform.

## 🔑 Microsoft Identity Platform

### Core Components

- **Azure Active Directory (Azure AD)**: Cloud-based identity and access management service
- **Endpoints**: OAuth 2.0 and OpenID Connect standard-compliant authentication service
- **Authentication Libraries**: MSAL and library support for different application types
- **Application Management Portal**: App registrations and configuration
- **Application Configuration API**: Programmatic interface to configure applications

### Key Concepts

- **Tenants**: An instance of Azure AD that represents an organization
- **Application Objects**: Definition of an application in Azure AD
- **Service Principals**: Instance of an application in a directory
- **OAuth 2.0 Scopes**: Permissions that an application can request
- **Consent**: Process by which resource owners grant permissions to applications

> 💡 **Exam Tip**: Understand the difference between application objects and service principals. An application object is the global representation of an application, while a service principal is the local representation within a specific tenant.

## 📝 App Registrations

### Registration Process

1. Sign in to the Azure portal
2. Navigate to Azure Active Directory
3. Select "App registrations" and "New registration"
4. Provide a name, supported account types, and optional redirect URI
5. Click Register

### Configuration Options

- **Authentication**: Configure platforms, redirect URIs, and token settings
- **Certificates & secrets**: Add client secrets or certificates for authentication
- **API permissions**: Define resources and permissions the app needs
- **Expose an API**: Define scopes and user consent settings
- **Branding**: Customize logo and publisher information
- **Owners**: Assign ownership for management purposes

```csharp
// C# Example: Creating an app registration using Microsoft Graph SDK
var application = new Application
{
    DisplayName = "My API Application",
    SignInAudience = "AzureADMyOrg",
    Web = new WebApplication
    {
        RedirectUris = new List<string>
        {
            "https://myapp.example.com/auth-callback"
        },
        ImplicitGrantSettings = new ImplicitGrantSettings
        {
            EnableIdTokenIssuance = true
        }
    }
};

await graphClient.Applications
    .Request()
    .AddAsync(application);
```

> ⚠️ **Gotcha**: Client secrets have expiration dates. In the exam, watch for scenarios involving expired secrets causing authentication failures.

## 🔄 Authentication Flows

### Authorization Code Flow

- Used by web applications that store tokens on server side
- Most secure OAuth 2.0 grant type
- Provides both access tokens and refresh tokens

**Flow Steps**:

1. App directs user to /authorize endpoint
2. Azure AD authenticates user and returns authorization code
3. App redeems code at /token endpoint for access and refresh tokens
4. App uses access token to call protected resources

```csharp
// C# Example: Authorization Code Flow with MSAL
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .Create(config.ClientId)
    .WithClientSecret(config.ClientSecret)
    .WithAuthority(new Uri(config.Authority))
    .WithRedirectUri(config.RedirectUri)
    .Build();

AuthorizationCodeProvider authProvider = new AuthorizationCodeProvider(app);
```

### Implicit Flow

- Legacy flow for SPAs (single-page applications)
- Access tokens returned directly from /authorize endpoint
- No refresh tokens provided

> 🚨 **Important**: Microsoft now recommends using Authorization Code flow with PKCE for SPAs instead of implicit flow.

### Client Credentials Flow

- Used for service-to-service authentication
- No user involved
- Application authenticates using client ID and secret or certificate

```csharp
// C# Example: Client Credentials Flow
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .Create(config.ClientId)
    .WithClientSecret(config.ClientSecret)
    .WithAuthority(new Uri(config.Authority))
    .Build();

var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

### Device Code Flow

- Used for devices with limited input capabilities
- User authenticates on a separate device

### Resource Owner Password Credentials (ROPC)

- Legacy flow where app collects username/password
- Should be avoided when possible

> 💡 **Exam Tip**: Know which flow to use for different application types. The exam often tests your ability to choose the right authentication flow for a given scenario.

## 📚 Microsoft Authentication Library (MSAL)

### Library Variants

- **MSAL.NET**: For .NET applications (Framework, Core, Xamarin)
- **MSAL.js**: For JavaScript/TypeScript applications
- **MSAL for Python, Java, Android, iOS**, etc.

### Key Features

- Acquires tokens for users signing in with Azure AD accounts
- Maintains token cache and handles token refreshing
- Provides silent authentication when possible
- Helps with token acquisition for daemon/service applications

```csharp
// C# Example: Using MSAL.NET to acquire token interactively
IPublicClientApplication app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

string[] scopes = { "user.read" };
AuthenticationResult result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();

// Use the access token
Console.WriteLine($"Access token: {result.AccessToken}");
```

> ⚠️ **Gotcha**: Mixing ADAL (legacy) and MSAL libraries in the same application can cause unexpected token cache issues.

## 👤 Managed Identities

### Types

- **System-assigned**: Tied to the lifecycle of an Azure resource
- **User-assigned**: Created as standalone Azure resources

### Supported Services

- Azure Virtual Machines
- Azure App Service
- Azure Functions
- Azure Logic Apps
- Azure API Management
- Azure Data Factory
- Many other Azure services

### Usage

```csharp
// C# Example: Using managed identity with Azure SDK
// No credentials needed in code - uses the managed identity
var credential = new DefaultAzureCredential();
var blobServiceClient = new BlobServiceClient(
    new Uri("https://mystorageaccount.blob.core.windows.net"),
    credential);
```

> 💡 **Exam Tip**: Know how to enable and use both system-assigned and user-assigned managed identities, and understand when each is appropriate.

## 🏢 Azure AD B2C

### Key Features

- Customer identity access management (CIAM) solution
- Customizable user experiences
- Support for social identity providers
- Progressive user profiling
- Supports MFA and conditional access

### User Flows vs. Custom Policies

- **User Flows**: Predefined, configurable policies for common identity tasks
- **Custom Policies**: Advanced configuration using the Identity Experience Framework

```csharp
// C# Example: Configuring MSAL for Azure AD B2C
IPublicClientApplication app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithB2CAuthority("https://contoso.b2clogin.com/contoso.onmicrosoft.com/B2C_1_signupsignin")
    .Build();
```

> ⚠️ **Gotcha**: Azure AD B2C tenants are different from regular Azure AD tenants. Make sure you understand the differences in configuration and capabilities.

## 🔒 Secure Access to Resources

### Access Token Validation

```csharp
// C# Example: Validating access tokens in an API
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(options =>
    {
        Configuration.Bind("AzureAd", options);
    });
```

### OAuth Scopes

#### What Are Scopes?

Scopes (OAuth 2.0 permissions) represent the permission to access a protected resource. They define what an application can do, not who can access it (which is handled by roles).

#### Types of Scopes

- **Delegated permissions**: Used when an app acts on behalf of a signed-in user
- **Application permissions**: Used when an app acts as itself without a user present

#### Scope Format

- **Microsoft Graph scopes**: `https://graph.microsoft.com/{scope}` (e.g., `https://graph.microsoft.com/User.Read`)
- **Custom API scopes**: `api://{appId}/{scope}` or `{appUri}/{scope}`

#### Defining Scopes in Your API

```csharp
// C# Example: Defining a scope in an app registration using Microsoft Graph SDK
var appRegistration = new Application
{
    // Other properties
    Api = new ApiApplication
    {
        OAuth2PermissionScopes = new List<PermissionScope>()
        {
            new PermissionScope
            {
                Id = Guid.NewGuid(),
                AdminConsentDescription = "Allows the app to read user data on behalf of the signed-in user",
                AdminConsentDisplayName = "Read user data",
                IsEnabled = true,
                Type = "User",
                Value = "user.read",
                UserConsentDescription = "Allows this app to read your user data",
                UserConsentDisplayName = "Read your user data"
            }
        }
    }
};
```

#### Requesting Scopes in Client Applications

```csharp
// C# Example: Requesting scopes using MSAL
string[] scopes = new string[] { "api://myapi.example.com/user.read", "api://myapi.example.com/data.write" };

// For interactive authentication
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();

// For client credentials flow
var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

#### Validating Scopes in APIs

```csharp
// C# Example: Validating scopes in an API controller
[Authorize]
[RequiredScope(RequiredScopesConfigurationKey = "AzureAd:Scopes:Required")]
public IActionResult GetUserData()
{
    // Only accessible if the token contains the required scope
    return Ok(new { message = "Data accessed successfully" });
}

// More granular scope validation
[Authorize]
public IActionResult UpdateData()
{
    // Manual scope validation
    if (!User.HasScope("data.write"))
    {
        return Forbid();
    }
    
    // Process the update
    return Ok();
}

// Extension method for scope validation
public static class ClaimsPrincipalExtensions
{
    public static bool HasScope(this ClaimsPrincipal principal, string scope)
    {
        var scopes = principal.FindFirst("http://schemas.microsoft.com/identity/claims/scope")?.Value.Split(' ');
        return scopes != null && scopes.Contains(scope);
    }
}
```

#### Incremental Consent

- Users can consent to scopes incrementally, rather than all at once
- Your app should request only the scopes it needs at any given time
- Additional scopes can be requested when needed

> 💡 **Exam Tip**: Understand the difference between static and dynamic scopes. Static scopes are defined at application registration, while dynamic scopes can be requested at runtime (particularly relevant for Microsoft Graph).

> ⚠️ **Gotcha**: The exam may test your knowledge of handling consent in multi-tenant applications. Remember that admin consent may be required for certain scopes in organizational contexts.

### App Roles

- Define roles for authorization decisions
- Unlike scopes, roles apply to users and groups, not to applications

```csharp
// C# Example: Requiring specific roles
[Authorize(Roles = "Admin,Manager")]
public IActionResult AdminFunction()
{
    // Only users in Admin or Manager roles can access
}
```

### Conditional Access

- Enforce additional controls based on signals like device, location, or risk

## 💻 Exam Tips & Gotchas

### Top Tips

1. **Terminology Precision**: Understand the exact meanings of terms like client ID, tenant ID, application ID, object ID.
    
2. **Authentication Flow Selection**: Know which flow to use for each scenario:
    
    - Web app: Authorization code flow
    - SPA: Auth code with PKCE (or legacy implicit)
    - Native app: Auth code with PKCE
    - Daemon/service: Client credentials
    - Limited input device: Device code
3. **Token Handling**: Understand token lifetimes, refresh tokens, and token caching strategies.
    
4. **Permission Types**:
    
    - Delegated permissions: App acts on behalf of signed-in user
    - Application permissions: App acts as itself (no user)
5. **Library Selection**: Use MSAL libraries for new applications, not the legacy ADAL.
    

### Common Gotchas

1. **Application vs. Delegated Permissions**: Using the wrong type of permission for a scenario.
    
2. **Redirect URI Mismatches**: Not configuring the exact same redirect URI in both code and app registration.
    
3. **Missing API Permissions**: Forgetting to grant and consent to required permissions.
    
4. **Consent Issues**: Not understanding when admin consent is required.
    
5. **Secret Management**: Hard-coding secrets or not rotating them properly.
    
6. **Scope Formats**: Using incorrect scope formats:
    
    - Microsoft Graph: `https://graph.microsoft.com/User.Read`
    - Custom API: `api://[appId]/[scope]`
7. **Scope vs. Roles Confusion**: Using scopes for user authorization or roles for app permissions.
    
8. **Scope Claim Format**: The scope claim in the token is a space-delimited string, not an array.
    
9. **Managed Identity Limitations**: Not all Azure services support managed identities.
    
10. **Multi-tenant vs. Single-tenant**: Confusion about account types supported by the application.
    
11. **Incremental Consent**: Requesting too many permissions upfront rather than incrementally.
    

## ❓ Practice Questions

1. Your web API needs to authenticate service-to-service calls without a user present. Which flow should you use?
    
    - A) Authorization code flow
    - B) Implicit flow
    - C) Client credentials flow
    - D) Resource owner password credentials flow
2. You need to authenticate users in a SPA (single-page application). Which approach is currently recommended by Microsoft?
    
    - A) Implicit flow
    - B) Authorization code flow with PKCE
    - C) Client credentials flow
    - D) Resource owner password credentials flow
3. You have a system-assigned managed identity for an Azure Function. What happens to the identity when you delete the Function App?
    
    - A) The identity remains but is marked as orphaned
    - B) The identity is automatically deleted
    - C) The identity is moved to a recovery container for 30 days
    - D) The identity changes to user-assigned type
4. Which library should you use for authentication in a new .NET application?
    
    - A) ADAL.NET
    - B) MSAL.NET
    - C) Microsoft.Identity.Web
    - D) Both B and C are acceptable
5. You are developing a web API that must validate scopes in incoming access tokens. Which claim in the JWT token contains the delegated permissions granted to the client application?
    
    - A) `permissions`
    - B) `scp`
    - C) `roles`
    - D) `aud`
6. In an access token obtained through client credentials flow (application permissions), how are the granted permissions represented?
    
    - A) In the `scp` claim as a space-delimited string
    - B) In the `roles` claim as an array
    - C) In the `permissions` claim as an array
    - D) In the `grant_type` claim as "client_credentials"

> 💡 **Answers**: 1-C, 2-B, 3-B, 4-D, 5-B, 6-B, 7-B

## 📘 Further Resources

- [Microsoft Identity Platform Documentation](https://docs.microsoft.com/azure/active-directory/develop/)
- [Authentication flows in MSAL](https://docs.microsoft.com/azure/active-directory/develop/msal-authentication-flows)
- [Managed Identities Overview](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)
- [Azure AD B2C Documentation](https://docs.microsoft.com/azure/active-directory-b2c/)
- [Microsoft Graph API Reference](https://docs.microsoft.com/graph/api/overview)

---

## 🏆 Certification Success Checklist

- [ ] Understand all authentication flows and when to use each
- [ ] Practice configuring app registrations in Azure portal
- [ ] Implement MSAL in a C# application
- [ ] Configure and use managed identities
- [ ] Define and expose OAuth scopes in an API application
- [ ] Configure a client application to request appropriate scopes
- [ ] Implement token validation including scope validation in APIs
- [ ] Understand incremental consent and admin consent requirements
- [ ] Set up role-based access control using app roles
- [ ] Test token validation in an API
- [ ] Review all common gotchas listed in this guide
- [ ] Complete practice questions and validate your knowledge