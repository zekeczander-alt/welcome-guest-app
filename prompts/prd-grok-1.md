# Product Requirement Document (PRD): Airbnb Management App for Small-Scale Owners

## 1. Document Information
- **Product Name**: Airbnb Host Manager (tentative; can be customized)
- **Version**: 1.4 (Updated Iteration with Automated WhatsApp Integration and Enhanced Guest Pages)
- **Date**: September 23, 2025
- **Author**: Grok 4 (based on user requirements and clarifications)
- **Stakeholders**: Small-scale Airbnb hosts/owners, guests (via unique URLs and WhatsApp groups), staff (defined via contact details and added to groups)
- **Purpose**: This PRD outlines the requirements for an app that enables Airbnb hosts to manage properties, reservations, and guest interactions. The simplified version supports multi-tenancy for scalability, starting with one host/business but designed for extension to multiple hosts. Access to the app is limited to the host, with guests receiving time-limited unique URLs for viewing configurable stay details, including WhatsApp group invites. Hosts can automatically create, manage, and archive WhatsApp groups, and send URLs/receipts via integrated email and WhatsApp messaging.

## 2. Introduction
### 2.1 Product Overview
The app is a web-based platform built with Next.js, allowing small-scale Airbnb owners to create and manage property listings, define staff contact details, handle reservations, configure guest-facing pages, automatically manage WhatsApp groups for reservations, and provide guests with access to stay information via unique URLs. It focuses on ease of use for hosts while ensuring secure, time-limited access for guests. Unauthenticated users see static public information. The app serves as a recordkeeping tool, with additional features like cancellations to be added later.

Key differentiators:
- Seamless integration with Google for host authentication.
- Multi-tenant architecture from the start for future scalability.
- Mobile-first design with desktop responsiveness.
- Generation of unique, time-limited URLs for guest access, with configurable content and automated WhatsApp group management.
- Integrated email and WhatsApp messaging for sending URLs and receipts.
- Automated WhatsApp group creation, member addition, archiving, and deletion/deactivation.

### 2.2 Objectives
- Enable hosts to set up and customize property pages efficiently.
- Streamline reservation management, guest information sharing, and communication setup via automated WhatsApp groups.
- Provide convenient, time-limited access for guests via unique URLs, showing configurable content including stay summaries, receipts, and WhatsApp invites.
- Ensure a static, informative experience for public visitors.
- Support multi-tenancy for easy expansion to multiple properties/hosts.
- Allow hosts to define staff and guest WhatsApp details, automatically create groups, archive chats, and deactivate groups post-stay.

### 2.3 Assumptions
- Reservations are manually created by hosts only (no self-booking).
- No billing PDFs or downloads; bills and receipts displayed as dynamic values on guest URLs; final receipts sent via email as text or simple attachment.
- WhatsApp integration uses WhatsApp Business API for automated group creation, member addition (via phone numbers), messaging, archiving (export chat contents), and deletion/deactivation.
- Guest and staff details, including first/last name, email, and WhatsApp numbers, entered by host; users responsible for valid contact details (e.g., WhatsApp numbers must be registered and consent to group addition).
- Guest URLs expire post-stay, limiting access.
- No offline access, cancellations, or advanced edge case handling in v1.
- Compliance with basic privacy (e.g., consent for data storage and group participation), but detailed data handling ignored for now.
- Group names follow format: [Room Number]-[Lastname1]-[Lastname2-...]-[DD-MM-Duration(in days)] (e.g., "101-Smith-Johnson-15-10-5" for room 101, guests Smith and Johnson, start 15-10, 5 days); auto-generated based on reservation data.
- Host's WhatsApp number stored in profile for group addition.

## 3. Scope
### 3.1 In Scope (v1)
- Host dashboard for property setup, staff definition, reservation creation, viewing past reservations, field definitions, guest page configuration, and WhatsApp group management.
- Google OAuth for host sign-in.
- Multi-tenant architecture (e.g., shared database with isolation via tenant IDs).
- Generation of unique, time-limited URLs for guests to view configurable stay summaries, receipts, and WhatsApp invite links.
- Storage of WhatsApp numbers for host, guests, and staff; automated group creation/addition/archiving/deletion via WhatsApp Business API.
- Buttons for hosts to send guest URLs and final receipts via integrated email and WhatsApp.
- Static public property page for unauthenticated users.
- Basic access controls (e.g., host-only access to dashboard).

