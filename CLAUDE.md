# Lucky PAW 🐾

Online pet store (pet toys, pet food, pet events). The full specification is in SPEC.md. Read SPEC.md before starting any task and follow it exactly.

## Working rules
1. Build only the phase I ask for. Do not jump ahead to later phases.
2. Before writing code for a phase, show me a short plan and wait for my approval.
3. At the end of each phase, run the app or the tests to confirm everything works, then give me a short summary and a checklist of things to test manually.
4. I am new to PostgreSQL and Prisma. When you use a new Prisma or PostgreSQL concept, explain it in one or two simple sentences.
5. Do not add libraries that are not listed in SPEC.md without asking me first.
6. Never commit .env files or secrets.
7. Do not use Docker. PostgreSQL runs locally through Postgres.app on macOS.
8. Keep this file updated with the commands below as they are created.

## Stack
Server: Express 5, TypeScript, Prisma, PostgreSQL (folder: server, port 4000)
Client: Next.js App Router, TypeScript, Tailwind CSS, Motion (folder: client, port 3000)

## Commands
Server (run inside `server`):
1. Start the API in dev mode: `npm run dev` (http://localhost:4000/api/health)
2. Run migrations: `npx prisma migrate dev` (production: `npx prisma migrate deploy`)
3. Run the seed: `npx prisma db seed` (deletes and recreates demo data)
4. Run tests: `npm test` (uses TEST_DATABASE_URL, a separate `luckypaw_test` database)
5. Open Prisma Studio: `npx prisma studio`
6. Lint, type check, format: `npm run lint`, `npm run typecheck`, `npm run format`
7. Production build and start: `npm run build`, `npm start`

Client (run inside `client`):
1. Start the website in dev mode: `npm run dev` (http://localhost:3000)
2. Production build and start: `npm run build`, `npm start`
3. Lint, type check, format: `npm run lint`, `npm run typecheck`, `npm run format`

Demo logins from the seed: admin@luckypaw.com / Admin@12345 and customer@luckypaw.com / Customer@12345.

## Notes for future work
1. Next.js 16 with Cache Components is enabled. Data sections fetch at request time through `lib/server-api.ts` (which calls `connection()`) inside `<Suspense>` with skeleton fallbacks. Client hooks that read the URL (`usePathname`, `useSearchParams`) must sit inside `<Suspense>`. Read `client/node_modules/next/dist/docs` before changing data fetching.
2. Admin pages export `instant = false` because they wait for a client side login check.
3. Prisma is pinned to version 6 because SPEC uses `url` plus `directUrl` in the datasource.
