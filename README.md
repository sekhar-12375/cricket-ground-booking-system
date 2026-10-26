# Cricket Ground Booking System

Full-stack responsive web application built for a **B.Tech Software Engineering / UML Architecture** semester project. The system provides an end-to-end platform for cricket enthusiasts to browse stadiums, verify live turf slot availability, reserve match hours, simulate payments, and obtain official admission vouchers. An administrator portal provides live oversight, pitch management, and booking audits.

---

## 1. Project Objective & Problem Statement

### Problem Statement
Traditional sports venue reservations suffer from fragmented phone scheduling, manual record-keeping errors, and frequent **double-booking conflicts** where two teams arrive for the same turf slot simultaneously.

### Project Objective
To engineer a robust, UML-compliant web application with:
1. Strict server-side mathematical verification ensuring **zero double-booking overlap**.
2. Atomic database transitions moving reservations from `PENDING_PAYMENT` to `CONFIRMED` only after verified transaction completion.
3. Separation of duties via **Role-Based Access Control (RBAC)** between `USER` and `ADMIN` actors.
4. Professional academic presentation quality suitable for collegiate viva demonstrations.

---

## 2. UML Architecture & Design Mapping

The implementation directly maps to standard UML engineering diagrams:

### Actors
- **User**: Authenticates, browses grounds, verifies slot availability, books pitch, completes payment, views/cancels bookings.
- **Admin**: Authenticates, adds and updates cricket grounds, manages availability states, and audits bookings/revenue.

### Use Cases
- **User Use Cases**: `Login`, `List/View Grounds`, `Check Ground Availability`, `Booking`, `Payment`, `View Booking Confirmation`, `Logout`.
- **Admin Use Cases**: `Login`, `Add Ground`, `Update Ground`, `Manage Ground Availability`, `View Bookings`, `Logout`.

### Class Diagram Mapping
| UML Class | Attributes | Methods / Operations |
| :--- | :--- | :--- |
| **User** | `userId`, `name`, `phone`, `email`, `password_hash` | `login()`, `register()`, `bookGround()`, `logout()` |
| **Admin** | `adminId`, `name`, `email`, `password_hash` | `login()`, `addGround()`, `updateGround()`, `viewBooking()` |
| **Ground** | `groundId`, `groundName`, `location`, `description`, `hourlyPrice`, `openingTime`, `closingTime`, `availability` | `checkAvailability()`, `updateAvailability()` |
| **Booking** | `bookingId`, `userId`, `groundId`, `bookingDate`, `startTime`, `endTime`, `numberOfPlayers`, `amount`, `bookingStatus`, `paymentStatus` | `createBooking()`, `cancelBooking()`, `makePayment()` |
| **Payment** | `paymentId`, `bookingId`, `transactionId`, `amount`, `paymentMethod`, `status`, `paidAt` | `processPayment()`, `verifyPayment()` |

### Activity Diagram Flow
```
[START]
   ↓
[LOGIN / REGISTER] → [VALIDATE CREDENTIALS]
   ↓ (Valid)
[DISPLAY AVAILABLE GROUNDS]
   ↓
[SELECT GROUND & TIME SLOT]
   ↓
[CHECK AVAILABILITY]
   ├── (Conflict / Unavailable) → [SHOW UNAVAILABLE MESSAGE] → [SELECT NEW SLOT]
   └── (Available)
         ↓
      [ENTER MATCH DETAILS]
         ↓
      [CALCULATE TOTAL AMOUNT (Server-Side)]
         ↓
      [SUBMIT PAYMENT (DEMO GATEWAY)]
         ↓
      [PROCESS PAYMENT]
         ├── (Failed) → [DISPLAY PAYMENT FAILED] → [RETRY]
         └── (Success)
               ↓
            [CONFIRM BOOKING (Atomically update status)]
               ↓
            [GENERATE FORMAL BOOKING VOUCHER]
               ↓
            [VIEW / PRINT CONFIRMATION] → [LOGOUT] → [END]
```

### Sequence Diagram Interaction
1. `User` → `Login Interface`: `enterLogin(email, password)`
2. `Login Interface` → `Booking System`: `validateLogin()`
3. `Booking System` → `Ground Database`: `verifyUser()`
4. `Ground Database` → `Booking System`: `loginValid`
5. `Booking System` → `Login Interface`: `loginSuccess(token)`
6. `User` → `Booking System`: `requestGroundAvailability(groundId, date, startTime, endTime)`
7. `Booking System` → `Ground Database`: Execute overlap query `(start < reqEnd AND end > reqStart)`
8. `Ground Database` → `Booking System`: `available`
9. `User` → `Booking System`: `makeBooking(details)`
10. `Booking System` → `Payment Interface`: `processPayment(bookingId)`
11. `Payment Interface` → `Booking System`: `paymentSuccess(transactionId)`
12. `Booking System` → `Ground Database`: `confirmBooking()`
13. `Booking System` → `User`: `bookingConfirmation()`

---

## 3. Technology Stack