### 3.2 Out of Scope (Future Iterations)
- Self-booking by guests.
- Payment processing or billing downloads (beyond email sending of final receipt).
- In-app messaging or chat beyond WhatsApp integration.
- Offline access, push notifications, PWA support, or calendar integrations.
- Cancellations, modifications, or overbooking handling.
- Advanced analytics beyond viewing past reservations and archived chats.
- Detailed edge cases (e.g., guests without WhatsApp or email; group addition failures due to privacy settings).

## 4. User Roles and Personas
- **Host/Owner**: Primary and only authenticated user; full admin access to setup, manage reservations, view past reservations, define content, configure guest pages, generate/send URLs/receipts, and manage WhatsApp groups (create, archive, delete). Persona: Small business owner, tech-savvy but not expert, needs intuitive UI with one-click actions for automation.
- **Staff (e.g., Kitchen, Housekeeping)**: Defined by name, role, and WhatsApp number; no app access. Automatically added to reservation-specific WhatsApp groups. Persona: On-site employee, interacts via WhatsApp.
- **Guest**: No app login; accesses configurable stay details via unique, time-limited URL provided by host. Page includes personalized content and WhatsApp invite; automatically added to group. Persona: Traveler, expects seamless mobile experience via shared link and group chat.
- **Unauthenticated Visitor**: View-only static page. Persona: Potential booker browsing properties.

## 5. Functional Requirements
This section provides detailed specifications, including data fields, UI elements, user flows, validation rules, error handling, and integration points for each feature.

### 5.1 Property Setup
- **Overview**: Hosts create and edit the core property listing, which serves as the basis for public and guest pages.
- **Detailed Features**:
  - UI: Dashboard tab with form-based editor (e.g., tabs for "Basics," "Images," "Amenities").
  - Fields:
    - Property Name (text, required, max 100 chars).
    - Description (rich text editor with formatting toolbar for bold/italic/lists, max 2000 chars).
    - Upload Pictures: Drag-and-drop uploader supporting multiple images (JPEG/PNG, max 10MB each, up to 20 images); fields per image: Caption (text, optional), Alt Text (text, for accessibility).
    - Amenities List: Dynamic list builder; host adds items via input (e.g., "Wi-Fi", "Pool"); each with checkbox for enable/disable, icon selector (from predefined set), and description (text, optional).
    - Things to Do: Separate list for resort/city activities; fields per item: Title (text), Description (rich text), Links/Images (optional).
  - Actions: Save (validates required fields), Preview (opens modal showing public view), Delete (confirmation prompt).
  - Validation: Required fields highlighted in red; error messages like "Property name is required."
  - Error Handling: Upload failures show retry button; network errors prompt "Try again later."

### 5.2 Staff Definition
- **Overview**: Hosts add staff details for automatic addition to WhatsApp groups.
- **Detailed Features**:
  - UI: Dashboard section with table view (columns: Name, Role, WhatsApp Number, Actions); "Add Staff" button opens modal form.
  - Fields per Staff:
    - First Name (text, required), Last Name (text, optional).
    - Role (dropdown: Kitchen, Housekeeping, Other; with custom text input for "Other").
    - WhatsApp Number (phone input with international format validation, e.g., +1-XXX-XXX-XXXX, required).
  - Actions: Edit (reopens modal), Delete (confirmation; warns if staff is in active groups).
  - Validation: Phone number checked for format (e.g., using regex); duplicates prevented.
  - Integration: Numbers stored for auto-addition to groups via WhatsApp API.
  - Error Handling: Invalid phone shows inline error; save fails if incomplete.

### 5.3 User Authentication
- **Overview**: Limited to hosts; guests use URLs without login.
- **Detailed Features**:
  - Google OAuth sign-in button on login page; redirects to dashboard upon success.
  - Session management: Persistent login with JWT; auto-logout after 30min inactivity.
  - Host Profile: Includes host's WhatsApp number (required for group creation).
  - No guest/staff auth.

