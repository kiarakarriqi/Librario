# Library Management System (LMS)

A web-based library management platform built as the Software Engineering course project.
Members can search the catalog, request borrows, reserve books, book study rooms, donate
books, register for events, and track personal reading statistics. Staff approve borrow
requests, process returns with book-condition assessment, settle cash deposits, confirm
fine payments, and moderate reviews. Administrators manage users, fine policy, and
library-wide statistics.

---

## Team

| Name | 
|---|---|
| 
| Kiara Karriqi (team lead) 
| Xhim Zotaj 
| Migert Blloshmi 
| Haris Kockici 
| Meriklesta Hasanaj 
| Kasem Bilal Kasemi 


---

## Technology Stack

**Backend**
- .NET 7 (ASP.NET Core Web API)
- Entity Framework Core 7.0.20
- SQLite (Microsoft.EntityFrameworkCore.Sqlite 7.0.7)
- BCrypt.Net-Next 4.0.3 — password hashing
- Swashbuckle.AspNetCore 6.5.0 — Swagger / OpenAPI

**Frontend**
- Static HTML5 + CSS + vanilla JavaScript
- No build step, no client-side framework

**Database**
- SQLite (single `library.db` file)
- EF Core migrations (6 versioned migrations)

---

## Repository Structure

```
LibraryProject/
├── LibraryAPI/                   # ASP.NET Core Web API project
│   ├── Controllers/              # REST controllers grouped by domain
│   │   ├── AuthController.cs
│   │   ├── BooksController.cs
│   │   ├── LoansController.cs
│   │   ├── BorrowRequestController.cs
│   │   └── AllControllers.cs     # Reservations, Fines, Reviews, Wishlist,
│   │                             # Rooms, Donations, Events, Notifications,
│   │                             # Stats, Admin, Authors, Categories
│   ├── Data/
│   │   └── LibraryDbContext.cs   # EF Core context, all DbSets, unique indexes
│   ├── Models/
│   │   └── Models.cs             # All entity classes + enums
│   ├── Services/
│   │   └── OverdueProcessorService.cs  # Background worker (runs every 6h)
│   ├── Migrations/               # 6 EF Core migrations
│   ├── Program.cs                # Startup, seed data, DI configuration
│   └── LibraryAPI.csproj
│
├── FrontEnd/                     # Static HTML/CSS/JS frontend
│   ├── login.html / register.html
│   ├── index.html                # Member dashboard
│   ├── catalog.html              # Book search & request
│   ├── loans.html                # Member loans + pending requests
│   ├── reservations.html
│   ├── wishlist.html
│   ├── fines.html
│   ├── rooms.html                # Study room booking
│   ├── donations.html
│   ├── events.html
│   ├── notifications.html
│   ├── stats.html                # Personal reading stats
│   ├── account.html
│   ├── staff-dashboard.html      # Staff & admin console
│   ├── auth.js                   # Session helpers
│   ├── dialog.js                 # Modal helpers
│   ├── notifications-widget.js
│   └── style.css
│
├── LibraryManagementSystemReport_finalme_ndryshime.docx
│                                 # Full software documentation
│                                 # (requirements, diagrams, design, screenshots)
└── README.md                     # This file
```

---

## Getting Started

### Prerequisites

