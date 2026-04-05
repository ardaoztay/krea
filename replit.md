# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server
│   └── postcraft/          # React + Vite frontend (KREA SaaS — rebranded from PostCraft)
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   └── db/                 # Drizzle ORM schema + DB connection
├── scripts/                # Utility scripts (single workspace package)
│   └── src/                # Individual .ts scripts, run via `pnpm --filter @workspace/scripts run <script>`
├── pnpm-workspace.yaml     # pnpm workspace (artifacts/*, lib/*, lib/integrations/*, scripts)
├── tsconfig.base.json      # Shared TS options (composite, bundler resolution, es2022)
├── tsconfig.json           # Root TS project references
└── package.json            # Root package with hoisted devDeps
```

## PWA (Progressive Web App)

KREA is a fully installable PWA:
- **Manifest**: `artifacts/postcraft/public/manifest.json` — name, icons, theme color (#FF3C00), display: standalone, shortcuts
- **Icons**: `artifacts/postcraft/public/icons/icon-512.svg` (standard) + `icon-maskable.svg` (with safe-zone padding for maskable)
- **Service worker**: `artifacts/postcraft/public/sw.js` — cache-first for static assets, network-first for navigation; registered in `src/main.tsx` using `import.meta.env.BASE_URL`
- **Meta tags**: theme-color, apple-mobile-web-app-capable, apple-mobile-web-app-status-bar-style, apple-touch-icon — all in `index.html`
- **Safe area insets**: `env(safe-area-inset-*)` applied in standalone mode via CSS `@media (display-mode: standalone)` for notched phones
- **Touch optimizations**: `touch-action: manipulation` on all interactive elements; non-passive touch listeners in PostImageCard for drag + pinch-to-zoom; `touch-action: none` on the card when interactive

## KREA App (formerly PostCraft)

KREA is a premium AI-first social media post generator SaaS for multi-vertical businesses. Rebranded from PostCraft — all UI, translations, and branding updated to KREA.

### Pages
- `/` — Landing page (hero, features, CTA)
- `/dashboard` — Dashboard with stats and post history
- `/generate` — 4-step post generator wizard (Platform+Size → Content → Tone → Result)
- `/profile` — Account page: My Businesses management + profile details

### Features
- **Multi-business architecture**: Each user can create and manage multiple business profiles; active business shown in Navbar switcher
- **BusinessContext**: `useBusiness()` hook provides `businesses`, `activeBusiness`, `setActiveBusiness`, `createBusiness`, `updateBusiness`, `deleteBusiness`; active business ID persisted in `localStorage`
- **Business switcher in Navbar**: dropdown to switch active business, quick-add link to profile
- **My Businesses section in Profile**: inline add/edit/delete forms; category-grid picker, logo upload, brand color; first-business onboarding prompt
- **Category-aware content engine**: tailors headline/subtext/caption by business type (Logistics, Restaurant, Auto Gallery, Night Club, Bar, Pub, Butcher, Pharmacy)
- **Post size selector**: Square (1080×1080) for all platforms; Story (1080×1920) only for Instagram and Facebook
- **Logo integration**: logo renders on card corner; falls back to initial letter badge
- **Platform selection**: Twitter/X, LinkedIn, Instagram, Facebook
- **Tone selection**: Professional, Casual, Witty, Inspirational, Educational
- **Category-specific example prompts** on the content step
- **Post generation with hashtags** — category-based hashtag base
- **Three card templates**: Bold, Minimal, Split (AI-selected based on category + value signal)
- **Edit mode**: inline edits for headline, subtext, caption with live preview
- **Font system**: 10 fonts with searchable dropdown; separate headline/subtext fonts
- **Text controls**: Auto/White/Dark/Brand color; Top/Center/Bottom position
- **Dynamic font scaling**: headline font size auto-reduces to prevent overflow; `fitTextToWidth` for canvas export
- **AI image generation**: "AI Generate" tab in the background image section sends a prompt to `POST /api/images/generate-ai` → `gpt-image-1` via OpenAI integration → returns base64 PNG set as `uploadedImage`; Upload tab still available; tab state is `imageSource: "upload"|"ai"`
- **Step 4 two-column layout**: page container expands `max-w-3xl→max-w-5xl` on step 4; left column holds controls panel + edit mode + caption + action buttons; right column is sticky with post-size selector, live card preview, pan/zoom hint, and quick Export PNG button
- **Image upload**: background image with smart scrim overlay + auto text positioning
- **Image drag/zoom (enhanced)**: drag to pan freely (NO clamping, image can go beyond frame showing background), scroll-wheel zoom 0.2×–5×; `imagePos` is fraction-based offset (0=centered), CSS uses `transform: translate(x*100%, y*100%) scale(zoom)`; canvas export mirrors exact same math
- **Background color control**: 6 preset swatches (Navy/Black/White/Slate/Stone/Brand) + custom color picker; `bgColor` state (null=NAVY default); applied to card and canvas export
- **Separate headline/subtext colors**: `headlineColor` and `subtextColor` states; 7 color swatches + custom picker per text layer; reset per-layer independently; canvas export uses these overrides
- **Layout variation thumbnails**: 5 mini-preview buttons (Center/Top/Bottom/Minimal/Split) override AI template with one click; AI suggestion shown by default
- **Typography manual override**: "AI Auto" vs "Manual" toggle; headline 16–160px slider+input; subtext 10–60px; both synced to canvas export
- **Logo-only on cards**: brand name text and platform badge removed from card header; only logo (or monogram badge) shown
- **Yellow accent removed**: accent bars, decorative elements and glows now use `brandColor` (falls back to `#F5C842` if no brand color set); YELLOW constant kept for compatibility
- **Headline word-breaking fixed**: `wordBreak: "normal"`, `overflowWrap: "anywhere"`, `hyphens: "none"` — no more mid-word splits; removed `maxWidth: 14ch` constraint
- **Export**: high-quality PNG at selected size (1080×1080 or 1080×1920)
- **Share dialog**: Instagram (download+copy), Twitter/X, LinkedIn, Facebook; Copy caption button
- Save / delete generated posts; Real-time dashboard stats

