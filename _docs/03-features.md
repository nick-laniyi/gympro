# Feature Documentation

## Overview

GymCRM provides a complete gym management solution across four user types: Super Admin, Gym Owner, Staff (Trainers/Receptionists), and Members. Features are tiered based on subscription plans.

---

## Subscription Tiers

| Feature | Free | Core | Portal | Enterprise |
|---------|------|------|--------|------------|
| Max Members | 20 | 100 | Unlimited | Unlimited |
| Staff Accounts | 1 | 3 | 10 | Unlimited |
| Member Portal | No | No | Yes | Yes |
| White-Label | No | No | Yes | Yes |
| Custom Domain | No | No | Yes | Yes |
| API Access | No | No | No | Yes |
| SMS Credits | 0 | 100/mo | 500/mo | 2000/mo |
| Support | Email | Email | Priority | Dedicated |

---

## Super Admin Features

**Gym Management**
- View all registered gyms
- Approve or reject new registrations
- Suspend or activate gym accounts
- Manually adjust subscription tiers
- View gym usage statistics

**System Administration**
- Configure system-wide settings
- Manage feature flags
- Review audit logs
- Handle enterprise demo requests
- Generate system reports

**Enterprise Sales**
- Capture demo request leads
- Contact information and requirements
- Track follow-up status
- Convert leads to paying customers

---

## Gym Owner Features

### Dashboard
- Daily check-in count
- Active members
- Expiring memberships (next 7 days)
- Recent payments
- Revenue chart (weekly/monthly)
- Quick actions (add member, check-in, payment)

### Member Management

**Member List**
- Sortable, filterable table
- Search by name, phone, email, member code
- Status indicators (active, expired, suspended)
- Bulk actions (export, email, SMS)
- Member count by plan type

**Member Profile**
- Personal information
- Membership details with expiry date
- Payment history
- Attendance history
- Workout assignments
- Staff notes
- Digital ID card generation

**Add/Edit Member**
- Form validation
- Duplicate phone/email checking
- Optional photo upload
- Member code auto-generation
- Custom membership plans

**Member Import/Export**
- CSV import for existing members
- CSV export for backup or external use
- PDF export for ID cards

### Payment Processing

**Record Payment**
- Member lookup by code or phone
- Amount and payment method
- Payment date (defaults to today)
- Membership period extension
- Receipt generation

**Paystack Integration**
- Online payment links
- Automatic verification via webhook
- Support for cards, bank transfer, USSD
- Transaction history with Paystack reference
- Refund handling

**Payment Reports**
- Daily collection summary
- Monthly revenue breakdown
- Payment method distribution
- Outstanding invoices
- Export to CSV/PDF

**Invoice Management**
- Auto-generation on payment
- Custom invoice numbering
- PDF download
- Email delivery option
- Paid/Unpaid status tracking

### White-Label Portal (Portal/Enterprise)

**Branding Settings**
- Logo upload (PNG, JPG, SVG)
- Favicon upload
- Primary color picker
- Secondary color picker
- Custom footer text
- Custom support email and phone

**Customization Options**
- Hide GymCRM branding
- Custom CSS editor with live preview
- Save and activate branding
- Reset to defaults
- Multiple brand presets

**Custom Domain**
- Domain verification
- SSL certificate guidance
- DNS configuration instructions
- Automatic domain detection
- Fallback to subdomain

**Member Portal Branding**
- Branded login page
- Customized email templates
- Branded invoice PDFs
- Consistent color scheme throughout

### Attendance Tracking

**Check-In/Out**
- Quick check-in by member code or phone
- Automatic timestamp
- Optional notes
- Check-out for current session
- Duration calculation

**Attendance Reports**
- Today's attendance list
- Historical attendance by member
- Peak hours analysis
- Average visit duration
- Daily/Monthly check-in counts

**QR Code Check-In (Planned)**
- Member-specific QR codes
- Scanner interface
- Instant check-in

### Staff Management

**Staff Accounts**
- Add trainers and receptionists
- Set user type and permissions
- Reset passwords
- Deactivate staff access

