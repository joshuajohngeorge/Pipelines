# Pipelines
Pipelines aims to provide formerly incarcerated individuals with a consolidated and accessible platform for long-term employment opportunities in skilled trades.


# Pipelines

**Connecting formerly incarcerated people in Champaign County to skilled-trades training, apprenticeships and jobs.**

Pipelines is an Enactus UIUC project (FA26). This repo holds the website and its backend. The site has a searchable job board, a directory of reentry resources, and a contact/intake form for job seekers and partner organizations.

> **Principle:** job seekers never need an account, and the site never asks about conviction history. Browsing is anonymous. Only the team and partner orgs log in.

---

## Features

- **Job board:** jobs, apprenticeships and training programs, with keyword search and filters (type, trade, city, fair-chance employer, pay or cost, schedule, certification, date posted)
- **Resource directory:** housing, legal/record help, IDs and documents, transportation, food, health and reentry orgs, filterable by category
- **Contact & partner forms:** submissions are stored, the team gets an email, and the forms are spam-protected
- **Admin panel:** add, edit, approve and expire listings and resources without touching code
- **Listing freshness:** listings expire automatically, and an admin view shows which ones need re-verifying

## Tech stack

| Layer | Tool |
|---|---|
| Frontend | React (generated with Figma Make) |
| Database + API | [Supabase](https://supabase.com) (Postgres, auto REST API, row-level security) |
| Auth | Supabase Auth (email magic link, admins and partners only) |
| File storage | Supabase Storage (logos, flyers) |
| Serverless | Supabase Edge Functions (form email notifications) |
| Scheduled jobs | Supabase cron (`pg_cron`) for listing expiry |
| Email | Resend |
| Spam protection | Cloudflare Turnstile |
| Hosting | Vercel |

## Project structure

```
pipelines/
├── frontend/                # React app (Figma Make export)
│   ├── src/
│   │   ├── pages/           # JobBoard, ListingDetail, Resources, Contact, Admin
│   │   ├── components/
│   │   └── lib/supabase.ts  # Supabase client
│   └── .env.example
├── supabase/
│   ├── migrations/          # SQL schema + RLS policies
│   ├── seed.sql             # trades, categories, sample data
│   └── functions/
│       └── notify-submission/   # Edge Function: email on new form submission
└── README.md
```

## Getting started

### Prerequisites

- Node.js 18+
- [Supabase CLI](https://supabase.com/docs/guides/cli)
- A Supabase project (use the shared Enactus account, not a personal one)

### 1. Clone and install

```bash
git clone https://github.com/<org>/pipelines.git
cd pipelines/frontend
npm install
```

### 2. Environment variables

Copy `.env.example` to `.env` and fill in:

```
VITE_SUPABASE_URL=your-project-url
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_TURNSTILE_SITE_KEY=your-turnstile-key
```

Set these as Supabase secrets for the Edge Function (never commit them):

```bash
supabase secrets set RESEND_API_KEY=... TEAM_INBOX=...
```

### 3. Set up the database

```bash
supabase link --project-ref <project-ref>
supabase db push          # runs migrations
psql "$DB_URL" -f supabase/seed.sql   # optional sample data
```

### 4. Run locally

```bash
npm run dev
```

## Data model

| Table | Purpose |
|---|---|
| `organizations` | Employers, training providers, unions and nonprofits (name, type, contact info, `fair_chance`, logo) |
| `opportunities` | Jobs, apprenticeships and training programs: trade, city, pay or cost, schedule, requirements, `background_policy`, `transit_accessible`, `apply_url`, `status`, `expires_at` |
| `trades` | Lookup table for the trade filter (electrical, plumbing, HVAC, carpentry, welding, CDL…) |
| `resources` | Resource directory entries, with `last_verified_at` |
| `resource_categories` | Housing, Legal/Record, IDs and documents, Transportation, Food, Health, Reentry orgs |
| `submissions` | All public form input (contact, partner interest, listing submissions) |
| `profiles` | Links a login to a role (`admin` / `partner`) and an org |

**Listing lifecycle:** `draft` → `pending_review` → `published` → `expired` / `archived`

The public job board reads from the `public_opportunities` view. It joins opportunities, organizations and trades, and returns only published, unexpired listings.

## API

Supabase generates the REST API from the schema.

| Endpoint | Access | Purpose |
|---|---|---|
| `GET /opportunities` (via `public_opportunities`) | Public | Job board list with filters |
| `GET /opportunities/:id` | Public | Listing detail |
| `GET /organizations/:id` | Public | Partner profile |
| `GET /trades`, `GET /resource_categories` | Public | Filter options |
| `GET /resources?category=` | Public | Resource directory |
| `POST /submissions` | Public | Forms (CAPTCHA + rate limit) |
| `POST / PATCH / DELETE` on content tables | Admin, partner (own org) | Content management |

## Security & privacy

- Row-level security is on for **every** table:
  - The public can read published content only
  - The public can insert into `submissions` but can never read it
  - Partners can edit only rows where `org_id` matches their org
- No personal data is collected from job seekers. Analytics stay aggregate and anonymous.
- `fair_chance` and `background_policy` are set only from what the employer confirms, and each listing shows its last-verified date.
- Never commit `.env` files or service-role keys.

## Roadmap

**Foundation**
- [ ] Supabase project + repo connected to the frontend
- [ ] Schema migration + RLS policies
- [ ] Seed trades, categories and sample listings
- [ ] `public_opportunities` view + filter queries

**MVP**
- [ ] Job board wired to live data (list, filters, detail)
- [ ] Resources page
- [ ] Contact Us form → `submissions` + email notification
- [ ] Admin page (login, add/edit, approve)
- [ ] Expiry cron job + re-verification view

**Later**
- [ ] Partner logins and self-serve listings
- [ ] Anonymous apply-click tracking for impact metrics
- [ ] Spanish-language content

## Contributing

1. Branch off `main`: `git checkout -b feature/<short-name>`
2. Put every schema change in a new migration (`supabase migration new <name>`). Never edit the database by hand in production.
3. Open a PR and get one review before merging.

## Team

Built by the Enactus UIUC Pipelines team (FA26).
