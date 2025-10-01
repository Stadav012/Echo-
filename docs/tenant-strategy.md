# Tenant Strategy: Build Without IT Involvement, Ship With Admin Consent

## Goal
Develop and test end-to-end now without Ashesi IT. Transition to production later with a single admin consent.

## Approach

1. Dev tenant (multi-tenant app)
   - Create a multi-tenant Entra ID app in your own tenant or a Microsoft 365 Developer Program sandbox.
   - Implement OAuth (PKCE) and Microsoft Graph calls.
   - Test end-to-end (mail read, calendar create, webhooks) in dev tenant.

2. Production rollout (single admin approval)
   - Keep app multi-tenant.
   - Prepare Admin Consent URL:
     - `https://login.microsoftonline.com/common/adminconsent?client_id=<CLIENT_ID>&redirect_uri=<REDIRECT_URI>`
   - Provide scopes rationale: `User.Read`, `offline_access`, `Mail.Read`, `Calendars.ReadWrite` (optional `Mail.ReadWrite`).
   - After approval, enforce Ashesi Tenant ID in code.

## IT-less prototypes

- Browser extension (client-side Outlook web parse): avoids server Graph tokens but is brittle.
- Mail forwarding to capture inbox: quick demo; ensure clear privacy messaging; disable for sensitive senders.
- ICS generation: allow adding calendar events without Graph.

## Security and privacy

- Least privilege scopes; encrypt tokens; redact PII in logs.
- No permanent raw email storage unless required for features.

## Next steps

- Register dev app, set env vars, run local web/api.
- Build ingestion (delta + webhook), summaries, and notifications.
- Prepare consent pack for Ashesi IT.
