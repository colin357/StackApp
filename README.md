# Stack 📊

A personal finance app for people working toward big purchases — built for users referred by loan officers. Track net worth, build a monthly budget ("Game Plan"), set savings and debt-payoff goals, and monitor your credit, with real bank connections powered by [Plaid](https://plaid.com) and data stored in Firebase.

Built with Expo (React Native + TypeScript), Expo Router, Firebase Auth + Firestore, and Plaid Link (WebView flow — works in Expo Go, no dev build needed).

## Features

- **Sign up with a loan officer referral code** — stored on the user profile so officers can be credited.
- **Accounts** — link real bank accounts via Plaid (balances import automatically, pull-to-refresh updates them), or add assets/debts manually (checking, savings, investments, real estate, credit cards, mortgages, loans…).
- **Net worth dashboard** — live assets − debts, monthly spending vs. budget, goal progress.
- **Game Plan (budget)** — set monthly income and per-category limits; log transactions and watch category progress bars.
- **Goals** — savings goals (house down payment, emergency fund) and debt-payoff goals, with required-monthly-pace hints toward a target date.
- **Credit** — log credit scores over time with bureau, trend delta, and rating band (Good / Very Good / …).

## Setup

### 1. Install & run

```bash
npm install
cp .env.example .env   # then fill it in (see below)
npx expo start
```

Open in Expo Go (iOS/Android) or press `w` for web.

### 2. Firebase

1. Create a project at [console.firebase.google.com](https://console.firebase.google.com).
2. Enable **Authentication → Email/Password**.
3. Create a **Firestore** database and publish the rules in [firestore.rules](firestore.rules).
4. Add a **Web app**, copy its config into the `EXPO_PUBLIC_FIREBASE_*` vars in `.env`.

### 3. Plaid

For development/sandbox, just drop your keys in `.env`:

```
EXPO_PUBLIC_PLAID_ENV=sandbox
EXPO_PUBLIC_PLAID_CLIENT_ID=...
EXPO_PUBLIC_PLAID_SECRET=...   # sandbox secret
```

In Plaid sandbox, link any institution with credentials `user_good` / `pass_good`.

**Before production:** the secret must move server-side. Deploy a small backend (Firebase Cloud Function works well) exposing `/link/token/create`, `/item/public_token/exchange`, and `/accounts/balance/get` that verifies the Firebase ID token and calls Plaid with your production secret, then set `EXPO_PUBLIC_PLAID_BACKEND_URL` — the app switches to it automatically and the direct-call vars can be removed.

## Data model (Firestore)

```
users/{uid}                    profile: name, email, referralCode, createdAt
users/{uid}/accounts/{id}      name, type, balance (cents), source: manual|plaid, apr?
users/{uid}/plaidItems/{id}    Plaid item access token + institution
users/{uid}/transactions/{id}  description, amount (cents, neg = spend), category, date
users/{uid}/budgets/{YYYY-MM}  monthlyIncome, limits: {category: cents}
users/{uid}/goals/{id}         kind: savings|debt_payoff, targetAmount, currentAmount, targetDate?
users/{uid}/creditScores/{id}  score, bureau, date
```

All money values are stored as integer cents. Plaid access tokens live under the user's own Firestore subtree (rule-protected); move them behind the backend before going to production.
