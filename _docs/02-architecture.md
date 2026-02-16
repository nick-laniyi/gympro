# Database Schema Documentation

## Overview

The GymCRM database consists of 52 tables and 8 views. Multi-tenancy is implemented via `gym_id` foreign keys on all tenant-specific tables.

---

## Core Tables

### gyms
Root table for each tenant. All other tables reference this table.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | Auto-increment |
| name | varchar(255) | Gym name |
| slug | varchar(255) UNIQUE | URL-friendly name |
| address | text | Physical address |
| city | varchar(100) | City |
| state | varchar(100) | State |
| phone | varchar(20) | Contact number |
| email | varchar(255) | Contact email |
| logo_path | varchar(500) | Path to logo file |
| subscription_plan | enum | basic, pro, enterprise |
| subscription_end | date | When subscription expires |
| status | enum | active, suspended, cancelled |
| subscription_tier | enum | free, core, portal, enterprise |
| max_active_members | int | Limit based on tier |
| feature_flags | json | Enabled features |
| created_at | timestamp | |
| updated_at | timestamp | |

### users
System users including gym owners, trainers, and receptionists.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| email | varchar(255) UNIQUE | Login credential |
| username | varchar(100) UNIQUE | Optional username |
| password_hash | varchar(255) | Bcrypt hash |
| first_name | varchar(100) | |
| last_name | varchar(100) | |
| phone | varchar(20) | |
| user_type | enum | super_admin, gym_owner, trainer, receptionist |
| status | enum | active, suspended, inactive |
| permissions | text | JSON or comma-separated |
| last_login | datetime | |
| created_at | timestamp | |
| updated_at | timestamp | |

---

## Member Management

### members
Core member records. Each member belongs to one gym.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| member_code | varchar(50) UNIQUE | Gym-specific member ID |
| first_name | varchar(100) | |
| last_name | varchar(100) | |
| email | varchar(255) | |
| phone | varchar(20) | |
| date_of_birth | date | |
| gender | enum | male, female, other |
| address | text | |
| emergency_contact | varchar(20) | Phone number |
| medical_notes | text | Allergies, conditions |
| photo_path | varchar(500) | Profile photo |
| membership_type | enum | monthly, quarterly, yearly, day_pass, pay_per_visit |
| join_date | date | |
| membership_end | date | Expiration date |
| status | enum | active, expired, suspended, cancelled |
| password_hash | varchar(255) | For portal access |
| signup_status | enum | pending, approved, rejected |
| signup_source | enum | portal, admin, import |
| fitness_goals | text | |
| created_at | timestamp | |
| updated_at | timestamp | |

**Indexes:**
- `member_code` (UNIQUE)
- `gym_id`, `status`
- `email`
- `phone`
- `membership_end`

**Triggers:**
- `update_membership_status` - Automatically sets status to 'expired' when membership_end passes

### membership_plans
Custom plans created by each gym.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| name | varchar(255) | Plan name |
| description | text | |
| price | decimal(10,2) | |
| duration_days | int | Days of access |
| features | json | Included features |
| status | enum | active, inactive |
| created_at | timestamp | |
| updated_at | timestamp | |

### member_notes
Staff notes about members.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| member_id | int | References members.id |
| user_id | int | References users.id |
| note | text | |
| category | enum | general, medical, payment, behavior, progress |
| is_private | tinyint(1) | 0 = visible to all staff, 1 = owner only |
| created_at | timestamp | |

---

## Attendance & Classes

### attendance
Member check-in/out records.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| member_id | int | References members.id |
| gym_id | int | References gyms.id |
| check_in | datetime | |
| check_out | datetime | NULL if still checked in |
| duration_minutes | int | Calculated on check-out |
| check_in_method | enum | manual, biometric, qr_code, mobile_app |
| notes | text | |
| created_at | timestamp | |

**Indexes:**
- `member_id`, `check_in`
- `gym_id`, `check_in`

