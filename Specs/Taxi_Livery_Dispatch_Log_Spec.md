# 🚖 Taxi / Livery Dispatch Log — Product Specification

**Version:** 1.0  
**Date:** February 19, 2026  
**Status:** Draft  

---

## 1. Executive Summary

This document specifies a **Taxi and Livery Dispatch Log System** — a comprehensive software solution for managing ride requests, driver assignments, fleet tracking, customer accounts, and fare collection for taxi, livery, and car service companies. The system is designed to serve as the operational backbone for dispatchers, fleet managers, drivers, and administrative staff.

---

## 2. Problem Statement

Small-to-mid-size taxi and livery companies often rely on paper logs, whiteboards, or fragmented spreadsheets to manage dispatch. This leads to:

- Missed or double-booked rides
- No centralized customer history
- Difficulty tracking driver availability in real time
- Poor visibility into revenue, utilization, and fleet health
- Inability to produce reports for compliance or business planning

---

## 3. Goals & Objectives

| # | Objective |
|---|-----------|
| G1 | Provide a single source of truth for all trip activity |
| G2 | Enable dispatchers to book, assign, and track rides in real time |
| G3 | Maintain a searchable database of customers and their ride history |
| G4 | Track driver status and vehicle availability at a glance |
| G5 | Automate fare calculation and payment logging |
| G6 | Generate daily, weekly, and monthly operational reports |
| G7 | Support compliance record-keeping (driver logs, vehicle inspections) |

---

## 4. Users & Roles

### 4.1 Dispatcher
The primary user. Creates trips, assigns drivers, monitors the live board, and handles customer calls. Needs speed and clarity above all else.

### 4.2 Driver
Views assigned trips, updates trip status (en route, picked up, completed), and logs fare/payment info. May use a simplified mobile view.

### 4.3 Fleet Manager
Manages vehicles, driver rosters, and maintenance schedules. Reviews utilization and performance reports.

### 4.4 Admin / Owner
Full access. Manages rate cards, user accounts, company settings, and financial reports.

---

## 5. Core Data Model

### 5.1 Trips (Central Record)

| Field | Type | Notes |
|-------|------|-------|
| Trip ID | Auto-ID | e.g. `TRP-00142` |
| Status | Select | `Pending` · `Dispatched` · `En Route` · `In Progress` · `Completed` · `Cancelled` · `No-Show` |
| Trip Type | Select | `One-Way` · `Round Trip` · `Hourly` · `Airport` · `Medical` · `Corporate` · `Charter` |
| Customer | Relation | → Customers table |
| Passenger Name | Text | If different from account holder |
| Passenger Count | Number | Default: 1 |
| Pickup Address | Text | Full street address |
| Pickup Date/Time (Requested) | DateTime | When the customer wants to be picked up |
| Pickup Date/Time (Actual) | DateTime | When driver actually arrived |
| Dropoff Address | Text | Full street address |
| Dropoff Date/Time | DateTime | When ride ended |
| Assigned Driver | Relation | → Drivers table |
| Assigned Vehicle | Relation | → Vehicles table |
| Estimated Distance (mi) | Number | Pre-trip estimate |
| Actual Distance (mi) | Number | Post-trip actual |
| Estimated Duration (min) | Number | Pre-trip estimate |
| Actual Duration (min) | Number | Post-trip actual |
| Base Fare | Currency | From rate card |
| Mileage Charge | Currency | Distance × rate |
| Wait Time Charge | Currency | If applicable |
| Tolls / Surcharges | Currency | Airport fees, after-hours, etc. |
| Total Fare | Currency | Calculated or manual override |
| Tip | Currency | |
| Payment Method | Select | `Cash` · `Credit Card` · `Account/Invoice` · `Voucher` · `App` |
| Payment Status | Select | `Unpaid` · `Paid` · `Invoiced` · `Partial` |
| Special Instructions | Text | e.g. "Ring doorbell, 2nd floor apt" |
| Internal Notes | Text | Dispatcher-only notes |
| Created By | User | Dispatcher who booked it |
| Created At | DateTime | Auto-timestamp |

