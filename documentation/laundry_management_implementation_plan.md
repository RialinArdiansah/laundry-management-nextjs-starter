# Rencana Implementasi Sistem Manajemen Laundry Berbasis Web

## 1. Pendahuluan

Dokumen ini merinci rencana implementasi lengkap untuk Sistem Informasi Manajemen Laundry berbasis web menggunakan teknologi modern sesuai spesifikasi yang diminta. Sistem ini akan dikembangkan dengan Next.js App Router, TypeScript, Tailwind CSS, dan PostgreSQL dengan Prisma/Drizzle ORM.

## 2. Spesifikasi Teknologi

### Frontend Stack
- **Next.js**: Versi terbaru dengan App Router dan React Server Components
- **TypeScript**: Untuk keamanan kode dan kemudahan maintenance
- **Tailwind CSS**: Untuk styling yang cepat, ringan, dan responsif
- **React Query atau Zustand**: Untuk pengelolaan data client

### Backend Stack
- **Next.js API Routes atau Server Actions**: Untuk logika server
- **PostgreSQL**: Database utama
- **Prisma atau Drizzle ORM**: Manajemen database yang lebih mudah dan terstruktur
- **Auth.js atau Clerk**: Autentikasi dan manajemen pengguna

## 3. Struktur Database

Berdasarkan spesifikasi yang diberikan, database akan memiliki 4 tabel utama:

### 3.1 Tabel Akun
```sql
- id (Primary Key, UUID)
- email/username (String, Unique)
- password (String, Hashed)
- hak_akses (Enum: 'Pegawai', 'Owner')
- created_at (Timestamp)
- updated_at (Timestamp)
```

### 3.2 Tabel Pelanggan
```sql
- id_pelanggan (Primary Key, UUID)
- nama_pelanggan (String)
- alamat (Text)
- nomor_hp (String)
- created_at (Timestamp)
- updated_at (Timestamp)
```

### 3.3 Tabel Pesanan
```sql
- nomor_order (Primary Key, String, Auto-generated)
- id_pelanggan (Foreign Key ke Pelanggan)
- nama_pelanggan (String)
- jenis_layanan (Enum: 'express', 'regular')
- berat_cucian (Decimal)
- total_biaya (Decimal)
- tanggal_masuk (Date)
- tanggal_estimasi_selesai (Date)
- status_pesanan (Enum: 'Diterima', 'Proses', 'Selesai', 'Siap Diambil')
- tanggal_update_status (Timestamp)
- catatan (Text, Optional)
- created_at (Timestamp)
- updated_at (Timestamp)
```

### 3.4 Tabel Pegawai
```sql
- id_pegawai (Primary Key, UUID)
- nama (String)
- username (String, Unique)
- password (String, Hashed)
- jabatan (String)
- nomor_hp (String)
- id_akun (Foreign Key ke Akun)
- created_at (Timestamp)
- updated_at (Timestamp)
```

## 4. Daftar Screen Sistem Informasi Manajemen Laundry Berbasis Web

### A. SCREEN UMUM (AKSES PUBLIK)

#### 1. Halaman Utama Situs
**Komponen UI/UX:**
- Hero section dengan branding laundry
- Tombol Login yang jelas terlihat
- Menu Cek Status Laundry yang mudah diakses
- Informasi singkat tentang layanan

**Komponen React:**
```typescript
// components/LandingPage.tsx
export default function LandingPage() {
  return (
    <div>
      <Hero />
      <Services />
      <CallToAction>
        <Link href="/login">Login</Link>
        <Link href="/cek-status">Cek Status Laundry</Link>
      </CallToAction>
    </div>
  )
}
```

#### 2. Halaman Login
**Elemen UI/UX:**
- Form dengan field Username/email dan Password
- Tombol Masuk dengan loading state
- Link lupa password (untuk implementasi masa depan)

**Validasi dan Pesan Sistem:**
- Pesan error: "Username atau password salah."
- Pesan sukses: "Login berhasil."

