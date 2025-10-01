# Azure AD App Registration (Ashesi Tenant)

Follow these steps to register the Echo application and configure OAuth:

1. Create an app registration in Azure AD (Entra ID)
   - Supported account types: Accounts in this organizational directory only (single tenant)
   - Redirect URIs (add later for web):
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
4. Expose tenant enforcement
   - Record Tenant ID of Ashesi; enforce at sign-in
5. Configure application IDs
   - Copy `Application (client) ID`, `Directory (tenant) ID`
6. App settings placeholders
   - API `.env`:
     - `AZURE_AD_CLIENT_ID=`
     - `AZURE_AD_CLIENT_SECRET=`
     - `AZURE_AD_TENANT_ID=`
     - `GRAPH_BASE_URL=https://graph.microsoft.com/v1.0`
     - `API_BASE_URL=http://localhost:3001`
   - Web `.env`:
     - `NEXT_PUBLIC_API_BASE_URL=http://localhost:3001`
     - `NEXT_PUBLIC_AZURE_AD_CLIENT_ID=`
     - `NEXT_PUBLIC_AZURE_AD_TENANT_ID=`

Ensure least-privilege and store secrets securely.
