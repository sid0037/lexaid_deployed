---

<h1 align="center">LexAid — Legal Aid Access Platform</h1>

<p align="center">
  <b>A static, multi-page platform connecting clients who need legal help with lawyers who can take their case — built entirely on Firebase Authentication and Cloud Firestore, with no custom backend.</b>
  <br>
  <b>Client-side matching, real-time per-case chat, and full case lifecycle management, all running from plain HTML, CSS, and JavaScript.</b>
</p>

---

<p align="center">
  <img src="https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white" alt="HTML5"/>
  <img src="https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white" alt="CSS3"/>
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black" alt="JavaScript"/>
  <img src="https://img.shields.io/badge/GSAP-88CE02?style=for-the-badge&logo=greensock&logoColor=white" alt="GSAP"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Firebase%20Auth-FFCA28?style=for-the-badge&logo=firebase&logoColor=black" alt="Firebase Auth"/>
  <img src="https://img.shields.io/badge/Cloud%20Firestore-FFA000?style=for-the-badge&logo=firebase&logoColor=black" alt="Cloud Firestore"/>
  <img src="https://img.shields.io/badge/No%20Backend-000000?style=for-the-badge&logo=vercel&logoColor=white" alt="No Backend"/>
</p>

---

## Overview

People who need legal help — especially those without existing connections to lawyers — often don't know where to start: who to contact, whether a given lawyer even handles their type of case, or how to talk to them securely once connected. LexAid targets that discovery-and-communication gap. Clients submit a description of their legal issue, get matched with lawyers by practice area, send a case request, and — once accepted — communicate through a private, per-case chat. Lawyers get a dashboard to review, accept or decline requests, and manage active cases through to closure. This is an academic prototype built for a Software Engineering course at JIIT (Jaypee Institute of Information Technology), as credited on the landing page — not a production legal-services product.

## Key Features

**Fully implemented**
- **Email/password authentication** via Firebase Auth, with separate registration flows for "Client" and "Lawyer" roles
- **Role-based dashboards** — distinct UI and permitted actions per role, enforced via client-side redirects based on the role stored in each user's Firestore document
- **Case request submission** — clients create a request (title, type/practice area, description) written to Firestore
- **Lawyer discovery & matching** — a client-side scoring algorithm ranks lawyers by specialization match, historical success rate, and years of experience
- **Targeted requests** — a client can send a case directly to a specific lawyer, who can accept or decline
- **Open case pool** — untargeted or declined-and-reopened cases are visible to all lawyers; one active case per lawyer is enforced
- **Case lifecycle management** — pending → accepted → closed, plus decline, "leave case," and "get new lawyer" paths
- **Real-time per-case chat** — Firestore subcollection synced live via `onSnapshot`; read-only once a case closes
- **Ratings & feedback** — clients rate (1–5 stars) and review a lawyer after closure, one review per case
- **Public/own profile pages** — sensitive fields (phone, email, Bar Council ID) redacted on public views
- **Profile editing & account management** — password change with re-authentication, typed-confirmation account deletion

**Partially implemented / notable gaps**
- **Lawyer verification** — Bar Council ID is optional, self-reported, with no admin approval or verification step
- **Email verification** — the flow exists but doesn't gate any functionality, only toggles a profile badge
- **"Encrypted" chat** — landing page copy says "encrypted," but messages are plain Firestore documents secured only by transport-layer TLS

## User Roles

| Capability | Client | Lawyer |
|---|---|---|
| Submit a legal aid request | ✅ | — |
| Browse/search lawyers, view match score | ✅ | — |
| Send a request to a specific lawyer | ✅ | — |
| View & accept/decline incoming requests | — | ✅ |
| Browse the open case pool | — | ✅ |
| Hold more than one active case at a time | — | ❌ (limited to 1) |
| Chat on an accepted case | ✅ | ✅ |
| Close a case | — | ✅ |
| Rate/review a lawyer after case closure | ✅ | — |

## Architecture & Workflow

```text
+---------------------------------------------------------+
|              Static HTML / CSS / JS Pages               |
|               (No framework, no bundler)                |
|                                                         |
|   login.html              register.html                 |
|   client-dashboard.html   lawyer-dashboard.html         |
|   find-lawyer.html        chat.html                     |
|   profile.html                                          |
+----------------------------+----------------------------+
                             |
                             | Firebase JS SDK v10.7.1
                             | (ES modules via gstatic.com)
                             v
+---------------------------------------------------------+
|             Firebase Project (legal-aid-system)         |
|                                                         |
|   +------------------------+  +---------------------+   |
|   |     Firebase Auth      |  |   Cloud Firestore   |   |
|   +------------------------+  +---------------------+   |
|   | * Email/Password       |  | * users             |   |
|   | * onAuthStateChanged   |  | * requests          |   |
|   |                        |  |   └─ /messages      |   |
|   |                        |  | * feedback          |   |
|   +------------------------+  +---------------------+   |
+---------------------------------------------------------+
```

