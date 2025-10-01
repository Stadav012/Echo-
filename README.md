## Echo – AI Triage & Notification Agent for Students

Echo helps students connect their email so the agent can detect and prioritize important messages, summarize them, optionally create calendar invites, and push concise notifications to WhatsApp and the web app/extension—so they don’t miss urgent/important university messages or other chosen priorities.

Reference repo: [Echo main repository](https://github.com/Stadav012/Echo-.git)

---

### Current POC Focus: Gmail

For fast testing without any institutional IT involvement, the MVP POC uses Gmail OAuth + Gmail API.
- See `docs/gmail-setup.md` for step-by-step setup.
- Microsoft 365/Outlook integration is planned next and documented for later rollout.

---

### 1) Product Scope and UX

- **User roles**
  - **Student**: Authorizes Gmail (POC) or Microsoft later, sets preferences, receives notifications.
  - **Admin**: Manages allowlists/weights for senders and categories, investigates issues.

- **MVP user flows**
  - **Onboarding**
    - Sign in with Google (POC). Later: Microsoft SSO when admin consent is available.
    - Scopes (POC): Gmail read-only, plus openid/email/profile.
    - Set notification preferences: WhatsApp opt-in (phone verification), web notifications, digest frequency, categories to prioritize.
  - **Email triage**
    - Ingest mails (poll Gmail initially).
    - Prioritize (model + rules): e.g., Registrar, Finance, Faculty, Deadlines, Exams, Course comms.
    - Summarize and extract actions: date/time, RSVP, attachments, deadlines.
  - **Notifications**
    - WhatsApp: concise summaries with CTA links (View, Add to Calendar).
    - Web app: inbox-like feed with filters, “Why is this important?”, feedback controls.
  - **Calendar**
    - POC: provide `.ics` download for quick add; later: Google Calendar API or Microsoft Calendar.
  - **Feedback loop**
    - Thumbs up/down on priority/summaries; retrain weights per user and globally.

- **Non-goals (POC)**
  - No sending email replies.
  - English only.
  - WhatsApp and Web only (no Slack/Telegram).

---

### 2) Prioritization Strategy

- **Heuristics + lightweight model hybrid**
  - Sender/domain allowlist and weights (Registrar, Faculty, Dean, Finance).
  - Subject keywords/NER for deadlines, dates, “Action required”.
  - Recipient targeting (to:you vs list), thread participation, attachments.
  - Time sensitivity (relative to detected date/time windows).
- **Model**
  - Start: zero-shot LLM scoring with a compact prompt, guarded by a rules floor.
  - Later: fine-tune small classifier (e.g., MiniLM/DistilBERT) with features and feedback labels.
- **Output**
  - Priority score 0–100, labels (category), reason string.

---

### 3) Summarization & Action Extraction

- **LLM summarizer**: 1–2 sentence synopsis + structured JSON: { entities, dates, actions, RSVP, location, links }.
- **Guardrails**
  - Token limits, progressive summarization.
  - PII redaction before logs.
  - Hallucination checks: cite spans; validate dates/links.
- **Determinism**
  - Cache templates per recurring senders; keep variance low.

---

### 4) System Architecture

- **Frontend**
  - Next.js 14 (App Router), TypeScript, Tailwind, shadcn/ui.
  - Pages: `/` marketing, `/dashboard` feed/settings, `/onboarding` auth + preferences.
- **Browser extension**
  - MV3 React popup; same APIs; optional Outlook-on-web client parser (future option without Graph).
- **Backend** (NestJS, TypeScript)
  - Services
    - Auth service: Google OAuth (POC) + token storage; later add Microsoft OAuth.
    - Ingestion service: Gmail polling initially; later Microsoft Graph delta + webhooks.
    - Prioritizer: rules engine + LLM classifier.
    - Summarizer: LLM with structured output.
    - Notifications: WhatsApp (Twilio) + Web push/SSE.
    - Calendar service: `.ics` generation; later Calendar APIs.
- **Data**
  - Postgres (users, tokens, mail index, priority, summaries, events, preferences, feedback).
  - Redis (queues, rate limits, dedupe, sessions).
  - Optional blob storage for snapshots; prefer IDs and normalized text.
- **Infra**
  - Docker; Azure/AWS; Terraform minimal stack.
  - Observability: OpenTelemetry, Sentry, metrics.

---

### 5) Gmail Integration (POC)

- **Scopes**: `openid`, `email`, `profile`, `https://www.googleapis.com/auth/gmail.readonly`
- **Auth**
  - Google OAuth External (Testing) with test users.
  - PKCE + Authorization Code; refresh tokens stored encrypted.
- **Mail ingestion**
  - Poll `users/me/messages` with query filters; fetch bodies as needed.
  - Store messageId, threadId, sender, subject, snippet, internalDate, labelIds.
- **Calendar (POC)**
  - Generate `.ics` for add-to-calendar; later: Google Calendar API events.

---

### 6) WhatsApp Integration

- **Provider**: Twilio WhatsApp Business API.
- **Flow**
  - Phone verification; opt-in terms and frequency.
  - Template messages for high-priority; session messages fallback.
  - Per-user and global rate limits; quiet hours.
- **Content**
  - Title, 1-line summary, reason, CTA links (view, calendar), unsubscribe link.

---

### 7) Data Model (MVP)

- `User(id, provider, provider_user_id, email, phone, whatsapp_opt_in, scopes, created_at)`
- `OAuthToken(user_id, provider, access_token_enc, refresh_token_enc, expires_at, key_id)`
- `Message(id, provider, provider_message_id, sender, subject, body_excerpt, received_at, thread_id, has_attachments, raw_size)`
- `Priority(id, message_id, score, labels[], reason, created_at)`
- `Summary(id, message_id, text, actions_json, created_at)`
- `Preference(user_id, categories[], notify_channels[], auto_calendar, quiet_hours)`
- `CalendarEvent(id, message_id, provider_event_id, when_start, when_end, location, status)`
- `Feedback(id, user_id, message_id, type, value, comment, created_at)`

---

### 8) Security & Privacy

- Store minimum data; encrypt tokens and sensitive fields (KMS-managed).
- Least privilege scopes; clear disconnect/revoke flow.
- PII redaction in logs; no raw body storage unless required.
- Secrets in Key Vault; service mTLS.
- DPO-ready data export and delete.

---

### 9) Observability & SLAs

- Error budgets and retries on polling/API errors.
- Dead letter queues for failed LLM/WhatsApp sends.
- Metrics: ingestion latency, classification accuracy, open rate, notification delivery rate.

---

### 10) Delivery Plan and Milestones

- **Milestone 0: Foundations (1 week)**
  - Repo scaffolds, CI, IaC, OAuth base, DB schema, basic auth.
- **Milestone 1: Ingestion + Dashboard (1–2 weeks)**
  - Gmail polling and prioritized feed in dashboard.
- **Milestone 2: Summaries + Actions (1 week)**
  - LLM summaries, action extraction, `.ics` create.
- **Milestone 3: Notifications (1 week)**
  - WhatsApp integration + web real-time notifications.
- **Milestone 4: Feedback + Model (1 week)**
  - Feedback loop; refine scoring.
- **Milestone 5: Microsoft 365 Integration (later)**
  - Switch/extend ingestion to Microsoft Graph with admin consent.

---

### 11) Tech Stack

- **Frontend**: Next.js 14, TypeScript, Tailwind, shadcn/ui, Zustand/React Query, Vercel preview.
- **Backend**: NestJS, TypeScript, Prisma ORM, Zod validation, OpenAI/Anthropic for LLM.
- **Infra**: Azure App Service/Container Apps, Azure Postgres, Azure Cache for Redis, Key Vault, Terraform.
- **Messaging**: BullMQ (Redis).
- **Testing**: Vitest/Jest, Playwright, contract tests for WhatsApp templates.

---

### 12) Risks and Mitigations

- **WhatsApp template approvals**: Request early; fallback to web notifications.
- **API quotas**: Implement backoff and caching; use minimal scopes.
- **LLM costs**: Cache summaries, batch, use small models; extractive fallback.
- **Privacy concerns**: Transparent settings and opt-ins; configurable retention.

---

### References

- Gmail setup guide: `docs/gmail-setup.md`
- Microsoft Graph (future): `docs/azure-ad-app-registration.md`, `docs/tenant-strategy.md`