### 5.2 Customers / Accounts

| Field | Type | Notes |
|-------|------|-------|
| Customer ID | Auto-ID | e.g. `CUS-0034` |
| Full Name | Text (Title) | |
| Phone (Primary) | Phone | Main contact number |
| Phone (Secondary) | Phone | Alternate |
| Email | Email | |
| Company / Account Name | Text | For corporate accounts |
| Account Type | Select | `Individual` · `Corporate` · `Medical` · `Government` |
| Default Pickup Address | Text | Pre-fill for repeat bookings |
| Default Dropoff Address | Text | Optional |
| VIP | Checkbox | Priority flag |
| Billing Terms | Select | `Pay Per Ride` · `Weekly Invoice` · `Monthly Invoice` |
| Notes / Preferences | Text | e.g. "Prefers SUV", "Wheelchair accessible" |
| Trips | Relation | → Trips table (rollup for count & total spend) |
| Created At | DateTime | |

### 5.3 Drivers

| Field | Type | Notes |
|-------|------|-------|
| Driver ID | Auto-ID | e.g. `DRV-012` |
| Full Name | Text (Title) | |
| Phone | Phone | |
| Email | Email | |
| License Number | Text | State/local hack license or TLC # |
| License Expiry | Date | |
| Status | Select | `Available` · `On Trip` · `Off Duty` · `Break` · `Inactive` |
| Assigned Vehicle | Relation | → Vehicles table |
| Hire Date | Date | |
| Emergency Contact | Text | Name + phone |
| Notes | Text | |
| Trips | Relation | → Trips table (rollup for count & revenue) |

### 5.4 Vehicles

| Field | Type | Notes |
|-------|------|-------|
| Unit Number | Text (Title) | Internal fleet ID, e.g. `V-07` |
| Make | Text | e.g. Toyota |
| Model | Text | e.g. Camry |
| Year | Number | e.g. 2023 |
| Color | Text | |
| License Plate | Text | |
| VIN | Text | |
| Vehicle Type | Select | `Sedan` · `SUV` · `Van` · `Luxury` · `Wheelchair Accessible` · `Minibus` |
| Passenger Capacity | Number | Max riders |
| Status | Select | `Active` · `Maintenance` · `Out of Service` · `Retired` |
| Insurance Expiry | Date | |
| Inspection Expiry | Date | |
| Odometer (Last Recorded) | Number | Miles |
| Notes | Text | |
| Assigned Driver | Relation | → Drivers table |
| Trips | Relation | → Trips table |

### 5.5 Dispatch Log (Activity Journal)

| Field | Type | Notes |
|-------|------|-------|
| Log ID | Auto-ID | |
| Timestamp | DateTime | Auto |
| Dispatcher | User | Who performed the action |
| Action | Select | `Booked` · `Assigned` · `Reassigned` · `Status Change` · `Cancelled` · `Note` · `Fare Adjusted` |
| Related Trip | Relation | → Trips table |
| Related Driver | Relation | → Drivers table |
| Details | Text | Free-text description of what happened |

---

## 6. Workflow & Trip Lifecycle

