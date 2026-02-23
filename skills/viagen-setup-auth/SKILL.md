---
name: viagen-setup-auth
description: >
  Configure sandbox auth bypass. Detects the auth framework in the user's Vite
  app and modifies it to auto-authenticate in viagen sandbox mode — bypassing
  OAuth flows that break with ephemeral sandbox URLs.
allowed-tools: Bash, Read, Edit, Grep, Glob, Write
---

# Sandbox Auth Bypass Setup

You are helping the user configure their Vite app to auto-authenticate when
running inside a viagen sandbox. Sandboxes get ephemeral `*.vercel.run` URLs
that can't be pre-registered with OAuth providers, so OAuth flows fail. The
fix is to detect sandbox mode and bypass OAuth entirely, creating a mock user
session from environment variables.

## How it works

When `VIAGEN_AUTH_TOKEN` is present in the environment the app is running in a
viagen sandbox. In that mode, OAuth flows are skipped and a mock user session
is created from these env vars:

- `VIAGEN_AUTH_EMAIL` — the real email of the user who launched the sandbox
  (injected automatically by the platform)
- `VIAGEN_SANDBOX_USER_EMAIL` — override mock user email (falls back to
  `VIAGEN_AUTH_EMAIL` if not set)
- `VIAGEN_SANDBOX_USER_NAME` — mock user name (default: `"Sandbox User"`)
- `VIAGEN_SANDBOX_USER_ID` — mock user ID (default: `"sandbox-user-1"`)
- `VIAGEN_SANDBOX_USER_IMAGE` — mock user avatar URL (optional)

The bypass ONLY activates when `VIAGEN_AUTH_TOKEN` is set. It is never active
in production or normal local development.

---

## Step 1: Detect auth framework

Read `package.json` and check **dependencies** and **devDependencies** for:

| Package(s) | Framework |
|------------|-----------|
| `@auth/core`, `@auth/sveltekit`, `@auth/solid-start` | Auth.js |
| `@clerk/clerk-react`, `@clerk/remix`, `@clerk/tanstack-start` | Clerk |
| `@supabase/ssr`, `@supabase/supabase-js` (with auth usage) | Supabase Auth |
| `better-auth` | Better Auth |
| `arctic` | Custom / Arctic |

If multiple matches or none, **ask the user** which auth framework they use.
If they confirm no auth, tell them no changes are needed and stop.

Once detected, confirm with the user: _"It looks like you're using [framework].
Is that correct?"_

## Step 2: Ask for mock user details

The platform automatically injects `VIAGEN_AUTH_EMAIL` with the real email of
the user who launched the sandbox. This is used as the default mock email so
the sandbox user matches the actual platform user.

Ask the user:
1. **Email** for the sandbox mock user (suggest using `VIAGEN_AUTH_EMAIL` —
   the user's real email — which is the default; or `dev@sandbox.viagen.dev`
   if they prefer a generic one)
2. **Name** (suggest `Sandbox User`)
3. **User ID** (suggest `sandbox-user-1`)

Use these values in all code modifications below.

## Step 3: Implement bypass

Follow the section for the detected framework. In every case, the bypass MUST
be gated behind `process.env.VIAGEN_AUTH_TOKEN` so it never activates outside
a sandbox.

---

### Auth.js (SvelteKit / SolidStart)

**Find the config**: look for the Auth.js config — typically `src/auth.ts`,
`src/hooks.server.ts` (SvelteKit), or the file that calls `SvelteKitAuth()` /
`SolidStartAuth()`.

**1. Add the Credentials provider import** (if not already imported):

```ts
import Credentials from "@auth/core/providers/credentials";
```

**2. Add a conditional sandbox provider** to the `providers` array:

```ts
const isSandbox = !!process.env.VIAGEN_AUTH_TOKEN;

export const { handle, signIn, signOut } = SvelteKitAuth({
  providers: [
    // ... existing providers
    ...(isSandbox
      ? [
          Credentials({
            id: "viagen-sandbox",
            credentials: {},
            async authorize() {
              return {
                id: process.env.VIAGEN_SANDBOX_USER_ID ?? "sandbox-user-1",
                email: process.env.VIAGEN_SANDBOX_USER_EMAIL ?? process.env.VIAGEN_AUTH_EMAIL ?? "dev@sandbox.viagen.dev",
                name: process.env.VIAGEN_SANDBOX_USER_NAME ?? "Sandbox User",
                image: process.env.VIAGEN_SANDBOX_USER_IMAGE ?? null,
              };
            },
          }),
        ]
      : []),
  ],
  // ...
});
```

**3. Force JWT sessions in sandbox** (CredentialsProvider requires JWT):

If the config uses `session: { strategy: "database" }`, change it to:
```ts
session: { strategy: isSandbox ? "jwt" : "database" },
```

