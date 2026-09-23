<p align="center">
 Sage is a private repository. I'm happy to give a live demo or grant read-only access to anyone who's interested, just reach out at ejonofrio@gmail.com.
</p>

<p align="center">
  <img src="src/assets/sage-logo-ui.png" alt="Sage logo" width="128" />
</p>

<h1 align="center">Sage</h1>

<p align="center">
  <strong>Clear mind, clear inbox.</strong>
</p>

<p align="center">
  A calm, learning-first Gmail client that helps people focus on important messages,
  understand their inbox, and handle repetitive email with less effort.
</p>

<p align="center">
  <a href="https://github.com/Firefly-2953/sage/actions/workflows/quality.yml">
    <img src="https://github.com/Firefly-2953/sage/actions/workflows/quality.yml/badge.svg" alt="Quality workflow" />
  </a>
</p>

## Overview

Sage is a full-stack Gmail productivity application built around a simple principle: email software should reduce decisions instead of creating more of them.

Gmail remains the source of truth for messages, labels, and mailbox state. Sage provides a calmer interface on top, adding prioritization, explainable categorization, behavior learning, reversible actions, and carefully controlled automation.

Sage learns gradually. It observes repeated decisions, presents understandable suggestions, and waits for the user’s approval before remembering a preference or changing Gmail.

## Highlights

| Area | What Sage provides |
|---|---|
| **Today** | A focused overview of messages requiring attention, configurable Focus Areas, and a Daily Brief |
| **Inbox** | Gmail-backed mailboxes, search, progressive synchronization, bulk actions, Compose, and a shared email reader |
| **Important** | Explainable Sage priority that remains distinct from Gmail Important and Starred |
| **Learning** | Behavior suggestions, category corrections, accepted preferences, clarification questions, and reviewed automation |
| **Handled** | Explicit one-click shortcuts for qualified Archive, Mark read, and existing-label actions |
| **Adaptive actions** | Opt-in quick-action personalization based on strong, independent evidence |
| **Activity** | Human-readable action history with durable, relationship-aware Undo |
| **Accounts and Settings** | Separate Google identity and Gmail consent, synchronized preferences, and conservative account isolation |

## Learning without losing control

Sage’s learning flow is intentionally gradual:

```text
Observe repeated behavior
        ↓
Identify a supported pattern
        ↓
Explain the evidence
        ↓
Ask the user to review it
        ↓
Remember only after approval
        ↓
Offer an explicit, reversible shortcut
```

A remembered preference does not automatically grant permission to mutate Gmail.

When Sage offers a shortcut, it names the exact effect:

- **Handle · Archive**
- **Handle · Mark read**
- **Handle · Label: Receipts**

Each message must still pass current evidence, policy, ownership, classification, and protection checks. Conflicting or uncertain situations cause Sage to abstain.

## Gmail experience

Sage currently supports:

- Gmail-backed Inbox, Sent, Drafts, All Mail, Spam, and Trash
- Quick, Recent, and Full Inbox synchronization
- Gmail History incremental updates
- Provider-backed Gmail search
- Plain-text Compose with multiple recipients
- Mark read and unread
- Archive and Move to Inbox
- Star and unstar
- Mark and remove Gmail Important
- Move to and restore from Trash
- Mark and remove Spam
- Apply, remove, and create Gmail labels
- Explicitly reviewed Gmail filters through Teach Sage
- Optimistic updates, rollback, feedback, Activity, and Undo

Opening a message does not implicitly change its read state. Permanent deletion is not implemented.

## Architecture

```mermaid
flowchart LR
    UI[React + TypeScript UI]
    Provider[Shared Sage provider]
    Learning[Learning and decision services]
    Persistence[Supabase Auth and persistence]
    Edge[Supabase Edge Functions]
    Gmail[Gmail API]

    UI --> Provider
    Provider --> Learning
    Provider --> Persistence
    Provider --> Edge
    Edge --> Gmail
    Gmail -->|Source of truth| Provider
```