**Komponen React:**
```typescript
// components/LoginForm.tsx
"use client"

import { useState } from 'react'
import { signIn } from 'next-auth/react'

export default function LoginForm() {
  const [error, setError] = useState('')
  const [loading, setLoading] = useState(false)

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setLoading(true)
    setError('')

    // Implementasi login logic
    // ...
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      {error && <div className="error">{error}</div>}
      <button type="submit" disabled={loading}>
        {loading ? 'Memproses...' : 'Masuk'}
      </button>
    </form>
  )
}
```

#### 3. Halaman Cek Status Laundry
**Elemen UI/UX:**
- Form dengan kolom Nomor Order dan Nomor Telepon
- Tombol Cari dengan validasi
- Hasil pencarian dengan informasi lengkap:
  - Nama Pelanggan
  - Jenis Layanan
  - Berat Cucian
  - Tanggal Diterima
  - Estimasi Selesai
  - Status Pesanan
  - Catatan tambahan (batas penyimpanan)

**Pesan Sistem:**
- Pesan sukses: "Data pesanan ditemukan."
- Pesan error: "Nomor order tidak ditemukan."

**Implementasi Server Action:**
```typescript
// app/actions/orderActions.ts
'use server'

import { db } from '@/lib/db'
import { z } from 'zod'

const orderCheckSchema = z.object({
  nomorOrder: z.string().min(1),
  nomorTelepon: z.string().min(1)
})

export async function checkOrderStatus(formData: FormData) {
  try {
    const { nomorOrder, nomorTelepon } = orderCheckSchema.parse({
      nomorOrder: formData.get('nomorOrder'),
      nomorTelepon: formData.get('nomorTelepon')
    })

    const order = await db.query.pesanan.findFirst({
      where: and(
        eq(pesanan.nomorOrder, nomorOrder),
        // Logika pencocokan nomor telepon
      )
    })

    if (!order) {
      return { error: 'Nomor order tidak ditemukan.' }
    }

    return {
      success: 'Data pesanan ditemukan.',
      data: order
    }
  } catch (error) {
    return { error: 'Terjadi kesalahan dalam pencarian.' }
  }
}
```

### B. SCREEN DASHBOARD (SESUAI HAK AKSES)

#### 4. Dashboard Pegawai
**Menu Utama:**
- Order Laundry
- Data Pelanggan
- Laporan Transaksi
- Status Cucian

**Struktur Layout:**
```typescript
// app/dashboard/pegawai/layout.tsx
export default function PegawaiLayout({ children }) {
  return (
    <div className="flex">
      <Sidebar>
        <SidebarItem href="/dashboard/pegawai/order">Order Laundry</SidebarItem>
        <SidebarItem href="/dashboard/pegawai/pelanggan">Data Pelanggan</SidebarItem>
        <SidebarItem href="/dashboard/pegawai/laporan">Laporan Transaksi</SidebarItem>
        <SidebarItem href="/dashboard/pegawai/status">Status Cucian</SidebarItem>
      </Sidebar>
      <main>{children}</main>
    </div>
  )
}
```

#### 5. Dashboard Owner
**Menu Utama:**
- Data Pegawai
- Laporan
- Monitoring

**Middleware untuk Role-based Access:**
```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token')?.value

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  // Logic untuk role-based access control
  // ...
}
```

### C. SCREEN MANAJEMEN ORDER (ROLE: PEGAWAI)

#### 6. Halaman Input Pesanan
**Fitur UI/UX:**
- Filter nama pelanggan dengan autocomplete
- Tombol tambah Pelanggan inline
- Form lengkap untuk data pelanggan:
  - Nama pelanggan
  - Alamat
  - Nomor Hp
  - Tombol simpan pelanggan
- Pilihan jenis layanan: express (1 hari), regular (3 hari)
- Input berat cucian dengan kalkulasi otomatis
- Tanggal masuk pesanan (auto-hari ini)
- Tanggal estimasi selesai (auto-berdasarkan layanan)
- Catatan opsional
- Total biaya dengan kalkulasi otomatis
- Tombol simpan order

