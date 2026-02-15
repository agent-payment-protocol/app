# Agent Payment Protocol (APP)

**An open protocol that lets AI agents pay on behalf of users, with user-defined rules.**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Twitter Follow](https://img.shields.io/twitter/follow/agentpayprotocol?style=social)](https://twitter.com/agentpayprotocol)

---

## The Problem

AI agents can browse, search, recommend, and negotiate — but they cannot pay.

When an agent needs to complete a purchase on behalf of a user:
- There's no standard way for agents to access payment credentials
- Users have no way to authorize agents with spending controls
- Each platform builds proprietary solutions (or nothing at all)
- Users can't see or control agent spending across platforms

**Result:** Users can't confidently delegate spending. Agents can't complete purchases. Commerce stays manual.

---

## The Solution

**Agent Payment Protocol (APP)** is an open protocol that lets AI agents pay on behalf of users, with user-defined rules.

| Actor | What They Do |
|-------|--------------|
| **Users** | Store payment credentials once, authorize agents with limits |
| **Agents** | Request payments through a standard API |
| **APP** | Verifies identity, checks rules, executes payment |
| **Merchants** | Receive normal payments (no integration required) |

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. USER SETUP (Once)                                            │
│    User adds card → Sets rules → "Claude can spend $50/day"     │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. PAYMENT (Every Transaction)                                  │
│    Agent requests payment → APP checks rules → Executes or denies│
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. USER CONTROL (Anytime)                                       │
│    View transactions → Change limits → Revoke access            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Features

- **Federated Identity** — Users don't need a separate account. APP trusts the AI platform's authentication.
- **User-Defined Rules** — Spending limits per transaction, daily, monthly. Instant revocation.
- **Open Protocol** — Specification and reference implementation are open source.
- **PCI Compliant** — Cards tokenized via Stripe. APP never sees raw card numbers.
- **Audit Trail** — Every transaction logged. Users have full visibility.

---

## What APP Is

- ✅ An open protocol specification
- ✅ A reference implementation (Apache 2.0)
- ✅ A card vault with user-defined spending rules
- ✅ An authorization layer between users and agents

## What APP Is NOT

- ❌ A payment processor (Stripe, Adyen handle this)
- ❌ A merchant of record
- ❌ An AI platform
- ❌ A shopping experience

---

## Documentation

- [Design Document](docs/DESIGN.md) — Architecture, components, decisions
- [Protocol Specification](docs/PROTOCOL.md) — API contracts (coming soon)
- [Contributing](CONTRIBUTING.md) — How to contribute

---

## Status

🚧 **Early Development** — Protocol specification in progress.

We're looking for:
- AI platforms interested in integrating
- Contributors to the protocol spec
- Feedback on the design

---

## License

Apache 2.0 — See [LICENSE](LICENSE)

---

## Contact

- **GitHub:** [github.com/agent-payment-protocol](https://github.com/agent-payment-protocol)
- **Twitter/X:** [@agentpayprotocol](https://twitter.com/agentpayprotocol)
