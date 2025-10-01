# Gmail POC Setup Guide

Use this to enable Echo POC without any tenant admin involvement.

## 1) Create a Google Cloud project
- Go to Google Cloud Console
- Create a new project (e.g., `echo-poc`)
- Enable APIs: Gmail API (and optionally Google Calendar API later)

## 2) Configure OAuth consent screen
- User Type: External
- App name, support email
- Scopes (add later during client creation)
- Test users: add the Gmail accounts that will test the app
- Publish status: Testing (keeps it limited to test users)

## 3) Create OAuth client (Web)
- Credentials → Create Credentials → OAuth client ID
- Application type: Web application
- Authorized redirect URIs (examples):
  - `http://localhost:3000/api/auth/callback/google`
  - `https://<your-domain>/api/auth/callback/google`
- Save `Client ID` and `Client Secret`

## 4) Scopes to use (MVP)
- `openid`
- `email`
- `profile`
- `https://www.googleapis.com/auth/gmail.readonly`
- (Optional later) `https://www.googleapis.com/auth/calendar.events`

## 5) Local environment variables
- Web `.env.local` (example):
  - `NEXT_PUBLIC_API_BASE_URL=http://localhost:3001`
  - `NEXT_PUBLIC_GOOGLE_CLIENT_ID=...`
- API `.env` (example):
  - `GOOGLE_CLIENT_ID=...`
  - `GOOGLE_CLIENT_SECRET=...`
  - `GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/callback/google`

## 6) Minimal flow in Echo
- Web app: “Connect Gmail” button → Google OAuth → receive auth code → send to API
- API exchanges code for tokens → stores encrypted refresh token for background polling
- API polling job: fetch new messages via Gmail API, prioritize + summarize, notify
- Web dashboard shows prioritized feed; `.ics` download for calendar add

## 7) Notes & limits
- Testing mode limits to 100 test users by default
- Gmail API quota is generous for read-only; implement exponential backoff
- Avoid storing full raw messages; keep normalized fields and message IDs

## 8) Useful endpoints
- Gmail List messages: `GET https://gmail.googleapis.com/gmail/v1/users/me/messages?q=...`
- Message detail: `GET https://gmail.googleapis.com/gmail/v1/users/me/messages/{id}`

## 9) Security & privacy
- Encrypt tokens at rest (e.g., Key Vault or env for local)
- Do not log PII; redact subject/body in logs
- Offer disconnect to revoke tokens