### gym_classes
Class definitions.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| class_name | varchar(255) | |
| description | text | |
| trainer_id | int | References users.id |
| max_capacity | int | |
| duration_minutes | int | |
| class_type | enum | yoga, zumba, spinning, boxing, etc |
| status | enum | active, cancelled, completed |
| created_at | timestamp | |
| updated_at | timestamp | |

### class_schedule
Recurring class times.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| class_id | int | References gym_classes.id |
| day_of_week | enum | monday through sunday |
| start_time | time | |
| end_time | time | |
| recurrence | enum | weekly, biweekly, monthly |
| status | enum | active, cancelled |
| created_at | timestamp | |

### class_bookings
Member class reservations.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| class_id | int | References gym_classes.id |
| member_id | int | References members.id |
| schedule_date | date | Date of class |
| status | enum | confirmed, cancelled, attended, absent |
| notes | text | |
| created_at | timestamp | |

**Unique Constraint:** `class_id`, `member_id`, `schedule_date`

---

## Payments & Billing

### payments
All payment transactions.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| member_id | int | References members.id |
| gym_id | int | References gyms.id |
| amount | decimal(10,2) | |
| payment_date | date | |
| payment_method | enum | cash, bank_transfer, pos, paystack, flutterwave, ussd |
| transaction_reference | varchar(255) | Gateway reference |
| period_start | date | Membership period start |
| period_end | date | Membership period end |
| recorded_by | int | References users.id |
| notes | text | |
| status | enum | completed, pending, failed |
| created_at | timestamp | |

**Indexes:**
- `member_id`, `payment_date`
- `gym_id`, `payment_date`
- `transaction_reference`

### invoices
Generated invoices for members.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| member_id | int | References members.id |
| invoice_number | varchar(50) UNIQUE | |
| description | text | |
| amount | decimal(10,2) | |
| due_date | date | |
| status | enum | pending, paid, overdue, cancelled |
| payment_method | varchar(50) | |
| transaction_reference | varchar(255) | |
| notes | text | |
| created_at | timestamp | |
| updated_at | timestamp | |

### gym_payment_config
Payment gateway configuration per gym.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| payment_gateway | enum | paystack, flutterwave, bank_transfer, cash, pos, ussd |
| config_data | longtext | Encrypted keys, settings |
| is_default | tinyint(1) | Default payment method |
| status | enum | active, inactive, pending_verification, suspended |
| verified_at | datetime | |
| created_at | timestamp | |
| updated_at | timestamp | |

**Unique Constraint:** `gym_id`, `payment_gateway`

### payment_sessions
Temporary storage for in-progress payments.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| session_id | varchar(128) | Unique session identifier |
| payment_data | json | Cart data, member info |
| user_type | enum | member, staff, guest |
| user_id | int | |
| gym_id | int | |
| created_at | timestamp | |
| expires_at | timestamp | |

---

## White Label & Branding

### white_label_settings
Per-gym branding configuration.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int UNIQUE | References gyms.id |
| brand_name | varchar(255) | Custom business name |
| primary_color | varchar(7) | Hex color |
| secondary_color | varchar(7) | Hex color |
| accent_color | varchar(7) | Hex color |
| custom_logo_path | varchar(500) | |
| favicon_path | varchar(500) | |
| custom_footer_text | text | |
| custom_support_email | varchar(255) | |
| custom_support_phone | varchar(20) | |
| custom_css | text | Advanced customization |
| hide_gympro_branding | tinyint(1) | Remove system branding |
| custom_domain | varchar(255) | Custom domain name |
| is_active | tinyint(1) | Whether white-label is enabled |
| created_at | timestamp | |
| updated_at | timestamp | |

---

## API & Integrations

### api_keys
Third-party API access.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| api_key | varchar(255) | Generated key |
| name | varchar(100) | Key description |
| status | enum | active, revoked, expired |
| expires_at | datetime | |
| revoked_at | datetime | |
| last_used | datetime | |
| created_at | timestamp | |