- [.NET 7 SDK](https://dotnet.microsoft.com/download/dotnet/7.0)
- A modern browser (Chrome, Safari, Firefox, Edge)

### Running the project

```bash
# 1. Clone the repo
git clone <repo-url>
cd LibraryProject/LibraryAPI

# 2. Restore packages and run
dotnet restore
dotnet run
```

The API starts on `http://localhost:5273`. On first run, EF Core applies all 6
migrations and seeds:
- 3 test users (Admin / Staff / Member)
- 6 categories, 5 authors, 9 books, 4 study rooms
- Default fine policy (€0.50/day, €5 borrow-block threshold)

### Access the application

Open in browser:
- **Main app:** http://localhost:5273/login.html
- **Swagger UI:** http://localhost:5273/swagger

### Test accounts (seeded on first run)

| Role | Email | Password |
|---|---|---|
| Admin | admin@librario.com | Admin123! |
| Staff | staff@librario.com | Staff123! |
| Member | member@librario.com | Member123! |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  Browser (Member / Staff / Admin)                       │
│  ─ HTML pages + auth.js + dialog.js                     │
└────────────────────┬────────────────────────────────────┘
                     │ HTTP / JSON
┌────────────────────▼────────────────────────────────────┐
│  ASP.NET Core Web API (port 5273)                       │
│  ─ Controllers grouped by domain                        │
│  ─ Static file middleware (serves FrontEnd/)            │
│  ─ OverdueProcessorService (IHostedService, every 6h)   │
└────────────────────┬────────────────────────────────────┘
                     │ EF Core
┌────────────────────▼────────────────────────────────────┐
│  LibraryDbContext                                       │
│  ─ 20+ DbSets, unique indexes, FK cascade rules         │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│  SQLite (library.db)                                    │
└─────────────────────────────────────────────────────────┘
```

---

## Key Features

### Member features (FR-01 through FR-22)
- Account registration & authentication
- Catalog search (title / author / category)
- Book detail view with reviews and ratings
- Borrow request submission (1–30 days, max 5 active loans)
- Loan history & renewal (max 2 renewals, +14 days each)
- Book reservations (FIFO queue, max 3 active, 3-day expiry)
- Book reviews (1–5 stars, only after returning the book)
- Wishlist management
- Personal fine view & cash payment request
- Study room booking (max 2 active, 2-hour slots)
- Book donations
- Event registration with waitlist & auto-promotion
- Personal reading statistics & recommendations

### Staff features (FR-23 through FR-31)
- View pending borrow requests
- Approve / reject requests with deposit handling
- Process book returns with condition assessment
  (Good / MinorWear / SignificantDamage / MajorDamage / Lost)
- Settle held deposits (return or forfeit)
- Confirm cash fine payments
- Moderate book reviews (soft-delete)
- Approve / reject book donations
- Add new books to the catalog

### Admin features (FR-32 through FR-35)
- User management (search, activate/deactivate, change role)
- Fine waiver with documented reason
- Fine policy configuration (daily rate, grace period, threshold)
- Library-wide statistics dashboard

---

## API Documentation

All endpoints follow REST conventions:
- Routes: `/api/[controller]` (e.g. `GET /api/Books`, `POST /api/borrowrequest`)
- Bodies: JSON
- Status codes: 200 / 201 / 400 / 401 / 404 / 409

Full interactive documentation is available at `/swagger` when the API is running.

---

## Database Schema

The schema is defined in `LibraryAPI/Data/LibraryDbContext.cs` and managed through
6 EF Core migrations:

1. `InitialCreate` — base schema (User, Book, Author, Category, Loan, Reservation,
   Fine, FinePolicy, BookReview, Wishlist, Room, RoomBooking, BookDonation,
   LibraryEvent, EventRegistration, Notification)
2. `BorrowRequestAndDeposit` — BorrowRequest & Deposit entities
3. `FixupBorrowRequestAndDeposit` — schema fixes
4. `AddLoanRenewal` — RenewalCount & LastRenewedAt on Loan
5. `AddSuspensionAndOverdueFields` — BorrowSuspendedUntil on User, suspension
   policy on FinePolicy
6. `AddDamageFeesAndConditionTracking` — damage fee fields on FinePolicy,
   ReturnCondition & DamageFee on Loan

Unique indexes enforce key business invariants:
- `Wishlist(UserId, BookId)` — one wishlist entry per user-book
- `BookReview(UserId, BookId)` — one review per user-book
- `EventRegistration(EventId, UserId)` — one registration per user-event
- `DismissedRecommendation(UserId, BookId)` — one dismissal per user-book

---

## Development Process

The project was developed using an **iterative-incremental approach** with Agile
practices (informal). Each migration in the `Migrations/` folder corresponds to
a feature increment that was designed, implemented, tested, and integrated before
moving to the next. Team communication was informal (daily messaging) rather than
formal Scrum ceremonies.

Feature evolution timeline (visible in migration history):
1. Core schema with users, books, basic loans
2. Borrow request workflow with cash deposit
3. Schema refinements
4. Loan renewal feature
5. Member suspension policy
6. Book condition tracking & damage fees

---

## Known Limitations

These are intentionally out of scope for the academic version and identified as
hardening items before any production deployment:

- **Authorization is client-side only.** The Web API does not currently apply
  `[Authorize]` attributes or JWT/cookie middleware. Role enforcement is performed
  in `auth.js` based on the role stored in `sessionStorage`.
- **No automated tests.** Unit and integration tests are out of scope for this
  version.
- **Cash-only payments.** No online payment gateway integration.
- **Single-server deployment.** No multi-server replication, load balancing, or
  Docker packaging.
- **English / Albanian UI only.** No formal i18n framework.

---

## Documentation

The full software documentation is in LibraryReport

- Section 1: Executive Summary
- Section 2: System Context & Stakeholders
- Section 3: Requirements (35 functional + non-functional)
- Section 4: Software Design (use case, activity, sequence, state, class, ERD,
  component, and deployment diagrams)
- Section 5: Implementation
- Section 6: Screenshots

---

## License

Academic project — not licensed for commercial use.
