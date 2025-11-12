# Backend Structure Document

## 1. Backend Architecture

**Overall Design**
- We use a **Next.js** backend based on the App Router. This means our server-side logic lives alongside our pages and components. We rely on **Server Actions** for data mutations (create, update, delete) and **Server Components** for data fetching.
- **NextAuth (Auth.js)** handles authentication, providing session management and role-based access control (RBAC) for `Pegawai` and `Owner` users.
- **Drizzle ORM** connects our code to a **PostgreSQL** database via type‐safe query builders and migrations.

**Supporting Scalability, Maintainability, and Performance**
- **Component-driven structure**: co-locating server code with UI components reduces boilerplate and keeps logic organized.
- **Server Actions**: minimize API overhead by running mutations on the server directly from components.
- **Edge Functions and Serverless**: hosted on Vercel, scaling automatically as traffic increases.
- **Strict typing** (TypeScript + Drizzle schema types) catches errors early and makes refactoring safer.

## 2. Database Management

**Technologies**
- Type: SQL
- System: PostgreSQL (hosted on AWS RDS or a managed Postgres service)
- ORM: Drizzle ORM for migrations, schema definitions, and type-safe queries

**Data Organization & Access**
- We separate entities into dedicated tables (Users, Pelanggan, Pegawai, Pesanan) with clear relationships (foreign keys).
- All queries go through Drizzle, ensuring consistent SQL generation and preventing injection.
- Migrations track schema changes. Seed scripts populate sample data for dev environments.
- Connection pooling (via the managed database service) keeps queries performant under load.

## 3. Database Schema

Below is a human-readable overview, followed by actual PostgreSQL table definitions.

**Entities and Relationships**
- **Users**: authentication table storing login credentials and RBAC roles.
- **Pegawai**: employee profile, linked to a User account.
- **Pelanggan**: customer profile.
- **Pesanan**: laundry orders, referencing Pelanggan and Pegawai.

**PostgreSQL Schema**
```sql
-- 1. Users (Authentication & Roles)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL CHECK(role IN ('pegawai', 'owner')),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- 2. Pegawai (Employees)
CREATE TABLE pegawai (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  phone TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- 3. Pelanggan (Customers)
CREATE TABLE pelanggan (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  phone TEXT NOT NULL,
  address TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- 4. Pesanan (Orders)
CREATE TABLE pesanan (
  id SERIAL PRIMARY KEY,
  pelanggan_id INT NOT NULL REFERENCES pelanggan(id) ON DELETE RESTRICT,
  pegawai_id INT REFERENCES pegawai(id) ON DELETE SET NULL,
  weight_kg NUMERIC(5,2) NOT NULL,
  service_type TEXT NOT NULL CHECK(service_type IN ('kiloan', 'express', 'cuci kering')),
  status TEXT NOT NULL CHECK(status IN ('baru', 'diproses', 'selesai', 'diambil')),
  price_cents INT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);
```

## 4. API Design and Endpoints

We blend **REST-style** endpoints (for non-mutation fetches) with **Next.js Server Actions** for all writes. Authentication and authorization run via middleware.

**Key Endpoints**
- `GET /api/pelanggan` : Fetch list of customers (paginated)
- `GET /api/pelanggan/{id}` : Fetch a single customer’s details
- `GET /api/pegawai` : Fetch list of employees (Owner-only)
- `GET /api/pesanan` : Fetch list of orders with filters (status, date range)
- `GET /api/pesanan/{id}` : Fetch a specific order

**Server Actions (POST/PUT/DELETE)**
- `createPelanggan(data)` : Adds a new customer
- `updatePelanggan(id, data)` : Updates an existing customer
- `createPesanan(data)` : Creates a new order
- `updatePesananStatus(id, status)` : Updates order status
- `createPegawai(data)` : Owner-only: adds a new employee
- `updatePegawai(id, data)` : Owner-only: edits an employee
- `deletePegawai(id)` : Owner-only: removes an employee

**Communication Flow**
1. Frontend calls a React Query hook to `GET /api/pesanan`.
2. Data arrives as JSON and populates tables in the UI.
3. On form submit, frontend calls a Server Action (`createPesanan`).
4. Server Action writes to the DB, returns updated data or error.
5. React Query invalidates cache and refetches fresh data.

## 5. Hosting Solutions

**Application Hosting**
- Deployed on **Vercel** (serverless functions & edge network).
- Benefits: zero-config deploys, global CDN, automatic scaling, built-in HTTPS.

**Database Hosting**
- **AWS RDS (PostgreSQL)** or an equivalent managed Postgres service (e.g., DigitalOcean Managed DB, Supabase).
- Benefits: automatic backups, multi-AZ failover, connection pooling, predictable pricing.

## 6. Infrastructure Components

- **Load Balancer**: Vercel’s edge routing balances requests across serverless instances.
- **CDN**: Vercel’s global CDN caches static assets and optimizes page streaming.
- **Caching Layer**: React Query caches API responses in memory. For more aggressive caching or cross-instance caching, you can add **Redis**.
- **Queue (Optional)**: If you introduce background jobs (e.g., sending emails or generating reports), you can add **BullMQ** backed by Redis.
- **Documentation & API Explorer**: We can integrate **Swagger UI** or **OpenAPI** definitions for easy API testing.

## 7. Security Measures

- **Transport Security**: Enforce HTTPS/TLS on all endpoints via Vercel certificates.
- **Authentication**: NextAuth with secure cookies and refresh tokens.
- **Authorization**: Middleware checks user roles (`pegawai`, `owner`) before granting access to protected routes.
- **Input Validation**: Zod schemas in Server Actions ensure incoming data matches expectations.
- **Data Encryption**: PostgreSQL at-rest encryption (managed by the cloud provider) and SSL for connections.
- **Secret Management**: Environment variables stored securely in Vercel and AWS Secrets Manager.
- **HTTP Headers**: Security headers (CSP, HSTS, X-Frame-Options) set via Vercel or custom middleware.

## 8. Monitoring and Maintenance

- **Error Tracking**: Sentry captures runtime errors in server and client code.
- **Performance Monitoring**: Vercel’s built-in analytics + optional Datadog or New Relic integration.
- **Logs**: Centralized logging via Vercel logs or forwarding to a service like Logflare.
- **Database Metrics**: Cloud provider’s dashboard (AWS RDS console) monitors CPU, connections, slow queries.
- **Health Checks**: Scheduled pings to a `/api/health` endpoint to verify uptime.
- **Maintenance**
  - Regular Drizzle migrations handle schema evolution.
  - Monthly dependency updates via Dependabot.
  - Continuous Integration (GitHub Actions) runs tests, lints, and builds on every PR.

## 9. Conclusion and Overall Backend Summary

This backend is built for reliability, security, and easy evolution. By combining Next.js App Router, Server Actions, PostgreSQL, and Drizzle ORM, we achieve:

- **Clear separation of concerns**: UI components, server logic, and data models are co-located but well-scoped.
- **High performance**: Edge functions and global CDN minimize latency.
- **Scalability**: Serverless functions and managed database scale as demand grows.
- **Maintainability**: Type-safe schemas, migrations, and automated tests keep the codebase healthy.

Unique to this project is the use of Next.js Server Actions to simplify data mutations, giving us a leaner, more maintainable codebase and a smoother developer experience. With robust RBAC, monitoring, and automated workflows, this backend is ready to support your laundry management system now and in the future.