**4. Add auto-login on first load**: Find the root layout or a top-level
component that runs on every page. Add logic that checks for an active session
and, if none exists and `VIAGEN_AUTH_TOKEN` is set (passed as a page prop or
server data), triggers `signIn("viagen-sandbox")` automatically.

For SvelteKit this would be in `+layout.server.ts` or `+layout.svelte`.
For SolidStart this would be in the root route.

**Required secret**: `AUTH_SECRET` — ask the user for their Auth.js secret (or
generate a random one for sandbox use). Write it to `.env`.

---

### Clerk

Clerk is a hosted auth service — sessions are managed by Clerk's servers, so
we cannot create sessions from env vars alone. This approach requires the user
to generate a token from the Clerk dashboard.

**1. Tell the user** to:
- Go to Clerk Dashboard → Users → create a test user (or use an existing one)
- Go to that user → Sessions → create a long-lived session token
- Copy the session JWT

**2. Ask the user** to paste the session JWT.

**3. Find the middleware** file (`middleware.ts` or similar with `clerkMiddleware`).

**4. Add a sandbox bypass** before the Clerk middleware runs:

```ts
import { clerkMiddleware } from "@clerk/remix"; // or @clerk/tanstack-start etc.

export default function middleware(request: Request) {
  // Sandbox bypass: inject Clerk session from env
  if (process.env.VIAGEN_AUTH_TOKEN && process.env.VIAGEN_SANDBOX_CLERK_SESSION) {
    const response = new Response(null, { status: 200 });
    response.headers.set(
      "Set-Cookie",
      `__session=${process.env.VIAGEN_SANDBOX_CLERK_SESSION}; Path=/; HttpOnly; SameSite=Lax`
    );
    // Only set cookie on first request (check if already set)
    if (!request.headers.get("cookie")?.includes("__session")) {
      return response;
    }
  }

  return clerkMiddleware()(request);
}
```

**Note**: Tell the user that the session token will expire based on their Clerk
settings and will need to be regenerated periodically.

**Required secrets**: `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY` (likely
already set), plus `VIAGEN_SANDBOX_CLERK_SESSION` (the JWT from above).

---

### Supabase Auth

Supabase supports email/password sign-in, so we can auto-authenticate using a
test user's credentials.

**1. Tell the user** to create a test user in their Supabase dashboard:
- Go to Authentication → Users → Add user
- Use email/password (not magic link)
- Disable email confirmation for this user or confirm them manually

**2. Ask the user** for the test user's email and password.

**3. Create a sandbox auth helper**. Find the directory where Supabase client
utilities live (e.g. `lib/`, `utils/`, `src/lib/`) and create
`viagen-sandbox-auth.ts`:

```ts
import type { SupabaseClient } from "@supabase/supabase-js";

/**
 * In viagen sandbox mode, auto-authenticate with a test user.
 * Call this from your root layout's server-side logic.
 */
export async function ensureSandboxSession(supabase: SupabaseClient) {
  if (!process.env.VIAGEN_AUTH_TOKEN) return;

  const { data: { session } } = await supabase.auth.getSession();
  if (session) return;

  const email = process.env.VIAGEN_SANDBOX_SUPABASE_EMAIL ?? process.env.VIAGEN_AUTH_EMAIL;
  const password = process.env.VIAGEN_SANDBOX_SUPABASE_PASSWORD;
  if (!email || !password) return;

  const { error } = await supabase.auth.signInWithPassword({ email, password });
  if (error) {
    console.warn("[viagen] sandbox auto-login failed:", error.message);
  }
}
```

**4. Call the helper** from the root server layout or middleware. Find where the
Supabase server client is created (e.g. `+layout.server.ts` in SvelteKit,
`root.tsx` loader in React Router) and add:

```ts
import { ensureSandboxSession } from "~/lib/viagen-sandbox-auth";

// In the loader or server function:
await ensureSandboxSession(supabase);
```

**Required secrets**: `SUPABASE_URL`, `SUPABASE_ANON_KEY` (likely already set),
plus `VIAGEN_SANDBOX_SUPABASE_EMAIL` and `VIAGEN_SANDBOX_SUPABASE_PASSWORD`.

---

### Better Auth

**Find the config**: look for the file that calls `betterAuth()` — typically
`lib/auth.ts`, `server/auth.ts`, or similar.

**1. Add a sandbox bypass hook** in the Better Auth config. Better Auth
supports `onRequest` or `before` hooks depending on version. Add a hook that
creates a mock session when `VIAGEN_AUTH_TOKEN` is set:

```ts
import { betterAuth } from "better-auth";

const isSandbox = !!process.env.VIAGEN_AUTH_TOKEN;

export const auth = betterAuth({
  // ... existing config
  advanced: {
    ...(isSandbox && {
      generateId: () => process.env.VIAGEN_SANDBOX_USER_ID ?? "sandbox-user-1",
    }),
  },
  databaseHooks: {
    session: {
      create: {
        before: isSandbox
          ? async (session) => {
              // Ensure sandbox sessions get the mock user
              return { data: session };
            }
          : undefined,
      },
    },
  },
});
```

