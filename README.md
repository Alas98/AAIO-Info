# AAIO — AI-Powered WhatsApp Automation for Tuition Centres

![Status](https://img.shields.io/badge/status-active%20development-blue)
![Stage](https://img.shields.io/badge/stage-approaching%20beta-orange)
![Platform](https://img.shields.io/badge/platform-Singapore-red)
![Stack](https://img.shields.io/badge/stack-FastAPI%20%7C%20React%2019%20%7C%20PostgreSQL-informational)
![WhatsApp](https://img.shields.io/badge/messaging-WhatsApp-25D366?logo=whatsapp&logoColor=white)

---

## Overview

**AAIO** (All-In-One) is a commercial SaaS platform that automates customer communication, class scheduling, and student enrollment for tuition centres in Singapore — entirely through WhatsApp.

Tuition centre owners receive hundreds of WhatsApp messages every week: parents asking about available slots, pricing, subjects, and trial class bookings. Managing all of this manually is time-consuming, error-prone, and difficult to scale. AAIO replaces that manual workload with an AI-powered conversational agent that handles enquiries 24/7, books trial classes, manages student enrollment, and surfaces everything to centre staff through a clean web dashboard.

---

## The Problem

Independent and boutique tuition centres in Singapore operate almost entirely over WhatsApp. Typical pain points include:

- **Unstructured enquiries** — parents ask about schedules, fees, and vacancies in unstructured natural language across dozens of concurrent chats
- **Manual booking** — trial class bookings are tracked in spreadsheets, WhatsApp-noted reminders, or physical registers
- **No CRM** — there is no centralised record of leads, enrolled students, or communication history
- **Zero out-of-hours coverage** — enquiries sent at night go unanswered until the next morning, causing drop-off
- **Broadcast fatigue** — sending fee reminders, schedule changes, or promotions to parents requires manually messaging each contact

AAIO solves all of the above in a single product, purpose-built for this market.

---

## Key Features

### AI Conversational Agent
- WhatsApp-native AI bot handles parent enquiries end-to-end in natural language
- Understands questions about class schedules, subjects offered, fees, available slots, and centre policies
- Books trial classes directly within the conversation
- Gracefully escalates complex or sensitive conversations to a human staff member
- Configurable escalation triggers (e.g. keywords that always hand off to staff)
- Smart bot pause/resume: staff can take over a conversation and AAIO steps aside automatically

### Class Scheduling & Calendar
- Class schedule management with per-subject, per-day, per-time configuration
- Google Calendar integration: trial bookings are automatically created as calendar events
- Class availability checking in real time — the bot only offers slots with remaining capacity
- Per-class capacity overrides for fine-grained control
- School closure and holiday management to suppress bookings on non-teaching days

### Student Enrollment
- Full enrollment lifecycle: create, view, and cancel student enrollments
- Conflict detection prevents double-booking a student into overlapping classes
- Enrollment records linked to customer profiles for a complete view of each family

### Customer & Leads CRM
- Every parent who messages is automatically captured as a customer record
- Tracks name, phone number, children's details, subjects of interest, and current status (lead, enrolled, etc.)
- Full conversation history per customer
- Staff can add notes and update customer information from the dashboard
- Flagged and escalated conversations surfaced separately for priority follow-up

### Staff Dashboard
- Responsive single-page web application for centre staff
- Conversation viewer with full message history
- Booking management and trial class overview
- Enrollment management interface
- Analytics: message volume, customer counts, booking rates, escalation frequency
- Configurable business settings: pricing, schedule, subjects, policies, and more

### Broadcast Messaging
- Send bulk WhatsApp messages to any selection of customers
- Use cases: fee reminders, schedule changes, promotions, holiday notices
- Targets existing customer phone numbers — no third-party marketing platform required

### Security & Resilience
- Rate limiting per phone number (hourly and daily message caps) to prevent abuse
- Circuit breaker pattern on the AI layer — automatically pauses AI responses during upstream failures and resumes when service recovers
- Prompt injection defences on all inbound messages before they reach the AI
- Session-based dashboard authentication with bcrypt password hashing
- Input validation on all API endpoints with strict schema enforcement

---

## Tech Stack

### Backend
| Component | Technology |
|-----------|-----------|
| Language | Python |
| API Framework | FastAPI |
| ASGI Server | Uvicorn |
| Data Validation | Pydantic v2 |
| Database ORM | SQLAlchemy 2 |
| Database | PostgreSQL |
| DB Driver | psycopg2 |
| Auth | bcrypt, `secrets` (token generation) |

### Frontend
| Component | Technology |
|-----------|-----------|
| Framework | React 19 |
| Routing | React Router v7 |
| Build Tool | Vite 7 |
| Styling | Tailwind CSS 3 |
| HTTP Client | Axios |
| Icons | Lucide React |
| Linting | ESLint 9 |

### Infrastructure & Deployment
| Component | Technology |
|-----------|-----------|
| Cloud | DigitalOcean (VPS) |
| Web Server / Reverse Proxy | Nginx |
| Process Management | systemd |
| Operating System | Linux (Ubuntu) |

### Integrations & External Services
| Integration | Purpose |
|-------------|---------|
| WhatsApp gateway API | Sends and receives WhatsAPP messages, manages sessions |
| Google Calendar (via `gog` CLI) | Trial class event creation and calendar management |
| Webhook Hooks | Real-time inbound message delivery into AAIO's processing pipeline |

---

## Architecture

AAIO is composed of four main layers that work together to deliver a seamless automation loop from WhatsApp message to calendar event.

```
                          ┌─────────────────────────────┐
                          │       Parent (WhatsApp)     │
                          └────────────┬────────────────┘
                                       │
                            ┌──────────▼──────────┐
                            │   WhatsApp Gateway  │
                            │  (message delivery) │
                            └──────────┬──────────┘
                                       │ webhook hook
                            ┌──────────▼───────────┐
                            │    AAIO Backend      │
                            │    (FastAPI/Python)  │◄──── Staff Dashboard
                            │                      │      (React SPA)
                            └────┬──────────┬──────┘
                                 │          │
                   ┌─────────────▼──┐   ┌───▼───────────────┐
                   │   PostgreSQL   │   │  Google Calendar  │
                   │  (businesses,  │   │  (trial bookings  │
                   │  customers,    │   │   as events)      │
                   │  conversations,│   └───────────────────┘
                   │  enrollments,  │
                   │  bookings)     │
                   └────────────────┘
```

**How it works end-to-end:**

1. **Inbound message** — A parent sends a WhatsApp message. The WhatsApp gateway receives it and fires a webhook to AAIO's internal API to AAIO's internal API.
2. **Message processing** — AAIO logs the message, upserts the customer record, applies rate limiting, and checks whether the bot is paused for this conversation.
3. **AI response** — If the bot is active, AAIO retrieves the business context (schedule, pricing, policies, closures) and passes the conversation to the AI layer. The AI generates a contextually aware reply.
4. **Action handling** — If the AI determines the parent wants to book a trial class, AAIO checks availability, creates a database record, and creates a Google Calendar event — all automatically.
5. **Escalation** — If a trigger keyword is detected or the AI cannot confidently resolve the enquiry, the conversation is flagged and the designated staff member is notified via WhatsApp.
6. **Staff dashboard** — Centre staff use the React dashboard to view conversations, manage enrollments, update business settings, and send broadcast messages at any time.

---

## Security

AAIO is built with the security requirements of a multi-tenant SaaS product in mind:

- **Authentication** — Dashboard access is protected by session tokens issued on login, with bcrypt-hashed passwords and 24-hour token expiry
- **Rate limiting** — Inbound WhatsApp messages are rate-limited per phone number (50 messages/hour, 200 messages/day) to prevent abuse and runaway AI usage
- **Circuit breaker** — The AI integration layer implements a circuit breaker: after a configurable number of consecutive failures, the AI is automatically suspended and traffic falls back to human escalation. It re-enables automatically once the service recovers
- **Prompt injection defence** — All inbound message content is sanitised before being passed to the AI to prevent prompt injection attacks
- **Input validation** — All API endpoints enforce strict schema validation via Pydantic, including E.164 phone number format checks
- **Parameterised queries** — Database access uses bound parameters throughout to prevent SQL injection
- **Timeout enforcement** — All external subprocess calls (WhatsApp, calendar) are wrapped with timeouts to prevent hanging requests

---

## Project Status

AAIO is currently in **active development**, approaching **private beta**. The core automation loop — inbound message handling, AI response generation, trial booking, and Google Calendar integration — is functional. The staff dashboard covers the full operational surface for a single tuition centre.

**Roadmap priorities before beta launch:**
- Hardened internal API authentication (shared secrets / network ACL)
- External session store for zero-downtime restarts
- Production CORS policy enforcement
- Payments integration for automated fee collection

**Target market:** Independent and boutique tuition centres in Singapore.

---

## Demo Video

https://drive.google.com/file/d/1ZgIx_sVHGQI1a6gyc0mtP-21Xl4vz6NP/view?usp=sharing

---

## About

AAIO is an independent software project designed and built from the ground up as a vertical SaaS product for the Singapore education market. It is built by a solo developer with the goal of giving small tuition centre operators access to automation tooling that has previously only been available to large enterprises.

---

<div align="center">

**This is a private commercial repository. Source code is not publicly available.**

*For enquiries, please reach out via GitHub.*

</div>