### Design
- Minimal, premium aesthetic; Plus Jakarta Sans font; Deep indigo accent
- NAVY (#0B1F3A) card backgrounds; accent color uses `brandColor` (defaults to indigo)
- Soft text shadows (no harsh drop shadows); gradient scrim overlays

## API Endpoints

- `GET /api/healthz` — Health check
- `GET /api/brands` — List all businesses for current user
- `POST /api/brands` — Create a new business
- `PUT /api/brands/:id` — Update a business by ID
- `DELETE /api/brands/:id` — Delete a business by ID
- `GET /api/brand` — Legacy: get first business (backward compat)
- `POST /api/brand` — Legacy: create/update first business (backward compat)
- `POST /api/posts/generate` — Generate a social post
- `GET /api/posts` — List saved posts
- `POST /api/posts` — Save a post
- `DELETE /api/posts/:id` — Delete a post
- `GET /api/posts/stats` — Get post generation statistics

## Database Tables

- `brand` — Business settings (name, category, primaryColor, logoUrl)
- `posts` — Saved generated posts (postType, topic, tone, content, hashtags)

## TypeScript & Composite Projects

Every package extends `tsconfig.base.json` which sets `composite: true`. The root `tsconfig.json` lists all packages as project references. This means:

- **Always typecheck from the root** — run `pnpm run typecheck` (which runs `tsc --build --emitDeclarationOnly`). This builds the full dependency graph so that cross-package imports resolve correctly. Running `tsc` inside a single package will fail if its dependencies haven't been built yet.
- **`emitDeclarationOnly`** — we only emit `.d.ts` files during typecheck; actual JS bundling is handled by esbuild/tsx/vite...etc, not `tsc`.
- **Project references** — when package A depends on package B, A's `tsconfig.json` must list B in its `references` array. `tsc --build` uses this to determine build order and skip up-to-date packages.

## Root Scripts

- `pnpm run build` — runs `typecheck` first, then recursively runs `build` in all packages that define it
- `pnpm run typecheck` — runs `tsc --build --emitDeclarationOnly` using project references

## Packages

### `artifacts/api-server` (`@workspace/api-server`)

Express 5 API server. Routes live in `src/routes/` and use `@workspace/api-zod` for request and response validation and `@workspace/db` for persistence.

- Entry: `src/index.ts` — reads `PORT`, starts Express
- App setup: `src/app.ts` — mounts CORS, JSON/urlencoded parsing, routes at `/api`
- Routes: `src/routes/index.ts` mounts sub-routers; `src/routes/health.ts`, `brand.ts`, `posts.ts`
- Depends on: `@workspace/db`, `@workspace/api-zod`
- `pnpm --filter @workspace/api-server run dev` — run the dev server
- `pnpm --filter @workspace/api-server run build` — production esbuild bundle (`dist/index.cjs`)

### `artifacts/postcraft` (`@workspace/postcraft`)

React + Vite frontend. Uses shadcn/ui, Tailwind CSS, wouter for routing, TanStack Query.

- Entry: `src/main.tsx` → `src/App.tsx`
- Pages: `src/pages/landing.tsx`, `src/pages/dashboard.tsx`, `src/pages/generate.tsx`
- Depends on: `@workspace/api-client-react`

### `lib/db` (`@workspace/db`)

Database layer using Drizzle ORM with PostgreSQL. Exports a Drizzle client instance and schema models.

- `src/index.ts` — creates a `Pool` + Drizzle instance, exports schema
- `src/schema/brand.ts` — brand table
- `src/schema/posts.ts` — posts table
- `drizzle.config.ts` — Drizzle Kit config (requires `DATABASE_URL`, automatically provided by Replit)

Production migrations are handled by Replit when publishing. In development, we just use `pnpm --filter @workspace/db run push`, and we fallback to `pnpm --filter @workspace/db run push-force`.

### `lib/api-spec` (`@workspace/api-spec`)

Owns the OpenAPI 3.1 spec (`openapi.yaml`) and the Orval config (`orval.config.ts`). Running codegen produces output into two sibling packages.

Run codegen: `pnpm --filter @workspace/api-spec run codegen`

### `lib/api-zod` (`@workspace/api-zod`)

Generated Zod schemas from the OpenAPI spec.

### `lib/api-client-react` (`@workspace/api-client-react`)

Generated React Query hooks and fetch client from the OpenAPI spec.