### external_business_integrations
Connections to external platforms.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| external_platform | enum | naijabased, website, other |
| external_business_id | int | ID from external platform |
| oauth_access_token | varchar(512) | |
| oauth_refresh_token | varchar(512) | |
| connection_status | enum | pending, connected, disconnected, suspended |
| last_lead_received | datetime | |
| total_leads_received | int | |
| created_at | timestamp | |
| updated_at | timestamp | |

**Unique Constraint:** `gym_id`, `external_platform`

---

## Notifications

### notifications
In-app notifications.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| member_id | int | Optional - related member |
| title | varchar(255) | |
| message | text | |
| type | enum | info, warning, urgent, success |
| category | varchar(50) | payment, membership, attendance, etc |
| target_user_id | int | References users.id |
| is_read | tinyint(1) | |
| read_at | datetime | |
| status | enum | active, deleted |
| created_at | timestamp | |

### sms_logs
Outbound SMS records.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| member_id | int | |
| phone_number | varchar(20) | |
| message | text | |
| status | enum | sent, failed, pending |
| cost | decimal(6,2) | |
| sent_at | datetime | |
| created_at | timestamp | |

### sms_templates
Reusable SMS templates.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| template_name | varchar(255) | |
| template_text | text | With placeholders |
| template_type | enum | welcome, renewal, birthday, promotion, reminder, custom |
| status | enum | active, inactive |
| created_at | timestamp | |
| updated_at | timestamp | |

---

## Equipment & Inventory

### equipment
Gym equipment tracking.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | References gyms.id |
| name | varchar(255) | |
| category | varchar(100) | |
| quantity | int | |
| condition | enum | excellent, good, fair, poor, broken |
| last_maintenance | date | |
| next_maintenance | date | |
| status | enum | active, inactive, under_maintenance |
| notes | text | |
| created_at | timestamp | |
| updated_at | timestamp | |

### maintenance_logs
Equipment maintenance history.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| equipment_id | int | References equipment.id |
| maintenance_type | varchar(100) | |
| description | text | |
| cost | decimal(10,2) | |
| performed_by | varchar(255) | |
| maintenance_date | date | |
| next_maintenance | date | |
| created_at | timestamp | |

---

## Workouts

### workouts
Member workout plans.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| member_id | int | References members.id |
| trainer_id | int | References users.id |
| title | varchar(255) | |
| description | text | |
| workout_type | enum | strength, cardio, hiit, crossfit, yoga, custom |
| difficulty | enum | beginner, intermediate, advanced |
| duration_days | int | |
| status | enum | active, completed, paused |
| start_date | date | |
| end_date | date | |
| created_at | timestamp | |
| updated_at | timestamp | |

### workout_exercises
Individual exercises within a workout.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| workout_id | int | References workouts.id |
| exercise_name | varchar(255) | |
| sets | int | |
| reps | varchar(50) | |
| weight | varchar(50) | |
| rest_time | varchar(50) | |
| notes | text | |
| day_of_week | enum | monday through sunday |
| exercise_order | int | Display order |
| created_at | timestamp | |

---

## Audit & Logs

### audit_logs
System audit trail.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | |
| user_id | int | |
| action | varchar(100) | Created, updated, deleted, etc |
| entity_type | varchar(50) | member, payment, user, etc |
| entity_id | int | |
| details | json | Before/after values |
| ip_address | varchar(45) | |
| user_agent | text | |
| created_at | timestamp | |

### api_audit_logs
API request logging.

| Column | Type | Description |
|--------|------|-------------|
| id | int PRIMARY KEY | |
| gym_id | int | |
| api_key_id | int | |
| endpoint | varchar(255) | |
| method | varchar(10) | |
| status_code | int | |
| response_time_ms | int | |
| user_agent | text | |
| ip_address | varchar(45) | |
| created_at | timestamp | |

---

## Database Views

### active_members_summary
Aggregated member counts per gym.

