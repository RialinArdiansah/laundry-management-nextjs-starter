# Project Requirements Document: Laundry Management System

## 1. Project Overview
This web-based Laundry Management System is designed to help small to medium laundry businesses move from manual or spreadsheet-based order tracking to a streamlined digital workflow. Through a public portal, customers can check their laundry status, and through secure, role-based dashboards, employees (“Pegawai”) and owners (“Owner”) can manage orders, customers, employees, and financial reports.

By building this system, the goal is to reduce errors in order entry, speed up customer check-ins and status lookups, and give owners real-time visibility into operations and revenue. Success will be measured by faster order processing times, fewer lost or misplaced orders, and clear, actionable reports that help owners make data-driven decisions.

## 2. In-Scope vs. Out-of-Scope

In-Scope (Version 1):
- Public landing page and **Status Check** page (customers enter order ID to see progress)
- Authentication with NextAuth.js supporting two roles: **Pegawai** and **Owner**
- **Pegawai Dashboard**:
  - Create new orders (Pelanggan + Pesanan)
  - View and edit existing orders (update weight, service type, status)
  - Browse customer list (Pelanggan) with search and filter
- **Owner Dashboard**:
  - Manage employee accounts (add, edit, deactivate Pegawai)
  - View transaction history with date-range filters
  - View summary reports (daily orders, revenue, average turnaround time)
- Receipt (Nota) generation and printable view via `react-to-print`
- Responsive UI with Tailwind CSS and shadcn/ui component library
- PostgreSQL database with Drizzle ORM schema and migrations
- Server Actions for all create/update/delete operations, validated by Zod
- Basic unit tests (Vitest) and end-to-end tests (Playwright)

Out-of-Scope (Future Phases):
- Real-time push updates (e.g., Pusher/Supabase Realtime)
- Mobile app or native wrapper
- SMS/email notifications for status changes
- Advanced analytics or BI integrations
- Forgot-password and self-service password reset flows
- Multi-location or franchise support

## 3. User Flow
A public visitor lands on the home page and clicks “Check Laundry Status.” They enter an order ID, submit the form, and immediately see a timeline of status updates (Received → In Progress → Completed → Picked Up). If they enter an invalid ID, an error toast appears.

An employee logs in via email and password on the **Login** page. After authentication, they arrive at their dashboard, see a sidebar with menu items (“New Order,” “Customers,” “Transactions”), and a main panel showing today’s pending orders. They click “New Order,” fill out customer details and laundry weight/service, submit to save, then optionally print a receipt. To update status, they navigate to “Transactions,” select an order, change its status, and click “Save.”

An owner logs in and sees a different sidebar (“Employee Management,” “Reports,” “Monitoring”). They add or edit employee accounts under “Employee Management,” then browse “Reports” to view charts and tables summarizing revenue, order counts, and average processing times. They can filter by date or employee and export to CSV.

## 4. Core Features
- **Authentication & RBAC**: NextAuth.js with role-based protection (Pegawai vs. Owner).
- **Public Status Check**: Single-page form to query order status by ID.
- **Order Management**:
  • Create, read, update orders with fields: customer name, contact, weight, service type, status, total cost.
  • Drizzle ORM models for `Pelanggan`, `Pesanan`, `Pegawai`.
- **Customer List**: Table with search, sort, pagination.
- **Employee Management**: Owner-only CRUD for `Pegawai` accounts.
- **Transaction Reports**: Date-filtered tables and charts (using Recharts).
- **Printable Receipts**: `react-to-print` component styled with Tailwind.
- **Forms & Validation**: Zod schemas for all Server Actions and form inputs.
- **UI Components & Theming**: Tailwind CSS, shadcn/ui for inputs, buttons, tables, dialogs, toasts; dark mode support.
- **Testing**: Unit tests (Vitest) and E2E tests (Playwright).

## 5. Tech Stack & Tools
- **Frontend**: Next.js (App Router, Server & Client Components), TypeScript
- **Styling**: Tailwind CSS v4, shadcn/ui component library
- **State & Data Fetching**: React Query (TanStack Query) for server state, Zustand for complex form state
- **Backend & APIs**: Next.js Server Actions for mutations, Server Components for data fetching
- **Auth**: NextAuth.js (Auth.js) for session & role management
- **Database**: PostgreSQL, Drizzle ORM (schema, migrations, seed scripts)
- **Validation**: Zod for schema validation in Server Actions
- **Print**: react-to-print for generating receipts
- **Charts**: Recharts for report visualizations
- **Testing**: Vitest for unit tests, Playwright for end-to-end tests
- **IDE/Plugins**: VS Code (recommended), optional Cursor or Windsurf for AI-assisted coding

## 6. Non-Functional Requirements
- **Performance**: Page loads under 2 seconds on 3G; data updates reflect in UI under 1 second.
- **Security**: HTTPS everywhere; OWASP Top 10 mitigations; role-based access control enforced in middleware.
- **Usability**: Responsive design across desktop/tablet/mobile; WCAG 2.1 AA accessibility standards.
- **Scalability**: Support up to 1,000 daily orders; database indices on key fields (order ID, customer name).
- **Reliability**: 99.9% uptime; automatic retries for transient DB connection errors.
- **Compliance**: GDPR-friendly handling of personal data; audit logs for CRUD operations.

## 7. Constraints & Assumptions
- Deployment on a Node.js-compatible platform (Vercel, AWS, etc.) with PostgreSQL support.
- Next.js App Router and Server Actions available in the target runtime.
- Internet connectivity for employees and owners.
- Minimum Node.js version ≥ 18, PostgreSQL ≥ 14.
- Users have modern browsers (Chrome, Firefox, Edge, Safari).
- Auth.js secrets and database credentials managed via environment variables.

## 8. Known Issues & Potential Pitfalls
- **Form Double-Submit**: Mitigate by disabling buttons during Server Action calls.
- **Database Migration Conflicts**: Use Drizzle’s migration lock and consistent environment setups.
- **Cross-Browser PDF Print Layout**: Test `react-to-print` on all browsers; provide fallback PDF download if print fails.
- **API Rate Limits**: If integrating external services later (e.g., SMS), plan for exponential backoff.
- **Data Consistency on Concurrent Updates**: Use optimistic locking or version columns if simultaneous edits occur.
- **Visual Flicker on Data Refetch**: Use React Query’s `placeholderData` and `keepPreviousData` to smooth UI transitions.

---

This PRD captures all requirements for the initial release of the Laundry Management System. Subsequent documents (Tech Stack Details, Frontend Guidelines, Backend Structure, File Structure) can be generated directly from these clear specifications without ambiguity.