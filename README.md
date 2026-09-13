# TeamPulse — Weekly Team Reporting & Performance Dashboard

A full-stack enterprise MERN web application built for the **Sisenco Digital** technical assignment. Designed for structured weekly team reporting, manager evaluation workflows, version-controlled audit trails, and real-time team analytics.

---

## 1. Key Features

- **Authentication & RBAC**: Secure HTTP-only cookie-based authentication with strict role-based access control.
- **Weekly Report Generator**: Comprehensive progress tracking covering completed tasks, priorities, spent hours, deliverables, next week plans, blockers, achievements, and resource links.
- **State Machine Lifecycle**: Deterministic workflow progression (`DRAFT` $\rightarrow$ `SUBMITTED` $\rightarrow$ `NEEDS_CORRECTION` $\rightarrow$ `APPROVED`).
- **Immutable Version History**: Point-in-time version snapshots (`ReportVersion`) captured on every submission for audit and diffing.
- **Manager Review Inbox & Detail**: Centralized queue for pending reports with one-click approvals and mandatory-feedback revision requests (`ReviewHistory`).
- **Project & Allocation Management**: Project directory with category classification, team member assignment, and safe archiving.
- **Team Directory & Account Controls**: User management with activation/deactivation toggles and self-deactivation prevention guards.
- **Executive Analytics Dashboard**: High-level KPI summary cards, Recharts weekly submission velocity trends, and individual team member performance metrics.

---

## 2. Roles & Permissions

The system strictly enforces a dual-role access control model:

| Capability / Resource | `TEAM_MEMBER` | `MANAGER_ADMIN` |
| :--- | :---: | :---: |
| **Create & Edit Own Drafts** | ✅ | ❌ *(Reviews only)* |
| **Submit Weekly Reports** | ✅ | ❌ |
| **View Own Reports & Snapshots** | ✅ | ✅ |
| **View Other Members' Reports** | ❌ *(Protected)* | ✅ |
| **Approve Submitted Reports** | ❌ | ✅ |
| **Request Report Corrections** | ❌ | ✅ *(Mandatory comment)* |
| **Access Executive Dashboard** | ❌ | ✅ |
| **Manage Projects & Assignments**| ❌ *(Assigned only)* | ✅ |
| **Manage Team Directory & Access**| ❌ | ✅ *(Self-guard enabled)* |

---

## 3. Report Workflow & Lifecycle

The application enforces a strict, deterministic report state machine:

### Standard Approval Cycle
```text
[ DRAFT ] ──────────► [ SUBMITTED ] ──────────► [ APPROVED ]
 (Author creates)     (v1 Snapshot saved)     (Manager reviews & finalizes)
```

### Correction & Resubmission Cycle
```text
[ DRAFT ] ──────────► [ SUBMITTED ] ──────────► [ NEEDS_CORRECTION ]
                        │                       (Manager feedback required)
                        │                                  │
                        │                       (Author revises)
                        │                                  ▼
                        └──────── [ SUBMITTED ] ◄──────────┘
                              (v2+ Snapshot saved)
                                       │
                                (Manager approves)
                                       ▼
                                 [ APPROVED ]
```

---

## 4. Technology Stack

### Frontend
- **Framework**: React 18 (Vite)
- **Styling**: Tailwind CSS (Dark-mode optimized palette & design tokens)
- **Routing**: React Router DOM v6
- **HTTP Client**: Axios (Centralized interceptors & `withCredentials: true`)
- **Forms & Validation**: React Hook Form + Zod
- **Data Visualization**: Recharts (Weekly submission trends)
- **Icons**: Lucide React

### Backend
- **Runtime & Framework**: Node.js & Express.js
- **Database & ODM**: MongoDB & Mongoose
- **Authentication**: JWT (`jsonwebtoken`) & `cookie-parser`
- **Password Hashing**: `bcryptjs` (10 salt rounds)
- **Schema Validation**: Zod (Express validation middleware)
- **Testing**: Jest & Supertest

---

## 5. System Architecture & Layering