```sql
SELECT 
    gym_id,
    COUNT(*) as total_members,
    COUNT(CASE WHEN membership_type = 'monthly' THEN 1 END) as monthly_members,
    COUNT(CASE WHEN membership_type = 'quarterly' THEN 1 END) as quarterly_members,
    COUNT(CASE WHEN membership_type = 'yearly' THEN 1 END) as yearly_members,
    COUNT(CASE WHEN membership_end < CURDATE() THEN 1 END) as expired_members,
    COUNT(CASE WHEN membership_end BETWEEN CURDATE() AND CURDATE() + INTERVAL 7 DAY THEN 1 END) as expiring_soon
FROM members
WHERE status = 'active'
GROUP BY gym_id;
expiring_memberships
Members whose membership ends in the next 30 days.

sql
SELECT 
    m.*,
    DATEDIFF(m.membership_end, CURDATE()) as days_until_expiry
FROM members m
WHERE m.status = 'active'
    AND m.membership_end IS NOT NULL
    AND m.membership_end BETWEEN CURDATE() AND CURDATE() + INTERVAL 30 DAY
ORDER BY m.membership_end ASC;
monthly_revenue
Revenue grouped by month.

sql
SELECT 
    m.gym_id,
    YEAR(p.payment_date) as year,
    MONTH(p.payment_date) as month,
    COUNT(*) as total_payments,
    SUM(p.amount) as total_revenue,
    AVG(p.amount) as average_payment
FROM payments p
JOIN members m ON p.member_id = m.id
WHERE p.status = 'completed'
GROUP BY m.gym_id, YEAR(p.payment_date), MONTH(p.payment_date);
todays_attendance
Currently checked-in members with duration.

sql
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
pending_gym_registrations
Gym registrations awaiting approval.

sql
SELECT 
    id,
    registration_code,
    gym_name,
    gym_city,
    gym_state,
    owner_first_name,
    owner_last_name,
    owner_email,
    owner_phone,
    selected_plan,
    status,
    DATE(created_at) as registration_date,
    DATEDIFF(NOW(), created_at) as days_pending
FROM gym_registrations
WHERE status IN ('pending', 'verified')
ORDER BY created_at DESC;
gym_setup_status
Onboarding progress for each gym.

sql
SELECT 
    g.id as gym_id,
    g.name as gym_name,
    g.slug as gym_slug,
    g.status as gym_status,
    gs.completion_percentage,
    gs.current_step,
    gs.total_steps,
    CASE WHEN gs.profile_completed = 1 THEN '✓' ELSE '✗' END as profile_status,
    CASE WHEN gs.membership_plans_setup = 1 THEN '✓' ELSE '✗' END as plans_status,
    CASE WHEN gs.staff_added = 1 THEN '✓' ELSE '✗' END as staff_status,
    CASE WHEN gs.settings_configured = 1 THEN '✓' ELSE '✗' END as settings_status,
    gs.created_at as setup_started,
    gs.updated_at as last_updated
FROM gyms g
LEFT JOIN gym_setup_progress gs ON g.id = gs.gym_id
WHERE g.status = 'active'
ORDER BY gs.completion_percentage DESC;
Indexing Strategy
All foreign keys are indexed - This is non-negotiable for join performance.

Composite indexes are created for common query patterns:

sql
-- Member queries filtered by gym and status
CREATE INDEX idx_members_gym_status ON members(gym_id, status);

-- Attendance queries by date range
CREATE INDEX idx_attendance_checkin ON attendance(check_in);

-- Payment queries by gym and date
CREATE INDEX idx_payments_gym_date ON payments(gym_id, payment_date);
Unique constraints enforce data integrity:

member_code per gym

invoice_number globally

gym_id + payment_gateway per gym

Data Retention
Active data - Members, payments, attendance kept indefinitely

Audit logs - Retained for 2 years

Session data - Cleaned up after 30 days

Password reset tokens - Expire after 1 hour

Offline cache files - Deleted after successful sync

This schema has supported 50+ gyms and 15,000+ members with no performance degradation. The design prioritizes data integrity, query performance, and ease of development.