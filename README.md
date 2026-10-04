# Next.js Authentication System

A modern authentication system built with **Next.js, Better Auth, MongoDB, and HeroUI**. This project includes user registration, login, session management, protected routes, Google OAuth authentication, and email verification.

## 🚀 Features

* **User Sign Up** — Create an account using name, email, and password.
* **User Sign In** — Authenticate users using email and password.
* **Session Management** — Track authenticated users with Better Auth.
* **Sign Out** — Securely sign out and redirect users to the login page.
* **Protected Routes** — Restrict access to private pages using `proxy.js`.
* **Google OAuth** — Allow users to authenticate with their Google accounts.
* **Email Verification** — Verify user email addresses using Resend.
* **Modern UI** — Build responsive authentication pages with HeroUI.
* **MongoDB Integration** — Store authentication data in MongoDB.

## 🛠️ Technologies Used

* Next.js
* React
* Better Auth
* MongoDB
* `@better-auth/mongo-adapter`
* HeroUI
* Google OAuth 2.0
* Resend

## 📦 Installation and Setup

### 1. Create a Next.js Project

Create a new Next.js application:

```bash
npx create-next-app@latest my-auth-app
```

Navigate to the project directory:

```bash
cd my-auth-app
```

### 2. Install Dependencies

Install Better Auth:

```bash
npm install better-auth
```

Install MongoDB:

```bash
npm install mongodb
```

Install the Better Auth MongoDB adapter:

```bash
npm install @better-auth/mongo-adapter
```

Install HeroUI:

