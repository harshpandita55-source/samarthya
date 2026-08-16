# samarthya
website for disable
#!/bin/bash
set -e

echo "Creating Samarthya project directories..."
mkdir -p prisma src/lib src/components src/app/dashboard src/app/strategy src/app/isl-hub/[id] src/app/jobs/[id] src/app/employers/signup src/app/employers/post-job src/app/api/stats src/app/api/courses/[id] src/app/api/enrollments src/app/api/jobs/[id] src/app/api/applications src/app/api/employers

# -------------------------------------------------------------
# 1. Root Config Files
# -------------------------------------------------------------

cat << 'EOF' > package.json
{
  "name": "samarthya",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "db:push": "prisma db push",
    "db:seed": "node --loader ts-node/esm prisma/seed.ts"
  },
  "dependencies": {
    "@prisma/client": "^6.4.1",
    "lucide-react": "^0.475.0",
    "next": "15.1.7",
    "prisma": "^6.4.1",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@tailwindcss/postcss": "^4.0.0",
    "@types/node": "^20.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "eslint": "^9.0.0",
    "eslint-config-next": "15.1.7",
    "tailwindcss": "^4.0.0",
    "typescript": "^5.0.0"
  }
}
EOF

cat << 'EOF' > tsconfig.json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
EOF

cat << 'EOF' > next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  reactStrictMode: true,
};

export default nextConfig;
EOF

cat << 'EOF' > postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
EOF

cat << 'EOF' > prisma.config.ts
import path from 'node:path'
import { defineConfig } from 'prisma/config'

export default defineConfig({
  earlyAccess: true,
  schema: path.join(__dirname, 'prisma', 'schema.prisma'),
  migrate: {
    async url() {
      return `file:${path.join(__dirname, 'prisma', 'dev.db')}`
    },
  },
})
EOF

# -------------------------------------------------------------
# 2. Prisma Database Schema & Seed Script
# -------------------------------------------------------------

cat << 'EOF' > prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

model User {
  id             Int          @id @default(autoincrement())
  name           String
  email          String       @unique
  role           String       // CANDIDATE, EMPLOYER, LEARNER, ADMIN
  organization   String?
  city           String?
  tier           String?      // TIER1, TIER2, TIER3, RURAL
  disabilityType String?      // HEARING, VISUAL, MOTOR, COGNITIVE, NONE
  createdAt      DateTime     @default(now())
  enrollments    Enrollment[]
}

model ISLCourse {
  id          Int          @id @default(autoincrement())
  title       String
  description String
  sector      String       // HEALTHCARE, EDUCATION, PUBLIC_SERVICE, WORKPLACE
  level       String       // BEGINNER, INTERMEDIATE, ADVANCED
  duration    String       // e.g. "4 weeks"
  modules     Int          @default(0)
  language    String       @default("Hindi")
  imageUrl    String?
  enrollments Enrollment[]
  createdAt   DateTime     @default(now())
}

model Enrollment {
  id        Int       @id @default(autoincrement())
  userId    Int
  courseId  Int
  progress  Int       @default(0)
  certified Boolean   @default(false)
  user      User      @relation(fields: [userId], references: [id])
  course    ISLCourse @relation(fields: [courseId], references: [id])
  createdAt DateTime  @default(now())

  @@unique([userId, courseId])
}

model JobListing {
  id              Int           @id @default(autoincrement())
  title           String
  company         String
  location        String
  city            String
  tier            String        // TIER1, TIER2, TIER3
  type            String        // FULL_TIME, PART_TIME, INTERNSHIP, REMOTE
  sector          String
  description     String
  salary          String?
  accommodations  String        // Comma-separated: WHEELCHAIR,SCREEN_READER,ISL_INTERPRETER,FLEXIBLE_HOURS
  disabilityTypes String        // Comma-separated disability types welcome
  islCertified    Boolean       @default(false)
  isActive        Boolean       @default(true)
  postedAt        DateTime      @default(now())
  applications    Application[]
}

model Application {
  id                   Int        @id @default(autoincrement())
  jobId                Int
  job                  JobListing @relation(fields: [jobId], references: [id])
  name                 String
  email                String
  phone                String?
  city                 String?
  disabilityType       String?
  accommodationsNeeded String?
  coverNote            String?
  status               String     @default("SUBMITTED") // SUBMITTED, REVIEWED, SHORTLISTED, REJECTED, HIRED
  createdAt            DateTime   @default(now())
}

model Employer {
  id          Int      @id @default(autoincrement())
  companyName String
  contactName String
  email       String   @unique
  phone       String?
  city        String
  tier        String   // TIER1, TIER2, TIER3
  sector      String
  website     String?
  description String?
  status      String   @default("PENDING") // PENDING, VERIFIED
  createdAt   DateTime @default(now())
}

model Partner {
  id          Int      @id @default(autoincrement())
  name        String
  type        String   // NGO, CORPORATE, GOVERNMENT, EDUCATION
  description String
  city        String
  tier        String
  logoUrl     String?
  website     String?
  createdAt   DateTime @default(now())
}
EOF

cat << 'EOF' > prisma/seed.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  await prisma.application.deleteMany()
  await prisma.enrollment.deleteMany()
  await prisma.jobListing.deleteMany()
  await prisma.islCourse.deleteMany()
  await prisma.user.deleteMany()
  await prisma.employer.deleteMany()
  await prisma.partner.deleteMany()

  const courses = await Promise.all([
    prisma.islCourse.create({
      data: {
        title: 'ISL for Healthcare Professionals',
        description: 'Learn essential ISL signs for medical consultations, emergency triage, patient intake, and pharmacy interactions. Designed for doctors, nurses, and hospital staff.',
        sector: 'HEALTHCARE',
        level: 'BEGINNER',
        duration: '4 weeks',
        modules: 12,
        language: 'Hindi',
        imageUrl: '/images/healthcare.svg',
      },
    }),
    prisma.islCourse.create({
      data: {
        title: 'ISL for Bank & Financial Services',
        description: 'Master ISL vocabulary for account opening, loan processing, ATM assistance, and customer queries. Essential for bank tellers and managers.',
        sector: 'PUBLIC_SERVICE',
        level: 'BEGINNER',
        duration: '3 weeks',
        modules: 9,
        language: 'Hindi',
        imageUrl: '/images/banking.svg',
      },
    }),
    prisma.islCourse.create({
      data: {
        title: 'ISL for Classroom Teachers',
        description: 'Learn to create inclusive classrooms with ISL basics, classroom management signs, subject-specific vocabulary, and parent communication.',
        sector: 'EDUCATION',
        level: 'BEGINNER',
        duration: '6 weeks',
        modules: 18,
        language: 'Hindi',
        imageUrl: '/images/education.svg',
      },
    }),
    prisma.islCourse.create({
      data: {
        title: 'ISL for Corporate Workplaces',
        description: 'Build inclusive teams with ISL for meetings, onboarding, HR interactions, and daily office communication. Includes DEI compliance modules.',
        sector: 'WORKPLACE',
        level: 'BEGINNER',
        duration: '4 weeks',
        modules: 10,
        language: 'Hindi',
        imageUrl: '/images/workplace.svg',
      },
    }),
    prisma.islCourse.create({
      data: {
        title: 'ISL for Court & Legal Services',
        description: 'Essential legal ISL vocabulary for court proceedings, FIR filing, legal aid consultations, and rights communication.',
        sector: 'PUBLIC_SERVICE',
        level: 'INTERMEDIATE',
        duration: '5 weeks',
        modules: 15,
        language: 'Hindi',
        imageUrl: '/images/legal.svg',
      },
    }),
    prisma.islCourse.create({
      data: {
        title: 'Advanced ISL Interpreter Training',
        description: 'Professional-level ISL interpreter certification covering simultaneous interpretation, ethics, and specialized domains.',
        sector: 'WORKPLACE',
        level: 'ADVANCED',
        duration: '12 weeks',
        modules: 30,
        language: 'Hindi',
        imageUrl: '/images/advanced.svg',
      },
    }),
  ])

  await Promise.all([
    prisma.jobListing.create({
      data: {
        title: 'Customer Service Associate',
        company: 'State Bank of India',
        location: 'Lucknow Branch',
        city: 'Lucknow',
        tier: 'TIER2',
        type: 'FULL_TIME',
        sector: 'Banking',
        description: 'Handle customer queries, account services, and documentation. Training provided for assistive technology.',
        salary: '₹18,000 - ₹25,000/month',
        accommodations: 'WHEELCHAIR,SCREEN_READER,FLEXIBLE_HOURS',
        disabilityTypes: 'MOTOR,VISUAL',
        islCertified: true,
      },
    }),
    prisma.jobListing.create({
      data: {
        title: 'Data Entry Operator',
        company: 'Tata Consultancy Services',
        location: 'Jaipur Office',
        city: 'Jaipur',
        tier: 'TIER2',
        type: 'FULL_TIME',
        sector: 'IT Services',
        description: 'Data processing, verification, and report generation. Fully accessible workstation provided.',
        salary: '₹15,000 - ₹22,000/month',
        accommodations: 'WHEELCHAIR,SCREEN_READER,ISL_INTERPRETER',
        disabilityTypes: 'HEARING,MOTOR,VISUAL',
        islCertified: true,
      },
    }),
    prisma.jobListing.create({
      data: {
        title: 'Teaching Assistant',
        company: 'Kendriya Vidyalaya',
        location: 'Bhopal',
        city: 'Bhopal',
        tier: 'TIER2',
        type: 'FULL_TIME',
        sector: 'Education',
        description: 'Support classroom teaching, manage student records, and assist with inclusive education programs.',
        salary: '₹12,000 - ₹18,000/month',
        accommodations: 'ISL_INTERPRETER,FLEXIBLE_HOURS',
        disabilityTypes: 'HEARING,MOTOR',
        islCertified: false,
      },
    }),
    prisma.jobListing.create({
      data: {
        title: 'Graphic Designer (Remote)',
        company: 'Zoho Corporation',
        location: 'Remote',
        city: 'Chennai',
        tier: 'TIER1',
        type: 'REMOTE',
        sector: 'Technology',
        description: 'Create UI designs, marketing materials, and brand assets. Fully remote with flexible schedule.',
        salary: '₹30,000 - ₹45,000/month',
        accommodations: 'SCREEN_READER,FLEXIBLE_HOURS',
        disabilityTypes: 'HEARING,MOTOR,VISUAL',
        islCertified: false,
      },
    }),
    prisma.jobListing.create({
      data: {
        title: 'Pharmacy Assistant',
        company: 'Apollo Pharmacy',
        location: 'Indore',
        city: 'Indore',
        tier: 'TIER2',
        type: 'FULL_TIME',
        sector: 'Healthcare',
        description: 'Assist pharmacists with inventory, billing, and customer service. ISL-trained staff on premises.',
        salary: '₹14,000 - ₹20,000/month',
        accommodations: 'ISL_INTERPRETER,WHEELCHAIR',
        disabilityTypes: 'HEARING,MOTOR',
        islCertified: true,
      },
    }),
    prisma.jobListing.create({
      data: {
        title: 'Warehouse Associate',
        company: 'Flipkart',
        location: 'Patna Warehouse',
        city: 'Patna',
        tier: 'TIER2',
        type: 'FULL_TIME',
        sector: 'E-Commerce',
        description: 'Pick, pack, and dispatch operations. Accessible workstations and visual task guidance provided.',
        salary: '₹16,000 - ₹21,000/month',
        accommodations: 'WHEELCHAIR,FLEXIBLE_HOURS',
        disabilityTypes: 'HEARING,COGNITIVE',
        islCertified: false,
      },
    }),
  ])

  await Promise.all([
    prisma.partner.create({
      data: {
        name: 'National Association of the Deaf (NAD)',
        type: 'NGO',
        description: 'Apex body for deaf advocacy in India, providing ISL resources and interpreter training.',
        city: 'New Delhi',
        tier: 'TIER1',
        website: 'https://nadfindia.org',
      },
    }),
    prisma.partner.create({
      data: {
        name: 'Indian Sign Language Research & Training Centre (ISLRTC)',
        type: 'GOVERNMENT',
        description: 'Government body developing ISL curriculum and standardization.',
        city: 'New Delhi',
        tier: 'TIER1',
        website: 'https://islrtc.nic.in',
      },
    }),
  ])

  const users = await Promise.all([
    prisma.user.create({
      data: {
        name: 'Dr. Priya Sharma',
        email: 'priya@hospital.org',
        role: 'LEARNER',
        organization: 'District Hospital Jaipur',
        city: 'Jaipur',
        tier: 'TIER2',
        disabilityType: 'NONE',
      },
    }),
    prisma.user.create({
      data: {
        name: 'Rahul Verma',
        email: 'rahul@example.com',
        role: 'CANDIDATE',
        city: 'Lucknow',
        tier: 'TIER2',
        disabilityType: 'HEARING',
      },
    }),
  ])

  await Promise.all([
    prisma.enrollment.create({
      data: { userId: users[0].id, courseId: courses[0].id, progress: 75, certified: false },
    }),
    prisma.enrollment.create({
      data: { userId: users[0].id, courseId: courses[3].id, progress: 100, certified: true },
    }),
  ])

  console.log('Database seeded successfully.')
}