**Validasi dan Pesan Sistem:**
- Pesan pop up: "Order berhasil disimpan."
- Peringatan: "Harap lengkapi semua data pesanan sebelum menyimpan."
- Error: "Gagal menyimpan data order. Silakan coba lagi."

**Nota Transaksi:**
- Nomor order
- Nama pelanggan
- Jenis layanan
- Berat cucian
- Total biaya
- Tanggal masuk
- Tanggal estimasi selesai
- Tombol cetak nota

**Implementasi Server Action untuk Order:**
```typescript
// app/actions/orderActions.ts
export async function createOrder(formData: FormData) {
  try {
    const orderSchema = z.object({
      idPelanggan: z.string().uuid(),
      namaPelanggan: z.string().min(1),
      jenisLayanan: z.enum(['express', 'regular']),
      beratCucian: z.number().positive(),
      catatan: z.string().optional()
    })

    const validatedData = orderSchema.parse({
      idPelanggan: formData.get('idPelanggan'),
      namaPelanggan: formData.get('namaPelanggan'),
      jenisLayanan: formData.get('jenisLayanan'),
      beratCucian: parseFloat(formData.get('beratCucian') as string),
      catatan: formData.get('catatan')
    })

    // Kalkulasi total biaya
    const hargaPerKg = validatedData.jenisLayanan === 'express' ? 15000 : 10000
    const totalBiaya = validatedData.beratCucian * hargaPerKg

    // Kalkulasi estimasi selesai
    const tanggalMasuk = new Date()
    const estimasiSelesai = new Date(tanggalMasuk)
    estimasiSelesai.setDate(
      estimasiSelesai.getDate() + (validatedData.jenisLayanan === 'express' ? 1 : 3)
    )

    const nomorOrder = generateOrderNumber()

    await db.insert(pesanan).values({
      nomorOrder,
      idPelanggan: validatedData.idPelanggan,
      namaPelanggan: validatedData.namaPelanggan,
      jenisLayanan: validatedData.jenisLayanan,
      beratCucian: validatedData.beratCucian,
      totalBiaya,
      tanggalMasuk,
      tanggalEstimasiSelesai: estimasiSelesai,
      statusPesanan: 'Diterima',
      catatan: validatedData.catatan
    })

    return {
      success: 'Order berhasil disimpan.',
      orderId: nomorOrder
    }
  } catch (error) {
    return {
      error: 'Gagal menyimpan data order. Silakan coba lagi.'
    }
  }
}
```

### D. SCREEN MANAJEMEN DATA PELANGGAN (ROLE: PEGAWAI)

#### 7. Halaman Data Pelanggan
**Fitur UI/UX:**
- Tabel daftar pelanggan dengan kolom:
  - Nama
  - Alamat
  - Nomor HP
- Fitur pencarian dan filter
- Tombol Tambah Pelanggan
- Pagination untuk data yang banyak

#### 8. Form Tambah Pelanggan
**Fitur UI/UX:**
- Form dengan validasi real-time
- Isian Nama, Alamat, Nomor HP
- Tombol "Simpan" dan "Batal"

**Pesan Sistem:**
- Pop-up: "Data pelanggan berhasil disimpan."
- Peringatan: "Lengkapi semua data sebelum menyimpan."

### E. SCREEN MANAJEMEN DATA PEGAWAI (ROLE: OWNER)

#### 9. Halaman Data Pegawai
**Fitur UI/UX:**
- Tabel daftar pegawai dengan kolom:
  - Nama
  - Jabatan
  - Username
  - Nomor HP
- Tombol Tambah Pegawai
- Aksi Edit dan Delete

#### 10. Form Tambah/Edit Pegawai
**Fitur UI/UX:**
- Form dengan field:
  - Nama pegawai
  - Username
  - Password
  - Jabatan
  - Nomor HP
- Tombol "Simpan" dan "Batal"

**Pesan Sistem:**
- Pop-up: "Data pegawai berhasil disimpan."
- Peringatan: "Lengkapi semua data sebelum menyimpan."

### F. SCREEN UPDATE STATUS LAUNDRY (ROLE: PEGAWAI)

