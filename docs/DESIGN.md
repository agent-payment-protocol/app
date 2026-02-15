# Agent Payment Protocol (APP) — Design Document

**Version:** 2.0  
**Date:** February 15, 2026  
**Status:** Draft  
**License:** Apache 2.0  

---

## 1. What APP Is

### The Problem

AI agents can browse, search, recommend, and negotiate — but they cannot pay.

When an agent needs to complete a purchase on behalf of a user:
- There's no standard way for agents to access payment credentials
- Users have no way to authorize agents with spending controls
- Each platform builds proprietary solutions (or nothing at all)
- Users can't see or control agent spending across platforms

**Result:** Users can't confidently delegate spending. Agents can't complete purchases. Commerce stays manual.

### The Solution

**Agent Payment Protocol (APP)** is an open protocol that lets AI agents pay on behalf of users, with user-defined rules.

- **Users** store payment credentials once, authorize agents with limits
- **Agents** request payments through a standard API
- **APP** verifies identity, checks rules, executes payment
- **Merchants** receive normal payments (no integration required)

### What APP Is

- An open protocol specification
- A reference implementation (open source, Apache 2.0)
- A card vault with user-defined spending rules
- An authorization layer between users and agents

### What APP Is NOT

- ❌ A payment processor (Stripe, Adyen do this)
- ❌ A merchant of record (merchants/platforms handle this)
- ❌ An AI platform (OpenAI, Anthropic do this)
- ❌ A shopping experience (agents handle this)
- ❌ A fraud detection system (PSPs handle this)

---

## 2. How It Works

### Key Actors

| Actor | Role |
|-------|------|
| **User** | Person who owns the money. Stores card, sets rules, reviews history. |
| **Agent** | AI that acts on user's behalf. Requests payments through APP. |
| **Platform** | Where user interacts with agent (OpenAI, Anthropic, etc.). Authenticates user. |
| **APP** | Verifies identity, enforces rules, executes payment. |
| **PSP** | Payment processor (Stripe). Handles actual card charging. |
| **Merchant** | Receives payment. No integration with APP required. |

### Core Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. USER SETUP (Once)                                                        │
│                                                                             │
│    User is on AI platform (e.g., ChatGPT)                                   │
│    Agent wants to make a purchase                                           │
│    User is directed to APP setup page                                       │
│    User adds card (tokenized via Stripe — APP never sees card number)       │
│    User sets rules: "This agent can spend up to $X per day"                 │
│    Setup complete — user returns to agent                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 2. PAYMENT (Every Transaction)                                              │
│                                                                             │
│    Agent requests payment from APP                                          │
│    APP verifies: Is this request really from the platform? Is user real?    │
│    APP checks rules: Is amount within limits? Is agent still authorized?    │
│                                                                             │
│    If rules pass → APP charges card via Stripe → Success                    │
│    If rules fail → APP denies request → Agent informs user                  │
│                                                                             │
│    Everything is logged.                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 3. USER CONTROL (Anytime)                                                   │
│                                                                             │
│    User can view all transactions                                           │
│    User can change limits                                                   │
│    User can revoke agent access (immediate)                                 │
│    User can add/remove cards                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Identity Model

APP uses **federated identity** — users don't create a separate APP account.

```
User logs into ChatGPT (OpenAI authenticates them)
    ↓
Agent sends payment request with platform's proof of user identity
    ↓
APP verifies the proof (trusts OpenAI's authentication)
    ↓
APP knows who the user is without separate login
```

**Benefit:** Zero friction. User is already logged into their AI platform.

---

## 3. Core Components

### 3.1 Identity Verification

Verifies that payment requests are legitimate.

- Validates platform identity (is this really from OpenAI?)
- Validates user identity (is this really user X?)
- Maps platform user IDs to APP user records
- Maintains registry of trusted platforms

### 3.2 Card Vault

Stores user payment credentials securely.

- Cards tokenized via Stripe (APP never sees raw card numbers)
- Only stores: token reference, last 4 digits, expiry, brand
- Users can have multiple cards
- Users can add/remove cards anytime
- PCI-DSS compliant (SAQ-A scope)

### 3.3 Rules Engine

Enforces user-defined spending controls.

- Checks all rules before every payment
- Tracks spending against limits
- Atomic operations (no race conditions)
- Immediate rule updates (no delay)

### 3.4 Payment Execution

Processes approved payments.

- Integrates with Stripe (MVP)
- Handles authorization, capture, refunds
- Idempotent (no double charges)
- Returns clear success/failure responses

### 3.5 Audit Trail

Logs everything, forever.