main()
  .catch((e) => {
    console.error(e)
    process.exit(1)
  })
  .finally(async () => {
    await prisma.$disconnect()
  })
EOF

# -------------------------------------------------------------
# 3. Lib & Components
# -------------------------------------------------------------

cat << 'EOF' > src/lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  })

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
EOF

cat << 'EOF' > src/components/Navbar.tsx
'use client'

import { useState } from 'react'
import Link from 'next/link'
import { usePathname } from 'next/navigation'
import { HandMetal, Briefcase, BarChart3, Compass, Menu, X, Building2 } from 'lucide-react'

const navLinks = [
  { href: '/isl-hub', label: 'ISL Hub', icon: HandMetal },
  { href: '/jobs', label: 'Jobs', icon: Briefcase },
  { href: '/strategy', label: 'Strategy & Scale', icon: Compass },
  { href: '/dashboard', label: 'Dashboard', icon: BarChart3 },
]

export default function Navbar() {
  const [open, setOpen] = useState(false)
  const pathname = usePathname()

  return (
    <header className="sticky top-0 z-50 bg-white/90 backdrop-blur-md border-b border-gray-200">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          <Link href="/" className="flex items-center gap-2.5">
            <div className="w-9 h-9 rounded-xl bg-indigo-600 flex items-center justify-center text-white font-bold text-lg shadow-sm">
              स
            </div>
            <div className="flex flex-col">
              <span className="font-bold text-gray-900 leading-none text-lg tracking-tight">
                Samarthya
              </span>
              <span className="text-[10px] text-indigo-600 font-medium tracking-wide uppercase">
                Inclusive India
              </span>
            </div>
          </Link>

          <nav className="hidden md:flex items-center gap-1">
            {navLinks.map(({ href, label, icon: Icon }) => {
              const active = pathname === href || pathname.startsWith(`${href}/`)
              return (
                <Link
                  key={href}
                  href={href}
                  className={`flex items-center gap-1.5 px-3.5 py-2 rounded-lg text-sm font-medium transition-colors ${
                    active
                      ? 'bg-indigo-50 text-indigo-700'
                      : 'text-gray-600 hover:text-gray-900 hover:bg-gray-100'
                  }`}
                >
                  <Icon className="w-4 h-4" />
                  {label}
                </Link>
              )
            })}
          </nav>

          <div className="hidden md:flex items-center gap-3">
            <Link
              href="/employers/signup"
              className="flex items-center gap-1.5 text-xs font-semibold px-3.5 py-2 rounded-lg border border-gray-300 text-gray-700 hover:bg-gray-50 transition-colors"
            >
              <Building2 className="w-3.5 h-3.5" />
              For Employers
            </Link>
            <Link
              href="/jobs"
              className="text-xs font-semibold px-3.5 py-2 rounded-lg bg-indigo-600 text-white hover:bg-indigo-700 transition-colors shadow-sm"
            >
              Find Inclusive Work
            </Link>
          </div>

          <button
            onClick={() => setOpen(!open)}
            aria-label="Toggle menu"
            className="md:hidden p-2 rounded-lg text-gray-600 hover:bg-gray-100"
          >
            {open ? <X className="w-5 h-5" /> : <Menu className="w-5 h-5" />}
          </button>
        </div>
      </div>

      {open && (
        <div className="md:hidden border-b border-gray-200 bg-white px-4 pt-2 pb-4 space-y-1">
          {navLinks.map(({ href, label, icon: Icon }) => {
            const active = pathname === href || pathname.startsWith(`${href}/`)
            return (
              <Link
                key={href}
                href={href}
                onClick={() => setOpen(false)}
                className={`flex items-center gap-2.5 px-3 py-2.5 rounded-lg text-sm font-medium ${
                  active ? 'bg-indigo-50 text-indigo-700' : 'text-gray-700 hover:bg-gray-50'
                }`}
              >
                <Icon className="w-4 h-4" />
                {label}
              </Link>
            )
          })}
          <div className="pt-3 flex flex-col gap-2">
            <Link
              href="/employers/signup"
              onClick={() => setOpen(false)}
              className="text-center text-sm font-medium px-3 py-2 rounded-lg border border-gray-300 text-gray-700"
            >
              For Employers
            </Link>
            <Link
              href="/jobs"
              onClick={() => setOpen(false)}
              className="text-center text-sm font-medium px-3 py-2 rounded-lg bg-indigo-600 text-white"
            >
              Find Inclusive Work
            </Link>
          </div>
        </div>
      )}
    </header>
  )
}
EOF

cat << 'EOF' > src/components/Footer.tsx
import Link from 'next/link'
import { Heart } from 'lucide-react'

export default function Footer() {
  return (
    <footer className="bg-gray-900 text-gray-400 border-t border-gray-800">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-8 mb-8">
          <div className="md:col-span-1">
            <div className="flex items-center gap-2 mb-3">
              <div className="w-7 h-7 rounded-lg bg-indigo-500 flex items-center justify-center text-white font-bold text-sm">
                स
              </div>
              <span className="font-bold text-white text-lg">Samarthya</span>
            </div>
            <p className="text-xs leading-relaxed text-gray-400">
              National Disability Inclusion & Indian Sign Language Mainstreaming Platform.
              Built under the RPwD Act 2016 framework targeting Tier 2/3 cities.
            </p>
          </div>

          <div>
            <h4 className="text-xs font-semibold text-gray-200 uppercase tracking-wider mb-3">
              ISL Learning
            </h4>
            <ul className="text-xs space-y-2">
              <li><Link href="/isl-hub" className="hover:text-white transition-colors">Healthcare ISL</Link></li>
              <li><Link href="/isl-hub" className="hover:text-white transition-colors">Banking & Finance ISL</Link></li>
              <li><Link href="/isl-hub" className="hover:text-white transition-colors">Education & Schools</Link></li>
              <li><Link href="/isl-hub" className="hover:text-white transition-colors">Legal Aid ISL</Link></li>
            </ul>
          </div>

          <div>
            <h4 className="text-xs font-semibold text-gray-200 uppercase tracking-wider mb-3">
              Inclusive Jobs
            </h4>
            <ul className="text-xs space-y-2">
              <li><Link href="/jobs?tier=TIER2" className="hover:text-white transition-colors">Tier 2 City Jobs</Link></li>
              <li><Link href="/jobs?tier=TIER3" className="hover:text-white transition-colors">Tier 3 City Jobs</Link></li>
              <li><Link href="/jobs?type=REMOTE" className="hover:text-white transition-colors">Remote Opportunities</Link></li>
              <li><Link href="/employers/signup" className="hover:text-white transition-colors">Employer Network</Link></li>
            </ul>
          </div>

          <div>
            <h4 className="text-xs font-semibold text-gray-200 uppercase tracking-wider mb-3">
              Scale & Impact
            </h4>
            <ul className="text-xs space-y-2">
              <li><Link href="/strategy" className="hover:text-white transition-colors">CSC Delivery Model</Link></li>
              <li><Link href="/strategy" className="hover:text-white transition-colors">Offline-First Tech</Link></li>
              <li><Link href="/dashboard" className="hover:text-white transition-colors">Live Impact Dashboard</Link></li>
            </ul>
          </div>
        </div>

        <div className="pt-8 border-t border-gray-800 flex flex-col sm:flex-row items-center justify-between gap-4 text-xs">
          <p>© {new Date().getFullYear()} Samarthya. Built for inclusive empowerment.</p>
          <p className="flex items-center gap-1">
            Mainstreaming ISL & Equal Livelihoods <Heart className="w-3.5 h-3.5 text-rose-500 fill-rose-500" />
          </p>
        </div>
      </div>
    </footer>
  )
}
EOF

cat << 'EOF' > src/components/StatCard.tsx
import { LucideIcon } from 'lucide-react'

interface StatCardProps {
  icon: LucideIcon
  value: string
  label: string
  description?: string
  color?: 'indigo' | 'emerald' | 'purple' | 'amber' | 'rose'
}

const colorMap = {
  indigo: 'bg-indigo-50 text-indigo-600 border-indigo-100',
  emerald: 'bg-emerald-50 text-emerald-600 border-emerald-100',
  purple: 'bg-purple-50 text-purple-600 border-purple-100',
  amber: 'bg-amber-50 text-amber-600 border-amber-100',
  rose: 'bg-rose-50 text-rose-600 border-rose-100',
}

export default function StatCard({
  icon: Icon,
  value,
  label,
  description,
  color = 'indigo',
}: StatCardProps) {
  return (
    <div className="bg-white border border-gray-200 rounded-xl p-5 shadow-sm hover:shadow-md transition-shadow">
      <div className="flex items-center justify-between mb-3">
        <span className="text-xs font-semibold text-gray-500 uppercase tracking-wider">{label}</span>
        <div className={`p-2 rounded-lg border ${colorMap[color]}`}>
          <Icon className="w-5 h-5" />
        </div>
      </div>
      <div className="text-3xl font-bold text-gray-900 mb-1">{value}</div>
      {description && <p className="text-xs text-gray-500">{description}</p>}
    </div>
  )
}
EOF

cat << 'EOF' > src/components/CourseCard.tsx
import Link from 'next/link'
import { Clock, BookOpen, Users, ArrowRight } from 'lucide-react'

interface CourseCardProps {
  id: number
  title: string
  description: string
  sector: string
  level: string
  duration: string
  modules: number
  enrollmentCount?: number
}

