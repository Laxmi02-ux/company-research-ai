# Dossier — AI Company Research Assistant

An AI-powered research assistant that takes a **company name** or a **website URL**,
crawls the company's site, searches the public web, runs AI analysis, identifies
competitors, and produces a downloadable PDF dossier — all through a ChatGPT-style
chat interface.

Built as a single unified Next.js 14 app (frontend + backend API routes), ready to
deploy on Vercel.

---

## Features

- **Dual input** — accepts a company name (`Tesla`) or a URL (`https://tesla.com`).
- **Website crawler** — discovers and ranks internal pages (About, Products, Pricing,
  Contact, etc.), skips login/duplicate/irrelevant pages, extracts clean text.
- **Serper.dev search integration** — resolves the official website from a company
  name, gathers contact details, and seeds competitor discovery.
- **OpenRouter AI integration** — model-selectable (dropdown in the header); returns
  a structured JSON dossier: summary, products, pain points, competitors.
- **Competitor analysis** — same-country, same-industry competitors with name +
  website.
- **PDF report generation** — one-click download, built server-side with `pdfkit`.
- **ChatGPT-style UI** — chat history, live progress states, inline report card,
  download button.
- **Discord integration (bonus)** — a `/settings` page for Bot Token + Channel ID
  (session-only, never stored server-side) and applicant name/email. After a report
  generates, it's automatically posted to the configured channel with the PDF
  attached.
- **No database, no auth** — fully stateless; only `sessionStorage` in the browser
  is used to remember Discord settings and the last report during a session.

---

## Tech Stack

| Layer       | Choice                                   |
|-------------|-------------------------------------------|
| Framework   | Next.js 14 (App Router, single project)   |
| UI          | React + Tailwind CSS                      |
| Crawling    | `fetch` + `cheerio`                       |
| Search      | Serper.dev REST API                       |
| AI          | OpenRouter REST API (model-selectable)    |
| PDF         | `pdfkit`                                  |
| Messaging   | Discord Bot REST API (`discord.com/api`)  |
| Deployment  | Vercel (or Netlify / Cloudflare Pages)    |

---

## Project Structure

```
app/
  page.tsx               # ChatGPT-style research UI
  settings/page.tsx       # Discord integration settings page
  api/research/route.ts   # Orchestrates search -> crawl -> AI analysis
  api/pdf/route.ts        # Generates the downloadable PDF
  api/discord/route.ts    # Sends report + PDF to a Discord channel
lib/
  serper.ts               # Serper.dev search helpers
  crawler.ts               # Website crawler (page discovery + text extraction)
  openrouter.ts             # OpenRouter AI call + model list
  pdfGenerator.ts           # PDF layout/generation (pdfkit)
  types.ts                  # Shared TypeScript types
```

---

## Setup

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Copy `.env.example` to `.env.local` and fill in your keys:

```bash
cp .env.example .env.local
```

| Variable              | Required | Description                                      |
|-----------------------|----------|---------------------------------------------------|
| `SERPER_API_KEY`      | Yes      | From [serper.dev](https://serper.dev)              |
| `OPENROUTER_API_KEY`  | Yes      | From [openrouter.ai/keys](https://openrouter.ai/keys) |

Discord **Bot Token** and **Channel ID** are *not* environment variables — they're
entered live on the `/settings` page, since the evaluator supplies them at test
time and the app has no persistent storage.

### 3. Run locally

```bash
npm run dev
```

Visit `http://localhost:3000`.

### 4. Build for production

```bash
npm run build
npm start
```

---

## Deployment (Vercel)

1. Push this repo to GitHub.
2. Import the repo in [vercel.com/new](https://vercel.com/new).
3. Add `SERPER_API_KEY` and `OPENROUTER_API_KEY` as Environment Variables in the
   Vercel project settings.
4. Deploy. Vercel auto-detects Next.js — no extra config needed.

The same steps apply to Netlify or Cloudflare Pages with their Next.js adapters.

---

## How It Works (Request Flow)

1. User submits a company name or URL in the chat.
2. If a name was given, the app calls Serper.dev to resolve the official website.
3. The crawler visits the homepage, discovers internal links, ranks them by
   relevance (about/products/pricing/contact), and extracts text from up to 6 pages.
4. In parallel, Serper.dev is queried for contact details and competitor seeds.
5. All crawled text + search snippets are sent to OpenRouter with a structured-JSON
   system prompt; the selected model returns summary, products, pain points, phone,
   address, industry, and competitors.
6. The result renders as a report card in the chat, with a **Download PDF** button.
7. If Discord is configured (`/settings`), the report + PDF are automatically
   posted to the channel right after generation.

---

## Known Limitations / Notes

- The crawler uses plain `fetch` (no headless browser), so heavily JS-rendered
  sites (client-side-only React/Vue sites with no server-rendered content) may
  yield limited text. This keeps the app fast and serverless-friendly.
- AI-generated pain points and competitor suggestions are inferences based on
  available public data — they're a research aid, not verified fact.
- No data is persisted anywhere; refreshing the page clears chat history (by
  design, per the "no database" requirement).