Core architectural boundaries:

- Gmail owns messages and mailbox state.
- Sage stores preferences, learning records, settings, Activity, and processing metadata.
- Gmail actions use one shared execution pipeline.
- Meaningful actions flow through the centralized Activity system.
- Account and session ownership are revalidated across asynchronous work.
- Suggestions, remembered knowledge, Handled shortcuts, and Gmail automation represent separate levels of authority.
- No LLM determines whether a Gmail mutation is authorized.

## Technology

- React 19
- TypeScript
- Vite
- React Router
- Supabase Auth, Postgres, Row Level Security, and Edge Functions
- Google Identity Services
- Gmail API
- Lucide icons
- Plain CSS
- Node’s native test runner
- PGlite migration and persistence-contract testing
- Chrome DevTools Protocol for mounted browser and accessibility tests
- GitHub Actions

## Trust and privacy

Sage is designed to avoid becoming a competing email database.

- Gmail OAuth credentials stay server-side.
- Refresh and short-lived access tokens are encrypted at rest.
- Browser roles cannot read server credential tables.
- Raw Gmail API responses are not persisted.
- Message bodies, snippets, attachments, recipient fields, and full headers are not stored in Sage’s persistence layer.
- Activity and learning records may retain limited sender, subject, action, and relationship metadata when required for explanation and Undo.
- Gmail filters are created only after explicit review and approval.
- Sage does not silently adopt or delete externally managed Gmail filters.
- Autonomous background Gmail actions remain outside the current product scope.

## Quality

Sage has more than **1,900 automated tests** covering:

- Gmail actions and synchronization
- Authentication and account isolation
- Persistence revisions and offline replay
- Learning qualification and evidence
- Activity and durable Undo
- Filter ownership and recovery
- Responsive layouts
- Native keyboard interaction
- Dialog containment and focus restoration
- Mounted Chrome behavior
- Migration and security contracts

The GitHub Actions workflow runs the project on Node 24 and checks:

```bash
npm ci
npm run lint
npm run build
npm test
```

## Current status

Sage V1 and its first Learning V2 milestones have completed bounded automated and authenticated acceptance in a personal/test-account environment.

The application is under active development and has not yet been prepared for broad public distribution. Public launch work will require production-origin configuration, OAuth verification, privacy and terms review, monitoring, and an intentional rollout plan.

Current development is focused on letting users explicitly choose which single Handled shortcut Sage should offer when multiple strongly supported habits apply to the same sender and category.

## Run locally

### Requirements

- Node.js 24 or newer
- npm
- A Google OAuth web client
- A Supabase project for authentication, persistence, and server-side Gmail access

Install dependencies:

```bash
npm install
```

Create the local environment file:

```bash
cp .env.example .env.local
```

Add the public client configuration:

```dotenv
VITE_GOOGLE_CLIENT_ID=your-web-client-id.apps.googleusercontent.com
VITE_SUPABASE_URL=https://your-project-ref.supabase.co
VITE_SUPABASE_ANON_KEY=your-public-anon-or-publishable-key
```

Start Sage:

```bash
npm run dev
```

Vite serves the application at:

```text
http://localhost:5173
```

Never place a Google client secret, Supabase service-role key, Gmail token, or another private credential in a `VITE_` variable.

## Google and Supabase setup

Sage separates Google identity from Gmail authorization:

1. **Sign in with Google** establishes the Sage/Supabase identity.
2. **Connect Gmail** begins a separate server-side OAuth authorization flow.
3. Gmail credentials remain encrypted behind Supabase Edge Functions.
4. Explicit Gmail actions are performed through that server boundary.

The Gmail integration uses:

```text
https://www.googleapis.com/auth/gmail.modify
https://www.googleapis.com/auth/gmail.settings.basic
```

Apply the migrations in `supabase/migrations/` in filename order.

