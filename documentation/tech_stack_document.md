# Laundry Management System: Tech Stack Document

This document explains the technology choices behind our Laundry Management System in simple terms. You don’t need a technical background to understand how each part works and why we picked it.

## 1. Frontend Technologies
These are the tools and libraries that build the part of the app you see and interact with.

- **Next.js (App Router & Server Components)**  
  A framework that lets us build fast web pages and organize our code neatly. Server Components help deliver content to you faster by doing some work on the server before it reaches your browser.

- **TypeScript**  
  A version of JavaScript with extra checks. It catches mistakes early, so the app is more reliable.

- **Tailwind CSS v4**  
  A styling tool that provides ready-made design bits (like colors, spacing, and fonts). It lets us style the interface quickly and consistently without writing lots of custom CSS.

- **shadcn/ui**  
  A collection of pre-built user interface pieces (buttons, forms, tables, dialogs). This library works on top of Tailwind CSS, giving us a polished look out of the box.

- **React Query (TanStack Query)**  
  Manages data fetching and caching behind the scenes. It helps keep data like customer lists and transaction reports up to date without writing extra code for loading states.

- **Zustand**  
  Manages complex on-screen states, such as multi-field forms. It keeps track of what you type and select in the order entry screens.

## 2. Backend Technologies
These components handle data processing, storage, and business logic behind the scenes.

- **Next.js API Routes & Server Actions**  
  Two ways to handle data operations (like saving an order). Server Actions let us write data-updating code right next to the form components, making the flow simpler and cleaner.

- **NextAuth (Auth.js)**  
  Handles user sign-up, login, and role-based access (Employee vs. Owner). It ensures only authorized users see the right dashboards.

- **PostgreSQL**  
  A reliable database where we store all information: customers, orders, employees, and more.

- **Drizzle ORM**  
  A tool that connects our code with the PostgreSQL database in a safe, type-checked way. We define our data structure once, and Drizzle ensures consistency when reading or writing data.

- **Zod**  
  Validates data coming from forms before it reaches the database. It ensures required fields are filled out and values are in the correct format.

## 3. Infrastructure and Deployment
This covers where the app lives, how we deploy updates, and how we manage the code.

- **Vercel (Hosting Platform)**  
  A cloud service optimized for Next.js applications. It automatically builds and serves our app with global distribution so users get fast load times.

- **Git & GitHub (Version Control)**  
  We store and track all code changes in GitHub. This helps multiple developers work together safely and review each other’s code.

- **Continuous Integration / Continuous Deployment (CI/CD)**  
  Automated pipelines that run tests and deploy the app whenever we push new code. This ensures new features or fixes go live quickly and reliably.

- **Drizzle Migrations & Seed Scripts**  
  Tools to manage database structure and initial data. Migrations update the database schema safely, and seed scripts fill in sample data for development.

## 4. Third-Party Integrations
External services that extend our app’s functionality without reinventing the wheel.

- **react-to-print**  
  Enables printing of transaction receipts (Notas) directly from the browser in a printable format.

- **Pusher or Supabase Realtime (optional)**  
  Provides live updates on order status for dashboards, removing the need for users to manually refresh the page.

- **Recharts**  
  A chart library we use to display graphs and visual reports in the Owner dashboard.

## 5. Security and Performance Considerations
Measures we put in place to keep data safe and ensure a smooth user experience.

- **Role-Based Access Control (RBAC)**  
  Using NextAuth, we define Employee and Owner roles. Middleware checks protect routes so only the right users can access each dashboard.

- **Data Validation (Zod)**  
  Ensures form inputs meet expected rules before data is saved, reducing errors and malicious input.

- **Data Caching (React Query)**  
  Stores recent data in memory, so pages load faster and network requests are minimized.

- **Server-Side Rendering & Server Components**  
  Prepares pages on the server to reduce load times and improve SEO.

- **Automated Testing**  
  • Unit tests with Vitest (or Jest) to check individual functions like price calculations.  
  • End-to-end tests with Playwright (or Cypress) to verify key user flows (order entry, status checks, employee management).

## 6. Conclusion and Overall Tech Stack Summary
In building our Laundry Management System, we chose modern, reliable tools that work well together:

- Frontend: Next.js, TypeScript, Tailwind CSS, shadcn/ui, React Query, Zustand
- Backend: Next.js API & Server Actions, NextAuth, PostgreSQL, Drizzle ORM, Zod
- Infrastructure: Vercel hosting, GitHub, CI/CD pipelines, database migrations & seeds
- Integrations: react-to-print, optional real-time updates, Recharts for charts
- Security & Performance: Role checks, data validation, caching, server-side optimizations, automated tests

These choices strike the right balance between developer productivity, application speed, and data safety. They align with project goals—delivering a user-friendly system for employees and owners—and set our Laundry Management System apart with a solid, future-proof foundation.