const sectorBadgeColors: Record<string, string> = {
  HEALTHCARE: 'bg-rose-100 text-rose-700 border-rose-200',
  EDUCATION: 'bg-blue-100 text-blue-700 border-blue-200',
  PUBLIC_SERVICE: 'bg-amber-100 text-amber-700 border-amber-200',
  WORKPLACE: 'bg-purple-100 text-purple-700 border-purple-200',
}

const levelBadgeColors: Record<string, string> = {
  BEGINNER: 'bg-emerald-50 text-emerald-700 border-emerald-200',
  INTERMEDIATE: 'bg-amber-50 text-amber-700 border-amber-200',
  ADVANCED: 'bg-red-50 text-red-700 border-red-200',
}

export default function CourseCard({
  id,
  title,
  description,
  sector,
  level,
  duration,
  modules,
  enrollmentCount = 0,
}: CourseCardProps) {
  return (
    <div className="bg-white border border-gray-200 rounded-xl overflow-hidden shadow-sm hover:shadow-md transition-all flex flex-col justify-between">
      <div className="p-6">
        <div className="flex items-center justify-between gap-2 mb-3">
          <span className={`text-[11px] font-semibold px-2.5 py-0.5 rounded-full border ${sectorBadgeColors[sector] || 'bg-gray-100 text-gray-700'}`}>
            {sector.replace('_', ' ')}
          </span>
          <span className={`text-[11px] font-medium px-2 py-0.5 rounded-md border ${levelBadgeColors[level] || 'bg-gray-50 text-gray-600'}`}>
            {level}
          </span>
        </div>

        <h3 className="text-base font-bold text-gray-900 mb-2 line-clamp-1">{title}</h3>
        <p className="text-xs text-gray-600 leading-relaxed line-clamp-3 mb-4">{description}</p>

        <div className="flex items-center gap-4 text-xs text-gray-500 pt-3 border-t border-gray-100">
          <span className="flex items-center gap-1">
            <Clock className="w-3.5 h-3.5 text-gray-400" />
            {duration}
          </span>
          <span className="flex items-center gap-1">
            <BookOpen className="w-3.5 h-3.5 text-gray-400" />
            {modules} modules
          </span>
          <span className="flex items-center gap-1">
            <Users className="w-3.5 h-3.5 text-gray-400" />
            {enrollmentCount}
          </span>
        </div>
      </div>

      <div className="px-6 py-3 bg-gray-50 border-t border-gray-100">
        <Link
          href={`/isl-hub/${id}`}
          className="flex items-center justify-between text-xs font-semibold text-indigo-600 hover:text-indigo-700"
        >
          View Course & Enroll
          <ArrowRight className="w-3.5 h-3.5" />
        </Link>
      </div>
    </div>
  )
}
EOF

cat << 'EOF' > src/components/JobCard.tsx
import Link from 'next/link'
import { MapPin, Building2, Shield, IndianRupee, ArrowRight } from 'lucide-react'

interface JobCardProps {
  id: number
  title: string
  company: string
  city: string
  tier: string
  type: string
  sector: string
  description: string
  salary?: string | null
  accommodations: string
  islCertified: boolean
}

const accommodationLabels: Record<string, string> = {
  WHEELCHAIR: 'Wheelchair',
  SCREEN_READER: 'Screen Reader',
  ISL_INTERPRETER: 'ISL Interpreter',
  FLEXIBLE_HOURS: 'Flexible Hours',
}

export default function JobCard({
  id,
  title,
  company,
  city,
  tier,
  type,
  sector,
  description,
  salary,
  accommodations,
  islCertified,
}: JobCardProps) {
  const accomList = accommodations.split(',').map((a) => a.trim()).filter(Boolean)

  return (
    <div className="bg-white border border-gray-200 rounded-xl p-6 shadow-sm hover:shadow-md transition-all flex flex-col justify-between">
      <div>
        <div className="flex items-start justify-between gap-3 mb-2">
          <div>
            <h3 className="text-base font-bold text-gray-900 leading-snug">{title}</h3>
            <p className="text-xs text-gray-600 flex items-center gap-1 mt-0.5">
              <Building2 className="w-3.5 h-3.5 text-gray-400" />
              {company} • <span className="text-gray-500">{sector}</span>
            </p>
          </div>
          {islCertified && (
            <span className="flex items-center gap-1 px-2 py-0.5 bg-emerald-50 border border-emerald-200 rounded text-[10px] font-semibold text-emerald-700 shrink-0">
              <Shield className="w-3 h-3" />
              ISL Certified
            </span>
          )}
        </div>

        <div className="flex flex-wrap gap-2 text-xs text-gray-600 my-3">
          <span className="flex items-center gap-1 bg-gray-100 px-2 py-0.5 rounded">
            <MapPin className="w-3 h-3 text-gray-500" />
            {city} ({tier.replace('TIER', 'Tier ')})
          </span>
          <span className="bg-gray-100 px-2 py-0.5 rounded">{type.replace('_', ' ')}</span>
          {salary && (
            <span className="flex items-center gap-0.5 bg-emerald-50 text-emerald-700 px-2 py-0.5 rounded font-medium">
              <IndianRupee className="w-3 h-3" />
              {salary.replace('₹', '')}
            </span>
          )}
        </div>

        <p className="text-xs text-gray-600 leading-relaxed line-clamp-2 mb-4">{description}</p>

        <div className="space-y-1 mb-4">
          <span className="text-[10px] font-semibold text-gray-400 uppercase tracking-wider block">Accommodations</span>
          <div className="flex flex-wrap gap-1.5">
            {accomList.map((a) => (
              <span key={a} className="text-[11px] bg-indigo-50 text-indigo-700 px-2 py-0.5 rounded-full border border-indigo-100">
                {accommodationLabels[a] || a}
              </span>
            ))}
          </div>
        </div>
      </div>

      <div className="pt-3 border-t border-gray-100 flex items-center justify-end">
        <Link
          href={`/jobs/${id}`}
          className="inline-flex items-center gap-1 text-xs font-semibold text-emerald-600 hover:text-emerald-700"
        >
          View Details & Apply
          <ArrowRight className="w-3.5 h-3.5" />
        </Link>
      </div>
    </div>
  )
}
EOF

# -------------------------------------------------------------
# 4. App Pages
# -------------------------------------------------------------

cat << 'EOF' > src/app/globals.css
@import "tailwindcss";

:root {
  --background: #ffffff;
  --foreground: #171717;
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}

html {
  scroll-behavior: smooth;
  color-scheme: light;
}

.animate-fade-in {
  animation: fadeIn 0.4s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
EOF

cat << 'EOF' > src/app/layout.tsx
import type { Metadata } from 'next'
import './globals.css'
import Navbar from '@/components/Navbar'
import Footer from '@/components/Footer'

export const metadata: Metadata = {
  title: 'Samarthya — Inclusive India Platform',
  description:
    'Mainstreaming Indian Sign Language and building inclusive employment pipelines for persons with disabilities across India.',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className="antialiased bg-gray-50 text-gray-900 flex flex-col min-h-screen">
        <Navbar />
        <main className="flex-grow">{children}</main>
        <Footer />
      </body>
    </html>
  )
}
EOF

cat << 'EOF' > src/app/page.tsx
import Link from 'next/link'
import {
  HandMetal,
  Briefcase,
  ArrowRight,
  CheckCircle2,
} from 'lucide-react'