- **Frontend**: React 19, TypeScript, Tailwind CSS, Lucide React icons.
- **Backend**: Node.js, Express, TypeScript (`tsx`).
- **Database**: PostgreSQL (Native PostgreSQL support via `pg.Pool` or embedded WASM PostgreSQL `@electric-sql/pglite` engine for instant zero-dependency execution).
- **Authentication**: Salted Bcrypt password hashing + JSON Web Tokens (JWT) with HTTP-only cookies and Bearer tokens.
- **Payments**: Payment Gateway Service Abstraction (`DEMO` mode default, with hooks for Razorpay / Stripe).

---

## 4. Double-Booking Overlap Algorithm (Server Enforced)

Two bookings for the same ground on the same date overlap when:
$$\text{requestedStart} < \text{existingEnd} \quad \text{AND} \quad \text{requestedEnd} > \text{existingStart}$$

Implemented in backend SQL as:
```sql
SELECT id FROM bookings
WHERE ground_id = $1
  AND booking_date = $2
  AND booking_status = 'CONFIRMED'
  AND start_time < $3
  AND end_time > $4;
```
If any rows are returned, the slot is immediately rejected and double-booking is prevented.

---

## 5. Seed Data & Credentials

### Default Administrator
- **Email**: `admin@cricketbook.demo`
- **Password**: `Admin@12345`
- **Role**: `ADMIN`

### Default Demo Users
- **User 1**: `rahul@cricketbook.demo` | Password: `User@12345` (Rahul Sharma)
- **User 2**: `priya@cricketbook.demo` | Password: `User@12345` (Priya Patel)

### Seeded Cricket Grounds
1. **Green Valley Cricket Ground** (Kadapa) - ₹1000/hr (Turf pitch, floodlights, pavilion)
2. **City Sports Cricket Arena** (Visakhapatnam) - ₹1500/hr (Red soil pitch, digital scoreboard)
3. **Champions Cricket Ground** (Vijayawada) - ₹1200/hr (Synthetic turf, shaded dugouts)
4. **Royal Cricket Arena** (Hyderabad) - ₹1800/hr (International dimension boundaries, VIP lounge)

---

## 6. Installation & Execution Commands

### Prerequisites
- Node.js >= 18
- npm

### 1. Install Dependencies
```bash
npm install
```

### 2. Run Test Suite
```bash
npm run test
```
Executes automated unit and integration tests verifying authentication, hashing, database connectivity, overlap detection, pricing arithmetic, and transaction states.

### 3. Start Development Server
```bash
npm run dev
```
The server will start at `http://localhost:3000` hosting both the Express REST API (`/api/*`) and Vite client application.

### 4. Build for Production
```bash
npm run build
```

### 5. Production Start
```bash
npm run start
```

---

## 7. REST API Reference

### Auth
- `POST /api/auth/register` - Create new User account (`name, phone, email, password, confirmPassword`)
- `POST /api/auth/login` - Authenticate User or Admin
- `POST /api/auth/logout` - Clear session token
- `GET /api/auth/me` - Fetch authenticated profile

### Grounds
- `GET /api/grounds` - Query grounds with `search`, `location`, `availability`, `sort`
- `GET /api/grounds/:id` - Fetch ground details
- `GET /api/grounds/:id/availability` - Query slot availability with `date, startTime, endTime`

### Bookings
- `POST /api/bookings` - Create reservation (creates `PENDING_PAYMENT` booking)
- `GET /api/bookings/my` - Fetch current user's bookings
- `GET /api/bookings/:id` - Fetch booking details (strict ownership check)
- `PUT /api/bookings/:id/cancel` - Cancel booking (initiates refund if paid)

### Payments
- `POST /api/payments/create` - Generate payment invoice
- `POST /api/payments/verify` - Process demo transaction (`simulateOutcome: 'SUCCESS' | 'FAILURE'`)
- `GET /api/payments/:bookingId` - Fetch payment receipt

### Admin
- `GET /api/admin/dashboard` - Retrieve aggregated metrics (`totalGrounds`, `revenue`, `bookings`)
- `GET /api/admin/grounds` - Administrative ground listing
- `POST /api/admin/grounds` - Add new cricket facility
- `PUT /api/admin/grounds/:id` - Update ground information or toggle availability
- `GET /api/admin/bookings` - Comprehensive booking ledger with search and filters
- `PUT /api/admin/bookings/:id` - Update booking status

---

## 8. Security Highlights

- **Ownership Isolation**: USER A cannot access USER B's booking by changing the URL (enforced on server).
- **Protected Backoffice**: Non-admin users attempting to access `/admin/*` routes or API endpoints receive HTTP 403 Forbidden.
- **Bcrypt Hash Protection**: Passwords are never stored or logged in plain text.
- **Parametrized Queries**: All SQL statements use parametrized placeholders (`$1, $2, ...`) eliminating SQL injection vulnerabilities.

---

## 9. Project Screenshots Placeholder
*(Include high-resolution screenshots of Home, Ground Listing, Availability Verification, Printable Booking Voucher, and Admin Analytics Dashboard in submission report).*
