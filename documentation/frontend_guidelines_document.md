# Frontend Guideline Document

## 1. Frontend Architecture

**Frameworks & Libraries**
- **Next.js (App Router + Server Components)**: Provides file-based routing, built-in SSR/SSG, and React Server Components for fast page loads.
- **TypeScript**: Offers compile-time type checking, reducing runtime bugs and improving maintainability.
- **Tailwind CSS v4** & **shadcn/ui**: Utility-first CSS for rapid styling and a ready-made component library for forms, tables, dialogs, and toasts.
- **NextAuth.js**: Manages authentication, sessions, and role-based access control for `Pegawai` and `Owner`.
- **React Query (TanStack Query)**: Handles server-state fetching, caching, and synchronization for data tables and reports.
- **Zustand**: Manages complex client-side form state (e.g., multi-step order entry).
- **Drizzle ORM + PostgreSQL**: Type-safe database models and migrations to store customers, orders, and staff data.
- **Next.js Server Actions**: Co-located data mutations (create, update, delete) with automatic revalidation and built-in Zod validation.

**Scalability, Maintainability, Performance**
- **Modular File Structure**: Public pages under `app/`, role-protected groups under `app/dashboard/(pegawai)/` and `app/dashboard/(owner)/`, shared layouts for consistency.
- **Component-Driven**: Small, focused components in `components/` allow independent development and testing.
- **Type Safety**: TypeScript + Drizzle schemas reduce errors when evolving the data model.
- **Server Components**: Move UI rendering to the server where possible, minimizing client bundle size.
- **Caching & Code Splitting**: React Query caches requests; Next.js splits code by route to speed up initial loads.

## 2. Design Principles

1. **Usability**
   - Clear, consistent layouts with obvious actions (e.g., “Simpan” buttons, navigation links).
   - Immediate feedback via dialogs and toasts (success, error messages).
2. **Accessibility**
   - Semantic HTML elements (<button>, <nav>, <form>) and ARIA labels for interactive components.
   - Keyboard navigable components and high-contrast color choices.
3. **Responsiveness**
   - Mobile-first design employing Tailwind’s responsive utilities (sm:, md:, lg: prefixes).
   - Flexible grids and cards that adapt to different screen sizes.

## 3. Styling and Theming

**Approach**: Utility-first with Tailwind CSS and custom theming via CSS variables.

**CSS Methodology**: No global CSS classes—styles are applied through Tailwind utility classes. Component-level styles use `@apply` in SASS files if needed.

**Theming**
- Light & Dark modes configured in `tailwind.config.js` with the `darkMode: 'class'` strategy.
- CSS variables (e.g., `--color-primary`, `--bg-card`) allow theme overrides.

**Visual Style**: Modern flat design with subtle glassmorphism on cards and dialogs.

**Color Palette**
- Primary: #3B82F6 (blue-500) / Dark: #2563EB (blue-600)
- Secondary: #F59E0B (amber-500)
- Success: #10B981 (green-500)
- Danger: #EF4444 (red-500)
- Neutral Light: #F3F4F6 (gray-100)
- Neutral Dark: #1F2937 (gray-800)
- Glass Card Background: rgba(255, 255, 255, 0.3) with `backdrop-filter: blur(10px)`

**Font**: “Inter”, sans-serif (imported via Google Fonts).

## 4. Component Structure

**Organization**
- `components/atoms/`: Buttons, Inputs, Labels, Icons
- `components/molecules/`: FormGroups, Tables, DialogWrappers
- `components/organisms/`: OrderForm, CustomerTable, EmployeeList
- `components/pages/`: Page-specific containers (wrapped in layouts)

**Reusability & Naming**
- PascalCase file names (e.g., `OrderForm.tsx`).
- Single responsibility: each component has one purpose and props define variations.

**Benefits**
- Promotes consistency, simplifies cross-team collaboration, and speeds up development by recombining existing pieces.

## 5. State Management

**Server State (React Query)**
- Queries: `useQuery(['orders'], fetchOrders)` to fetch and cache order lists.
- Mutations: `useMutation(createOrder)` with automatic invalidation of relevant queries.

**Client State (Zustand)**
- Local store for multi-field forms (e.g., `useOrderFormStore` to hold selected customer, service type, weight).
- Lightweight and decoupled from the UI components.

**Form Mutations (Server Actions)**
- Co-located action functions handle validation (Zod) and database writes, then trigger React Query invalidations.

## 6. Routing and Navigation

**Next.js App Router** with folder-based routes:
- Public pages:
  - `/`: Landing page
  - `/cek-status`: Laundry status lookup
  - `/signin`: Login page
- Protected dashboards under `app/dashboard/`:
  - `(pegawai)/order`, `(pegawai)/pelanggan`
  - `(owner)/pegawai`, `(shared)/laporan`

**Role-Based Middleware**
- `middleware.ts` inspects JWT session, checks user role, and redirects unauthorized access attempts.

**Layouts**
- `app/layout.tsx`: Global header and footer.
- `app/dashboard/layout.tsx`: Sidebar + header for all dashboards.
- Nested layouts per role group to adjust menu items.

## 7. Performance Optimization

- **Server Components** reduce client JS bundle.
- **Static Rendering & ISR**: Pre-build public pages; revalidate dashboards periodically.
- **Code Splitting**: Next.js automatically splits per page; dynamic imports for large components.
- **Lazy Loading**: Use `next/dynamic` for heavy chart components.
- **Image Optimization**: `next/image` for automatic resizing and modern formats.
- **Asset Compression**: Brotli/Gzip on the server; optimize SVG icons with SVGO.

## 8. Testing and Quality Assurance

**Unit Tests**
- **Vitest** + **React Testing Library** for components and business logic (e.g., price calculation).

**Integration / E2E Tests**
- **Playwright** for full user flows:
  1. Employee creates a new order.
  2. Public user checks status.
  3. Owner adds a staff member.

**Linting & Formatting**
- **ESLint** with recommended React/Next rules.
- **Prettier** for consistent code style.

**Continuous Integration**
- GitHub Actions pipeline runs: type check, lint, unit tests, E2E tests, and builds migrations.

## 9. Conclusion and Overall Frontend Summary

This guideline describes a modern, scalable, and maintainable frontend built with Next.js, TypeScript, Tailwind CSS, and a component-driven approach. By adhering to clear design principles—usability, accessibility, and responsiveness—and leveraging Next.js Server Components, React Query, and Server Actions, the laundry management system delivers fast, secure, and robust user experiences for both employees and owners. The combination of unit and E2E testing, theme support, and performance best practices ensures a production-ready application that can evolve gracefully as new features arise.