export default function HomePage() {
  return (
    <div className="animate-fade-in">
      <section className="relative overflow-hidden bg-gradient-to-br from-indigo-600 via-purple-600 to-indigo-800 text-white">
        <div className="relative max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-20 md:py-28">
          <div className="max-w-3xl">
            <div className="flex items-center gap-2 mb-6 flex-wrap">
              <span className="px-3 py-1 bg-white/20 backdrop-blur-sm rounded-full text-xs font-medium">
                Inclusion for Tier 2/3 India
              </span>
              <span className="px-3 py-1 bg-white/20 backdrop-blur-sm rounded-full text-xs font-medium">
                RPwD Act 2016 Compliant
              </span>
            </div>
            <h1 className="text-4xl md:text-6xl font-bold leading-tight mb-6">
              Making India
              <span className="block text-yellow-300">Truly Inclusive</span>
            </h1>
            <p className="text-lg md:text-xl text-indigo-100 mb-8 max-w-2xl leading-relaxed">
              Samarthya bridges the divide for over 18 million deaf and hard-of-hearing citizens by mainstreaming standardized Indian Sign Language (ISL) across critical sectors while creating accessible job pipelines outside metro cities.
            </p>
            <div className="flex flex-wrap gap-4">
              <Link
                href="/isl-hub"
                className="inline-flex items-center gap-2 px-6 py-3 bg-white text-indigo-700 rounded-lg font-semibold hover:bg-indigo-50 transition-colors shadow-lg text-sm"
              >
                <HandMetal className="w-4 h-4" />
                Explore ISL Courses
              </Link>
              <Link
                href="/jobs"
                className="inline-flex items-center gap-2 px-6 py-3 bg-indigo-900/40 text-white border border-white/30 rounded-lg font-semibold hover:bg-indigo-900/60 transition-colors text-sm"
              >
                <Briefcase className="w-4 h-4" />
                Inclusive Job Board
              </Link>
            </div>
          </div>
        </div>
      </section>

      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-16">
        <div className="text-center max-w-3xl mx-auto mb-12">
          <h2 className="text-3xl font-bold text-gray-900 mb-3">Our Two-Pillar Model</h2>
          <p className="text-gray-600 text-sm md:text-base">
            True inclusion requires institutional communication accessibility combined with sustainable livelihood pipelines.
          </p>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
          <div className="bg-gradient-to-br from-indigo-50 to-purple-50 border border-indigo-100 rounded-2xl p-8 flex flex-col justify-between">
            <div>
              <div className="w-12 h-12 rounded-xl bg-indigo-600 text-white flex items-center justify-center mb-5 shadow-sm">
                <HandMetal className="w-6 h-6" />
              </div>
              <span className="text-xs font-semibold text-indigo-600 uppercase tracking-wider">Pillar I</span>
              <h3 className="text-2xl font-bold text-gray-900 mt-1 mb-3">ISL Sectoral Mainstreaming</h3>
              <p className="text-sm text-gray-600 leading-relaxed mb-6">
                Modular, sector-tailored Indian Sign Language modules for healthcare staff, bank tellers, and educators, standardized to ISLRTC benchmarks.
              </p>
              <ul className="space-y-2 text-xs text-gray-700 mb-6">
                <li className="flex items-center gap-2"><CheckCircle2 className="w-4 h-4 text-indigo-600" /> Medical triage and emergency intake signs</li>
                <li className="flex items-center gap-2"><CheckCircle2 className="w-4 h-4 text-indigo-600" /> Banking, KYC, and loan documentation modules</li>
                <li className="flex items-center gap-2"><CheckCircle2 className="w-4 h-4 text-indigo-600" /> Common Service Centre (CSC) frontline training</li>
              </ul>
            </div>
            <Link href="/isl-hub" className="inline-flex items-center gap-1.5 text-sm font-semibold text-indigo-700 hover:text-indigo-800">
              Browse ISL Hub <ArrowRight className="w-4 h-4" />
            </Link>
          </div>

          <div className="bg-gradient-to-br from-emerald-50 to-teal-50 border border-emerald-100 rounded-2xl p-8 flex flex-col justify-between">
            <div>
              <div className="w-12 h-12 rounded-xl bg-emerald-600 text-white flex items-center justify-center mb-5 shadow-sm">
                <Briefcase className="w-6 h-6" />
              </div>
              <span className="text-xs font-semibold text-emerald-600 uppercase tracking-wider">Pillar II</span>
              <h3 className="text-2xl font-bold text-gray-900 mt-1 mb-3">Tier 2/3 Inclusive Employment</h3>
              <p className="text-sm text-gray-600 leading-relaxed mb-6">
                Matching with verified employers providing accommodations alongside clear guidance on tax deductions under Section 80DD and 80U.
              </p>
              <ul className="space-y-2 text-xs text-gray-700 mb-6">
                <li className="flex items-center gap-2"><CheckCircle2 className="w-4 h-4 text-emerald-600" /> Accommodation-first job filters</li>
                <li className="flex items-center gap-2"><CheckCircle2 className="w-4 h-4 text-emerald-600" /> SME inclusion onboarding blueprints</li>
                <li className="flex items-center gap-2"><CheckCircle2 className="w-4 h-4 text-emerald-600" /> Retention tracking and buddy mentorship frameworks</li>
              </ul>
            </div>
            <Link href="/jobs" className="inline-flex items-center gap-1.5 text-sm font-semibold text-emerald-700 hover:text-emerald-800">
              Explore Job Listings <ArrowRight className="w-4 h-4" />
            </Link>
          </div>
        </div>
      </section>

      <section className="bg-white border-y border-gray-200 py-14">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8 text-center">
            <div className="p-6">
              <div className="text-4xl font-extrabold text-indigo-600 mb-2">18M+</div>
              <div className="text-sm font-bold text-gray-900 mb-1">Deaf & Hard-of-Hearing Citizens</div>
              <p className="text-xs text-gray-500">Addressing fundamental communication barriers in critical public services.</p>
            </div>
            <div className="p-6 border-y md:border-y-0 md:border-x border-gray-200">
              <div className="text-4xl font-extrabold text-rose-600 mb-2">&lt; 35%</div>
              <div className="text-sm font-bold text-gray-900 mb-1">Employment Rate for PwDs</div>
              <p className="text-xs text-gray-500">Unlocking employment pathways beyond primary tier-1 metros.</p>
            </div>
            <div className="p-6">
              <div className="text-4xl font-extrabold text-emerald-600 mb-2">600K+</div>
              <div className="text-sm font-bold text-gray-900 mb-1">CSC Centers in India</div>
              <p className="text-xs text-gray-500">Utilizing grassroot distribution networks for offline-assisted delivery.</p>
            </div>
          </div>
        </div>
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/dashboard/page.tsx
'use client'

import { useState, useEffect } from 'react'
import {
  BarChart3, Users, Briefcase, BookOpen, Award, MapPin,
  TrendingUp, Building2, GraduationCap, Target
} from 'lucide-react'
import StatCard from '@/components/StatCard'

interface Stats {
  totalCourses: number
  totalJobs: number
  totalUsers: number
  totalEnrollments: number
  certifiedCount: number
  tier2Jobs: number
  tier3Jobs: number
  tier2_3Percentage: number
}

export default function DashboardPage() {
  const [stats, setStats] = useState<Stats | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch('/api/stats')
      .then((res) => res.json())
      .then((data) => {
        setStats(data)
        setLoading(false)
      })
      .catch(() => setLoading(false))
  }, [])

  if (loading) {
    return (
      <div className="max-w-7xl mx-auto px-4 py-16">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
          {[1, 2, 3, 4].map((i) => (
            <div key={i} className="h-36 bg-gray-100 rounded-xl animate-pulse" />
          ))}
        </div>
      </div>
    )
  }

  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-violet-600 to-fuchsia-600 text-white py-16">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex items-center gap-3 mb-4">
            <div className="w-12 h-12 bg-white/20 rounded-xl flex items-center justify-center">
              <BarChart3 className="w-6 h-6" />
            </div>
            <h1 className="text-3xl md:text-4xl font-bold">Impact Dashboard</h1>
          </div>
          <p className="text-violet-100 max-w-2xl text-lg">
            Real-time metrics tracking ISL adoption rates and employment outcomes for persons with disabilities.
          </p>
        </div>
      </section>

      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <h2 className="text-2xl font-bold text-gray-900 mb-6">Platform Overview</h2>
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
          <StatCard
            icon={BookOpen}
            value={String(stats?.totalCourses || 0)}
            label="ISL Courses"
            description="Across 4 essential sectors"
            color="indigo"
          />
          <StatCard
            icon={Briefcase}
            value={String(stats?.totalJobs || 0)}
            label="Active Job Listings"
            description="Accessible & verified"
            color="emerald"
          />
          <StatCard
            icon={Users}
            value={String(stats?.totalUsers || 0)}
            label="Registered Users"
            description="Candidates & learners"
            color="purple"
          />
          <StatCard
            icon={Award}
            value={String(stats?.certifiedCount || 0)}
            label="ISL Certifications"
            description="Verified completions"
            color="amber"
          />
        </div>
      </section>

      <section className="bg-white py-12 border-y border-gray-200">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <h2 className="text-2xl font-bold text-gray-900 mb-6">Tier 2/3 City Focus</h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            <div className="p-6 rounded-xl bg-gradient-to-br from-indigo-50 to-purple-50 border border-indigo-200">
              <MapPin className="w-8 h-8 text-indigo-600 mb-3" />
              <div className="text-4xl font-bold text-indigo-600">{stats?.tier2_3Percentage || 0}%</div>
              <div className="text-sm font-semibold text-indigo-700 mt-1">Jobs in Tier 2/3 Cities</div>
              <p className="text-xs text-gray-500 mt-2">Targeting 70%+ non-metro footprint</p>
            </div>
            <div className="p-6 rounded-xl bg-gradient-to-br from-emerald-50 to-teal-50 border border-emerald-200">
              <Building2 className="w-8 h-8 text-emerald-600 mb-3" />
              <div className="text-4xl font-bold text-emerald-600">{stats?.tier2Jobs || 0}</div>
              <div className="text-sm font-semibold text-emerald-700 mt-1">Tier 2 City Listings</div>
              <p className="text-xs text-gray-500 mt-2">Jaipur, Lucknow, Bhopal, Indore</p>
            </div>
            <div className="p-6 rounded-xl bg-gradient-to-br from-amber-50 to-orange-50 border border-amber-200">
              <Target className="w-8 h-8 text-amber-600 mb-3" />
              <div className="text-4xl font-bold text-amber-600">{stats?.tier3Jobs || 0}</div>
              <div className="text-sm font-semibold text-amber-700 mt-1">Tier 3 City Listings</div>
              <p className="text-xs text-gray-500 mt-2">Varanasi, Ranchi, Dehradun</p>
            </div>
          </div>
        </div>
      </section>

      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <h2 className="text-2xl font-bold text-gray-900 mb-6">Skill-to-Job Pipeline Model</h2>
        <div className="bg-white rounded-xl border border-gray-200 p-6">
          <div className="grid grid-cols-1 md:grid-cols-5 gap-4">
            {[
              { icon: Users, title: 'Identify', desc: 'PwD candidate self-assessment', color: 'text-indigo-600' },
              { icon: GraduationCap, title: 'Upskill', desc: 'Sector ISL certification courses', color: 'text-purple-600' },
              { icon: Target, title: 'Match', desc: 'Accommodation-ready matching', color: 'text-emerald-600' },
              { icon: Building2, title: 'Place', desc: 'Interview support & onboarding', color: 'text-amber-600' },
              { icon: TrendingUp, title: 'Retain', desc: 'Buddy mentorship & check-ins', color: 'text-rose-600' },
            ].map((stage, i) => (
              <div key={i} className="text-center p-3 rounded-lg bg-gray-50">
                <stage.icon className={`w-6 h-6 ${stage.color} mx-auto mb-2`} />
                <div className="text-sm font-bold text-gray-900">{stage.title}</div>
                <div className="text-xs text-gray-500 mt-1">{stage.desc}</div>
              </div>
            ))}
          </div>
        </div>
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/strategy/page.tsx
import {
  Compass,
  Network,
  Cpu,
  CheckCircle2,
} from 'lucide-react'

export default function StrategyPage() {
  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-slate-900 to-indigo-950 text-white py-16">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex items-center gap-3 mb-4">
            <div className="w-12 h-12 bg-indigo-500/20 rounded-xl flex items-center justify-center border border-indigo-400/30">
              <Compass className="w-6 h-6 text-indigo-300" />
            </div>
            <h1 className="text-3xl md:text-4xl font-bold">Scalability & Adoption Strategy</h1>
          </div>
          <p className="text-slate-300 max-w-3xl text-base md:text-lg leading-relaxed">
            Achieving population-scale impact across India&apos;s Tier 2, Tier 3, and rural clusters by leveraging existing public digital infrastructure and an offline-first architecture.
          </p>
        </div>
      </section>

      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-16">
        <h2 className="text-2xl font-bold text-gray-900 mb-8">4-Phase Implementation Roadmap</h2>
        <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
          <div className="bg-white border border-gray-200 rounded-xl p-6 relative flex flex-col justify-between">
            <div>
              <span className="text-xs font-bold text-indigo-600 bg-indigo-50 px-2.5 py-1 rounded-md">PHASE 1 (M1-M3)</span>
              <h3 className="text-lg font-bold text-gray-900 mt-3 mb-2">Pilot & Standardization</h3>
              <p className="text-xs text-gray-600 leading-relaxed mb-4">
                Roll out across 5 pilot Tier 2 hubs (Jaipur, Lucknow, Bhopal, Indore, Patna). Align sectoral ISL curricula with ISLRTC guidelines.
              </p>
            </div>
            <div className="text-[11px] text-gray-500 border-t border-gray-100 pt-3">
              Target: 500 PwD candidates, 50 local SMEs
            </div>
          </div>

          <div className="bg-white border border-gray-200 rounded-xl p-6 relative flex flex-col justify-between">
            <div>
              <span className="text-xs font-bold text-emerald-600 bg-emerald-50 px-2.5 py-1 rounded-md">PHASE 2 (M4-M8)</span>
              <h3 className="text-lg font-bold text-gray-900 mt-3 mb-2">CSC Network Integration</h3>
              <p className="text-xs text-gray-600 leading-relaxed mb-4">
                Partner with Common Service Centres (CSCs) where Village Level Entrepreneurs (VLEs) serve as assisted-registration facilitators.
              </p>
            </div>
            <div className="text-[11px] text-gray-500 border-t border-gray-100 pt-3">
              Target: 2,500 CSCs activated across 3 states
            </div>
          </div>

          <div className="bg-white border border-gray-200 rounded-xl p-6 relative flex flex-col justify-between">
            <div>
              <span className="text-xs font-bold text-amber-600 bg-amber-50 px-2.5 py-1 rounded-md">PHASE 3 (M9-M14)</span>
              <h3 className="text-lg font-bold text-gray-900 mt-3 mb-2">SME Consortium</h3>
              <p className="text-xs text-gray-600 leading-relaxed mb-4">
                Deploy the SME Inclusion Toolkit and Section 80DD/80U tax advisory modules across industrial clusters.
              </p>
            </div>
            <div className="text-[11px] text-gray-500 border-t border-gray-100 pt-3">
              Target: 5,000+ placements, 500+ active employers
            </div>
          </div>

          <div className="bg-white border border-gray-200 rounded-xl p-6 relative flex flex-col justify-between">
            <div>
              <span className="text-xs font-bold text-purple-600 bg-purple-50 px-2.5 py-1 rounded-md">PHASE 4 (M15+)</span>
              <h3 className="text-lg font-bold text-gray-900 mt-3 mb-2">National Scale</h3>
              <p className="text-xs text-gray-600 leading-relaxed mb-4">
                Integrate with Skill Council for Persons with Disability (SCPwD) and state employment exchanges via open APIs.
              </p>
            </div>
            <div className="text-[11px] text-gray-500 border-t border-gray-100 pt-3">
              Target: Pan-India coverage across 28 states
            </div>
          </div>
        </div>
      </section>

      <section className="bg-gray-100 border-t border-gray-200 py-16">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-10">
            <div>
              <div className="flex items-center gap-2 mb-3">
                <Network className="w-5 h-5 text-indigo-600" />
                <h3 className="text-xl font-bold text-gray-900">Distribution via CSCs</h3>
              </div>
              <p className="text-sm text-gray-600 mb-4 leading-relaxed">
                Village Level Entrepreneurs (VLEs) at CSC kiosks act as physical points of contact for assisted profile registration, video comprehension, and document verification.
              </p>
              <ul className="space-y-2 text-xs text-gray-700">
                <li className="flex items-start gap-2">
                  <CheckCircle2 className="w-4 h-4 text-indigo-600 shrink-0 mt-0.5" />
                  <span>VLE commission incentive linked to candidate upskilling and verification</span>
                </li>
                <li className="flex items-start gap-2">
                  <CheckCircle2 className="w-4 h-4 text-indigo-600 shrink-0 mt-0.5" />
                  <span>In-person verification of UDID (Unique Disability ID)</span>
                </li>
              </ul>
            </div>

            <div>
              <div className="flex items-center gap-2 mb-3">
                <Cpu className="w-5 h-5 text-emerald-600" />
                <h3 className="text-xl font-bold text-gray-900">Offline-First Tech Architecture</h3>
              </div>
              <p className="text-sm text-gray-600 mb-4 leading-relaxed">
                Engineered for low-bandwidth networks with cached lesson data and lightweight payload transmissions.
              </p>
              <ul className="space-y-2 text-xs text-gray-700">
                <li className="flex items-start gap-2">
                  <CheckCircle2 className="w-4 h-4 text-emerald-600 shrink-0 mt-0.5" />
                  <span>Vector sign illustrations instead of resource-heavy video downloads</span>
                </li>
                <li className="flex items-start gap-2">
                  <CheckCircle2 className="w-4 h-4 text-emerald-600 shrink-0 mt-0.5" />
                  <span>Local client caching with background synchronization upon reconnect</span>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/isl-hub/page.tsx
'use client'

import { useState, useEffect } from 'react'
import { HandMetal, Filter, Search, BookOpen, Award, TrendingUp, Users } from 'lucide-react'
import CourseCard from '@/components/CourseCard'

interface Course {
  id: number
  title: string
  description: string
  sector: string
  level: string
  duration: string
  modules: number
  language: string
  _count?: { enrollments: number }
}

const sectors = ['ALL', 'HEALTHCARE', 'EDUCATION', 'PUBLIC_SERVICE', 'WORKPLACE']
const levels = ['ALL', 'BEGINNER', 'INTERMEDIATE', 'ADVANCED']

export default function ISLHubPage() {
  const [courses, setCourses] = useState<Course[]>([])
  const [loading, setLoading] = useState(true)
  const [sectorFilter, setSectorFilter] = useState('ALL')
  const [levelFilter, setLevelFilter] = useState('ALL')
  const [searchQuery, setSearchQuery] = useState('')

  useEffect(() => {
    const params = new URLSearchParams()
    if (sectorFilter !== 'ALL') params.set('sector', sectorFilter)
    if (levelFilter !== 'ALL') params.set('level', levelFilter)

    fetch(`/api/courses?${params}`)
      .then((res) => res.json())
      .then((data) => {
        setCourses(data)
        setLoading(false)
      })
      .catch(() => setLoading(false))
  }, [sectorFilter, levelFilter])

  const filteredCourses = courses.filter((c) =>
    c.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
    c.description.toLowerCase().includes(searchQuery.toLowerCase())
  )

  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-indigo-600 to-purple-700 text-white py-16">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex items-center gap-3 mb-4">
            <div className="w-12 h-12 bg-white/20 rounded-xl flex items-center justify-center">
              <HandMetal className="w-6 h-6" />
            </div>
            <h1 className="text-3xl md:text-4xl font-bold">ISL Learning Hub</h1>
          </div>
          <p className="text-indigo-100 max-w-2xl text-lg">
            Sector-specific Indian Sign Language courses designed for professionals. Learn ISL in context from medical triage to banking desks.
          </p>

          <div className="grid grid-cols-1 sm:grid-cols-3 gap-4 mt-8">
            <div className="bg-white/10 backdrop-blur-sm rounded-xl p-4 border border-white/20">
              <BookOpen className="w-5 h-5 mb-2 text-yellow-300" />
              <div className="text-2xl font-bold">{courses.length}</div>
              <div className="text-sm text-indigo-200">Courses Available</div>
            </div>
            <div className="bg-white/10 backdrop-blur-sm rounded-xl p-4 border border-white/20">
              <Award className="w-5 h-5 mb-2 text-yellow-300" />
              <div className="text-2xl font-bold">4</div>
              <div className="text-sm text-indigo-200">Sectors Covered</div>
            </div>
            <div className="bg-white/10 backdrop-blur-sm rounded-xl p-4 border border-white/20">
              <TrendingUp className="w-5 h-5 mb-2 text-yellow-300" />
              <div className="text-2xl font-bold">Standardized</div>
              <div className="text-sm text-indigo-200">ISLRTC-Aligned</div>
            </div>
          </div>
        </div>
      </section>

      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-10">
        <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
          <h2 className="text-2xl font-bold text-gray-900 flex items-center gap-2">
            <Filter className="w-5 h-5 text-indigo-500" /> Browse Courses
          </h2>
          <div className="relative">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400" />
            <input
              type="text"
              placeholder="Search courses..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className="pl-10 pr-4 py-2 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 w-full md:w-64"
            />
          </div>
        </div>

        <div className="flex flex-wrap gap-6 mb-8">
          <div>
            <label className="text-xs font-semibold text-gray-500 uppercase tracking-wider block mb-2">Sector</label>
            <div className="flex flex-wrap gap-2">
              {sectors.map((s) => (
                <button
                  key={s}
                  onClick={() => setSectorFilter(s)}
                  className={`px-3 py-1.5 rounded-full text-xs font-medium transition-all ${
                    sectorFilter === s
                      ? 'bg-indigo-600 text-white shadow-md'
                      : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
                  }`}
                >
                  {s === 'ALL' ? 'All Sectors' : s.replace('_', ' ')}
                </button>
              ))}
            </div>
          </div>
          <div>
            <label className="text-xs font-semibold text-gray-500 uppercase tracking-wider block mb-2">Level</label>
            <div className="flex flex-wrap gap-2">
              {levels.map((l) => (
                <button
                  key={l}
                  onClick={() => setLevelFilter(l)}
                  className={`px-3 py-1.5 rounded-full text-xs font-medium transition-all ${
                    levelFilter === l
                      ? 'bg-indigo-600 text-white shadow-md'
                      : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
                  }`}
                >
                  {l === 'ALL' ? 'All Levels' : l}
                </button>
              ))}
            </div>
          </div>
        </div>

        {loading ? (
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {[1, 2, 3].map((i) => (
              <div key={i} className="bg-white rounded-xl border border-gray-200 h-72 animate-pulse p-6">
                <div className="h-4 bg-gray-200 rounded w-1/3 mb-4" />
                <div className="h-6 bg-gray-200 rounded w-2/3 mb-3" />
                <div className="h-4 bg-gray-200 rounded w-full mb-2" />
                <div className="h-4 bg-gray-200 rounded w-3/4" />
              </div>
            ))}
          </div>
        ) : filteredCourses.length === 0 ? (
          <div className="text-center py-16 text-gray-400">
            <Users className="w-12 h-12 mx-auto mb-3 opacity-50" />
            <p className="text-lg">No courses found matching your filters.</p>
          </div>
        ) : (
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {filteredCourses.map((course) => (
              <CourseCard
                key={course.id}
                id={course.id}
                title={course.title}
                description={course.description}
                sector={course.sector}
                level={course.level}
                duration={course.duration}
                modules={course.modules}
                enrollmentCount={course._count?.enrollments || 0}
              />
            ))}
          </div>
        )}
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/isl-hub/[id]/page.tsx
'use client'

import { useEffect, useState, use } from 'react'
import Link from 'next/link'
import {
  ArrowLeft, Clock, BookOpen, Users, Award, CheckCircle2, Loader2, HandMetal,
} from 'lucide-react'

interface Course {
  id: number
  title: string
  description: string
  sector: string
  level: string
  duration: string
  modules: number
  language: string
  _count?: { enrollments: number }
}

export default function CourseDetailPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = use(params)
  const [course, setCourse] = useState<Course | null>(null)
  const [loading, setLoading] = useState(true)
  const [notFound, setNotFound] = useState(false)
  const [showForm, setShowForm] = useState(false)
  const [submitting, setSubmitting] = useState(false)
  const [submitted, setSubmitted] = useState(false)
  const [error, setError] = useState('')

  const [form, setForm] = useState({ name: '', email: '', city: '', disabilityType: '' })

  useEffect(() => {
    fetch(`/api/courses/${id}`)
      .then((res) => {
        if (!res.ok) throw new Error('not found')
        return res.json()
      })
      .then((data) => {
        setCourse(data)
        setLoading(false)
      })
      .catch(() => {
        setNotFound(true)
        setLoading(false)
      })
  }, [id])

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setError('')
    if (!form.name || !form.email) {
      setError('Name and email are required.')
      return
    }
    setSubmitting(true)
    try {
      const res = await fetch('/api/enrollments', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ courseId: Number(id), ...form }),
      })
      if (!res.ok) {
        const data = await res.json()
        throw new Error(data.error || 'Something went wrong')
      }
      setSubmitted(true)
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Something went wrong')
    } finally {
      setSubmitting(false)
    }
  }

  if (loading) {
    return (
      <div className="max-w-4xl mx-auto px-4 py-16 animate-pulse">
        <div className="h-6 bg-gray-200 rounded w-32 mb-8" />
        <div className="h-8 bg-gray-200 rounded w-2/3 mb-4" />
        <div className="h-40 bg-gray-200 rounded" />
      </div>
    )
  }

  if (notFound || !course) {
    return (
      <div className="max-w-4xl mx-auto px-4 py-24 text-center">
        <HandMetal className="w-12 h-12 mx-auto mb-4 text-gray-300" />
        <h1 className="text-xl font-semibold text-gray-900 mb-2">Course not found</h1>
        <Link href="/isl-hub" className="text-indigo-600 font-medium hover:underline text-sm">
          ← Back to ISL Hub
        </Link>
      </div>
    )
  }

  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-indigo-600 to-purple-700 text-white py-12">
        <div className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
          <Link href="/isl-hub" className="inline-flex items-center gap-1.5 text-indigo-100 hover:text-white text-sm mb-6">
            <ArrowLeft className="w-4 h-4" /> Back to ISL Hub
          </Link>
          <h1 className="text-2xl md:text-3xl font-bold mb-4">{course.title}</h1>
          <div className="flex flex-wrap gap-4 text-sm text-indigo-100">
            <span className="flex items-center gap-1"><Clock className="w-4 h-4" /> {course.duration}</span>
            <span className="flex items-center gap-1"><BookOpen className="w-4 h-4" /> {course.modules} modules</span>
            <span className="flex items-center gap-1"><Users className="w-4 h-4" /> {course._count?.enrollments || 0} enrolled</span>
            <span>Language: {course.language}</span>
          </div>
        </div>
      </section>

      <section className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 py-10 grid grid-cols-1 md:grid-cols-3 gap-8">
        <div className="md:col-span-2 space-y-6">
          <div>
            <h2 className="text-lg font-semibold text-gray-900 mb-2">Course Overview</h2>
            <p className="text-sm text-gray-600 leading-relaxed">{course.description}</p>
          </div>
          <div>
            <h2 className="text-lg font-semibold text-gray-900 mb-2">Key Outcomes</h2>
            <ul className="space-y-2 text-xs text-gray-600">
              <li className="flex items-start gap-2">
                <CheckCircle2 className="w-4 h-4 text-indigo-500 shrink-0 mt-0.5" />
                Sector-specific vocabulary aligned with national accessibility guidelines
              </li>
              <li className="flex items-start gap-2">
                <CheckCircle2 className="w-4 h-4 text-indigo-500 shrink-0 mt-0.5" />
                Practical scenario simulations for daily operational workflows
              </li>
            </ul>
          </div>
        </div>

        <div className="md:col-span-1">
          <div className="bg-white border border-gray-200 rounded-xl p-6 shadow-sm">
            {submitted ? (
              <div className="text-center py-4">
                <CheckCircle2 className="w-10 h-10 text-indigo-500 mx-auto mb-3" />
                <h3 className="font-semibold text-gray-900 mb-1">Enrolled Successfully</h3>
                <Link href="/dashboard" className="inline-block mt-4 text-xs font-semibold text-indigo-600 hover:underline">
                  View in Dashboard →
                </Link>
              </div>
            ) : showForm ? (
              <form onSubmit={handleSubmit} className="space-y-3">
                <h3 className="font-semibold text-gray-900 mb-1 text-sm">Enroll in Course</h3>
                <input
                  type="text" placeholder="Full name *" required
                  value={form.name}
                  onChange={(e) => setForm({ ...form, name: e.target.value })}
                  className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                />
                <input
                  type="email" placeholder="Email address *" required
                  value={form.email}
                  onChange={(e) => setForm({ ...form, email: e.target.value })}
                  className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                />
                <input
                  type="text" placeholder="City"
                  value={form.city}
                  onChange={(e) => setForm({ ...form, city: e.target.value })}
                  className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                />
                {error && <p className="text-xs text-red-600">{error}</p>}
                <button
                  type="submit"
                  disabled={submitting}
                  className="w-full py-2 bg-indigo-600 text-white rounded-lg text-xs font-semibold hover:bg-indigo-700 flex items-center justify-center gap-1.5"
                >
                  {submitting && <Loader2 className="w-3.5 h-3.5 animate-spin" />}
                  Confirm Enrollment
                </button>
              </form>
            ) : (
              <div>
                <h3 className="font-semibold text-gray-900 mb-1 text-sm">Start Learning</h3>
                <p className="text-xs text-gray-500 mb-4">Free self-paced training module.</p>
                <button
                  onClick={() => setShowForm(true)}
                  className="w-full py-2 bg-indigo-600 text-white rounded-lg text-xs font-semibold hover:bg-indigo-700 flex items-center justify-center gap-1.5"
                >
                  <Award className="w-3.5 h-3.5" /> Enroll Now
                </button>
              </div>
            )}
          </div>
        </div>
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/jobs/page.tsx
'use client'