**Permission Controls**
- View-only access for receptionists
- Check-in/out permissions
- Member management restrictions
- Payment processing restrictions
- Report access controls

**Staff Attendance**
- Staff check-in/out tracking
- Duty hours monitoring
- Commission calculation basis (trainers)

### Class Management

**Class Scheduling**
- Create recurring classes
- Set capacity limits
- Assign trainers
- Schedule by day and time

**Member Bookings**
- Members book available slots
- View class attendance
- Waitlist when full
- Cancellation handling

### Equipment Management

**Inventory Tracking**
- Equipment list with quantities
- Condition monitoring
- Maintenance scheduling
- Service history

### Settings

**Gym Profile**
- Business name and contact details
- Address and location
- Business hours
- Social media links

**Membership Plans**
- Create custom plans
- Set price and duration
- Activate/deactivate plans
- Plan descriptions

**SMS Templates**
- Welcome message
- Renewal reminders
- Birthday greetings
- Payment confirmations
- Custom templates with placeholders

**API Access (Enterprise)**
- Generate API keys
- Set expiration dates
- Monitor usage
- Revoke access

---

## Staff Features (Trainers/Receptionists)

**Receptionist Access**
- Check-in/out members
- Look up member information
- Add new members
- Record manual payments
- View today's attendance

**Trainer Access**
- View assigned members
- Create workout plans
- Record workout completion
- View class schedule
- View member progress

**Limited Permissions**
- Cannot edit membership plans
- Cannot access financial reports
- Cannot manage staff accounts
- Cannot access white-label settings
- Cannot delete records

---

## Member Portal Features

**Self-Registration**
- Multi-step registration form
- Personal information collection
- Photo upload (optional)
- Membership plan selection
- Email verification
- Admin approval required (configurable)

**Member Dashboard**
- Membership status and expiry
- Check-in history
- Payment history
- Digital ID card
- Profile information

**Online Payments**
- View outstanding balance
- Select payment method
- Paystack checkout page
- Payment confirmation
- Email receipt

**Account Management**
- Update contact information
- Change password
- View workout assignments
- Cancel membership (request)

**PWA Features**
- Install on home screen (Android/iOS)
- Offline access to membership card
- Offline check-in (queued)
- Push notifications (planned)

---

## Communication Features

### Email Notifications

**Automated Emails**
- Welcome email on registration
- Payment receipt
- Membership expiry reminder (7 days before)
- Membership expiry notice
- Password reset
- Email verification

**Email Templates**
- Branded templates (white-label enabled)
- Placeholder variables
- HTML formatting
- Plain text fallback

### SMS Notifications

**Automated SMS**
- Welcome message
- Payment confirmation
- Renewal reminders
- Birthday greetings
- Class reminders

**Bulk SMS**
- Filter members by plan, expiry, activity
- Compose message
- Preview and send
- Delivery reports
- Cost tracking

### In-App Notifications

**Notification Center**
- Bell icon with unread count
- Notification list with categories
- Mark as read/unread
- Click to action links
- Read receipts

**Notification Triggers**
- Member signup (admin)
- Payment received
- Membership expiring
- Class booking
- System announcements

---

## Reporting & Analytics

### Financial Reports

**Revenue Report**
- Daily, weekly, monthly, yearly views
- Comparison with previous periods
- Payment method breakdown
- Export to CSV, PDF
- Chart visualization

**Payment Method Analysis**
- Cash vs online vs bank transfer
- Trend analysis
- Gateway fees calculation

**Expense Tracking**
- Record gym expenses
- Categorize expenses
- Profit calculation

### Member Reports

**Member Growth**
- New members over time
- Active vs expired trend
- Retention rate
- Member churn analysis

**Membership Distribution**
- Plan type breakdown
- Plan revenue contribution
- Popular plans

**Expiry Forecast**
- Members expiring this week
- Members expiring this month
- Renewal probability estimates

### Attendance Reports