```
┌─────────────────────────────────────────────────────────┐
│                    TRIP LIFECYCLE                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   CUSTOMER CALLS / BOOKS ONLINE                         │
│          │                                              │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │   PENDING    │  Trip created, awaiting assignment    │
│   └──────┬──────┘                                       │
│          │  Dispatcher assigns driver + vehicle          │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │  DISPATCHED  │  Driver notified of assignment       │
│   └──────┬──────┘                                       │
│          │  Driver departs for pickup                    │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │  EN ROUTE    │  Driver heading to pickup location   │
│   └──────┬──────┘                                       │
│          │  Passenger picked up                          │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │ IN PROGRESS  │  Ride underway                       │
│   └──────┬──────┘                                       │
│          │  Passenger dropped off                        │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │  COMPLETED   │  Fare logged, payment recorded       │
│   └─────────────┘                                       │
│                                                         │
│   ALTERNATE ENDINGS:                                    │
│   • CANCELLED — Trip cancelled before pickup            │
│   • NO-SHOW  — Driver arrived, passenger absent         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 7. Key Views & Screens

### 7.1 Live Dispatch Board
The dispatcher's primary workspace. A real-time table/board of all active and upcoming trips for the current day, sorted by pickup time. Color-coded by status. Allows quick assignment of drivers and status updates.

**Columns:** Time · Status · Customer · Pickup → Dropoff · Driver · Vehicle · Actions

**Filters:** Today / Tomorrow / Date Range · Status · Driver · Trip Type

### 7.2 Trip History & Search
A searchable, filterable archive of all trips. Supports full-text search across customer name, address, notes. Exportable to CSV for accounting.

### 7.3 Customer Lookup
Search by name or phone number. View full ride history, total spend, preferences. One-click "Book Again" to create a new trip pre-filled with the customer's default addresses.

### 7.4 Driver Dashboard
Per-driver view showing today's assignments, trip count, revenue generated, and current status. Weekly/monthly rollup available.

### 7.5 Fleet Overview
Grid of all vehicles with status indicators. Highlights upcoming insurance/inspection expirations. Links to assigned driver and recent trip history.

### 7.6 Daily Summary Report
Auto-generated end-of-day report:

- Total trips completed
- Total revenue collected
- Breakdown by payment method
- Cancellations and no-shows
- Driver leaderboard (trips completed, revenue)
- Average fare, average trip duration

---

## 8. Rate Card / Fare Calculation

The system should support a configurable rate card:

| Parameter | Example Value |
|-----------|--------------|
| Base fare | $3.50 |
| Per mile rate | $2.75 / mi |
| Per minute rate (wait time) | $0.50 / min |
| Minimum fare | $8.00 |
| Airport surcharge | $5.00 |
| After-hours surcharge (10pm–6am) | +20% |
| Holiday surcharge | +25% |
| Wheelchair accessible vehicle | No surcharge |
| Flat rate zones | Configurable (e.g. Downtown ↔ Airport = $45) |

**Fare formula:**  
`Total = max(Minimum, Base + (Miles × Per-Mile) + (Wait Min × Per-Min)) + Surcharges`

Dispatchers can override the calculated fare with a manual amount when needed.

---

## 9. Reporting & Analytics

### 9.1 Operational Reports

| Report | Frequency | Description |
|--------|-----------|-------------|
| Daily Dispatch Summary | Daily | Trips, revenue, cancellations, driver stats |
| Weekly Performance | Weekly | Trends, comparisons to prior week |
| Monthly Revenue Report | Monthly | Revenue by type, payment method, account |
| Driver Utilization | Weekly | % of shift time on active trips per driver |
| Vehicle Mileage Log | Monthly | Odometer tracking, maintenance triggers |

### 9.2 Key Metrics (KPIs)

- **Trips per day** — volume trend
- **Revenue per trip** — average fare health
- **Cancellation rate** — % of booked trips cancelled
- **No-show rate** — % where passenger didn't appear
- **Average response time** — booking to driver arrival
- **Driver utilization** — active trip time vs. idle time
- **Top 10 customers** — by trip count and by revenue
- **Revenue by trip type** — which segments are growing

---

## 10. Compliance & Record-Keeping

Many jurisdictions require taxi/livery operators to maintain trip logs. This system supports:

- Complete trip records with timestamps, addresses, driver, and vehicle
- Driver license and expiration tracking with alerts
- Vehicle insurance and inspection expiry alerts (30-day, 7-day warnings)
- Exportable logs in CSV/PDF format for regulatory submission
- Immutable dispatch log (audit trail of all actions)

---

## 11. Phase 2 — Future Enhancements

| Feature | Description |
|---------|-------------|
| **Recurring / Standing Orders** | Auto-generate trips for repeat bookings (e.g. "Every Mon & Wed, 8am, Mr. Johnson → Office") |
| **Customer SMS Notifications** | "Your driver is 5 minutes away" / "Your ride has arrived" |
| **Driver Mobile App** | Accept/decline trips, update status, navigate, log fare from phone |
| **Online Booking Portal** | Web form for customers to self-book rides |
| **Google Maps Integration** | Auto-calculate distance, duration, and route for fare estimation |
| **Invoice Generation** | Auto-generate PDF invoices for corporate/account customers |
| **Shift Management** | Clock-in/out, shift scheduling, break tracking |
| **Two-Way Radio / VoIP Log** | Link dispatch audio to trip records |
| **Multi-Location Support** | Manage multiple bases/garages from one system |
| **Dynamic Pricing** | Surge pricing based on demand/time/availability |

---

## 12. Technical Requirements

### 12.1 Non-Functional Requirements

- **Response time:** Dispatch board updates within 2 seconds
- **Availability:** 99.5% uptime (system is operationally critical)
- **Concurrent users:** Support 10+ simultaneous dispatchers
- **Data retention:** Minimum 7 years of trip records (regulatory)
- **Backup:** Daily automated backups with point-in-time recovery
- **Access control:** Role-based permissions (Dispatcher, Driver, Manager, Admin)

### 12.2 Platform Options

| Approach | Pros | Cons | Best For |
|----------|------|------|----------|
| **Web Application (React + API)** | Real-time updates, multi-user, mobile-friendly | Development effort, hosting costs | Mid-to-large fleets, growth-oriented |
| **Notion Workspace** | Fast setup, no-code, collaborative | Limited real-time, no native fare calc | Small fleets (<15 vehicles), low budget |
| **Excel / Google Sheets** | Familiar, offline-capable, zero cost | No real-time, manual processes, fragile | Solo dispatchers, minimal operations |
| **Desktop App (Electron / WPF)** | Fast, works offline, native feel | Single-machine, harder to share | Single-base, single-dispatcher setups |

### 12.3 Integrations (Optional)

- Google Maps / Mapbox — address autocomplete, distance/time estimates
- Stripe / Square — credit card processing
- Twilio — SMS notifications to customers
- QuickBooks / Xero — accounting sync
- Google Calendar — driver shift scheduling

---

## 13. Glossary

| Term | Definition |
|------|-----------|
| **Dispatch** | The act of assigning a driver and vehicle to a trip |
| **Livery** | A pre-arranged car service (as opposed to street-hail taxi) |
| **Hack License** | A taxi/livery driver's permit issued by local authority |
| **TLC** | Taxi & Limousine Commission (e.g. NYC TLC) |
| **Standing Order** | A recurring, pre-scheduled trip |
| **No-Show** | Customer was not present at pickup location |
| **Dead-head** | A trip where the vehicle drives empty (no passenger) |
| **Base** | The physical location / office from which a livery company operates |
| **Rate Card** | The published schedule of fares and surcharges |
| **Flat Rate** | A fixed fare for a specific origin-destination pair |

---

## 14. Open Questions

- [ ] What jurisdiction-specific regulations apply (TLC, PUC, etc.)?
- [ ] Do drivers own their vehicles or are they company-owned?
- [ ] Is credit card processing handled in-vehicle or at dispatch?
- [ ] What is the current fleet size and expected growth?
- [ ] Are there existing systems (accounting, scheduling) to integrate with?
- [ ] Is GPS/AVL (Automatic Vehicle Location) tracking desired in Phase 1?

---

*This specification is a living document. Feedback and revisions are welcome as requirements are refined.*