import { useState, useEffect } from 'react'
import { Briefcase, Search, Filter } from 'lucide-react'
import JobCard from '@/components/JobCard'

interface Job {
  id: number
  title: string
  company: string
  location: string
  city: string
  tier: string
  type: string
  sector: string
  description: string
  salary: string | null
  accommodations: string
  disabilityTypes: string
  islCertified: boolean
}

const tiers = ['ALL', 'TIER1', 'TIER2', 'TIER3']
const types = ['ALL', 'FULL_TIME', 'PART_TIME', 'REMOTE', 'INTERNSHIP']
const accommodationOptions = ['WHEELCHAIR', 'SCREEN_READER', 'ISL_INTERPRETER', 'FLEXIBLE_HOURS']

export default function JobsPage() {
  const [jobs, setJobs] = useState<Job[]>([])
  const [loading, setLoading] = useState(true)
  const [tierFilter, setTierFilter] = useState('ALL')
  const [typeFilter, setTypeFilter] = useState('ALL')
  const [accomFilter, setAccomFilter] = useState('')
  const [islOnly, setIslOnly] = useState(false)
  const [searchQuery, setSearchQuery] = useState('')

  useEffect(() => {
    const params = new URLSearchParams()
    if (tierFilter !== 'ALL') params.set('tier', tierFilter)
    if (typeFilter !== 'ALL') params.set('type', typeFilter)
    if (accomFilter) params.set('accommodation', accomFilter)
    if (islOnly) params.set('islCertified', 'true')

    fetch(`/api/jobs?${params}`)
      .then((res) => res.json())
      .then((data) => {
        setJobs(data)
        setLoading(false)
      })
      .catch(() => setLoading(false))
  }, [tierFilter, typeFilter, accomFilter, islOnly])

  const filteredJobs = jobs.filter(
    (j) =>
      j.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
      j.company.toLowerCase().includes(searchQuery.toLowerCase()) ||
      j.city.toLowerCase().includes(searchQuery.toLowerCase())
  )

  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-emerald-600 to-teal-700 text-white py-16">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex items-center gap-3 mb-4">
            <div className="w-12 h-12 bg-white/20 rounded-xl flex items-center justify-center">
              <Briefcase className="w-6 h-6" />
            </div>
            <h1 className="text-3xl md:text-4xl font-bold">Inclusive Job Board</h1>
          </div>
          <p className="text-emerald-100 max-w-2xl text-lg">
            Connecting persons with disabilities to accessible, accommodation-ready opportunities with an emphasis on Tier 2/3 cities.
          </p>
        </div>
      </section>

      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-10">
        <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-6">
          <h2 className="text-2xl font-bold text-gray-900 flex items-center gap-2">
            <Filter className="w-5 h-5 text-emerald-500" /> Browse Openings
          </h2>
          <div className="relative">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400" />
            <input
              type="text"
              placeholder="Search by role, company, city..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className="pl-10 pr-4 py-2 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-emerald-500 w-full md:w-72"
            />
          </div>
        </div>

        <div className="mb-8 space-y-4">
          <div className="flex flex-wrap gap-6">
            <div>
              <label className="text-xs font-semibold text-gray-500 uppercase tracking-wider block mb-2">City Tier</label>
              <div className="flex flex-wrap gap-2">
                {tiers.map((t) => (
                  <button
                    key={t}
                    onClick={() => setTierFilter(t)}
                    className={`px-3 py-1.5 rounded-full text-xs font-medium transition-all ${
                      tierFilter === t
                        ? 'bg-emerald-600 text-white shadow-md'
                        : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
                    }`}
                  >
                    {t === 'ALL' ? 'All Tiers' : t.replace('TIER', 'Tier ')}
                  </button>
                ))}
              </div>
            </div>
            <div>
              <label className="text-xs font-semibold text-gray-500 uppercase tracking-wider block mb-2">Job Type</label>
              <div className="flex flex-wrap gap-2">
                {types.map((t) => (
                  <button
                    key={t}
                    onClick={() => setTypeFilter(t)}
                    className={`px-3 py-1.5 rounded-full text-xs font-medium transition-all ${
                      typeFilter === t
                        ? 'bg-emerald-600 text-white shadow-md'
                        : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
                    }`}
                  >
                    {t === 'ALL' ? 'All Types' : t.replace('_', ' ')}
                  </button>
                ))}
              </div>
            </div>
            <div>
              <label className="text-xs font-semibold text-gray-500 uppercase tracking-wider block mb-2">Accommodation</label>
              <div className="flex flex-wrap gap-2">
                <button
                  onClick={() => setAccomFilter('')}
                  className={`px-3 py-1.5 rounded-full text-xs font-medium transition-all ${
                    !accomFilter ? 'bg-emerald-600 text-white shadow-md' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
                  }`}
                >
                  Any
                </button>
                {accommodationOptions.map((a) => (
                  <button
                    key={a}
                    onClick={() => setAccomFilter(a)}
                    className={`px-3 py-1.5 rounded-full text-xs font-medium transition-all ${
                      accomFilter === a ? 'bg-emerald-600 text-white shadow-md' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
                    }`}
                  >
                    {a.replace(/_/g, ' ')}
                  </button>
                ))}
              </div>
            </div>
          </div>
          <label className="flex items-center gap-2 text-xs font-medium text-gray-700 cursor-pointer">
            <input
              type="checkbox"
              checked={islOnly}
              onChange={(e) => setIslOnly(e.target.checked)}
              className="rounded text-emerald-600 focus:ring-emerald-500"
            />
            Show only ISL Certified employers
          </label>
        </div>

        {loading ? (
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            {[1, 2, 3, 4].map((i) => (
              <div key={i} className="bg-white rounded-xl border border-gray-200 h-48 animate-pulse" />
            ))}
          </div>
        ) : filteredJobs.length === 0 ? (
          <div className="text-center py-16 text-gray-400">
            <p>No jobs found matching your criteria.</p>
          </div>
        ) : (
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            {filteredJobs.map((job) => (
              <JobCard key={job.id} {...job} />
            ))}
          </div>
        )}
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/jobs/[id]/page.tsx
'use client'

import { useEffect, useState, use } from 'react'
import Link from 'next/link'
import {
  ArrowLeft, MapPin, Building2, IndianRupee, Shield, Accessibility,
  CheckCircle2, Loader2, Briefcase,
} from 'lucide-react'

interface Job {
  id: number
  title: string
  company: string
  location: string
  city: string
  tier: string
  type: string
  sector: string
  description: string
  salary: string | null
  accommodations: string
  disabilityTypes: string
  islCertified: boolean
}

export default function JobDetailPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = use(params)
  const [job, setJob] = useState<Job | null>(null)
  const [loading, setLoading] = useState(true)
  const [notFound, setNotFound] = useState(false)
  const [showForm, setShowForm] = useState(false)
  const [submitting, setSubmitting] = useState(false)
  const [submitted, setSubmitted] = useState(false)
  const [error, setError] = useState('')

  const [form, setForm] = useState({
    name: '', email: '', phone: '', city: '',
    disabilityType: '', accommodationsNeeded: '', coverNote: '',
  })

  useEffect(() => {
    fetch(`/api/jobs/${id}`)
      .then((res) => {
        if (!res.ok) throw new Error('not found')
        return res.json()
      })
      .then((data) => {
        setJob(data)
        setLoading(false)
      })
      .catch(() => {
        setNotFound(true)
        setLoading(false)
      })
  }, [id])

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setError('')
    if (!form.name || !form.email) {
      setError('Name and email are required.')
      return
    }
    setSubmitting(true)
    try {
      const res = await fetch('/api/applications', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ jobId: Number(id), ...form }),
      })
      if (!res.ok) {
        const data = await res.json()
        throw new Error(data.error || 'Something went wrong')
      }
      setSubmitted(true)
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Something went wrong')
    } finally {
      setSubmitting(false)
    }
  }

  if (loading) {
    return (
      <div className="max-w-4xl mx-auto px-4 py-16 animate-pulse">
        <div className="h-6 bg-gray-200 rounded w-32 mb-8" />
        <div className="h-8 bg-gray-200 rounded w-2/3 mb-4" />
        <div className="h-40 bg-gray-200 rounded" />
      </div>
    )
  }

  if (notFound || !job) {
    return (
      <div className="max-w-4xl mx-auto px-4 py-24 text-center">
        <Briefcase className="w-12 h-12 mx-auto mb-4 text-gray-300" />
        <h1 className="text-xl font-semibold text-gray-900 mb-2">Job listing not found</h1>
        <Link href="/jobs" className="text-emerald-600 font-medium hover:underline text-sm">
          ← Back to job board
        </Link>
      </div>
    )
  }

  const accomList = job.accommodations.split(',').map((a) => a.trim()).filter(Boolean)

  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-emerald-600 to-teal-700 text-white py-12">
        <div className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
          <Link href="/jobs" className="inline-flex items-center gap-1.5 text-emerald-100 hover:text-white text-sm mb-6">
            <ArrowLeft className="w-4 h-4" /> Back to job board
          </Link>
          <div className="flex flex-wrap items-start justify-between gap-4">
            <div>
              <h1 className="text-2xl md:text-3xl font-bold mb-2">{job.title}</h1>
              <div className="flex items-center gap-2 text-emerald-100 text-sm">
                <Building2 className="w-4 h-4" /> {job.company}
              </div>
            </div>
            {job.islCertified && (
              <span className="flex items-center gap-1 px-3 py-1 bg-white/20 rounded-full text-xs font-medium">
                <Shield className="w-3.5 h-3.5" /> ISL Certified Employer
              </span>
            )}
          </div>

          <div className="flex flex-wrap gap-4 mt-6 text-xs text-emerald-100">
            <span className="flex items-center gap-1">
              <MapPin className="w-3.5 h-3.5" /> {job.location} ({job.tier.replace('TIER', 'Tier ')})
            </span>
            <span className="px-2 py-0.5 bg-white/10 rounded-full">{job.type.replace('_', ' ')}</span>
            {job.salary && (
              <span className="flex items-center gap-0.5">
                <IndianRupee className="w-3.5 h-3.5" /> {job.salary}
              </span>
            )}
          </div>
        </div>
      </section>

      <section className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 py-10 grid grid-cols-1 md:grid-cols-3 gap-8">
        <div className="md:col-span-2 space-y-6">
          <div>
            <h2 className="text-base font-semibold text-gray-900 mb-2">Job Description</h2>
            <p className="text-sm text-gray-600 leading-relaxed">{job.description}</p>
          </div>

          <div>
            <h2 className="text-base font-semibold text-gray-900 mb-2 flex items-center gap-2">
              <Accessibility className="w-4 h-4 text-emerald-500" /> Accommodations Available
            </h2>
            <div className="flex flex-wrap gap-2">
              {accomList.map((a) => (
                <span key={a} className="px-2.5 py-1 bg-emerald-50 border border-emerald-200 rounded text-xs text-emerald-700">
                  {a.replace(/_/g, ' ')}
                </span>
              ))}
            </div>
          </div>
        </div>

        <div className="md:col-span-1">
          <div className="bg-white border border-gray-200 rounded-xl p-6 shadow-sm">
            {submitted ? (
              <div className="text-center py-4">
                <CheckCircle2 className="w-10 h-10 text-emerald-500 mx-auto mb-3" />
                <h3 className="font-semibold text-gray-900 mb-1 text-sm">Application Sent</h3>
                <p className="text-xs text-gray-500">The hiring team has received your application.</p>
              </div>
            ) : showForm ? (
              <form onSubmit={handleSubmit} className="space-y-3">
                <h3 className="font-semibold text-gray-900 mb-1 text-sm">Apply for Role</h3>
                <input
                  type="text" placeholder="Full name *" required
                  value={form.name}
                  onChange={(e) => setForm({ ...form, name: e.target.value })}
                  className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                />
                <input
                  type="email" placeholder="Email address *" required
                  value={form.email}
                  onChange={(e) => setForm({ ...form, email: e.target.value })}
                  className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                />
                <input
                  type="tel" placeholder="Phone number"
                  value={form.phone}
                  onChange={(e) => setForm({ ...form, phone: e.target.value })}
                  className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                />
                {error && <p className="text-xs text-red-600">{error}</p>}
                <button
                  type="submit"
                  disabled={submitting}
                  className="w-full py-2 bg-emerald-600 text-white rounded-lg text-xs font-semibold hover:bg-emerald-700 flex items-center justify-center gap-1.5"
                >
                  {submitting && <Loader2 className="w-3.5 h-3.5 animate-spin" />}
                  Submit Application
                </button>
              </form>
            ) : (
              <div>
                <h3 className="font-semibold text-gray-900 mb-1 text-sm">Apply for this Position</h3>
                <p className="text-xs text-gray-500 mb-4">Direct application to {job.company}.</p>
                <button
                  onClick={() => setShowForm(true)}
                  className="w-full py-2 bg-emerald-600 text-white rounded-lg text-xs font-semibold hover:bg-emerald-700"
                >
                  Apply Now
                </button>
              </div>
            )}
          </div>
        </div>
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/employers/signup/page.tsx
'use client'

