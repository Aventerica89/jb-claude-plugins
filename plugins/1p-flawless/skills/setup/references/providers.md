# OAuth Provider Reference

Environment variable names and dev app setup instructions per provider and framework.

---

## Environment Variable Names

### Better Auth

| Provider | Variable | Notes |
|----------|----------|-------|
| Google | `GOOGLE_CLIENT_ID` | |
| Google | `GOOGLE_CLIENT_SECRET` | |
| GitHub | `GITHUB_CLIENT_ID` | |
| GitHub | `GITHUB_CLIENT_SECRET` | |
| Todoist | `TODOIST_CLIENT_ID` | |
| Todoist | `TODOIST_CLIENT_SECRET` | |

Better Auth config pattern:
```ts
// src/lib/auth.ts
export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
  },
})
```

### NextAuth / Auth.js v5

| Provider | Variable | Notes |
|----------|----------|-------|
| Google | `AUTH_GOOGLE_ID` | Note: `AUTH_` prefix |
| Google | `AUTH_GOOGLE_SECRET` | |
| GitHub | `AUTH_GITHUB_ID` | |
| GitHub | `AUTH_GITHUB_SECRET` | |

Auth.js v5 config pattern:
```ts
// auth.ts
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [Google],  // reads AUTH_GOOGLE_ID / AUTH_GOOGLE_SECRET automatically
})
```

---

## Provider-Specific Dev App Setup

### Google OAuth

**Console:** https://console.cloud.google.com/apis/credentials

**Steps:**
1. Create a new project (or use existing)
2. Enable Google Identity API
3. OAuth consent screen → External → Testing mode
4. Add test users (required for Testing mode — only listed emails can auth)
5. Create OAuth client: Application type = **Web application**
6. Authorized redirect URIs (add all ports you use):
   - `http://localhost:3000/api/auth/callback/google`
   - `http://localhost:3001/api/auth/callback/google`
   - `http://localhost:3002/api/auth/callback/google`
7. Copy Client ID and Client Secret

**Critical:** Google only allows redirect URIs (not JavaScript origins) for server-side OAuth. Sensitive scopes (profile, email) require Testing mode + listed test users.

**Better Auth callback path:** `/api/auth/callback/google`
**NextAuth v5 callback path:** `/api/auth/callback/google`

---

### GitHub OAuth

**Console:** https://github.com/settings/developers → New OAuth App

**Steps:**
1. Application name: `{project} (dev)`
2. Homepage URL: `http://localhost:3000`
3. Authorization callback URL: `http://localhost:3000/api/auth/callback/github`
4. Register application
5. Copy Client ID; generate and copy Client Secret

**Note:** GitHub OAuth Apps support only a single callback URL. Create a separate "dev" app to avoid conflicts with production.

**Better Auth callback path:** `/api/auth/callback/github`
**NextAuth v5 callback path:** `/api/auth/callback/github`

---

### Todoist

**Console:** https://developer.todoist.com/appconsole.html → Create a new app

**Steps:**
1. App name: `{project} dev`
2. App service URL: `http://localhost:3000`
3. OAuth redirect URL: `http://localhost:3000/api/auth/callback/todoist`
4. Note the Client ID; generate and copy the Client Secret

**Note:** Todoist requires a separate app per environment. Do not reuse production app credentials for local dev.

**Better Auth callback path:** `/api/auth/callback/todoist`

---

## Detecting Providers from Auth Config

### Better Auth — Provider Detection

Search for `socialProviders` block in the auth config file:

```bash
grep -A 30 "socialProviders" src/lib/auth.ts
```

Provider keys in `socialProviders` map directly to credential variable names. Example:
- `google:` → needs `GOOGLE_CLIENT_ID` + `GOOGLE_CLIENT_SECRET`
- `github:` → needs `GITHUB_CLIENT_ID` + `GITHUB_CLIENT_SECRET`
- `todoist:` → needs `TODOIST_CLIENT_ID` + `TODOIST_CLIENT_SECRET`

### NextAuth v5 — Provider Detection

Search for imported providers:

```bash
grep -E "import .* from ['\"]next-auth/providers/" auth.ts src/auth.ts
```

Provider import name maps to env var prefix. Example:
- `import Google` → reads `AUTH_GOOGLE_ID` + `AUTH_GOOGLE_SECRET`
- `import GitHub` → reads `AUTH_GITHUB_ID` + `AUTH_GITHUB_SECRET`

---

## Adding an Unsupported Provider

For providers not in this list:

1. Ask the user: "What environment variable names does your auth config use for {provider}?"
2. Use those names directly as 1Password field names
3. Find the provider's OAuth dev console in their developer documentation
4. Follow the same pattern: create dev app, get client ID + secret, add to 1Password item
