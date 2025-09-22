# Product Requirement Document (PRD): Airbnb Management App for Small-Scale Owners

## 1. Document Information
- **Product Name**: Airbnb Host Manager (tentative; can be customized)
- **Version**: 1.1 (Updated Iteration)
- **Date**: September 22, 2025
- **Author**: Grok 4 (based on user requirements and clarifications)
- **Stakeholders**: Small-scale Airbnb hosts/owners, staff (kitchen, housekeeping), guests, unauthenticated visitors
- **Purpose**: This PRD outlines the requirements for an app that enables Airbnb hosts to manage properties, reservations, and guest interactions. The initial version supports multi-tenancy for scalability, starting with one host/business but designed for extension to multiple hosts.

## 2. Introduction
### 2.1 Product Overview
The app is a web-based platform built with Next.js, allowing small-scale Airbnb owners to create and manage property listings, onboard staff, handle reservations, and provide personalized guest experiences. It focuses on ease of use for hosts while ensuring secure access for guests via Google authentication. Unauthenticated users see static public information. The app serves as a recordkeeping tool, with additional features like cancellations to be added later.

Key differentiators:
- Seamless integration with Google for authentication.
- Multi-tenant architecture from the start for future scalability.
- Mobile-first design with desktop responsiveness.

### 2.2 Objectives
- Enable hosts to set up and customize property pages efficiently.
- Streamline reservation management and guest onboarding.
- Provide convenient access for guests with persistent login, but dynamic content changes post-stay.
- Ensure a static, informative experience for public visitors.
- Support multi-tenancy for easy expansion to multiple properties/hosts.

### 2.3 Assumptions
- Reservations are manually created by hosts only (no self-booking).
- No billing PDFs or downloads; bills displayed as dynamic values.
- No WhatsApp integration in v1; specifications to be provided later.
- Guest profiles created via Google accounts entered by host; users responsible for valid contact details.
- Guest access persists post-stay, but page information updates (e.g., to show past stay details or limited info).
- No offline access, cancellations, or advanced edge case handling in v1.
- Compliance with basic privacy (e.g., consent for data storage), but detailed data handling ignored for now.

## 3. Scope
### 3.1 In Scope (v1)
- Host dashboard for property setup, staff onboarding, reservation creation, viewing past reservations, and field definitions.
- Google OAuth for user sign-in (hosts, guests, staff).
- Multi-tenant architecture (e.g., shared database with isolation via tenant IDs).
- Custom guest pages with personalized and generic information.
- Static public property page for unauthenticated users.
- Basic access controls (e.g., role-based for hosts/staff/guests).

### 3.2 Out of Scope (Future Iterations)
- Self-booking by guests.
- Payment processing or billing downloads.
- WhatsApp integration (e.g., notifications, communication links).
- In-app messaging or chat.
- Offline access, push notifications, PWA support, or calendar integrations.
- Cancellations, modifications, or overbooking handling.
- Advanced analytics beyond viewing past reservations.
- ID verification or detailed edge cases (e.g., guests without Google).

## 4. User Roles and Personas
- **Host/Owner**: Primary user; full admin access to setup, manage reservations, view past reservations, and define content. Persona: Small business owner, tech-savvy but not expert, needs intuitive UI.
- **Staff (e.g., Kitchen, Housekeeping)**: Limited access; view public page only. Communication features (e.g., WhatsApp) in later phases. Persona: On-site employee, mobile-first user.
- **Guest**: Access via Google login; view stay details, update profile. Access persists post-stay with changed information. Persona: Traveler, expects seamless mobile experience.
- **Unauthenticated Visitor**: View-only static page. Persona: Potential booker browsing properties.

## 5. Functional Requirements
### 5.1 Property Setup
- Hosts can create/edit a landing page with:
  - Upload pictures (multiple, with captions).
  - Description (rich text).
  - Amenities list (e.g., Wi-Fi, pool; customizable checkboxes or tags).
- Define rentable parts (e.g., "Twin Room", "Lawn Area", "Tour Package #1"):
  - Fields: Name, description, capacity (adults/children), additional notes.
  - Tour packages: Simple text field.

