# Lucky PAW 🐾 Project Specification

This file is the single source of truth for the project. Read it before starting any task. Build the project one phase at a time (see section 10). Never build several phases in one go.

# 1. Project overview
Lucky PAW is an online pet store. Customers can browse and buy pet toys and pet food, and discover and register for pet events such as adoption days, training workshops, grooming sessions and pet shows. Admins manage products, events and orders from a dashboard.

Branding:
1. The logo is a wordmark reading "Lucky PAW", with "PAW" in bold using the accent colour, followed by a custom SVG paw print icon. Do not use the 🐾 emoji inside the logo, because emojis render differently on each device.
2. Use the emoji only in page titles and friendly text, for example the browser tab title "Lucky PAW 🐾 | Toys, Food and Events for Your Pets".
3. Use a paw print SVG as the favicon.
4. Use the name "Lucky PAW" consistently in the navbar, footer, metadata and README.

# 2. Tech stack
Backend: Node.js (LTS) with Express 5 and TypeScript, PostgreSQL as the database, Prisma as the ORM (schema, migrations and seeding), JWT authentication stored in an httpOnly cookie, bcrypt for password hashing, zod for request validation, helmet, cors, morgan, cookie parsing middleware and rate limiting middleware.
Frontend: Next.js (latest stable) with the App Router and TypeScript, Tailwind CSS, Motion (formerly Framer Motion) for animations, Lucide icons, Zustand for the cart store, next/image for all images, next/font for fonts.
Testing: Jest and Supertest for backend API tests.
Tooling: npm, ESLint and Prettier in both apps.

The repository root is a folder named "luckypaw" containing two separate apps: "client" (Next.js) and "server" (Express).

Ports: the server runs on port 4000 and the client on port 3000. Do not use port 5000, because macOS uses it for AirPlay.

# 3. Folder structure
luckypaw
  SPEC.md
  CLAUDE.md
  README.md
  .gitignore
  server
    prisma
      schema.prisma
      migrations
      seed.ts
    src
      config (Prisma client instance, environment loading)
      controllers
      routes
      middleware (auth, admin check, validation, error handler)
      validators
      utils
      app.ts and server.ts
    tests
    .env and .env.example
  client
    app (all routes)
    components (ui, layout, sections, product, event, animation)
    lib (API client, helpers)
    hooks
    store (cart, auth)
    types
    public/images
    .env.local and .env.example

# 4. Data models (Prisma schema)
Use Prisma enums for fixed values and Decimal for all money fields.

User: id, name, email (unique), passwordHash, role (CUSTOMER or ADMIN), createdAt. Has many orders and event registrations.

Product: id, name, slug (unique), description, category (TOYS or FOOD), petType (DOG, CAT, BIRD, FISH, SMALL_PET), price, salePrice (optional), images (string array), stock, brand, rating, isFeatured, weight (optional, food only), ageGroup (optional, food only: YOUNG, ADULT, SENIOR), createdAt, updatedAt.

Event: id, title, slug (unique), description, date, startTime, endTime, location, image, capacity, price (0 means free), category (ADOPTION, TRAINING, GROOMING, SHOW, MEETUP), createdAt. Has many registrations.

EventRegistration: id, userId, eventId, numberOfPets, createdAt. Add a unique constraint on (userId, eventId) so a user cannot register twice.

Order: id, userId, total, shipping address fields (fullName, phone, addressLine, city, postalCode), status (PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED), createdAt. Has many order items.

OrderItem: id, orderId, productId, productName, unitPrice, quantity. Store the name and price at the time of purchase so old orders stay correct when product prices change.

NewsletterSubscriber: id, email (unique), createdAt.

ContactMessage: id, name, email, message, createdAt.

Database rules:
1. Create orders inside a Prisma transaction: check stock, reduce stock and save the order together, so two customers can never buy the same last item.
2. Create event registrations inside a transaction that counts existing registrations and rejects the request when the event is full.
3. Add indexes on Product category, petType and isFeatured, and on Event date.
4. In the Prisma datasource, use DATABASE_URL for the app connection and DIRECT_URL for migrations, so Neon's pooled and direct connection strings both work in production. Locally both can be the same value.
5. The seed runs with "npx prisma db seed". It creates one admin account, one customer account, at least 12 products (6 toys and 6 foods across different pet types) and 6 events (4 upcoming, 2 past) with realistic data and Unsplash image URLs.

# 5. REST API
Prefix all routes with /api. Always return JSON in a consistent shape: { success, data, message } with correct HTTP status codes.

Health
GET /api/health

Auth
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
GET  /api/auth/me

Products
GET    /api/products (filters: category, petType, minPrice, maxPrice, search, sort, page, limit)
GET    /api/products/featured
GET    /api/products/:slug
POST   /api/products (admin only)
PUT    /api/products/:id (admin only)
DELETE /api/products/:id (admin only)

Events
GET    /api/events (upcoming by default, filter by category, include past events with a query flag)
GET    /api/events/:slug
POST   /api/events/:id/register (logged in users only)
GET    /api/events/registrations/mine (logged in users only)
POST   /api/events (admin only)
PUT    /api/events/:id (admin only)
DELETE /api/events/:id (admin only)

Orders
POST  /api/orders (check stock, reduce stock, calculate the total on the server, never trust prices sent from the client)
GET   /api/orders/mine
GET   /api/orders (admin only)
PATCH /api/orders/:id/status (admin only)

Uploads
POST /api/uploads (admin only, uploads an image to Cloudinary and returns the URL)

Other
POST /api/newsletter
POST /api/contact

