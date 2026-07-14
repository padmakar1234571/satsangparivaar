# Satsang Parivaar — Vercel deployment

This is a standalone export of the Satsang Parivaar website, restructured to
deploy on Vercel with plain npm (no pnpm workspace, no `catalog:` protocol).

## What changed from the Replit version

- All `catalog:` and `workspace:*` package versions were replaced with real
  pinned npm version numbers in `package.json`.
- The Express API server was rewritten as two Vercel serverless functions
  under `/api` (`api/enquiries.ts`, `api/healthz.ts`), since Vercel does not
  run a long-lived Express process.
- Replit-only Vite plugins (cartographer, dev banner, runtime error modal)
  were removed — they only work inside the Replit workspace.
- The generated API client (`@workspace/api-client-react`) was inlined into
  `src/lib/api-client`.

## Setup

1. Install dependencies:
   ```bash
   npm install
   ```
2. Copy `.env.example` to `.env` and fill in:
   - `DATABASE_URL` — a PostgreSQL connection string. Use
     [Vercel Postgres](https://vercel.com/docs/storage/vercel-postgres),
     [Neon](https://neon.tech), or [Supabase](https://supabase.com) (all have
     free tiers). Your existing Replit database's connection string will
     **not** work from Vercel — Replit's built-in Postgres is not reachable
     from outside the Replit network.
   - `ADMIN_PASSCODE` — the passcode you use to view enquiries at `/admin`.
3. Create the `enquiries` table in your new database:
   ```bash
   npm run db:push
   ```
4. Test locally:
   ```bash
   npm run dev
   ```
   The frontend runs on Vite's dev server. To exercise the `/api` functions
   locally the same way Vercel runs them, use the
   [Vercel CLI](https://vercel.com/docs/cli) instead: `npx vercel dev`.

## Deploy to Vercel

1. Push this project to a GitHub repo (or run `npx vercel` directly from this
   folder).
2. Import the repo in the [Vercel dashboard](https://vercel.com/new), or run
   `npx vercel --prod` from this folder.
3. In the Vercel project's **Settings → Environment Variables**, add
   `DATABASE_URL` and `ADMIN_PASSCODE`.
4. Deploy. Vercel auto-detects the Vite frontend (via `vercel.json`) and
   deploys `api/*.ts` as serverless functions automatically — no extra
   configuration needed.

## Notes

- The Aadhar numbers customers submit are sensitive personal data. Keep
  `ADMIN_PASSCODE` private, and only you should know the `/admin` URL.
- If you outgrow serverless functions per file, everything in `/api` can be
  consolidated back into an Express app later without touching the frontend.