- Every request logged (approved or denied)
- Every rule check logged
- Every payment outcome logged
- Immutable (append-only)
- User can query their own history

---

## 4. Guardrails

### User Controls

Users define rules for each agent they authorize:

| Control | Purpose |
|---------|---------|
| **Spending limits** | Cap how much an agent can spend (per transaction, daily, monthly) |
| **Confirmation thresholds** | Require user approval above certain amounts |
| **Instant revocation** | Stop an agent immediately |
| **Transaction visibility** | See everything an agent has spent |

### System Protections

Built-in safeguards that protect users automatically:

| Protection | Purpose |
|------------|---------|
| **Idempotency** | Prevent duplicate charges |
| **Atomic spending** | No overspending via race conditions |
| **Audit logging** | Full accountability |
| **Secure tokenization** | Never expose card data |

### Compliance

| Requirement | How APP Handles It |
|-------------|-------------------|
| **PCI-DSS** | Card data tokenized by Stripe; APP is SAQ-A scope |
| **User consent** | Explicit authorization stored with timestamp |
| **Data minimization** | Only store what's necessary |
| **Right to delete** | Users can remove cards and revoke access |

---

## 5. Open Protocol

### What's Open Source

| Component | Open Source? | Notes |
|-----------|--------------|-------|
| Protocol specification | ✅ Yes | How agents request payments, response formats |
| Reference implementation | ✅ Yes | Working code anyone can run |
| Hosted service | ❌ No | Optional managed service |

### License

**Apache 2.0**

Why:
- Explicit patent grant protects contributors and users
- Patent retaliation clause (if someone sues, their license terminates)
- Enterprise-friendly (legal teams are comfortable with it)
- Industry standard for infrastructure (Kubernetes, Terraform, Kafka)

### Governance

- Open development on GitHub
- Contributions welcome via pull requests
- Major changes via RFC process
- Maintainers approve changes to spec

---

## 6. Open Questions

### Identity & Platforms

| # | Question | Notes |
|---|----------|-------|
| 1 | How do platforms authenticate with APP? | API keys? OAuth client credentials? |
| 2 | What format is the user identity token? | JWT? Opaque token verified via callback? |
| 3 | How does a new platform join? | Open registration? Manual approval? |
| 4 | User on multiple platforms → same APP account? | Need to define linking mechanism |

### Rules & Limits

| # | Question | Notes |
|---|----------|-------|
| 5 | What timezone for daily/monthly resets? | UTC? User timezone? |
| 6 | What are default limits for new users? | Must set? Or sensible defaults? |
| 7 | Can user require confirmation for all purchases? | Maximum control option |

### Legal & Business

| # | Question | Notes |
|---|----------|-------|
| 8 | Money transmission licenses needed? | Likely no (not MoR), but needs legal review |
| 9 | What billing descriptor appears on card statement? | "APP*MERCHANT"? Just "MERCHANT"? |
| 10 | How does APP make money? | Transaction fee? Platform fee? Freemium? |

### Technical

| # | Question | Notes |
|---|----------|-------|
| 11 | What if APP service is down? | Fail closed (deny)? Or degrade gracefully? |
| 12 | How long to retain transaction history? | Compliance requirements vary |
| 13 | Multi-currency support? | USD only for MVP; future consideration |
| 14 | Multi-PSP support? | Stripe only for MVP; future consideration |

---

## 7. Out of Scope

These are explicitly NOT part of APP:

| Out of Scope | Why |
|--------------|-----|
| Payment processing | Stripe handles this |
| Fraud detection | PSP handles this |
| Merchant onboarding | Not needed (merchants receive normal payments) |
| Product catalog / discovery | Agent's responsibility |
| Order management | Agent/merchant responsibility |
| Shipping / fulfillment | Merchant responsibility |
| Shopping UI | Agent platform's responsibility |
| Multi-currency (MVP) | Future consideration |
| Multi-PSP (MVP) | Future consideration |

---

## 8. Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2026-02-15 | Complete rewrite. Federated identity. Open protocol focus. Conceptual design. |
| 1.0 | 2026-02-14 | Initial design. User-owned auth. Implementation-focused. |

---

## 9. Next Steps

1. **Finalize protocol spec** — Define exact request/response formats
2. **Resolve open questions** — Especially identity and platform authentication
3. **Build reference implementation** — Stripe integration, core flows
4. **Recruit first platform partner** — Validate with real integration
5. **Legal review** — Confirm licensing approach, no MTL needed

---

*This document is the source of truth for APP design decisions. Implementation details will be in separate technical documents.*