Backend rules:
1. Validate every request body and query with zod.
2. Use a central error handler middleware and catch all async errors.
3. Rate limit the auth routes.
4. Restrict CORS to the frontend URL with credentials enabled.
5. Keep secrets in .env and provide a .env.example with every variable listed.
6. Read the port from process.env.PORT (default 4000).
7. Write Jest and Supertest tests for auth, products, events and orders, including edge cases such as duplicate registration, a full event, insufficient stock and a customer calling an admin route.

# 6. Frontend pages
1. Home: hero section, category highlights (Toys, Food, Events), featured products, upcoming events, "why choose us", testimonials, stats counters, newsletter signup, footer.
2. Shop (/shop): product grid with a filter sidebar (category, pet type, price range), search, sort and pagination. Store filters in the URL search params so results can be shared.
3. Product details (/shop/[slug]): image gallery, price and sale price, stock status, quantity selector, add to cart button, related products.
4. Events (/events): upcoming events as cards with a date badge, filter by category, past events in a separate section below.
5. Event details (/events/[slug]): full details, a "spots left" indicator, register button.
6. Cart (/cart) and Checkout (/checkout): cart kept in a Zustand store saved to localStorage. Checkout collects the shipping address and creates an order. Use a mock payment step (no real payments).
7. Login and Register pages.
8. Account (/account): profile, order history, my event registrations.
9. Admin dashboard (/admin): protected route to manage products, events and orders using tables and forms, with image upload.
10. About and Contact pages.
11. A custom 404 page.

Use server components for data fetching where possible, and client components only where interaction or animation is needed. Add loading skeletons and error states for every data driven section. Add proper SEO metadata on every page.

API connection: the browser must only ever call the frontend domain. Add a Next.js rewrite so requests to /api are forwarded to the backend URL from the BACKEND_URL environment variable (http://localhost:4000 locally). This keeps the auth cookie first party in production.

# 7. Design direction
The website must look modern, clean and premium, not like a generic template.
1. Colours: warm cream background, deep teal or forest green as the primary colour, a playful coral or orange accent, dark charcoal for text. Define these as Tailwind theme tokens.
2. Typography: a rounded, friendly display font for headings (for example Fredoka or Baloo 2) and a clean sans serif for body text (for example Inter or DM Sans), loaded with next/font.
3. Layout: generous whitespace, large rounded corners (around 24px), soft shadows, subtle gradients, cards that lift slightly on hover, and a sticky glass style navbar that shrinks when the user scrolls.
4. Imagery: high quality pet photos (Unsplash URLs for seed data, Cloudinary for admin uploads; allow both domains in next.config), with playful decorative shapes such as paw prints or soft blobs used sparingly.
5. Fully responsive and mobile first, with a slide in mobile menu.
6. Accessible: semantic HTML, alt text on all images, visible focus states, good colour contrast and full keyboard navigation.

# 8. Scroll animations (very important)
Content must appear smoothly and slowly as the user scrolls down the page.
1. Create a reusable Reveal client component using Motion's whileInView. Default behaviour: start at opacity 0 and 40px below the final position, then animate to full opacity and the original position over 0.8 to 1 second with an ease out curve. Trigger once when about 20% of the element is visible.
2. Give Reveal props for direction (up, down, left, right), delay and duration.
3. Create a StaggerGroup component so grids of product and event cards appear one after another with a 0.1 to 0.15 second stagger between items.
4. Hero: headline words fade and rise in sequence on page load, and the hero image has a gentle floating motion plus a light parallax effect on scroll.
5. Section headings should reveal slightly before the content under them.
6. Stats counters (for example "5,000+ happy pets") should count up when they enter the viewport.
7. Keep animations smooth: animate only transform and opacity, avoid layout shift, and only the parallax effect should react to every scroll movement.
8. Respect the user's reduced motion setting: if it is enabled, show content instantly without movement.
Use these components in every section on every page so the whole site feels consistent.

# 9. Quality requirements
1. Clean, modular code with helpful comments where logic is not obvious.
2. No hard coded secrets.
3. Shared TypeScript types for API responses.
4. Friendly empty states (no products found, no upcoming events, empty cart).
5. Optimised images and fast loading. Target a Lighthouse score of 90 or above for performance and accessibility.

# 10. Build phases
Build one phase per request. At the end of each phase: run the app or tests to confirm it works, then give a short summary and a list of things I should check manually.

Phase 1: Project setup (both apps, configs, env files, .gitignore, README skeleton)
Phase 2: Database (Prisma schema, first migration, seed)
Phase 3: Backend API and tests
Phase 4: Frontend foundation (theme, fonts, layout, navbar, footer, UI components, Reveal, StaggerGroup, API rewrite)
Phase 5: Home page
Phase 6: Shop and product details
Phase 7: Events and event details
Phase 8: Auth pages, cart, checkout and account
Phase 9: Admin dashboard
Phase 10: About, Contact, 404 and final polish (responsiveness, accessibility, performance)
Phase 11: Deployment preparation and README

# 11. Local development and deployment
Local development:
1. PostgreSQL runs directly on macOS through Postgres.app. Do not create a compose.yaml or any Docker files.
2. Local DATABASE_URL format: postgresql://MAC_USERNAME@localhost:5432/luckypaw (no password).

Deployment (all free plans):
1. Frontend on Vercel, backend on Render, database on Neon (PostgreSQL), images on Cloudinary.
2. Server "build" script runs "prisma generate" and compiles TypeScript. Server "start" script runs "prisma migrate deploy" and then starts the compiled server.
3. Do not store uploaded files on the server disk, because Render's free plan loses files on every restart.
4. The README includes step by step deployment instructions for Neon, Render and Vercel, listing every environment variable each platform needs, plus the local setup steps, how to run the seed and the default admin login.