For complete OAuth, Supabase, Edge Function, redirect URI, secret, and deployment instructions, see [Gmail server authorization](docs/gmail-server-authorization.md).

## Useful commands

```bash
npm run dev
npm test
npm run build
npm run lint
npm run preview
```

Focused test suites are also available:

```bash
npm run test:classifier
npm run test:gmail
npm run test:activity
npm run test:patterns
npm run test:stability
npm run test:sync
npm run test:production
npm run test:auth
npm run test:accessibility
```

## Project documentation

Sage maintains a durable project knowledge vault alongside the implementation:

- [Product overview](docs/Sage/00%20-%20Sage%20Overview.md)
- [Current state](docs/Sage/01%20-%20Current%20State.md)
- [Product decisions](docs/Sage/02%20-%20Product%20Decisions.md)
- [Architecture](docs/Sage/03%20-%20Architecture.md)
- [Bugs and lessons](docs/Sage/04%20-%20Bugs%20and%20Lessons.md)
- [Backlog](docs/Sage/05%20-%20Backlog.md)
- [V1 release acceptance](docs/Sage/08%20-%20V1%20Release%20Acceptance.md)

## Roadmap

Future directions include:

- Richer preference management and additional Handled actions
- AI-assisted summaries and semantic importance
- Action-item and calendar extraction
- Package intelligence
- Contact-aware Compose
- Smart, explicitly reviewed automation
- Unified multi-account inboxes

Every addition should make Sage feel calmer, smarter, or more trustworthy.

## Home Page
<img width="1450" height="681" alt="Screenshot 2026-09-23 at 9 39 25 AM" src="https://github.com/user-attachments/assets/28bd12a9-9f65-4ee2-9d0c-ca3636823433" />

## Inbox
<img width="1440" height="729" alt="Screenshot 2026-09-19 at 6 08 20 PM" src="https://github.com/user-attachments/assets/121348cc-a3ad-4233-91e6-e220fb2c7fe9" />

## Important
<img width="1438" height="727" alt="Screenshot 2026-09-23 at 9 51 17 AM" src="https://github.com/user-attachments/assets/04089ed3-8f49-4c6d-93ce-14387444dbd1" />

## Learning 
<img width="1449" height="675" alt="Screenshot 2026-09-23 at 9 52 03 AM" src="https://github.com/user-attachments/assets/0507e419-6dc4-4d6f-af59-cf0eb6bd4336" />

## Teaching Sage
<img width="536" height="724" alt="Screenshot 2026-09-20 at 7 40 41 AM" src="https://github.com/user-attachments/assets/386805b1-13c9-418c-862f-06968aec19b9" />

<img width="547" height="801" alt="Screenshot 2026-09-20 at 7 41 20 AM" src="https://github.com/user-attachments/assets/21b91296-06ef-489c-996a-567cbb8e846d" />

<img width="540" height="796" alt="Screenshot 2026-09-20 at 7 41 59 AM" src="https://github.com/user-attachments/assets/7e1c119d-d69b-4585-9af8-4068f0e08ca3" />

## Activity
<img width="1442" height="633" alt="Screenshot 2026-09-23 at 9 52 54 AM" src="https://github.com/user-attachments/assets/c53bc2ff-99c7-47e4-816c-d10f73ba2709" />

## Settings 
<img width="1445" height="668" alt="Screenshot 2026-09-23 at 9 53 41 AM" src="https://github.com/user-attachments/assets/be29c133-2122-4eb7-855b-fba5031c40bb" />

<img width="686" height="604" alt="Screenshot 2026-09-23 at 9 54 16 AM" src="https://github.com/user-attachments/assets/ae18b4ce-96d5-42c3-97b2-a4173858c061" />

<img width="654" height="332" alt="Screenshot 2026-09-23 at 9 54 38 AM" src="https://github.com/user-attachments/assets/d7a756ef-86ab-4c9c-9144-157ac9e64956" />








