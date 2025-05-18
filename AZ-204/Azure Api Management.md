# 📘 Azure API Management — AZ-204 Study Summary

---

## 📌 What is Azure API Management (APIM)?
Azure API Management is a fully managed service for publishing, securing, transforming, maintaining, and monitoring APIs. It acts as a gateway between API consumers and backend services.

---

## 📖 Core Components

| Component        | Description                                                       |
|:----------------|:------------------------------------------------------------------|
| **API Gateway**  | Handles API requests, enforces policies, and routes to backends.  |
| **Publisher Portal (Azure Portal)** | Admin UI for managing APIs, policies, products, and users. |
| **Developer Portal** | Customizable website for developers to discover, test, and consume APIs. |
| **Management Plane** | Interfaces (APIs, Azure Portal, CLI) to configure and manage the APIM instance. |

---

## 📌 Key AZ-204 Concepts & Features

### ✅ API Management Tiers

- **Developer** — Non-production, for testing and development.
- **Basic** — Entry-level production.
- **Standard** — Medium-load production.
- **Premium** — High-load, multi-region, VNET support.
- **Consumption** — Serverless, pay-per-call.

---

### ✅ APIs in APIM

- Supports:
  - **HTTP, REST, SOAP**
  - Import from **OpenAPI, WSDL, Azure Functions, App Services**
  - Mock API responses for testing.

---

### ✅ API Policies

Policies are XML-based statements that control API behaviors like security, caching, and transformation.

**Common policies:**
- `validate-jwt` — Validate JWT tokens.
- `rate-limit-by-key` — Throttle requests per subscription.
- `set-header` — Add/modify headers.
- `rewrite-uri` — Change request URI before forwarding.
- `log-to-eventhub` — Log requests/responses to Azure Event Hub.

**Applied at:**
- **Global scope**
- **API scope**
- **Operation scope**

---

### ✅ Products

- Logical grouping of one or more APIs.
- Can be **published** (public) or **unpublished** (private).
- Requires **subscriptions** for access, unless marked as *open*.

---

### ✅ Subscription Keys

- **Primary & Secondary keys** generated when a developer subscribes to a product.
- Used for authenticating API requests via:
  - HTTP header: `Ocp-Apim-Subscription-Key`
  - Query string: `subscription-key`

---

### ✅ Developer Portal

- Self-service web portal.
- Developers can:
  - Discover APIs.
  - Subscribe to products.
  - Retrieve subscription keys.
  - Test APIs.

---

### ✅ Security Options

- **OAuth 2.0 & OpenID Connect (OIDC)** for authorization.
- **Validate-JWT policy** for token validation.
- **IP filtering, CORS**, and **rate limiting** for security and control.

---

### ✅ Monitoring & Analytics

- Built-in monitoring via **Azure Monitor**
- Export logs to:
  - **Azure Log Analytics**
  - **Azure Event Hub**
  - **Azure Storage**

---

### ✅ Versioning & Revisions

- **Versions** — Different versions of the same API (e.g., v1, v2)
- **Revisions** — Non-breaking updates to an API.

---

### ✅ Backends

- Can connect to:
  - Azure App Services
  - Azure Functions
  - On-premises APIs (via VPN/VNET integration)
  - Any public HTTP endpoint

---

## 📌 Common AZ-204 Exam Scenarios

- Import an API definition into APIM.
- Apply and test policies (e.g., JWT validation, rate limiting).
- Secure APIs with subscription keys and OAuth 2.0.
- Deploy APIM in a multi-region setup using Premium tier.
- Integrate APIM with Azure Functions or App Services.
- Use the Developer Portal for publishing API documentation.
- Manage API revisions and versions.
- Export APIM logs for monitoring.

---

## ✅ Quick Commands

```bash
# Create an APIM instance
az apim create --name <apim-name> --resource-group <resource-group> --publisher-email <email> --publisher-name <name>

# List APIM instances
az apim list --resource-group <resource-group>

# Import API from OpenAPI specification
az apim api import --resource-group <resource-group> --service-name <apim-name> --path <api-path> --specification-url <url>
