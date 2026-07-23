# Atlassian MCP API Proxy

This directory contains the Apigee API proxy bundle for the **Atlassian Model Context Protocol (MCP) Gateway**. The proxy acts as a secure, managed gateway between client applications (such as NextChat) and the downstream Atlassian MCP services.

## Overview

The Atlassian MCP Proxy facilitates:
- **OAuth 2.0 Flow Interception & Redirects**: Automates handling and rewriting of authorize, callback, and token requests.
- **Identity & Profiles**: Resolves user identifiers, caches user profiles, and logs requests using downstream callouts (e.g., Atlassian Me API).
- **JSON-RPC MCP Traffic Forwarding**: Parses and proxies MCP JSON-RPC 2.0 request payloads.
- **Dynamic Client Registration (DCR / Authv2)**: Supports client onboarding and registration flows.

---

## Directory Structure

```
atlassian-mcp/
├── apiproxy/
│   ├── atlassian-mcp.xml         # Main API proxy definition and policy list
│   ├── policies/                 # Policies executed in flows
│   │   ├── AM-Atlassian-OpenIdConfiguration.xml
│   │   ├── AM-Capture-Original-Host.xml
│   │   ├── AM-Capture-User-Metadata.xml
│   │   ├── AM-Clean-Accept-Encoding.xml
│   │   ├── AM-Hash-Token.xml
│   │   ├── AM-RedirectAuth.xml
│   │   ├── AM-RedirectToClient.xml
│   │   ├── AM-Rewrite-WWW-Authenticate.xml
│   │   ├── AM-Set-OAuth-Redirect-Uri-Global.xml
│   │   ├── AM-Set-OAuth-Redirect-Uri-Tenant.xml
│   │   ├── AM-Set-OAuth-Redirect-Uri.xml
│   │   ├── AM-Set-OAuth-Target-Path.xml
│   │   ├── AM-Set-OASM-Target-Path-Global.xml
│   │   ├── AM-Set-OASM-Target-Path.xml
│   │   ├── AM-Set-OIDC-Target-Path.xml
│   │   ├── AM-Set-PRM-Target-Path.xml
│   │   ├── AM-Set-Registration-Target-Path.xml
│   │   ├── AM-Set-User-Profile.xml
│   │   ├── CORS-Allow.xml
│   │   ├── EV-Atlassian-Me-Callback.xml
│   │   ├── EV-Atlassian-Me.xml
│   │   ├── EV-Atlassian-TokenExchange.xml
│   │   ├── EV-Cached-User-Profile.xml
│   │   ├── EV-Extract-Response-Token.xml
│   │   ├── EV-Extract-Tenant-Id.xml
│   │   ├── IC-Client-Redirect-Uri.xml
│   │   ├── JavaScript-RewriteAuthServer.xml
│   │   ├── JavaScript-RewriteOpenIdConfiguration.xml
│   │   ├── JavaScript-RewritePRM.xml
│   │   ├── JavaScript-RewriteRegistration.xml
│   │   ├── JavaScript-RewriteTokenRedirectUri.xml
│   │   ├── LC-Client-Id.xml
│   │   ├── LC-Client-Redirect-Uri.xml
│   │   ├── LC-User-Profile.xml
│   │   ├── MCP-SetExternalClientId.xml
│   │   ├── OAuthV2-GenerateAccessToken-AuthCode.xml
│   │   ├── PC-Client-Id.xml
│   │   ├── PC-Client-Redirect-Uri.xml
│   │   ├── PC-User-Profile-Callback.xml
│   │   ├── PC-User-Profile.xml
│   │   ├── ParsePayload-MCP.xml
│   │   ├── RF-Session-Expired.xml
│   │   ├── RF-Unauthorized.xml
│   │   ├── SC-Atlassian-Me-Callback.xml
│   │   ├── SC-Atlassian-Me.xml
│   │   └── SC-Atlassian-TokenExchange.xml
│   ├── proxies/                   # Entry points (ProxyEndpoints)
│   │   ├── mcp-resource.xml       # Main MCP resource routing flow definitions
│   │   ├── oasm-endpoint.xml      # OAuth Authorization Server metadata endpoint
│   │   └── prm-endpoint.xml       # Protected Resource Metadata endpoint
│   ├── resources/
│   │   └── jsc/                   # JavaScript callouts for payloads/redirects rewriting
│   │       ├── rewrite-auth-server.js
│   │       ├── rewrite-oidc.js
│   │       ├── rewrite-prm.js
│   │       ├── rewrite-registration.js
│   │       └── rewrite-token-redirect-uri.js
│   └── targets/                   # TargetEndpoints for routing
│       ├── atlassian-mcp.xml      # Targets downstream: https://mcp.atlassian.com
│       ├── atlassian-oauth.xml    # Targets downstream: https://auth.atlassian.com
│       └── atlassian-prm.xml      # Targets downstream: https://mcp.atlassian.com/.well-known/oauth-protected-resource
└── README.md                      # This documentation
```

---

## Proxy Configurations

### Base Paths
The proxy listens on three distinct base paths:
1. `/.well-known/oauth-protected-resource/atlassian-mcp` (Protected Resource Metadata)
2. `/.well-known/oauth-authorization-server/atlassian-mcp` (OAuth Server Metadata)
3. `/atlassian-mcp` (Main resource endpoints, authorize/callback redirects, and MCP JSON-RPC protocol messages)

### Primary Endpoint Flows (`mcp-resource.xml`)
- **OpenID Connect Discovery (`openid-configuration`)**: Exposes OIDC config and issuer information.
- **Authorization Server Discovery (`oauth-authorization-server`)**: Returns endpoints for client OAuth 2.0 authorization.
- **Authorize Intercept (`/authorize`)**: Intercepts clients redirected to authorize and handles redirects downstream.
- **Callback Intercept (`/callback`)**: Catches code redirect from Atlassian and redirects clients securely.
- **Token Dynamic Exchange (`/token`)**: Re-exchanges authentication code dynamically.
- **MCP Endpoint (`mcp-v1` at `/v1/mcp`)**: Expects POST requests with a JSON-RPC 2.0 payload matching the Model Context Protocol. Checks token headers, caches user profile, and proxies to downsteam server.
- **DCR / Dynamic Registration (`authv2-v1` at `/v1/mcp/authv2/*`)**: Passthrough routing to dynamically register client details.

---

## Deploying to Apigee

A convenience deployment script is provided at `scripts/deploy-atlassian-mcp.sh` to compile, package, and upload this proxy bundle to Apigee.

### Prerequisites
1. Logged into Google Cloud SDK:
   ```bash
   gcloud auth login
   ```
2. Make sure you have `apigeecli` installed in your home directory (`$HOME/.apigeecli/bin/apigeecli`).

### Command
From the project root directory, run:
```bash
./scripts/deploy-atlassian-mcp.sh
```

The script automatically:
1. Compiles and packages the `apiproxy` directory into a zip archive (`atlassian-mcp.zip`).
2. Generates an active credentials token using `gcloud`.
3. Connects to Google Cloud Apigee for project `om-edison` and imports a new revision.
4. Deploys the imported revision to the `eval` environment, overriding active deployments.

---

## Debugging and Tracing

If you need to trace request/response flow pipelines on Apigee:

1. **Trigger Debug Session**:
   ```bash
   ./scripts/get-apigee-trace.sh
   ```
   This will spin up a debug session on Apigee, wait for requests to flow through, and then retrieve and inspect detailed runtime transaction traces.

2. **Poll Debug Session**:
   ```bash
   ./scripts/poll-apigee-trace.sh
   ```
   Use this script to keep polling the live Apigee debug sessions.