#### 11. Halaman Daftar Transaksi Laundry/Status Cucian
**Fitur UI/UX:**
- Tabel transaksi dengan kolom:
  - Nomor Order
  - Nama Pelanggan
  - Status
  - Tanggal Masuk
- Tombol Update Status per baris
- Filter berdasarkan status

#### 12. Form Update Status
**Fitur UI/UX:**
- Dropdown pilihan status:
  - Diterima
  - Proses
  - Selesai
  - Siap Diambil
- Tombol "Simpan" dan "Batal"

**Pesan Sistem:**
- Pop-up: "Status berhasil diperbarui."
- Error: "Gagal memperbarui status. Silakan coba lagi."

### G. SCREEN LAPORAN (ROLE: PEGAWAI & OWNER)

#### 13. Halaman Laporan Transaksi (Pegawai)
**Fitur UI/UX:**
- Tabel laporan transaksi
- Filter tanggal
- Export ke CSV/PDF
- Ringkasan pendapatan harian/mingguan

#### 14. Halaman Laporan (Owner)
**Fitur UI/UX:**
- Laporan lengkap dengan grafik menggunakan Recharts
- Analisis pendapatan
- Laporan kinerja pegawai
- Monitoring operasional real-time

## 5. Implementasi Prisma Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Akun {
  id          String   @id @default(cuid())
  email       String   @unique
  password    String
  hakAkses    HakAkses
  pegawai     Pegawai?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@map("akun")
}

model Pelanggan {
  id          String   @id @default(cuid())
  namaPelanggan String
  alamat      String?
  nomorHp     String
  pesanan     Pesanan[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@map("pelanggan")
}

model Pesanan {
  id                    String   @id @default(cuid())
  nomorOrder            String   @unique
  idPelanggan           String
  namaPelanggan         String
  jenisLayanan          JenisLayanan
  beratCucian           Decimal
  totalBiaya            Decimal
  tanggalMasuk          DateTime
  tanggalEstimasiSelesai DateTime
  statusPesanan         StatusPesanan @default(DITERIMA)
  tanggalUpdateStatus   DateTime @default(now())
  catatan               String?
  pelanggan             Pelanggan @relation(fields: [idPelanggan], references: [id], onDelete: Cascade)
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt

  @@map("pesanan")
}

model Pegawai {
  id          String   @id @default(cuid())
  nama        String
  username    String   @unique
  password    String
  jabatan     String
  nomorHp     String
  idAkun      String   @unique
  akun        Akun     @relation(fields: [idAkun], references: [id], onDelete: Cascade)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@map("pegawai")
}

enum HakAkses {
  PEGAWAI
  OWNER
}

enum JenisLayanan {
  EXPRESS
  REGULAR
}

enum StatusPesanan {
  DITERIMA
  PROSES
  SELESAI
  SIAP_DIAMBIL
}
```

## 6. Struktur Folder dan File

```
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── layout.tsx
│   ├── dashboard/
│   │   ├── pegawai/
│   │   │   ├── order/
│   │   │   ├── pelanggan/
│   │   │   ├── laporan/
│   │   │   ├── status/
│   │   │   └── layout.tsx
│   │   ├── owner/
│   │   │   ├── pegawai/
│   │   │   ├── laporan/
│   │   │   ├── monitoring/
│   │   │   └── layout.tsx
│   │   └── layout.tsx
│   ├── cek-status/
│   ├── actions/
│   │   ├── orderActions.ts
│   │   ├── pelangganActions.ts
│   │   ├── pegawaiActions.ts
│   │   └── authActions.ts
│   ├── api/
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/ (shadcn/ui components)
│   ├── forms/
│   ├── layout/
│   └── dashboard/
├── lib/
│   ├── db.ts (Prisma client)
│   ├── auth.ts (NextAuth config)
│   ├── utils.ts
│   └── validations.ts (Zod schemas)
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── public/
├── types/
│   └── index.ts
└── tailwind.config.js
```

## 7. Tahapan Implementasi

### Phase 1: Setup Database & Authentication (Week 1-2)
1. Setup PostgreSQL database
2. Konfigurasi Prisma ORM
3. Implementasi schema database
4. Setup NextAuth.js dengan role-based access
5. Membuat middleware untuk proteksi route

### Phase 2: Public Pages & Basic Layout (Week 3)
1. Halaman utama (landing page)
2. Halaman login dengan validasi
3. Halaman cek status laundry
4. Layout dashboard untuk pegawai dan owner

### Phase 3: Pegawai Dashboard - Order Management (Week 4-5)
1. Form input pesanan lengkap
2. Manajemen data pelanggan
3. Update status laundry
4. Generate nota transaksi

### Phase 4: Owner Dashboard & Reporting (Week 6-7)
1. Manajemen data pegawai
2. Laporan transaksi pegawai
3. Laporan lengkap owner dengan grafik
4. Monitoring dashboard

### Phase 5: Testing & Deployment (Week 8)
1. Unit testing dengan Vitest
2. E2E testing dengan Playwright
3. Optimasi performa
4. Deployment ke production

## 8. Validasi Form dengan Zod

```typescript
// lib/validations.ts
import { z } from 'zod'

export const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6)
})