**Attendance Summary**
- Daily check-in count
- Average daily attendance
- Peak hours
- Day-of-week analysis

**Member Attendance**
- Individual visit frequency
- Last visit date
- Inactive member identification

### Custom Reporting
- Date range selection
- Multiple filter options
- Column selection
- Save report configurations
- Scheduled email reports

---

## Offline Capabilities

**Offline Detection**
- Network status indicator
- Automatic detection of connection loss
- Visual warning banner
- Pending operations counter

**Offline Operations**
- View cached member list
- Check-in members (stored locally)
- View member details
- Access digital ID card
- View cached attendance history

**Sync Process**
- Automatic when connection restored
- Visual progress indicator
- Conflict resolution (server wins)
- Retry on failure
- Manual sync option

**Data Storage**
- Member data cached as JSON
- Configurable cache duration
- Cache invalidation on updates
- Storage quota management

---

## API Features (Enterprise)

**Authentication**
- API key authentication
- Key generation with expiration
- Key revocation
- Usage tracking

**Endpoints**

Members:
- GET /api/v1/members
- GET /api/v1/members/{id}
- POST /api/v1/members
- PUT /api/v1/members/{id}

Attendance:
- POST /api/v1/attendance/checkin
- POST /api/v1/attendance/checkout
- GET /api/v1/attendance/today

Payments:
- GET /api/v1/payments
- POST /api/v1/payments

**Rate Limiting**
- Per-gym limits
- Configurable requests per minute
- Rate limit headers
- 429 response on exceed

**Response Format**
- JSON only
- Consistent error structure
- Pagination for lists
- HTTP status codes

---

## White-Label Implementation Details

**Portal Features**
The member portal is a separate application within the main system. It provides self-service functionality to members while maintaining the gym's brand identity.

**Portal Sections:**
- Login/Registration
- Dashboard
- Membership Details
- Payment History
- Check-in History
- ID Card
- Profile Settings

**Branding Application:**
1. Gym owner configures colors and logo in admin panel
2. Settings saved to white_label_settings table
3. Portal page loads, checks for active white-label
4. CSS variables injected into page header
5. Logo path updated in navigation
6. Favicon replaced

**Email Template Branding:**
Member-facing emails use the gym's logo and colors when white-label is enabled.

---

## Security Features

**Authentication Security**
- Password hashing with bcrypt
- Session timeout (configurable)
- Login attempt limiting
- Password complexity requirements
- Remember me with secure tokens

**Data Security**
- Prepared statements for all queries
- Input validation on all forms
- Output encoding for XSS prevention
- CSRF tokens on all forms
- File upload validation

**Access Control**
- User type-based permissions
- Gym-level data isolation
- Owner access only to own data
- Staff permission checks

**Audit Trail**
- All member updates logged
- Payment records immutable
- User login/logout tracking
- Permission change logging

---

## Feature Availability Matrix

| Feature | Free | Core | Portal | Enterprise |
|---------|------|------|--------|------------|
| Member Management | ✓ | ✓ | ✓ | ✓ |
| Payment Recording | ✓ | ✓ | ✓ | ✓ |
| Paystack Integration | ✓ | ✓ | ✓ | ✓ |
| Basic Reports | ✓ | ✓ | ✓ | ✓ |
| Attendance Tracking | ✓ | ✓ | ✓ | ✓ |
| Staff Accounts (1) | 1 | 3 | 10 | Unlimited |
| Member Portal | ✗ | ✗ | ✓ | ✓ |
| White-Label Branding | ✗ | ✗ | ✓ | ✓ |
| Custom Domain | ✗ | ✗ | ✓ | ✓ |
| API Access | ✗ | ✗ | ✗ | ✓ |
| Bulk SMS | ✗ | 100/mo | 500/mo | 2000/mo |
| Custom CSS | ✗ | ✗ | ✗ | ✓ |
| Priority Support | ✗ | ✗ | ✓ | ✓ |

---

This feature set represents eight months of development based on direct feedback from 50+ gym owners. Each feature was built to solve a specific operational problem.