**2. Create a sandbox bootstrap script**. Create `lib/viagen-sandbox-auth.ts`:

```ts
/**
 * In viagen sandbox mode, ensure a mock user and session exist.
 * Call this once on server startup or first request.
 */
export async function ensureSandboxUser(auth: ReturnType<typeof import("better-auth").betterAuth>) {
  if (!process.env.VIAGEN_AUTH_TOKEN) return;

  const email = process.env.VIAGEN_SANDBOX_USER_EMAIL ?? process.env.VIAGEN_AUTH_EMAIL ?? "dev@sandbox.viagen.dev";
  const name = process.env.VIAGEN_SANDBOX_USER_NAME ?? "Sandbox User";
  const password = "viagen-sandbox-" + (process.env.VIAGEN_AUTH_TOKEN ?? "dev");

  // VIAGEN_AUTH_EMAIL is injected by the platform with the real user's email

  // Try to sign in first; if user doesn't exist, sign up
  try {
    await auth.api.signInEmail({ body: { email, password } });
  } catch {
    await auth.api.signUpEmail({ body: { email, password, name } });
  }
}
```

**3. Wire it up** in the root server handler or middleware, calling
`ensureSandboxUser(auth)` on first request.

**Required secrets**: `BETTER_AUTH_SECRET` (or whatever the app uses for its
auth secret).

---

### Custom / Arctic

This covers apps with hand-rolled auth using Arctic or any other OAuth library.

**1. Find the session validation function**. Search for functions like
`getSession`, `requireAuth`, `validateSession`, `getUser`, or similar. These
typically read a cookie, validate a session token, and return a user object.

**2. Add an early return** at the top of the function:

```ts
export async function getSession(request: Request) {
  // Viagen sandbox bypass: auto-authenticate with mock user
  if (process.env.VIAGEN_AUTH_TOKEN) {
    return {
      user: {
        id: process.env.VIAGEN_SANDBOX_USER_ID ?? "sandbox-user-1",
        email: process.env.VIAGEN_SANDBOX_USER_EMAIL ?? process.env.VIAGEN_AUTH_EMAIL ?? "dev@sandbox.viagen.dev",
        name: process.env.VIAGEN_SANDBOX_USER_NAME ?? "Sandbox User",
        avatarUrl: process.env.VIAGEN_SANDBOX_USER_IMAGE ?? null,
      },
    };
  }

  // ... existing session validation logic
}
```

Adapt the returned object shape to match what the app's existing code expects.
Read the existing return type and field names carefully.

**3. Optionally bypass the login page**: If the app has a login route, add a
redirect at the top that sends sandbox users straight to the dashboard/home:

```ts
// In the login route loader
if (process.env.VIAGEN_AUTH_TOKEN) {
  return redirect("/");
}
```

**Required secrets**: Only the `VIAGEN_SANDBOX_USER_*` env vars — no
framework-specific secrets needed.

---

## Step 4: Write `.env`

Append the following to the project's `.env` file (create it if it doesn't
exist). Do NOT overwrite existing values — only add new keys.

For **all frameworks**, add:
```
# Viagen sandbox auth bypass (set to "test" for local testing)
VIAGEN_AUTH_TOKEN=test
VIAGEN_SANDBOX_USER_EMAIL=<user's chosen email>
VIAGEN_SANDBOX_USER_NAME=<user's chosen name>
VIAGEN_SANDBOX_USER_ID=<user's chosen ID>
```

For **framework-specific secrets**, add the values the user provided in the
previous steps (e.g. `AUTH_SECRET`, `VIAGEN_SANDBOX_SUPABASE_EMAIL`, etc.).

## Step 5: Verify & next steps

Tell the user:

1. **Test locally**: Run `npm run dev` and verify you are auto-logged in as the
   mock user. The auth bypass activates because `VIAGEN_AUTH_TOKEN=test` is in
   `.env`.

2. **Push to platform**: Run `npx viagen sync` to push the sandbox secrets to
   the viagen platform. Once synced, every sandbox launched for this project
   will auto-authenticate.

3. **Important**: When you're done testing locally, remove `VIAGEN_AUTH_TOKEN=test`
   from `.env` so normal OAuth works again in local development. The variable is
   automatically set by the sandbox system — you only need it in `.env` for
   local testing.

## Notes

- The bypass is gated behind `process.env.VIAGEN_AUTH_TOKEN` — it is **never
  active** in production or normal development
- Never deploy bypass code to production with `VIAGEN_AUTH_TOKEN` set in the
  environment
- If the user's app has role-based access, the mock user defaults to a basic
  user. Ask if they need a specific role and adjust accordingly
- For apps with database sessions, the mock user may need to exist in the
  database. Guide the user through creating a seed if necessary