```text
React Client (SPA)
       │ (JSON over HTTP with HTTP-only cookies)
       ▼
Express API Router (`/api/*`)
       │
Middleware Pipeline (CORS, CookieParser, AuthenticateUser, AuthorizeRoles, Validate)
       │
Controllers (HTTP input parsing & ApiResponse mapping)
       │
Services (Business logic & transaction rules)
       │
Mongoose ODM Models (Data validation & indexes)
       │
MongoDB (Persistent Collections)
```

### Directory Structure

```text
├── backend/
│   ├── src/
│   │   ├── config/          # Environment variables & MongoDB connection
│   │   ├── middleware/      # Auth, RBAC, validation, notFound, errorHandler
│   │   ├── models/          # User, Project, Report, ReportVersion, ReviewHistory
│   │   ├── modules/         # auth, projects, reports, reviews, users, dashboard
│   │   ├── utils/           # ApiResponse & ApiError utilities
│   │   ├── app.js           # Express configuration & route binding
│   │   └── server.js        # Server bootstrap
│   ├── tests/               # 90 Automated integration tests
│   ├── seed/                # Seed script with demo dataset
│   ├── .env.example
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/      # UI primitives, common layout, reports
│   │   ├── context/         # AuthContext & ToastContext
│   │   ├── hooks/           # useAuth, useToast
│   │   ├── layouts/         # AuthLayout, MainLayout, Sidebar, Navbar
│   │   ├── pages/           # Auth, Member (MyReports, Create, Edit, Detail), Manager (Dashboard, Inbox, Review, Projects, Team)
│   │   ├── routes/          # AppRoutes with ProtectedRoute guards
│   │   └── services/        # api.js, authService, reportService, reviewService, projectService, userService, dashboardService
│   ├── .env.example
│   └── package.json
├── docs/                    # Architecture, Database Design, API Specs, RBAC Specs
└── README.md
```

---

## 6. Prerequisites

- **Node.js**: `v18.x` or `v20.x`
- **npm**: `v9.x` or `v10.x`
- **MongoDB**: `v6.x` or `v7.x` running locally (`mongodb://localhost:27017`) or a MongoDB Atlas connection string.

---

## 7. Installation & Setup

### Step 1: Clone Repository
```bash
git clone https://github.com/VikumBhashitha02/TodoList.git
cd TodoList
```

### Step 2: Backend Setup
```bash
cd backend
npm install
cp .env.example .env
```

*Ensure your backend `.env` is configured:*
```env
PORT=5000
NODE_ENV=development
MONGODB_URI=mongodb://localhost:27017/team_report_dashboard
CORS_ORIGIN=http://localhost:5173
JWT_SECRET=replace_with_strong_secret_key
JWT_EXPIRES_IN=1d
COOKIE_NAME=auth_token
```

### Step 3: Frontend Setup
```bash
cd ../frontend
npm install
cp .env.example .env
```

*Ensure your frontend `.env` is configured:*
```env
VITE_API_URL=http://localhost:5000/api
```

---

## 8. Database Seeding & Development Accounts

Populate the database with realistic demonstration data (users, projects, draft reports, submitted reports, approved reports with version snapshots, and review history):

```bash
cd backend
npm run seed
```

### Development Demo Accounts

| Role | Email | Password | Description |
| :--- | :--- | :--- | :--- |
| **`MANAGER_ADMIN`** | `manager@example.com` | `Password123!` | Full manager & executive access |
| **`TEAM_MEMBER`** | `alex@example.com` | `Password123!` | Team member with approved & draft reports |
| **`TEAM_MEMBER`** | `sarah@example.com` | `Password123!` | Team member with pending review report |

---

## 9. Running the Application

### Start Backend API Server
```bash
cd backend
npm run dev
# Server runs on http://localhost:5000
```

### Start Frontend Dev Client
```bash
cd frontend
npm run dev
# Application accessible at http://localhost:5173
```

---

## 10. Automated Testing & Verification

Run the full automated backend test suite covering Authentication, RBAC, Projects, Reports, Versioning, Review Workflows, Users, and Dashboard Analytics:

```bash
cd backend
npm test
```

**Expected Result:**
```text
Test Suites: 6 passed, 6 total
Tests:       90 passed, 90 total
Snapshots:   0 total
Time:        ~10-11s
```

### Production Build Verification
```bash
cd frontend
npm run build
# Production assets built cleanly into frontend/dist/
```

---

## 11. API Endpoint Overview

| Module | Method | Endpoint | Access | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Auth** | `POST` | `/api/auth/register` | Public | Register new `TEAM_MEMBER` |
| | `POST` | `/api/auth/login` | Public | Login & set HTTP-only cookie |
| | `POST` | `/api/auth/logout` | Public | Logout & clear session cookie |
| | `GET` | `/api/auth/me` | Authenticated | Restore current session profile |
| **Reports** | `GET` | `/api/reports` | Authenticated | List reports (member/manager filtered) |
| | `GET` | `/api/reports/:id` | Authenticated | Get single report details |
| | `GET` | `/api/reports/:id/versions`| Authenticated | Get immutable version snapshots |
| | `POST` | `/api/reports` | `TEAM_MEMBER` | Create new draft report |
| | `PUT` | `/api/reports/:id` | `TEAM_MEMBER` | Edit draft or needs-correction report |
| | `PATCH` | `/api/reports/:id/submit` | `TEAM_MEMBER` | Submit report for review (snapshot v1+) |
| | `DELETE` | `/api/reports/:id` | `TEAM_MEMBER` | Delete draft report |
| **Reviews** | `GET` | `/api/reviews/pending` | `MANAGER_ADMIN` | Get pending review inbox queue |
| | `GET` | `/api/reviews/:id/history` | `MANAGER_ADMIN` | Retrieve review audit log timeline |
| | `PATCH` | `/api/reviews/:id/approve` | `MANAGER_ADMIN` | Approve submitted report |
| | `PATCH` | `/api/reviews/:id/request-correction` | `MANAGER_ADMIN` | Request changes (mandatory comment) |
| **Projects** | `GET` | `/api/projects` | Authenticated | List accessible projects |
| | `GET` | `/api/projects/categories`| Authenticated | List distinct categories |
| | `POST` | `/api/projects` | `MANAGER_ADMIN` | Create new project |
| | `PUT` | `/api/projects/:id` | `MANAGER_ADMIN` | Update project metadata |
| | `PATCH` | `/api/projects/:id/members`| `MANAGER_ADMIN` | Assign team members to project |
| | `DELETE` | `/api/projects/:id` | `MANAGER_ADMIN` | Delete project |
| **Users** | `GET` | `/api/users` | Authenticated | Team directory listing |
| | `PATCH` | `/api/users/:id/deactivate`| `MANAGER_ADMIN` | Deactivate user account |
| | `PATCH` | `/api/users/:id/reactivate`| `MANAGER_ADMIN` | Reactivate user account |
| **Dashboard**| `GET` | `/api/dashboard/stats` | `MANAGER_ADMIN` | Executive KPI overview metrics |
| | `GET` | `/api/dashboard/trend` | `MANAGER_ADMIN` | Weekly submission velocity trend |
| | `GET` | `/api/dashboard/members` | `MANAGER_ADMIN` | Team member performance table |
| **Health** | `GET` | `/api/health` | Public | System uptime & DB connection status |

---

## 12. Security & Compliance Measures

1. **Zero Client Token Storage**: JWTs are strictly transmitted via HTTP-only, `sameSite: 'lax'` cookies. No tokens exist in `localStorage` or `sessionStorage` (mitigating XSS token exfiltration).
2. **Password Protection**: Passwords are encrypted with bcrypt (10 rounds) and excluded from Mongoose query projections by default.
3. **Strict RBAC & Server Authority**: Authorization is enforced on every backend route via `authenticateUser` and `authorizeRoles`. Client-side route hiding is backed by server-side verification.
4. **Ownership Enforcement (Anti-IDOR)**: Team members cannot view, edit, submit, or delete reports owned by other users.
5. **Self-Deactivation Prevention**: Managers cannot deactivate their own active accounts, preventing organizational lockout.
6. **Payload Sanitization**: Zod validation schemas strictly filter unexpected fields on request bodies.

---

## 13. Documentation References

- [Architecture & Design Rationale](file:///d:/Sisenco%20Assignment/docs/ARCHITECTURE.md)
- [Database Schema & ER Diagram](file:///d:/Sisenco%20Assignment/docs/database-design.md)
- [API Endpoint Specification](file:///d:/Sisenco%20Assignment/docs/api-specification.md)
- [Authentication & RBAC Specification](file:///d:/Sisenco%20Assignment/docs/authentication-and-rbac.md)
- [Frontend Architecture Specification](file:///d:/Sisenco%20Assignment/docs/frontend-architecture.md)
- [Technical Presentation & Slide Deck](file:///d:/Sisenco%20Assignment/docs/PRESENTATION.md)