import { useState } from 'react'
import Link from 'next/link'
import { Building2, CheckCircle2, Loader2 } from 'lucide-react'

const tiers = ['TIER1', 'TIER2', 'TIER3']

export default function EmployerSignupPage() {
  const [form, setForm] = useState({
    companyName: '', contactName: '', email: '', phone: '',
    city: '', tier: 'TIER2', sector: '', website: '', description: '',
  })
  const [submitting, setSubmitting] = useState(false)
  const [submitted, setSubmitted] = useState(false)
  const [error, setError] = useState('')

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setError('')
    if (!form.companyName || !form.contactName || !form.email || !form.city || !form.sector) {
      setError('Please fill in all required fields.')
      return
    }
    setSubmitting(true)
    try {
      const res = await fetch('/api/employers', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(form),
      })
      if (!res.ok) {
        const data = await res.json()
        throw new Error(data.error || 'Something went wrong')
      }
      setSubmitted(true)
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Something went wrong')
    } finally {
      setSubmitting(false)
    }
  }

  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-gray-900 to-gray-800 text-white py-16">
        <div className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex items-center gap-3 mb-4">
            <div className="w-12 h-12 bg-white/10 rounded-xl flex items-center justify-center">
              <Building2 className="w-6 h-6" />
            </div>
            <h1 className="text-3xl font-bold">Employer Network Registration</h1>
          </div>
          <p className="text-gray-300 max-w-2xl text-sm md:text-base">
            Register your organization to list accessible job openings and access Section 80DD/80U inclusion incentives.
          </p>
        </div>
      </section>

      <section className="max-w-2xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <div className="bg-white border border-gray-200 rounded-xl p-8 shadow-sm">
          {submitted ? (
            <div className="text-center py-6">
              <CheckCircle2 className="w-12 h-12 text-emerald-500 mx-auto mb-4" />
              <h2 className="text-xl font-semibold text-gray-900 mb-2">Registration Complete</h2>
              <p className="text-gray-500 text-sm mb-6">You can now post your first accessible job listing.</p>
              <div className="flex gap-3 justify-center">
                <Link
                  href="/employers/post-job"
                  className="px-5 py-2.5 bg-emerald-600 text-white rounded-lg text-xs font-semibold hover:bg-emerald-700"
                >
                  Post a Job
                </Link>
                <Link
                  href="/jobs"
                  className="px-5 py-2.5 border border-gray-300 text-gray-700 rounded-lg text-xs font-semibold hover:bg-gray-50"
                >
                  View Job Board
                </Link>
              </div>
            </div>
          ) : (
            <form onSubmit={handleSubmit} className="space-y-4">
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Company Name *</label>
                  <input
                    type="text" required
                    value={form.companyName}
                    onChange={(e) => setForm({ ...form, companyName: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Contact Person *</label>
                  <input
                    type="text" required
                    value={form.contactName}
                    onChange={(e) => setForm({ ...form, contactName: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Work Email *</label>
                  <input
                    type="email" required
                    value={form.email}
                    onChange={(e) => setForm({ ...form, email: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">City *</label>
                  <input
                    type="text" required
                    value={form.city}
                    onChange={(e) => setForm({ ...form, city: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">City Tier *</label>
                  <select
                    value={form.tier}
                    onChange={(e) => setForm({ ...form, tier: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs text-gray-700"
                  >
                    {tiers.map((t) => (
                      <option key={t} value={t}>{t.replace('TIER', 'Tier ')}</option>
                    ))}
                  </select>
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Sector *</label>
                  <input
                    type="text" required placeholder="e.g. Banking, IT Services"
                    value={form.sector}
                    onChange={(e) => setForm({ ...form, sector: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
              </div>

              {error && <p className="text-xs text-red-600">{error}</p>}

              <button
                type="submit"
                disabled={submitting}
                className="w-full py-2.5 bg-emerald-600 text-white rounded-lg text-xs font-semibold hover:bg-emerald-700 flex items-center justify-center gap-2"
              >
                {submitting && <Loader2 className="w-3.5 h-3.5 animate-spin" />}
                Register Organization
              </button>
            </form>
          )}
        </div>
      </section>
    </div>
  )
}
EOF

cat << 'EOF' > src/app/employers/post-job/page.tsx
'use client'

import { useState } from 'react'
import Link from 'next/link'
import { Briefcase, CheckCircle2, Loader2 } from 'lucide-react'

const tiers = ['TIER1', 'TIER2', 'TIER3']
const types = ['FULL_TIME', 'PART_TIME', 'INTERNSHIP', 'REMOTE']
const accommodationOptions = ['WHEELCHAIR', 'SCREEN_READER', 'ISL_INTERPRETER', 'FLEXIBLE_HOURS']

export default function PostJobPage() {
  const [form, setForm] = useState({
    title: '', company: '', location: '', city: '', tier: 'TIER2', type: 'FULL_TIME',
    sector: '', description: '', salary: '', islCertified: false,
  })
  const [accommodations, setAccommodations] = useState<string[]>([])
  const [submitting, setSubmitting] = useState(false)
  const [submitted, setSubmitted] = useState(false)
  const [error, setError] = useState('')

  const toggle = (list: string[], setList: (v: string[]) => void, value: string) => {
    setList(list.includes(value) ? list.filter((v) => v !== value) : [...list, value])
  }

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setError('')
    if (!form.title || !form.company || !form.city || !form.sector || !form.description) {
      setError('Please fill in all required fields.')
      return
    }
    if (accommodations.length === 0) {
      setError('Select at least one accommodation you can offer.')
      return
    }
    setSubmitting(true)
    try {
      const res = await fetch('/api/jobs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          ...form,
          location: form.location || form.city,
          salary: form.salary || null,
          accommodations: accommodations.join(','),
          disabilityTypes: 'HEARING,VISUAL,MOTOR,COGNITIVE',
        }),
      })
      if (!res.ok) throw new Error('Something went wrong')
      setSubmitted(true)
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Something went wrong')
    } finally {
      setSubmitting(false)
    }
  }

  return (
    <div className="animate-fade-in">
      <section className="bg-gradient-to-br from-emerald-600 to-teal-700 text-white py-12">
        <div className="max-w-3xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex items-center gap-3 mb-3">
            <div className="w-12 h-12 bg-white/20 rounded-xl flex items-center justify-center">
              <Briefcase className="w-6 h-6" />
            </div>
            <h1 className="text-2xl md:text-3xl font-bold">Post an Accessible Job</h1>
          </div>
        </div>
      </section>

      <section className="max-w-3xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <div className="bg-white border border-gray-200 rounded-xl p-8 shadow-sm">
          {submitted ? (
            <div className="text-center py-6">
              <CheckCircle2 className="w-12 h-12 text-emerald-500 mx-auto mb-4" />
              <h2 className="text-xl font-semibold text-gray-900 mb-2">Job Posted Successfully</h2>
              <div className="flex gap-3 justify-center mt-6">
                <Link href="/jobs" className="px-5 py-2.5 bg-emerald-600 text-white rounded-lg text-xs font-semibold">
                  View on Job Board
                </Link>
              </div>
            </div>
          ) : (
            <form onSubmit={handleSubmit} className="space-y-4">
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Job Title *</label>
                  <input
                    type="text" required
                    value={form.title}
                    onChange={(e) => setForm({ ...form, title: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Company Name *</label>
                  <input
                    type="text" required
                    value={form.company}
                    onChange={(e) => setForm({ ...form, company: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">City *</label>
                  <input
                    type="text" required
                    value={form.city}
                    onChange={(e) => setForm({ ...form, city: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">City Tier *</label>
                  <select
                    value={form.tier}
                    onChange={(e) => setForm({ ...form, tier: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs text-gray-700"
                  >
                    {tiers.map((t) => <option key={t} value={t}>{t.replace('TIER', 'Tier ')}</option>)}
                  </select>
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Job Type *</label>
                  <select
                    value={form.type}
                    onChange={(e) => setForm({ ...form, type: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs text-gray-700"
                  >
                    {types.map((t) => <option key={t} value={t}>{t.replace('_', ' ')}</option>)}
                  </select>
                </div>
                <div>
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Sector *</label>
                  <input
                    type="text" required placeholder="e.g. Banking"
                    value={form.sector}
                    onChange={(e) => setForm({ ...form, sector: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
                <div className="sm:col-span-2">
                  <label className="text-xs font-semibold text-gray-500 uppercase block mb-1">Description *</label>
                  <textarea
                    rows={3} required
                    value={form.description}
                    onChange={(e) => setForm({ ...form, description: e.target.value })}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg text-xs"
                  />
                </div>
              </div>

              <div>
                <label className="text-xs font-semibold text-gray-500 uppercase block mb-2">Accommodations Provided *</label>
                <div className="flex flex-wrap gap-2">
                  {accommodationOptions.map((a) => (
                    <button
                      key={a} type="button"
                      onClick={() => toggle(accommodations, setAccommodations, a)}
                      className={`px-3 py-1.5 rounded-full text-xs font-medium ${
                        accommodations.includes(a)
                          ? 'bg-emerald-600 text-white shadow-sm'
                          : 'bg-gray-100 text-gray-600'
                      }`}
                    >
                      {a.replace(/_/g, ' ')}
                    </button>
                  ))}
                </div>
              </div>

              {error && <p className="text-xs text-red-600">{error}</p>}

              <button
                type="submit"
                disabled={submitting}
                className="w-full py-2.5 bg-emerald-600 text-white rounded-lg text-xs font-semibold hover:bg-emerald-700 flex items-center justify-center gap-2"
              >
                {submitting && <Loader2 className="w-3.5 h-3.5 animate-spin" />}
                Post Listing
              </button>
            </form>
          )}
        </div>
      </section>
    </div>
  )
}
EOF

# -------------------------------------------------------------
# 5. API Routes
# -------------------------------------------------------------

cat << 'EOF' > src/app/api/stats/route.ts
import { prisma } from '@/lib/prisma'
import { NextResponse } from 'next/server'

export async function GET() {
  const [totalCourses, totalJobs, totalUsers, totalEnrollments, certifiedCount, tier2Jobs, tier3Jobs] =
    await Promise.all([
      prisma.islCourse.count(),
      prisma.jobListing.count({ where: { isActive: true } }),
      prisma.user.count(),
      prisma.enrollment.count(),
      prisma.enrollment.count({ where: { certified: true } }),
      prisma.jobListing.count({ where: { tier: 'TIER2', isActive: true } }),
      prisma.jobListing.count({ where: { tier: 'TIER3', isActive: true } }),
    ])

  return NextResponse.json({
    totalCourses,
    totalJobs,
    totalUsers,
    totalEnrollments,
    certifiedCount,
    tier2Jobs,
    tier3Jobs,
    tier2_3Percentage: totalJobs > 0 ? Math.round(((tier2Jobs + tier3Jobs) / totalJobs) * 100) : 0,
  })
}
EOF

cat << 'EOF' > src/app/api/courses/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const sector = searchParams.get('sector')
  const level = searchParams.get('level')

  const where: Record<string, unknown> = {}
  if (sector) where.sector = sector
  if (level) where.level = level

  const courses = await prisma.islCourse.findMany({
    where,
    include: {
      _count: { select: { enrollments: true } },
    },
    orderBy: { createdAt: 'desc' },
  })

  return NextResponse.json(courses)
}
EOF

cat << 'EOF' > src/app/api/courses/[id]/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const course = await prisma.islCourse.findUnique({
    where: { id: Number(id) },
    include: { _count: { select: { enrollments: true } } },
  })

  if (!course) {
    return NextResponse.json({ error: 'Course not found' }, { status: 404 })
  }

  return NextResponse.json(course)
}
EOF

cat << 'EOF' > src/app/api/enrollments/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const body = await request.json()
  const { courseId, name, email, city, disabilityType } = body

  if (!courseId || !name || !email) {
    return NextResponse.json(
      { error: 'courseId, name, and email are required' },
      { status: 400 }
    )
  }

  const course = await prisma.islCourse.findUnique({ where: { id: Number(courseId) } })
  if (!course) {
    return NextResponse.json({ error: 'Course not found' }, { status: 404 })
  }

  let user = await prisma.user.findUnique({ where: { email } })
  if (!user) {
    user = await prisma.user.create({
      data: {
        name,
        email,
        role: 'LEARNER',
        city: city || null,
        disabilityType: disabilityType || null,
      },
    })
  }

  const existing = await prisma.enrollment.findUnique({
    where: { userId_courseId: { userId: user.id, courseId: Number(courseId) } },
  })
  if (existing) {
    return NextResponse.json(existing, { status: 200 })
  }

  const enrollment = await prisma.enrollment.create({
    data: {
      userId: user.id,
      courseId: Number(courseId),
      progress: 0,
      certified: false,
    },
  })

  return NextResponse.json(enrollment, { status: 201 })
}
EOF

cat << 'EOF' > src/app/api/jobs/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const city = searchParams.get('city')
  const tier = searchParams.get('tier')
  const type = searchParams.get('type')
  const sector = searchParams.get('sector')
  const accommodation = searchParams.get('accommodation')
  const islCertified = searchParams.get('islCertified')

  const where: Record<string, unknown> = { isActive: true }

  if (city) where.city = { contains: city }
  if (tier) where.tier = tier
  if (type) where.type = type
  if (sector) where.sector = { contains: sector }
  if (accommodation) where.accommodations = { contains: accommodation }
  if (islCertified === 'true') where.islCertified = true

  const jobs = await prisma.jobListing.findMany({
    where,
    orderBy: { postedAt: 'desc' },
  })

  return NextResponse.json(jobs)
}

export async function POST(request: NextRequest) {
  const body = await request.json()
  const job = await prisma.jobListing.create({ data: body })
  return NextResponse.json(job, { status: 201 })
}
EOF

cat << 'EOF' > src/app/api/jobs/[id]/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const job = await prisma.jobListing.findUnique({
    where: { id: Number(id) },
    include: { _count: { select: { applications: true } } },
  })

  if (!job) {
    return NextResponse.json({ error: 'Job not found' }, { status: 404 })
  }

  return NextResponse.json(job)
}
EOF

cat << 'EOF' > src/app/api/applications/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const body = await request.json()
  const { jobId, name, email, phone, city, disabilityType, accommodationsNeeded, coverNote } = body

  if (!jobId || !name || !email) {
    return NextResponse.json(
      { error: 'jobId, name, and email are required' },
      { status: 400 }
    )
  }

  const job = await prisma.jobListing.findUnique({ where: { id: Number(jobId) } })
  if (!job) {
    return NextResponse.json({ error: 'Job not found' }, { status: 404 })
  }

  const application = await prisma.application.create({
    data: {
      jobId: Number(jobId),
      name,
      email,
      phone: phone || null,
      city: city || null,
      disabilityType: disabilityType || null,
      accommodationsNeeded: accommodationsNeeded || null,
      coverNote: coverNote || null,
    },
  })

  return NextResponse.json(application, { status: 201 })
}

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const jobId = searchParams.get('jobId')

  const where: Record<string, unknown> = {}
  if (jobId) where.jobId = Number(jobId)

  const applications = await prisma.application.findMany({
    where,
    include: { job: true },
    orderBy: { createdAt: 'desc' },
  })

  return NextResponse.json(applications)
}
EOF

cat << 'EOF' > src/app/api/employers/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const body = await request.json()
  const { companyName, contactName, email, phone, city, tier, sector, website, description } = body

  if (!companyName || !contactName || !email || !city || !tier || !sector) {
    return NextResponse.json(
      { error: 'companyName, contactName, email, city, tier, and sector are required' },
      { status: 400 }
    )
  }

  const existing = await prisma.employer.findUnique({ where: { email } })
  if (existing) {
    return NextResponse.json(existing, { status: 200 })
  }

  const employer = await prisma.employer.create({
    data: {
      companyName,
      contactName,
      email,
      phone: phone || null,
      city,
      tier,
      sector,
      website: website || null,
      description: description || null,
    },
  })

  return NextResponse.json(employer, { status: 201 })
}

export async function GET() {
  const employers = await prisma.employer.findMany({ orderBy: { createdAt: 'desc' } })
  return NextResponse.json(employers)
}
EOF

echo "Samarthya codebase successfully written."
echo "Now run:"
echo "  1. npm install"
echo "  2. npx prisma generate"
echo "  3. npx prisma db push"
echo "  4. npx prisma db seed (or ts-node prisma/seed.ts)"
echo "  5. npm run dev"
