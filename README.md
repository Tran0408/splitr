# Splitr — AI-Powered Expense Splitting

Splitr is a full-stack Splitwise-style expense-splitting app. Users can track shared expenses, split bills equally or by percentage/exact amounts, settle up, and receive automated payment reminders and AI-generated spending insights. It's built with Next.js 15, Convex as the realtime backend/database, Clerk for authentication, Google Gemini for AI insights, Inngest for scheduled jobs, and Resend for transactional email.

![Splitr](https://github.com/user-attachments/assets/11e138c4-efcf-4a85-8586-f2993da118d8)

---

## Features

- **Expense Tracking** — record shared expenses with description, amount, category, and date; attach them to a group or to a one-on-one relationship.
- **Flexible Splits** — split by `equal`, `percentage`, or `exact` amounts, with a breakdown of who owes what and what's been paid.
- **Groups** — create groups (trip, household, team, etc.), invite members, and view group balances and member lists.
- **Contacts & One-on-One Balances** — see everyone you've split with, their running balance, and per-person expense/settlement history.
- **Settlements** — record partial or full settlements between users, optionally tied to specific expenses.
- **Dashboard** — at-a-glance view of total owed/owing, recent activity, and monthly spending charts (via `recharts`).
- **Automated Payment Reminders** — daily Inngest cron emails users who still owe money, listing their outstanding debts.
- **AI Spending Insights** — monthly Gemini-generated summary of each user's spending patterns, delivered by email.
- **Authentication** — Clerk sign-in / sign-up with protected app routes.
- **Realtime UI** — Convex reactive queries keep balances and activity in sync across clients.
- **Light / Dark Mode** — theme toggle via `next-themes`.

---

## Tech Stack

| Layer              | Tech                                                                     |
| ------------------ | ------------------------------------------------------------------------ |
| Framework          | [Next.js 15](https://nextjs.org/) (App Router, Turbopack dev)            |
| Language           | JavaScript / JSX, React 19                                               |
| Styling            | Tailwind CSS v4, shadcn/ui + Radix primitives, `tw-animate-css`          |
| Auth               | [Clerk](https://clerk.com/) (with Convex JWT integration)                |
| Database / Backend | [Convex](https://www.convex.dev/) (schema, queries, mutations, search)   |
| AI                 | [Google Generative AI](https://ai.google.dev/) (Gemini 1.5 Flash)        |
| Background Jobs    | [Inngest](https://www.inngest.com/) (daily reminders, monthly insights)  |
| Email              | [Resend](https://resend.com/)                                            |
| Forms / Validation | `react-hook-form` + `zod`                                                |
| Charts             | `recharts`                                                               |
| Webhooks           | `svix` (Clerk webhook signature verification)                            |

---

## Project Structure

```
splitr/
├── app/
│   ├── (auth)/             # Clerk sign-in / sign-up route group
│   ├── (main)/             # Protected app routes
│   │   ├── dashboard/
│   │   ├── expenses/
│   │   ├── groups/
│   │   ├── contacts/
│   │   ├── person/
│   │   └── settlements/
│   ├── api/inngest/        # Inngest webhook endpoint
│   ├── layout.js           # Root layout (Clerk + Convex providers)
│   └── page.jsx            # Marketing/landing page
├── components/
│   ├── convex-client-provider.jsx
│   ├── expense-list.jsx
│   ├── group-balances.jsx
│   ├── group-members.jsx
│   ├── settlement-list.jsx
│   ├── header.jsx
│   └── ui/                 # shadcn/ui primitives
├── convex/
│   ├── schema.js           # users, expenses, settlements, groups
│   ├── users.js            # user queries/mutations + search
│   ├── expenses.js
│   ├── settlements.js
│   ├── groups.js
│   ├── contacts.js
│   ├── dashboard.js
│   ├── email.js            # Resend integration
│   ├── inngest.js          # Server functions consumed by Inngest jobs
│   ├── seed.js
│   └── auth.config.js      # Clerk JWT config for Convex
├── lib/
│   ├── inngest/
│   │   ├── client.js
│   │   ├── payment-reminders.js    # daily 10:00 UTC
│   │   └── spending-insights.js    # monthly, 1st @ 08:00 UTC
│   ├── expense-categories.js
│   ├── landing.js
│   └── utils.js
├── hooks/
│   ├── use-convex-query.js
│   └── use-store-user.jsx
├── middleware.js           # Clerk auth + protected route matcher
└── next.config.mjs
```

### Data Model (Convex)

- **users** — `{ name, email, tokenIdentifier, imageUrl }`, indexed by token/email with search indexes on name and email.
- **expenses** — `{ description, amount, category, date, paidByUserId, splitType, splits[], groupId?, createdBy }`. `splits` is an array of `{ userId, amount, paid }`.
- **settlements** — `{ amount, note?, date, paidByUserId, receivedByUserId, groupId?, relatedExpenseIds?, createdBy }`.
- **groups** — `{ name, description?, createdBy, members[] }` where each member has `{ userId, role, joinedAt }`.

---

## Getting Started

### Prerequisites

- Node.js 18.18+ (Node 20 LTS recommended)
- A [Convex](https://www.convex.dev/) account
- A [Clerk](https://clerk.com/) application (configured to issue JWTs for Convex)
- A [Google AI Studio](https://aistudio.google.com/) API key for Gemini
- A [Resend](https://resend.com/) API key for outbound email

### 1. Clone and install

```bash
git clone https://github.com/Tran0408/splitr.git
cd splitr
npm install
```

### 2. Configure environment variables

Create a `.env` file in the repo root:

```env
# Convex
CONVEX_DEPLOYMENT=
NEXT_PUBLIC_CONVEX_URL=

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
CLERK_JWT_ISSUER_DOMAIN=

# Email (Resend)
RESEND_API_KEY=

# Gemini
GEMINI_API_KEY=
```

`CLERK_JWT_ISSUER_DOMAIN` must match the domain of the JWT template you configured in Clerk for Convex. See [convex/auth.config.js](convex/auth.config.js).

### 3. Start Convex

```bash
npx convex dev
```

This syncs `convex/schema.js` and the query/mutation files to your deployment and fills in `CONVEX_DEPLOYMENT` / `NEXT_PUBLIC_CONVEX_URL`.

### 4. Start the dev server

```bash
npm run dev
```

The app runs on [http://localhost:3000](http://localhost:3000) with Turbopack.

### 5. (Optional) Run Inngest locally

To exercise the scheduled functions locally:

```bash
npx inngest-cli@latest dev
```

Point it at `http://localhost:3000/api/inngest`.

---

## Available Scripts

| Script          | Description                                    |
| --------------- | ---------------------------------------------- |
| `npm run dev`   | Start the Next.js dev server with Turbopack.   |
| `npm run build` | Production build.                              |
| `npm run start` | Start the production server (after `build`).   |
| `npm run lint`  | Run ESLint with the Next.js config.            |

---

## Protected Routes

The following route groups require a signed-in Clerk user (see [middleware.js](middleware.js)):

- `/dashboard`
- `/expenses`
- `/contacts`
- `/groups`
- `/person`
- `/settlements`

Unauthenticated requests are redirected to Clerk's sign-in.

---

## Scheduled Jobs

| Job                      | Schedule (UTC)    | What it does                                                                                   |
| ------------------------ | ----------------- | ---------------------------------------------------------------------------------------------- |
| `send-payment-reminders` | `0 10 * * *`      | Emails every user a summary of their outstanding debts (see [lib/inngest/payment-reminders.js](lib/inngest/payment-reminders.js)). |
| `generate-spending-insights` | `0 8 1 * *`   | On the 1st of each month, asks Gemini to summarize the prior month's expenses per user and emails the insight (see [lib/inngest/spending-insights.js](lib/inngest/spending-insights.js)). |

Both functions pull data from Convex via `ConvexHttpClient` and send mail through Resend.

---

## Deployment

The app is optimized for Vercel:

1. Import the repo into Vercel.
2. Set all environment variables listed above.
3. Deploy Convex to production (`npx convex deploy`) and use the production `NEXT_PUBLIC_CONVEX_URL` / `CONVEX_DEPLOYMENT` in Vercel.
4. Register the `/api/inngest` endpoint in the Inngest dashboard and add your Inngest event/signing keys as env vars.
5. Configure a verified sender domain in Resend for reliable delivery.

---

## License

No license file is provided in this repository. All rights reserved by the original authors unless a license is added.

---

## Credits

Originally built following the "Full Stack AI Splitwise Clone" tutorial — see the [walkthrough video](https://youtu.be/Ce7O3p7-YDI).