export const pelangganSchema = z.object({
  namaPelanggan: z.string().min(1, 'Nama pelanggan wajib diisi'),
  alamat: z.string().optional(),
  nomorHp: z.string().regex(/^[0-9]{10,13}$/, 'Nomor HP tidak valid')
})

export const pesananSchema = z.object({
  idPelanggan: z.string().uuid(),
  jenisLayanan: z.enum(['express', 'regular']),
  beratCucian: z.number().positive('Berat cucian harus lebih dari 0'),
  catatan: z.string().optional()
})

export const pegawaiSchema = z.object({
  nama: z.string().min(1, 'Nama pegawai wajib diisi'),
  username: z.string().min(3, 'Username minimal 3 karakter'),
  password: z.string().min(6, 'Password minimal 6 karakter'),
  jabatan: z.string().min(1, 'Jabatan wajib diisi'),
  nomorHp: z.string().regex(/^[0-9]{10,13}$/, 'Nomor HP tidak valid')
})
```

## 9. Komponen UI Reusable dengan shadcn/ui

```typescript
// components/ui/Button.tsx
import { forwardRef } from 'react'
import { cn } from '@/lib/utils'

const Button = forwardRef(({ className, variant, size, ...props }, ref) => {
  return (
    <button
      className={cn(
        'inline-flex items-center justify-center rounded-md text-sm font-medium',
        'disabled:opacity-50 disabled:pointer-events-none',
        {
          'bg-blue-600 text-white hover:bg-blue-700': variant === 'default',
          'bg-gray-200 text-gray-900 hover:bg-gray-300': variant === 'secondary',
        },
        {
          'h-10 py-2 px-4': size === 'default',
          'h-9 px-3 rounded-md': size === 'sm',
        },
        className
      )}
      ref={ref}
      {...props}
    />
  )
})

Button.displayName = 'Button'

export { Button }
```

## 10. Kesimpulan

Rencana implementasi ini menyediakan kerangka kerja yang komprehensif untuk membangun Sistem Manajemen Laundry yang sesuai dengan spesifikasi yang diberikan. Dengan menggunakan teknologi modern dan arsitektur yang terstruktur, sistem ini akan memberikan:

1. **User Experience yang Optimal**: Interface yang intuitif dan responsif
2. **Keamanan Data**: Role-based access control dan validasi data
3. **Skalabilitas**: Arsitektur yang memungkinkan pengembangan lebih lanjut
4. **Performa yang Baik**: Server-side rendering dan optimasi database
5. **Maintenance yang Mudah**: TypeScript dan kode yang terstruktur dengan baik

Implementasi mengikuti persis spesifikasi DAFTAR SCREEN SISTEM INFORMASI MANAJEMEN LAUNDRY BERBASIS WEB dengan mempertahankan semua istilah dan fitur yang disebutkan tanpa perubahan.