### 5.2 Staff Onboarding
- Hosts add staff profiles:
  - Fields: Name, role (e.g., kitchen, housekeeping), contact (email/phone), permissions (public page access only by default).
- Staff sign-in via Google; access limited to public page.

### 5.3 User Authentication
- Google OAuth for sign-in (all roles).
- Guests: Login with Google account registered by host; option to update profile.
- Profiles: Created via Google; fields include name, email, preferences (WhatsApp number entry is user's responsibility; no integration).
- Unauthenticated: Static view only.

### 5.4 Reservation Management
- Hosts create reservations:
  - Fields: Duration (start/end dates), rooms/rentable parts assigned, adults/children count, special instructions, additional offerings (e.g., meals).
- Add adult profiles to reservation (link to Google for auto-fill).
- Hosts view all past reservations via dashboard (e.g., list with filters by date, guest).

### 5.5 Guest Experience
- After login: Custom page showing:
  - Personalized: Rooms reserved, duration of stay, bill (dynamic values, no download), communication links (WhatsApp specs later).
  - Generic: Facilities, kitchen menu, tour packages, attractions around, other public links/reviews.
- Access persists post-stay; page content changes (e.g., show archived details or message indicating stay ended).

### 5.6 Public View
- Static page for unauthenticated users: Description, generic fields (facilities, menu, etc.).
- No sign-in required; SEO-optimized.

### 5.7 Content Management
- Hosts define/edit values for all fields (e.g., amenities, menu items) via dashboard.
- Fields customizable by host (e.g., templates for generic/personalized content).

## 6. Non-Functional Requirements
### 6.1 Performance
- Page load < 2 seconds (Next.js SSR/SSG optimization).
- Handle up to 100 concurrent users (scalable via cloud hosting).

### 6.2 Security
- HTTPS enforcement.
- Data encryption (e.g., profiles in transit/at rest).
- Role-based access control (RBAC).
- Multi-tenancy isolation (e.g., via Supabase row-level security).

### 6.3 Usability
- Mobile-first design, responsive for desktop.
- Intuitive UI (e.g., drag-drop for images).
- Accessibility (WCAG 2.1 compliant).
- SEO optimization (e.g., meta tags, sitemaps via Next.js).

### 6.4 Reliability
- 99.9% uptime.
- Error handling (e.g., graceful degradation).

### 6.5 Scalability
- Multi-tenancy supported from start (e.g., shared Supabase database with tenant isolation).

## 7. Technical Stack
- **Frontend/Backend**: Next.js (with routing for API/backend logic).
- **Database**: Supabase (for auth, storage, and database).
- **Authentication**: Google OAuth 2.0 (via Supabase or Next.js).
- **Storage**: Supabase storage for images.
- **Hosting**: Vercel.
- **Other**: React for components, Tailwind CSS for styling.

## 8. User Flows
- **Host Setup Flow**: Login → Dashboard → Create Property → Add Staff → Define Rentables.
- **Reservation Flow**: Create Reservation → Add Guests (with Google accounts).
- **Guest Access Flow**: Login with Google → View Custom Page (content changes post-stay).
- **Public Flow**: Visit Site → View Static Page.

## 9. Risks and Mitigations
- Risk: Multi-tenancy data leaks. Mitigation: Use Supabase security features.
- Risk: Scalability issues. Mitigation: Design with modular components.
- Risk: Google auth failures. Mitigation: Error messages and retries.

## 10. Testing and Deployment
- Unit tests developed alongside features (e.g., for components, APIs).
- Milestones: Property setup (Week 1-2), Reservation management (Week 3-4), Guest pages (Week 5), Testing/Deployment (Week 6).
- Beta: Initial rollout for one host.

## 11. Appendix
- **Metrics for Success**: Host adoption rate, reservation completion time, guest satisfaction (NPS).
- **Future Enhancements**: WhatsApp integration, payments, cancellations, multi-host onboarding (e.g., admin approval).