Follow the official [HeroUI installation guide](https://www.heroui.com/docs/guide/installation) to configure the required packages and styles for your project.

### 3. Configure HeroUI Styles

Import HeroUI styles into your global CSS file, usually `app/globals.css`:

```css
@import "@heroui/styles";
```

## 🗄️ MongoDB Configuration

### 4. Set Up MongoDB

1. Create a MongoDB database using [MongoDB Atlas](https://www.mongodb.com/atlas) or a local MongoDB server.
2. Create a database user and configure the required network access.
3. Copy your MongoDB connection string.
4. Add the connection string to your `.env` file.

```env
MONGODB_URI=your_mongodb_connection_string
```

### 5. Configure Better Auth

Create an `auth.js` file in your `lib` directory and configure Better Auth with MongoDB.

Example configuration:

```javascript
import { betterAuth } from "better-auth";
import { MongoClient } from "mongodb";
import { mongodbAdapter } from "better-auth/adapters/mongodb";

const client = new MongoClient(process.env.MONGODB_URI);
const db = client.db("my-auth-app");

export const auth = betterAuth({
  database: mongodbAdapter(db),

  emailAndPassword: {
    enabled: true,
  },

  socialProviders: {
    google: {
      clientId: process.env.BETTER_AUTH_GOOGLE_CLIENT_ID,
      clientSecret: process.env.BETTER_AUTH_GOOGLE_CLIENT_SECRET,
    },
  },
});
```

**Note:** This example uses Better Auth's MongoDB adapter import. Follow the adapter documentation for the installed version and use the appropriate package and initialization method. Keep your database credentials private.

### 6. Configure the Authentication API Route

Create the following file:

`app/api/auth/[...all]/route.ts`

```typescript
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { GET, POST } = toNextJsHandler(auth);
```

This route handles Better Auth authentication requests, including sign-up, sign-in, and other supported authentication operations.

## 🔐 User Authentication

### 7. Create the Sign-Up Page

Create a registration page, for example:

`app/signup/page.tsx`

Design a responsive sign-up form with the following fields:

* Name
* Email
* Password
* Submit button

Use HeroUI components to create a clean and user-friendly interface.

### 8. Create the Authentication Client

Create `lib/auth-client.ts`:

```typescript
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient();
export const { signIn, signUp, signOut } = authClient;
```

Use this client to manage authentication actions from client components.

### 9. Handle User Sign-Up

Call `signUp.email()` when the user submits the registration form:

```typescript
const { data: signUpData, error } = await signUp.email({
  name: data.name,
  email: data.email,
  password: data.password,
  callbackURL: "/",
});

if (error) {
  console.error(error.message);
  return;
}

// Handle successful registration.
```

The form data is passed to Better Auth, which handles the registration process.

### 10. Handle User Sign-In

Call `signIn.email()` when the user submits the login form:

```typescript
const { data: signInData, error } = await signIn.email({
  email: data.email,
  password: data.password,
  callbackURL: "/",
});

if (error) {
  console.error(error.message);
  return;
}

// Handle successful sign-in.
```

The `callbackURL` specifies where users should be redirected after successful authentication.

### 11. Manage the User Session

Use `authClient.useSession()` inside a client component to access the current session:

```tsx
"use client";

import { authClient } from "@/lib/auth-client";

export default function Profile() {
  const { data: session, isPending } = authClient.useSession();

  if (isPending) {
    return <p>Loading...</p>;
  }

  if (!session) {
    return <p>Please sign in to continue.</p>;
  }

  return <p>Welcome, {session.user.name}!</p>;
}
```

The session provides information about the currently authenticated user.

### 12. Implement Sign-Out

Use `authClient.signOut()` to sign out the current user and redirect them to the login page:

```typescript
await authClient.signOut({
  fetchOptions: {
    onSuccess: () => {
      router.push("/login");
    },
  },
});
```

Make sure `router` is initialized using `useRouter` from `next/navigation` inside a client component.

## 🛡️ Protected Routes

### 13. Create `proxy.js`

Create a `proxy.js` file in the project root to protect private routes.

In Next.js versions that support the Proxy convention, this file can inspect requests and redirect unauthenticated users away from protected pages.

Example structure:

```javascript
import { NextResponse } from "next/server";

export function proxy(request) {
  // Add your authentication check here.

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*", "/profile/:path*"],
};
```

**Important:** This is only the Proxy structure, not a complete authentication check. Implement the session validation using a server-side Better Auth session check or another documented secure method. Do not rely only on client-side session state or the presence of a cookie to authorize access to sensitive data.

## 🌐 Google OAuth Authentication

### 14. Create Google OAuth Credentials

1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Create or select a project.
3. Configure the OAuth consent screen.
4. Open **APIs & Services → Credentials**.
5. Create an OAuth 2.0 Client ID.
6. Configure the authorized redirect URI according to your Better Auth base URL and Google provider configuration.
7. Copy your Google Client ID and Client Secret.

Add the credentials to your `.env` file:

```env
BETTER_AUTH_GOOGLE_CLIENT_ID=your_google_client_id
BETTER_AUTH_GOOGLE_CLIENT_SECRET=your_google_client_secret
```

### 15. Enable Google as a Social Provider

Add the following configuration to `auth.js`:

```javascript
socialProviders: {
  google: {
    clientId: process.env.BETTER_AUTH_GOOGLE_CLIENT_ID,
    clientSecret: process.env.BETTER_AUTH_GOOGLE_CLIENT_SECRET,
  },
},
```

This enables Google OAuth through Better Auth. You can then add a Google sign-in button to your login page.

## ✉️ Email Verification with Resend

### 16. Configure Resend

1. Visit [Resend](https://resend.com/).
2. Create an account.
3. Verify a sending domain as required for your deployment.
4. Navigate to the API Keys section.
5. Create an API key.
6. Add the key to your `.env` file.

```env
RESEND_API_KEY=your_resend_api_key
```

### 17. Enable Email Verification

Configure email verification in `auth.js` using Better Auth's email verification options and a supported email delivery method.

Conceptually, the configuration should:

* Enable email verification.
* Send a verification email when required.
* Generate a verification URL.
* Deliver the email using Resend.
* Handle successful verification and any required redirect.

**Note:** Adding `RESEND_API_KEY` alone does not enable email verification. You must also configure Better Auth's verification callbacks and integrate Resend with an email-sending implementation.

## 🔑 Environment Variables

Create a `.env` file in the project root:

```env
# MongoDB
MONGODB_URI=your_mongodb_connection_string

# Better Auth
BETTER_AUTH_SECRET=your_random_secret
BETTER_AUTH_URL=http://localhost:3000

# Google OAuth
BETTER_AUTH_GOOGLE_CLIENT_ID=your_google_client_id
BETTER_AUTH_GOOGLE_CLIENT_SECRET=your_google_client_secret

# Resend
RESEND_API_KEY=your_resend_api_key
```

Generate a strong, random `BETTER_AUTH_SECRET` and configure `BETTER_AUTH_URL` for your deployed application when deploying to production.

**Security:** Never commit `.env` files, API keys, database credentials, or OAuth client secrets to GitHub. Add `.env*` to `.gitignore`, while retaining any intentionally tracked example environment file.

## 📁 Suggested Project Structure

```text
my-auth-app/
├── app/
│   ├── api/
│   │   └── auth/
│   │       └── [...all]/
│   │           └── route.ts
│   ├── login/
│   │   └── page.tsx
│   ├── signup/
│   │   └── page.tsx
│   ├── dashboard/
│   │   └── page.tsx
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── lib/
│   ├── auth.js
│   └── auth-client.ts
├── proxy.js
├── .env
├── .gitignore
├── package.json
└── README.md
```

The structure is illustrative. Adjust file extensions and locations to match your project's configuration.

## ▶️ Run the Project

Start the development server:

```bash
npm run dev
```

Open http://localhost:3000 in your browser.

## 📚 Useful Resources

* [Next.js Documentation](https://nextjs.org/docs)
* [Better Auth Documentation](https://www.better-auth.com/docs)
* [Better Auth MongoDB Integration](https://www.better-auth.com/docs/adapters/mongodb)
* [HeroUI Documentation](https://www.heroui.com/docs)
* [MongoDB Atlas](https://www.mongodb.com/atlas)
* [Google Cloud Console](https://console.cloud.google.com/)
* [Resend Documentation](https://resend.com/docs)

## 👨‍💻 Author

**Jahid Hasib**

GitHub: [hasib095](https://github.com/hasib095)

---

*Built with Next.js, Better Auth, MongoDB, and HeroUI.*
