# Azure AD App Registration (Ashesi Tenant or Dev Tenant)

You have two practical paths:

- Development without IT involvement: create a multi-tenant app in your own tenant (or a Microsoft 365 Developer Program sandbox). Students won’t be able to consent in Ashesi if the tenant blocks it, but you can fully develop and test end-to-end.
- Production in Ashesi: request IT admin consent once for the required scopes so students can use it.

## App registration (Dev tenant recommended for now)

1. Create an app registration in Azure AD (Entra ID)
   - Supported account types: Accounts in any organizational directory (multitenant)
   - Redirect URIs (add for web):
     - `https://localhost:3000/api/auth/callback/azure-ad`
     - `https://<prod-domain>/api/auth/callback/azure-ad`
2. API permissions (Microsoft Graph)
   - `User.Read` (Delegated)
   - `offline_access` (Delegated)
   - `Mail.Read` (Delegated)
   - `Calendars.ReadWrite` (Delegated)
   - Optionally: `Mail.ReadWrite` (Delegated) if marking emails as read
3. Certificates & secrets
   - Create a client secret (store in Azure Key Vault)
4. Tenant enforcement
   - In dev, allow multitenant. In production, enforce Ashesi Tenant ID in code.
5. Configure application IDs
   - Copy `Application (client) ID`, `Directory (tenant) ID`
6. App settings placeholders
   - API `.env`:
     - `AZURE_AD_CLIENT_ID=`
     - `AZURE_AD_CLIENT_SECRET=`
     - `AZURE_AD_TENANT_ID=` (use `common` for multitenant or your dev tenant ID)
     - `GRAPH_BASE_URL=https://graph.microsoft.com/v1.0`
     - `API_BASE_URL=http://localhost:3001`
   - Web `.env`:
     - `NEXT_PUBLIC_API_BASE_URL=http://localhost:3001`
     - `NEXT_PUBLIC_AZURE_AD_CLIENT_ID=`
     - `NEXT_PUBLIC_AZURE_AD_TENANT_ID=`

## If Ashesi blocks user consent

- Prepare an Admin Consent URL for IT to approve once:
  - `https://login.microsoftonline.com/common/adminconsent?client_id=<CLIENT_ID>&redirect_uri=<REDIRECT_URI>`
- Scopes requested: `User.Read`, `offline_access`, `Mail.Read`, `Calendars.ReadWrite` (optionally `Mail.ReadWrite`).

## IT-less alternatives (for prototyping)

- Browser extension approach: process Outlook on the web client-side after user signs in; no Graph app in tenant required (limited and brittle).
- Mail forwarding rule: user forwards targeted emails to your capture inbox; process and notify (privacy and reliability tradeoffs).
- ICS links for calendar: generate `.ics` for user to add events manually without Graph.
