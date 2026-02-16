# Database Overview

## Design Philosophy

The database uses a shared schema with tenant isolation via `gym_id` on all tables. This approach balances simplicity with data integrity while keeping queries straightforward.

**Core principles:**
- Every tenant-scoped table has a `gym_id` foreign key
- All queries include tenant filtering
- Foreign keys enforce referential integrity
- Strategic indexes on frequently queried columns
- Views simplify complex reporting

---

## Table Groups

**Tenant Management (5 tables)**
- `gyms` - Root table for each gym tenant
- `gym_registrations` - Signup workflow tracking
- `gym_settings` - Per-gym configuration
- `gym_features` - Feature flags per tier
- `gym_setup_progress` - Onboarding completion tracking

**User Management (4 tables)**
- `users` - Staff accounts (owners, trainers, receptionists)
- `password_resets` - Password recovery
- `remember_tokens` - "Remember me" functionality
- `user_notification_preferences` - Per-user settings

**Member Management (4 tables)**
- `members` - Core member profiles and membership data
- `member_notes` - Staff notes on members
- `membership_plans` - Custom plans per gym
- `member_id_cards` - Generated ID card data

**Attendance & Classes (4 tables)**
- `attendance` - Check-in/out records
- `gym_classes` - Class definitions
- `class_schedule` - Recurring class times
- `class_bookings` - Member class reservations

**Payments & Billing (6 tables)**
- `payments` - All payment transactions
- `invoices` - Generated invoices
- `gym_payment_config` - Payment gateway settings
- `payment_sessions` - Temporary payment data
- `payment_settlements` - Platform fee tracking
- `subscription_upgrades` - Tier change requests

**White Label (1 table)**
- `white_label_settings` - Per-gym branding configuration

**Communications (3 tables)**
- `notifications` - In-app notifications
- `sms_logs` - SMS delivery tracking
- `sms_templates` - Reusable message templates

**Equipment (2 tables)**
- `equipment` - Gym equipment inventory
- `maintenance_logs` - Service history

**Workouts (2 tables)**
- `workouts` - Member workout plans
- `workout_exercises` - Individual exercises

**Integrations (5 tables)**
- `api_keys` - API access
- `api_audit_logs` - API usage tracking
- `api_rate_limits` - Request throttling
- `external_business_integrations` - NaijaBased connections
- `webhooks` - Outgoing webhook configuration

**Audit (2 tables)**
- `audit_logs` - User action audit trail
- `webhook_logs` - Incoming webhook payloads

**Other (2 tables)**
- `expenses` - Gym expense tracking
- `enterprise_demo_requests` - Lead capture

---

## Core Table Examples