**Client path:** `final_index.html` → `register.html`/`login.html` → `client-dashboard.html` (submit a request) → `find-lawyer.html` (get matched, send to a lawyer) → wait for acceptance → `chat.html` → rate the lawyer once the case is closed

**Lawyer path:** `final_index.html` → `register.html`/`login.html` → `lawyer-dashboard.html` (review requests/open pool, accept/decline) → `chat.html` → close the case when resolved

All role checks and permitted actions are re-derived on each page from the Firestore `users` document (not from cached `sessionStorage`), after `onAuthStateChanged` fires.

## Directory Structure

```text
lexaid_deployed/
├── index.html              # Redirects immediately to final_index.html
├── final_index.html        # Landing / marketing page
├── login.html              # Sign-in page
├── register.html           # Sign-up page (client or lawyer)
├── client-dashboard.html   # Client's case list, submission form, feedback modal
├── lawyer-dashboard.html   # Lawyer's request queue, open case pool, case actions
├── find-lawyer.html        # 3-step wizard: pick case → match & pick lawyer → confirm
├── chat.html               # Real-time per-case chat + post-close feedback
├── profile.html            # Own/public profile, stats, account settings
└── README.md               # Documentation
```
Every page is a single self-contained `.html` file with inline `<style>` and `<script type="module">` blocks — there are no subfolders, no `npm`, and no `package.json` in this repository.

## Getting Started & Local Setup

### Prerequisites
- Any modern web browser
- A local static file server (module imports require an HTTP origin, not `file://`)

### 1. Clone the repository
```bash
git clone https://github.com/sid0037/lexaid_deployed.git
cd lexaid_deployed
```

### 2. Serve it locally
```bash
# Using Python
python3 -m http.server 5500
```
Or use the VS Code "Live Server" extension — this repo's `.vscode/settings.json` already pins it to port 5501.

### 3. Open it
Navigate to:
http://localhost:5500/index.html

### Environment / Configuration
There is no `.env` file — the app connects to an already-provisioned Firebase project (`legal-aid-system`) whose config is hardcoded directly into each page's `<script>` block. To point the project at your **own** Firebase backend:

1. Create a Firebase project with **Authentication** (Email/Password) and **Cloud Firestore** enabled.
2. Replace the `firebaseConfig` object in each of the seven pages that contain one (`login.html`, `register.html`, `client-dashboard.html`, `lawyer-dashboard.html`, `find-lawyer.html`, `chat.html`, `profile.html`) with your own project's config.
3. Write Firestore Security Rules for the `users`, `requests`, `requests/{id}/messages`, and `feedback` collections — none are included in this repository.

> **Security note:** the current Firebase config (API key, project ID, app ID) is hardcoded in plaintext across all seven pages. Firebase Web API keys aren't secret the way traditional API keys are, but they should still be paired with proper Firestore Security Rules and App Check — neither is present here.

## Deployment

No deployment configuration (`netlify.toml`, `vercel.json`, GitHub Actions, GitHub Pages) is present in this repository, and no homepage URL is set on GitHub. Despite the repository's name, no live deployed URL could be verified from its contents. As a static site, it can be deployed as-is to Firebase Hosting, Netlify, Vercel, or GitHub Pages.

## Current Limitations

- No backend — all business logic (matching score, one-case-per-lawyer, feedback dedup) runs client-side and can be bypassed unless matching Firestore Security Rules exist server-side (unverifiable from this repo)
- No automated tests, CI/CD, or deployment pipeline
- Lawyer credentials are unverified/self-reported
- Lawyer dashboard loads the entire `requests` collection client-side rather than a scoped query
- "Encrypted" language on the landing page overstates the actual implementation

## Future Improvements

- Add Firestore Security Rules to enforce role-based access server-side
- Verify lawyer credentials against an authoritative source or add an admin approval step
- Gate sensitive actions behind email verification instead of just a badge
- Paginate/scope the lawyer dashboard's request feed
- Add automated tests and a CI pipeline
- Add a documented, repeatable deployment target

##  Team / Contributors

- **Siddharth Narang** ([@sid0037](https://github.com/sid0037))
- **Ojaswi Rao**

## 📄 License & Credits

No license file is present in this repository, so none is stated here. All rights are reserved by the contributors unless a license is added.

**Built with:** Firebase Authentication, Cloud Firestore, and GSAP — no other third-party frameworks or libraries.

---

