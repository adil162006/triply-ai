# Triply

Triply is a mobile-first AI travel planner. It turns a destination, dates, group size, budget, interests, and travel pace into a saved itinerary and budget breakdown, with a conversational assistant for travel questions.

## What it does

- Create, browse, and manage AI-generated trips.
- Generate itineraries and budget breakdowns asynchronously, with trip status updates.
- Ask a separate AI travel assistant questions and revisit or clear its transcript.
- Sign in with Clerk, including Google and Apple sign-in flows.
- Add a personal trip cover photo; generated and uploaded images are delivered through ImageKit.

## Technology

| Area | Technology | Role |
| --- | --- | --- |
| Mobile app | Expo SDK 56, React Native 0.85, React 19, TypeScript | Cross-platform native application |
| Navigation and UI | Expo Router, Expo native tabs, NativeWind v4, `@expo/ui` | File-based routes, native tab navigation, and styling/components |
| Authentication | Clerk (`@clerk/expo`) | User sessions and OAuth sign-in |
| Data | Neon Postgres, Drizzle ORM | Persistent users, trips, chat transcripts, and generation usage |
| AI | OpenAI | Trip plans, budgets, and streaming assistant responses |
| Background work | Inngest | Durable trip generation and Clerk user synchronization |
| Images | Unsplash, ImageKit | Trip imagery, uploads, optimization, and delivery |
| Monitoring | Sentry | Mobile error tracking, traces, logs, and session replay |

## Application flow

1. A user signs in through Clerk and enters the protected native-tab area.
2. The trip form submits preferences to `POST /api/trips`. The API validates the request, reserves the daily generation allowance, stores a pending trip, and dispatches an Inngest event.
3. An Inngest function generates an itinerary and budget with OpenAI, optionally obtains a destination cover image, then stores the result. The loading screen polls the trip status route.
4. Users view and delete saved trips. The travel companion has a per-user transcript that can be revisited or cleared.
5. Clerk user webhooks are verified and sent to Inngest for database synchronization.

## Project structure

```text
src/
  app/                 Expo Router screens and API route handlers
    (auth)/             Sign-in routes
    (home)/             Home, assistant, trips, and profile tabs
    api/                Trips, assistant, Inngest, and Clerk webhook endpoints
    trip/               Trip detail and generation loading screens
  components/           Shared UI components
  db/                   Drizzle schema and Neon database client
  hooks/                React hooks, including sign-in helpers
  inngest/              Inngest client and background functions
  lib/                  API clients, auth, AI, image, and trip utilities
assets/                 App images and icons
design/                 UI design references
legal/                  Privacy, terms, and support pages
```

## Prerequisites

- Node.js and npm (the repository includes `package-lock.json`).
- Expo-compatible iOS or Android development tooling for native builds; the project includes `expo-dev-client`.
- Credentials for Clerk, Neon Postgres, OpenAI, Unsplash, ImageKit, and the local Inngest development server.

## Setup

1. Install dependencies:

   ```sh
   npm install
   ```

2. Create a local environment file from the template:

   ```sh
   cp .env.example .env
   ```

   On Windows PowerShell, use `Copy-Item .env.example .env`.

3. Fill in the credentials described below. Keep `.env` private; it is ignored by Git.

4. Create a Neon Postgres database and set `DATABASE_URL` to its connection string. Apply the schema using Drizzle (see [Database](#database)).

5. Start the Expo development server and, in a separate terminal, start the Inngest development server:

   ```sh
   npm run start
   npm run inngest
   ```

   The Inngest script points to `http://localhost:8081/api/inngest`. If Expo uses another port, update that URL accordingly.

For a native development build, use `npm run ios` or `npm run android`. Web is available with `npm run web`.

## Environment variables

`.env.example` is the authoritative list of environment variable names. Add values to your local `.env`; do not commit secrets.

| Variable | Purpose |
| --- | --- |
| `EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk client configuration; required by the app at startup |
| `CLERK_SECRET_KEY` | Clerk server-side authentication |
| `CLERK_WEBHOOK_SIGNING_SECRET` | Verifies incoming Clerk webhooks |
| `DATABASE_URL` | Neon Postgres connection string |
| `OPENAI_API_KEY` | Trip plan and assistant generation |
| `UNSPLASH_ACCESS_KEY` | Search for destination cover images |
| `UNSPLASH_SECRET_KEY` | Present in the environment template for Unsplash credentials |
| `IMAGEKIT_PRIVATE_KEY` | Server-side ImageKit uploads |
| `IMAGEKIT_PUBLIC_KEY` | ImageKit configuration |
| `IMAGEKIT_URL_ENDPOINT` | ImageKit image delivery endpoint |
| `INNGEST_DEV` | Set to `1` for local Inngest development |
| `INNGEST_SIGNING_KEY`, `INNGEST_EVENT_KEY` | Inngest credentials marked in the template as production-only |

Configure Google and Apple OAuth providers in Clerk for the corresponding sign-in options. For deployed environments, configure Clerk’s webhook destination and Inngest keys to match the deployment.

## API routes

User-facing trip and assistant endpoints require a valid Clerk-authenticated request. The Clerk webhook verifies its signature, and the Inngest route is handled by the Inngest SDK. Responses and validation details are implemented in the route handlers.

| Method | Route | Purpose |
| --- | --- | --- |
| `GET`, `POST` | `/api/trips` | List the signed-in user’s trips; create a trip and queue generation |
| `GET`, `DELETE` | `/api/trips/:id` | Read or delete an owned trip |
| `GET` | `/api/trips/:id/status` | Read generation status and any safe failure message |
| `PATCH` | `/api/trips/:id/cover` | Upload and set a user-selected cover image |
| `POST` | `/api/assistant` | Stream an AI travel-assistant response |
| `GET`, `DELETE` | `/api/assistant/messages` | Read or clear the signed-in user’s assistant transcript |
| `GET`, `POST`, `PUT` | `/api/inngest` | Inngest function serving endpoint |
| `POST` | `/api/webhooks/clerk` | Verify Clerk user events and enqueue synchronization |

## Database

The Drizzle schema is in `src/db/schema.ts`. It defines users, trips, trip chat messages, assistant messages, and per-user daily generation usage. `drizzle.config.ts` reads `DATABASE_URL` and writes generated migrations to `drizzle/`.

Use the schema push command to apply the current schema directly to the configured database:

```sh
npm run db:push
```

To generate migration files instead, run `npm run db:generate`; apply generated migrations with `npx drizzle-kit migrate`. Inspect the database with `npm run db:studio`.

## Development commands

| Command | Description |
| --- | --- |
| `npm run start` | Start Expo development server |
| `npm run ios` | Run the iOS native development build |
| `npm run android` | Run the Android native development build |
| `npm run web` | Start Expo for web |
| `npm run inngest` | Start local Inngest dev server connected to the app endpoint |
| `npm run lint` | Run Expo ESLint |
| `npm run db:generate` | Generate Drizzle migration files |
| `npm run db:push` | Push schema changes to the configured database |
| `npm run db:studio` | Open Drizzle Studio |

No test script is currently defined in `package.json`.

## Screenshots

![Triply application screenshot](assets/images/screenshot-for-readme.png)

## License

See [LICENSE](LICENSE).