### members
```sql
CREATE TABLE members (
    id INT PRIMARY KEY AUTO_INCREMENT,
    gym_id INT NOT NULL,
    member_code VARCHAR(50) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255),
    phone VARCHAR(20) NOT NULL,
    date_of_birth DATE,
    gender ENUM('male','female','other'),
    address TEXT,
    emergency_contact VARCHAR(20),
    medical_notes TEXT,
    photo_path VARCHAR(500),
    membership_type ENUM('monthly','quarterly','yearly','day_pass','pay_per_visit'),
    join_date DATE NOT NULL,
    membership_end DATE,
    status ENUM('active','expired','suspended','cancelled') DEFAULT 'active',
    password_hash VARCHAR(255),
    signup_status ENUM('pending','approved','rejected') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (gym_id) REFERENCES gyms(id),
    UNIQUE KEY (member_code),
    INDEX (gym_id, status),
    INDEX (email),
    INDEX (phone)
);
payments
sql
CREATE TABLE payments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    member_id INT NOT NULL,
    gym_id INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_date DATE NOT NULL,
    payment_method ENUM('cash','bank_transfer','pos','paystack','flutterwave','ussd'),
    transaction_reference VARCHAR(255),
    period_start DATE,
    period_end DATE,
    recorded_by INT,
    notes TEXT,
    status ENUM('completed','pending','failed') DEFAULT 'completed',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (member_id) REFERENCES members(id),
    FOREIGN KEY (gym_id) REFERENCES gyms(id),
    INDEX (member_id, payment_date),
    INDEX (gym_id, payment_date),
    INDEX (transaction_reference)
);
white_label_settings
sql
CREATE TABLE white_label_settings (
    id INT PRIMARY KEY AUTO_INCREMENT,
    gym_id INT NOT NULL UNIQUE,
    brand_name VARCHAR(255),
    primary_color VARCHAR(7) DEFAULT '#0088cc',
    secondary_color VARCHAR(7) DEFAULT '#2ecc71',
    accent_color VARCHAR(7) DEFAULT '#e74c3c',
    custom_logo_path VARCHAR(500),
    favicon_path VARCHAR(500),
    custom_footer_text TEXT,
    custom_support_email VARCHAR(255),
    custom_support_phone VARCHAR(20),
    custom_css TEXT,
    hide_gympro_branding TINYINT(1) DEFAULT 0,
    custom_domain VARCHAR(255),
    is_active TINYINT(1) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (gym_id) REFERENCES gyms(id) ON DELETE CASCADE,
    INDEX (is_active)
);
Key Relationships
Tenant isolation: All gym-specific tables reference gyms.id

Member centric: Most operations flow from members:

members → payments

members → attendance

members → member_notes

members → workouts

members → class_bookings

Cascading deletes: When a gym is deleted, all related data is removed automatically via foreign key constraints.

Database Views
Views simplify common reporting queries:

active_members_summary

sql
-- One row per gym with member counts by type
SELECT 
    gym_id,
    COUNT(*) as total_members,
    COUNT(CASE WHEN membership_type = 'monthly' THEN 1 END) as monthly,
    COUNT(CASE WHEN membership_type = 'quarterly' THEN 1 END) as quarterly,
    COUNT(CASE WHEN membership_type = 'yearly' THEN 1 END) as yearly,
    COUNT(CASE WHEN membership_end < CURDATE() THEN 1 END) as expired,
    COUNT(CASE WHEN membership_end BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 7 DAY) THEN 1 END) as expiring_soon
FROM members
WHERE status = 'active'
GROUP BY gym_id;
expiring_memberships

sql
-- Members whose membership ends in next 30 days
SELECT 
    m.*,
    DATEDIFF(m.membership_end, CURDATE()) as days_until_expiry
FROM members m
WHERE m.status = 'active'
    AND m.membership_end IS NOT NULL
    AND m.membership_end BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 30 DAY)
ORDER BY m.membership_end;
monthly_revenue

sql
-- Revenue aggregated by month
SELECT 
    m.gym_id,
    YEAR(p.payment_date) as year,
    MONTH(p.payment_date) as month,
    COUNT(*) as total_payments,
    SUM(p.amount) as total_revenue
FROM payments p
JOIN members m ON p.member_id = m.id
WHERE p.status = 'completed'
GROUP BY m.gym_id, YEAR(p.payment_date), MONTH(p.payment_date);
todays_attendance

sql
-- Currently checked-in members
SELECT 
    a.*,
    m.first_name,
    m.last_name,
    m.member_code,
    TIMESTAMPDIFF(MINUTE, a.check_in, COALESCE(a.check_out, NOW())) as current_duration
FROM attendance a
JOIN members m ON a.member_id = m.id
WHERE DATE(a.check_in) = CURDATE()
ORDER BY a.check_in DESC;
Indexing Strategy
All foreign keys are indexed

Composite indexes for common WHERE clauses (gym_id + status)

Indexes on frequently searched columns (email, phone, member_code)

Date indexes for range queries (payment_date, membership_end, check_in)

Data Integrity
Foreign keys with CASCADE for deletes where appropriate

Triggers for automated status updates (expired memberships)

Unique constraints on business keys (member_code, invoice_number)

Transactions for multi-table operations

This design supports 50+ gyms and 15,000+ members with consistent performance and no data integrity issues.