### 5.4 Reservation Management
- **Overview**: Core feature for creating reservations, generating guest URLs, and managing WhatsApp groups.
- **Detailed Features**:
  - UI: Dashboard tab with list view (sortable table: columns for Reservation ID, Primary Guest(s), Start/End Dates, Status (Active/Completed/Archived), Group Status, Actions); filters (date range, guest name); "Create Reservation" button opens multi-step wizard form.
  - Creation Form Steps:
    - Step 1: Basics – Duration (date pickers for start/end, required; validation: end > start; auto-calculate days), Adults/Children Count (number inputs, min 1 adult), Special Instructions (textarea), Room/Parts Assigned (multi-select from property rentables; auto-populate room number for group name).
    - Step 2: Guests – Add multiple guests; fields per guest: First Name (text), Last Name (text, required), Email (email format, required for sending), WhatsApp Number (phone, required). Select primary if needed, but all last names used for group name.
    - Step 3: Additional Offerings – Checkbox list (e.g., meals, services); dynamic bill calculation (host inputs amounts per item; total displayed as "Amount Paid So Far"; editable throughout stay).
    - Step 4: Review & Automate – Preview summary including auto-generated group name (e.g., "101-Smith-Johnson-15-10-5"); "Create Group & Generate URL" button: 
      - Automatically creates WhatsApp group via API with generated name.
      - Adds host, all guests (via WhatsApp numbers), and relevant staff (e.g., all defined staff or selected per reservation).
      - Generates invite link if needed (for manual joins if auto-add fails).
      - Creates unique, time-limited URL (e.g., UUID-based, expires at end date +1 day).
  - Post-Creation Actions:
    - "Send URL" button: Triggers automated email and WhatsApp messages to all guests with URL and group invite (if applicable); template customizable (e.g., subject: "Your Stay Details at [Property]", body with placeholders like {guest_names}, {url}, {group_invite}).
    - "Update Bill" button: Edit offerings/amounts; updates dynamic receipt on URL.
    - "End Stay" button (available post-end date): Marks reservation as completed; option to "Generate Final Receipt" (locks bill, generates text/email-friendly version); "Send Final Receipt" button sends via email to guests (e.g., HTML email with itemized table).
    - "Archive Group" button: Exports chat contents via WhatsApp API (saves as text/JSON in app database, viewable in reservation details); then deletes/deactivates group (removes members and deletes via API).
  - Bill/Receipt: Dynamic JSON-stored structure (items, amounts, total); displayed as table; "Amount Paid So Far" updates in real-time; final version locked and emailable.
  - Validation: All required fields; date conflicts checked; WhatsApp numbers validated for API compatibility.
  - Error Handling: Group creation failures (e.g., invalid number) show alerts with manual fallback (e.g., generate invite link instead); API errors logged with retry button.
  - Integration: WhatsApp Business API for group create/add/archive/delete; email via Supabase/Postmark; WhatsApp messaging via API.

### 5.5 Guest Experience
- **Overview**: Time-limited URL page with configurable, structured content.
- **Detailed Features**:
  - Access: URL valid only during stay (checked server-side; redirects to error page post-expiration).
  - Page Structure (Fixed Order, Configurable Content):
    - Section 1: Personalized Greeting – Pre-configured template (e.g., "Welcome, [First1 Last1, First2 Last2]! We're excited for your stay."); host modifies text via rich editor with placeholders.
    - Section 2: Reservation and Room Details – Auto-populated (room number, duration, adults/children, special instructions, assigned rentables); host toggles sub-elements and adds custom notes.
    - Section 3: WhatsApp Group Link – Button/link: "Join Group for Your Stay ([Group Name])"; uses auto-generated invite if not auto-added; host can edit button text.
    - Section 4: Things to Do – List from property setup (resort/city activities); host adds/removes items, edits descriptions/links.
    - Section 5: Receipt Details – Dynamic table: "Amount Paid for Stay and Services So Far" (items, amounts, total); updates if host edits bill.
  - Configurable Aspects: Dashboard "Guest Page Config" section with drag-and-drop for minor reordering within sections (but overall order fixed); pre-configured templates per section (e.g., greeting options: formal/casual); host edits content via forms (rich text, placeholders like {room_number}, {duration_days}).
  - UI for Config: Accordion list; buttons to edit (opens editor), preview (simulates guest view with sample data).
  - Page Layout: Mobile-responsive; sections collapsible; includes property images/branding.
  - Error Handling: Invalid/expired URL shows "Access Denied" with contact host info.

