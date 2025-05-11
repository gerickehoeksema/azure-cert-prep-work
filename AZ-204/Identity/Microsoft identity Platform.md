# Scopes and permissions in the Microsoft identity platform

The Microsoft identity platform implements the [OAuth 2.0](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols) authorization protocol. OAuth 2.0 is a method through which a third-party app can access web-hosted resources on behalf of a user. Any web-hosted resource that integrates with the Microsoft identity platform has a resource identifier, or _application ID URI_.

The following list shows some examples of Microsoft web-hosted resources:

- Microsoft Graph: `https://graph.microsoft.com`
- Microsoft 365 Mail API: `https://outlook.office.com`
- Azure Key Vault: `https://vault.azure.net`

## Admin-restricted permissions

Permissions in the Microsoft identity platform can be set to admin restricted. For example, many higher-privilege Microsoft Graph permissions require admin approval. If your app requires admin-restricted permissions, an organization's administrator must consent to those scopes on behalf of the organization's users. The following section gives examples of these kinds of permissions:

- `User.Read.All`: Read all user's full profiles
- `Directory.ReadWrite.All`: Write data to an organization's directory
- `Groups.Read.All`: Read all groups in an organization's directory

## OpenID Connect scopes

### The `openid` scope

If an app signs in by using [OpenID Connect](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols), it must request the `openid` scope. The `openid` scope appears on the work account consent page as the **Sign you in** permission.

### The `email` scope

The `email` scope can be used with the `openid` scope and any other scopes. It gives the app access to the user's primary email address in the form of the `email` claim.

### The `profile` scope

The [`offline_access` scope](https://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess) gives your app access to resources on behalf of the user for an extended time. On the consent page, this scope appears as the **Maintain access to data you have given it access to** permission.

## The `.default` scope

The `.default` scope is used to refer generically to a resource service (API) in a request, without identifying specific permissions. If consent is necessary, using `.default` signals that consent should be prompted for all required permissions listed in the application registration (for all APIs in the list).

# OAuth 2.0 and OIDC authentication flow in the Microsoft identity platform

## Roles in OAuth 2.0

![[Pasted image 20250511192701.png]]

- **Authorization server** - The Microsoft identity platform is the authorization server. Also called an _identity provider_ or _IdP_, it securely handles the end-user's information, their access, and the trust relationships between the parties in the auth flow. The authorization server issues the security tokens your apps and APIs use for granting, denying, or revoking access to resources (authorization) after the user has signed in (authenticated).
    
- **Client** - The client in an OAuth exchange is the application requesting access to a protected resource. The client could be a web app running on a server, a single-page web app running in a user's web browser, or a web API that calls another web API. You'll often see the client referred to as _client application_, _application_, or _app_.
    
- **Resource owner** - The resource owner in an auth flow is usually the application user, or _end-user_ in OAuth terminology. The end-user "owns" the protected resource (their data) which your app accesses on their behalf. The resources owners can grant or deny your app (the client) access to the resources they own. For example, your app might call an external system's API to get a user's email address from their profile on that system. Their profile data is a resource the end-user owns on the external system, and the end-user can consent to or deny your app's request to access their data.
    
- **Resource server** - The resource server hosts or provides access to a resource owner's data. Most often, the resource server is a web API fronting a data store. The resource server relies on the authorization server to perform authentication and uses information in bearer tokens issued by the authorization server to grant or deny access to resources.