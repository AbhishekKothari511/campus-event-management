# 🎓 Campus Event Management System

> **Microsoft Hackathon Project** · Developed by **Abhishek Kothari — 24BCE2610**

---

## 📌 What is this project?

The **Campus Event Management System** is a full-stack web platform that solves a common campus problem: **event chaos**.

- Organizers waste time managing registrations manually via WhatsApp, Google Forms, or email.
- Students miss events because there's no single place to discover them.
- Admins have no visibility into attendance or seat availability.

This platform fixes all of that — one place for organizing, discovering, and registering for campus events.

---

## 🗺️ System Design Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (React.js)                       │
│  ┌──────────┐  ┌──────────────┐  ┌────────────────────────────┐ │
│  │  Login / │  │  Event List  │  │   Organizer / Admin        │ │
│  │  Signup  │  │  & Search    │  │   Dashboard                │ │
│  └────┬─────┘  └──────┬───────┘  └────────────┬───────────────┘ │
└───────┼───────────────┼──────────────────────┼──────────────────┘
        │               │  REST API calls       │
        ▼               ▼  (Axios + JWT token)  ▼
┌─────────────────────────────────────────────────────────────────┐
│                   BACKEND (Node.js + Express.js)                 │
│                                                                   │
│  ┌──────────────┐  ┌───────────────┐  ┌──────────────────────┐  │
│  │  Auth Routes │  │  Event Routes │  │  Registration Routes │  │
│  │  /api/auth   │  │  /api/events  │  │  /api/registrations  │  │
│  └──────┬───────┘  └───────┬───────┘  └──────────┬───────────┘  │
│         │                  │                      │               │
│         ▼                  ▼                      ▼               │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │              JWT Middleware (Role-Based Access)           │    │
│  │         Admin  |  Organizer  |  Student                  │    │
│  └──────────────────────────────┬───────────────────────────┘    │
└─────────────────────────────────┼──────────────────────────────  ┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                     DATABASE (PostgreSQL)                         │
│                                                                   │
│   ┌──────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│   │  users   │    │    events    │    │    registrations     │  │
│   │──────────│    │──────────────│    │──────────────────────│  │
│   │ id       │    │ id           │    │ id                   │  │
│   │ name     │    │ title        │    │ user_id (FK)         │  │
│   │ email    │◄───│ organizer_id │    │ event_id (FK)        │  │
│   │ role     │    │ date/time    │    │ registered_at        │  │
│   │ password │    │ location     │    │ status               │  │
│   └──────────┘    │ seat_limit   │    └──────────────────────┘  │
│                   │ seats_taken  │                                │
│                   └──────────────┘                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 User Flow

```
                    ┌──────────────┐
                    │  User visits  │
                    │  the website  │
                    └──────┬───────┘
                           │
               ┌───────────▼────────────┐
               │  Register / Login Page │
               └───────────┬────────────┘
                           │  JWT token issued
         ┌─────────────────┼─────────────────────┐
         │                 │                      │
    ┌────▼─────┐    ┌──────▼──────┐    ┌──────────▼────────┐
    │  STUDENT  │    │  ORGANIZER  │    │     ADMIN          │
    └────┬─────┘    └──────┬──────┘    └──────────┬────────┘
         │                 │                       │
    ┌────▼──────────┐  ┌───▼──────────────┐  ┌────▼──────────────┐
    │ Browse Events │  │  Create Event    │  │  Manage All Users  │
    │ Search Events │  │  Set Seat Limit  │  │  View All Events   │
    │ Register for  │  │  View Dashboard  │  │  Manage Roles      │
    │   an Event    │  │  See Who Signed  │  │  View Reports      │
    │ View My Regs  │  │     Up           │  │                    │
    └───────────────┘  └──────────────────┘  └────────────────────┘
```

---

## 🛠️ Tech Stack & Why We Used Each Technology

| Technology | Role | Why We Chose It |
|------------|------|-----------------|
| **React.js** | Frontend UI | Component-based architecture makes it easy to build reusable UI pieces (event cards, forms, dashboards). Vite gives instant hot-reload for fast development. |
| **Tailwind CSS** | Styling | Utility-first CSS means no context-switching between files. Responsive design out of the box. Soft pastel theme configured easily. |
| **Node.js** | Backend runtime | Non-blocking I/O handles many concurrent requests (students registering at the same time) without performance degradation. |
| **Express.js** | REST API framework | Minimal and flexible. Routes map cleanly to REST conventions (`GET /events`, `POST /register`). Middleware makes auth easy. |
| **PostgreSQL** | Database | Relational data is a perfect fit — events have organizers (FK), registrations link users to events (FK). ACID compliance prevents double-bookings. |
| **JWT (JSON Web Tokens)** | Authentication | Stateless auth — no session storage needed on the server. Token carries the user's role (admin/organizer/student) so the backend can enforce access control on every request. |

---

## 🔐 Role-Based Access Control

| Feature | Student | Organizer | Admin |
|---------|---------|-----------|-------|
| Browse events | ✅ | ✅ | ✅ |
| Register for event | ✅ | ❌ | ✅ |
| Create event | ❌ | ✅ | ✅ |
| Edit/delete own event | ❌ | ✅ | ✅ |
| View registrations | ❌ | ✅ (own events) | ✅ (all) |
| Manage users | ❌ | ❌ | ✅ |

---

## 🗄️ Database Schema

```sql
-- Users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role VARCHAR(20) DEFAULT 'student'  -- 'admin', 'organizer', 'student'
);

-- Events
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    location VARCHAR(200),
    event_date TIMESTAMP NOT NULL,
    seat_limit INT NOT NULL,
    seats_taken INT DEFAULT 0,
    organizer_id INT REFERENCES users(id)
);

-- Registrations
CREATE TABLE registrations (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    event_id INT REFERENCES events(id),
    registered_at TIMESTAMP DEFAULT NOW(),
    status VARCHAR(20) DEFAULT 'confirmed',
    UNIQUE(user_id, event_id)  -- prevent duplicate registrations
);
```

---

## 🚀 Getting Started

```bash
# Clone the repo
git clone https://github.com/AbhishekKothari511/campus-event-management.git
cd campus-event-management

# ── Backend ──────────────────────────────────
cd backend
npm install
cp .env.example .env       # add your DB URL and JWT secret
npm run dev                # runs on http://localhost:5000

# ── Frontend ─────────────────────────────────
cd ../frontend
npm install
npm run dev                # runs on http://localhost:3000
```

---

## 📁 Project Structure

```
campus-event-management/
├── backend/
│   ├── index.js           # Express server + JWT middleware
│   └── package.json
├── frontend/
│   ├── index.html
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── src/
│       ├── App.jsx        # Main layout (header, footer)
│       ├── main.jsx       # React entry point
│       ├── index.css      # Tailwind entry
│       └── components/
│           └── ExplanationCard.jsx  # Homepage explanation panel
├── .gitignore
└── README.md
```

---

*Built for the Microsoft Hackathon 2026 · Abhishek Kothari — 24BCE2610*