### 5.6 Public View
- **Overview**: Static landing page for marketing.
- **Detailed Features**:
  - Content: Pulls from property setup (description, images, amenities, things to do); no personalization.
  - UI: Hero image, sections for facilities/menu/etc.; contact form (optional, sends to host email).
  - Optimization: SSR for fast load; meta tags for SEO.

### 5.7 Content Management
- **Overview**: Centralized editing for reusable fields and templates.
- **Detailed Features**:
  - UI: Dashboard page with categorized editors (e.g., Amenities, Things to Do, Page Templates).
  - Actions: Add/edit/delete items; customize templates (e.g., greeting variants); changes reflect in public/guest pages and reservations.
  - Templates: Pre-filled examples for sections; host saves custom versions.

## 6. Non-Functional Requirements
### 6.1 Performance
- Page load < 2 seconds (Next.js SSR/SSG optimization).
- Handle up to 100 concurrent users (scalable via cloud hosting).

### 6.2 Security
- HTTPS enforcement.
- Data encryption (e.g., contact details, archived chats in transit/at rest).
- Role-based access control (RBAC) limited to host.
- Multi-tenancy isolation (e.g., via Supabase row-level security).
- Unique URLs with expiration and token validation for guest access security.
- API keys for email/WhatsApp integrations stored securely; consent prompts for group addition.

### 6.3 Usability
- Mobile-first design, responsive for desktop.
- Intuitive UI (e.g., drag-drop for images/sections, tooltips for fields, one-click buttons for automations).
- Accessibility (WCAG 2.1 compliant).
- SEO optimization for public page (e.g., meta tags, sitemaps via Next.js).

### 6.4 Reliability
- 99.9% uptime.
- Error handling (e.g., graceful degradation, logging for API failures).

### 6.5 Scalability
- Multi-tenancy supported from start (e.g., shared Supabase database with tenant isolation).

## 7. Technical Stack
- **Frontend/Backend**: Next.js (with routing for API/backend logic).
- **Database**: Supabase (for auth, storage, database, including archived chats).
- **Authentication**: Google OAuth 2.0 for host (via Supabase or Next.js).
- **Storage**: Supabase storage for images and archived chat exports.
- **Integrations**: WhatsApp Business API (for groups, messaging, archiving); Email (Postmark/SendGrid via API).
- **Hosting**: Vercel.
- **Other**: React for components, Tailwind CSS for styling, React-DnD for drag-drop.

## 8. User Flows
- **Host Setup Flow**: Login → Dashboard → Create Property → Define Staff → Define Rentables → Configure Guest Page Templates.
- **Reservation Flow**: Create Reservation (wizard) → Input Details → Create Group & Generate URL → Send via Email/WhatsApp Buttons.
- **Guest Access Flow**: Receive URL (via email/WhatsApp) → View Structured Page (expires post-stay) → Join WhatsApp Group via Link (if not auto-added).
- **End Stay Flow**: End Stay Button → Generate/Send Final Receipt → Archive & Delete Group.
- **Public Flow**: Visit Site → View Static Page.
- **Communication Flow**: Auto-group creation on reservation → Interactions during stay → Archive/Delete post-stay.

## 9. Risks and Mitigations
- Risk: Multi-tenancy data leaks. Mitigation: Use Supabase security features.
- Risk: Scalability issues. Mitigation: Design with modular components.
- Risk: Google auth failures for host. Mitigation: Error messages and retries.
- Risk: Unauthorized URL access. Mitigation: Time-limited expiration, token checks.
- Risk: WhatsApp API limitations/failures (e.g., rate limits, consent issues). Mitigation: Fallback to manual invites; error notifications; test with sandbox API.
- Risk: Privacy concerns with group addition. Mitigation: Require host to confirm consents; log additions.

## 10. Testing and Deployment
- Unit tests developed alongside features (e.g., for components, APIs, integrations including WhatsApp mocks).
- Milestones: Property setup (Week 1-2), Reservation management and URL/group creation (Week 3-4), Guest page config, messaging, and archiving (Week 5), Testing/Deployment (Week 6).
- Beta: Initial rollout for one host, with WhatsApp API verification.

## 11. Appendix
- **Metrics for Success**: Host adoption rate, reservation completion time, guest satisfaction (NPS), group usage rates, successful archive/deletions.
- **Future Enhancements**: Payments integration, cancellations, advanced group features (e.g., bots), multi-host onboarding (e.g., admin approval).