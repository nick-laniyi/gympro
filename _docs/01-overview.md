# System Architecture

## Overview

GymCRM uses a shared database, shared codebase multi-tenant architecture with tenant isolation at the query level. This approach was chosen for its simplicity, low overhead, and ease of deployment on standard shared hosting.

---

## Architecture Diagram
┌─────────────────────────────────────────────────────────────┐
│ CLIENT LAYER │
├─────────────┬───────────────┬───────────────┬───────────────┤
│ Web App │ PWA App │ Admin Panel │ API Clients │
│ (PHP/JS) │ (Service │ (Bootstrap) │ (REST) │
│ │ Worker) │ │ │
└─────────────┴───────────────┴───────────────┴───────────────┘
│
┌─────────────────────────────────────────────────────────────┐
│ APPLICATION LAYER │
├─────────────┬───────────────┬───────────────┬───────────────┤
│ Auth & │ Payment │ White-Label │ Offline │
│ Roles │ Processor │ Engine │ Sync │
├─────────────┼───────────────┼───────────────┼───────────────┤
│ Member │ Attendance │ Class/ │ Reporting │
│ Management │ Tracking │ Workout │ Engine │
└─────────────┴───────────────┴───────────────┴───────────────┘
│
┌─────────────────────────────────────────────────────────────┐
│ DATA LAYER │
├─────────────────────────────────────────────────────────────┤
│ MySQL 8.0 Database │
│ 52 Tables | 8 Views | Multi-tenant via gym_id │
└─────────────────────────────────────────────────────────────┘
│
┌─────────────────────────────────────────────────────────────┐
│ INTEGRATION LAYER │
├─────────────┬───────────────┬───────────────┬───────────────┤
│ Paystack │ NaijaBased │ PHPMailer │ SMS Gateway │
│ Payments │ Directory │ Email │ Text │
└─────────────┴───────────────┴───────────────┴───────────────┘

text

---

## Multi-Tenant Implementation

**Strategy:** Shared database, shared tables, row-level isolation

Every table that stores tenant-specific data contains a `gym_id` column:

```sql
CREATE TABLE members (
    id INT PRIMARY KEY,
    gym_id INT NOT NULL,
    member_code VARCHAR(50),
    -- other columns
    FOREIGN KEY (gym_id) REFERENCES gyms(id)
);
All queries include the tenant filter:

php
$query = "SELECT * FROM members 
          WHERE gym_id = ? 
          AND status = 'active'";
$stmt = $db->prepare($query);
$stmt->execute([$current_gym_id]);
Why this approach:

Single database, simple backups

Cross-tenant reporting possible

No complex schema changes

Works on any hosting environment

Easy to understand and maintain

Trade-offs:

No per-tenant database isolation

Must ensure gym_id is always included

Indexes must account for gym_id

White-Label Engine
Gym owners need their member portals to reflect their brand. The white-label system stores branding preferences per gym and injects them at runtime.

Database Schema:

sql
CREATE TABLE white_label_settings (
    id INT PRIMARY KEY,
    gym_id INT UNIQUE,
    brand_name VARCHAR(255),
    primary_color VARCHAR(7) DEFAULT '#0088cc',
    secondary_color VARCHAR(7) DEFAULT '#2ecc71',
    custom_logo_path VARCHAR(500),
    favicon_path VARCHAR(500),
    custom_css TEXT,
    hide_branding TINYINT(1) DEFAULT 0,
    is_active TINYINT(1) DEFAULT 0,
    FOREIGN KEY (gym_id) REFERENCES gyms(id)
);
Runtime CSS Injection:

php
// Simplified implementation
$settings = $db->query(
    "SELECT * FROM white_label_settings 
     WHERE gym_id = ? AND is_active = 1",
    [$gym_id]
)->fetch();

if ($settings) {
    echo "<style>
        :root {
            --primary: {$settings['primary_color']};
            --secondary: {$settings['secondary_color']};
        }
        .brand-logo {
            content: url('{$settings['custom_logo_path']}');
        }
        .brand-footer {
            display: " . ($settings['hide_branding'] ? 'none' : 'block') . ";
        }
    </style>";
    
    if ($settings['custom_css']) {
        echo "<style>{$settings['custom_css']}</style>";
    }
}
Benefits:

No file system writes

Instant activation across all pages

Per-gym caching possible

Simple backup and restore

Custom Domain Support:
Gyms can point their own domain to the platform. The system detects the domain via the Host header and maps it to the correct gym:

php
$domain = $_SERVER['HTTP_HOST'];
$gym = $db->query(
    "SELECT g.* FROM gyms g
     JOIN white_label_settings w ON g.id = w.gym_id
     WHERE w.custom_domain = ? AND w.is_active = 1",
    [$domain]
)->fetch();
Offline-First Architecture
Internet connectivity in Nigerian gyms is not guaranteed. The system must function during outages.

Client-Side Strategy:

Service Worker caches static assets (CSS, JS, images) on first visit

Application Shell loads from cache when offline

Offline Queue stores actions (check-ins, registrations) as JSON files

Background Sync processes the queue when connection returns

Offline Data Storage:

json
// cache/offline_checkins_1.json
[
    {
        "id": "offline_001",
        "member_id": 1234,
        "timestamp": "2026-02-12 10:23:45",
        "action": "check_in",
        "synced": false
    }
]
Sync Process:

System detects online status

Iterates through unsynced JSON files

Sends each action to server

Marks as synced or retries on failure

Visual Indicator:
A persistent banner shows offline status and pending sync count.

Limitations:

Read-only operations work fully offline

Write operations queued, processed when online

No real-time collaboration during outages

Payment Processing Pipeline
Payments flow through a state machine with verification at each step.

State Flow:

text
Init → Pending → Verify Webhook → Complete → Update Membership
        ↓
     Failed
Webhook Verification:

php
// Verify Paystack signature
$signature = $_SERVER['HTTP_X_PAYSTACK_SIGNATURE'];
$payload = file_get_contents('php://input');
$computed = hash_hmac('sha512', $payload, PAYSTACK_SECRET_KEY);

if ($signature !== $computed) {
    http_response_code(401);
    exit;
}
Idempotency:
Each webhook includes a reference that is checked before processing to prevent duplicates.

Membership Update:
Upon successful payment, the member's membership_end date is extended based on the plan purchased.

Database Design
Tenant Isolation:
Every tenant table has a gym_id foreign key. Composite indexes include gym_id first.

Views for Reporting:
Complex joins are encapsulated in database views:

active_members_summary - Member counts by type

expiring_memberships - Members expiring in next 30 days

monthly_revenue - Revenue grouped by month

todays_attendance - Currently checked-in members

Triggers:
A trigger automatically updates member status to 'expired' when membership_end passes.

sql
CREATE TRIGGER update_membership_status
BEFORE UPDATE ON members
FOR EACH ROW
BEGIN
    IF NEW.membership_end < CURDATE() 
       AND NEW.status = 'active' THEN
        SET NEW.status = 'expired';
    END IF;
END;
Indexing Strategy:

All foreign keys are indexed

Composite indexes for common WHERE clauses

Indexes on status columns for filtered queries

Security Implementation
Authentication:

Passwords hashed with password_hash() and PASSWORD_DEFAULT

Session regeneration after login

HttpOnly cookies with secure flags

Remember me tokens stored hashed in database

Authorization:

User types: super_admin, gym_owner, trainer, receptionist

Permission checks on every protected page

Gym owners cannot access other gyms' data

Input Validation:

All database queries use prepared statements

No dynamic table/column names in queries

Input sanitized based on context

API Security:

API keys required for third-party access

Rate limiting by gym

Request logging for audit

Payment Security:

Paystack handles card data (PCI compliance)

Webhooks verified with secret key

SSL enforced for all transactions

Framework Decision
The system uses vanilla PHP without Laravel, Symfony, or other frameworks.

Why:

Full control over every line of code

No framework-specific debugging

Deploys to any PHP hosting without special requirements

Minimal dependencies to maintain

Faster execution without framework bootstrapping

Easier for other developers to understand (no framework learning curve)

Trade-offs:

More boilerplate code

Manual routing and dependency injection

Custom ORM instead of Eloquent/Doctrine

Result: 50+ gyms running on standard shared hosting plans. No framework upgrade headaches.

Deployment
Current Infrastructure:

Shared hosting with cPanel

PHP 8.3

MySQL 8.0

Apache with mod_rewrite

Deployment Process:

Code pushed to private repository

Pulled to staging server for testing

Tested with sample data

Pulled to production

Migrations run manually

Scaling Considerations:

Vertical scaling currently sufficient

Read replicas planned for reporting queries

Redis for session storage if needed

Separate API server for mobile apps

Monitoring
Application Monitoring:

Error logging to files

Failed payment webhooks logged

Offline sync failures logged

Slow query identification

Business Monitoring:

Daily active gyms

New member registrations

Revenue by payment method

Expiring memberships

Alerting:

Payment gateway errors trigger email

Mass membership expirations reviewed

Offline queue backlog monitored

Future Architecture
Planned Improvements:

Separate API server from web application

Mobile native apps (Flutter)

Real-time attendance with WebSockets

Automated database backups

Containerization for easier deployment

This architecture has processed over 50 million naira in payments and handled 500+ daily offline check-ins without data loss. It prioritizes reliability and maintainability over trends.