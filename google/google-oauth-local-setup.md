# Google OAuth Local Setup

## Problem

Google sign-in for a local app shows authorization errors or never reaches the account chooser.

## Working Setup

### 1. Open Google Auth Platform

In Google Cloud Console, open:

```text
Google Auth Platform
```

### 2. Finish the initial setup if needed

If the page says the platform is not configured yet, click:

```text
Get started
```

Fill the required consent screen details:

- App name
- User support email
- Developer contact email

Choose:

```text
External
```

if you want to test with your own Gmail account.

Add your Gmail address under:

```text
Test users
```

### 3. Create the OAuth client

Open:

```text
Clients
```

Then click:

```text
Create client
```

Choose:

```text
Web application
```

Use a name like:

```text
Local Google OAuth
```

Add this exact redirect URI:

```text
http://localhost:8000/api/v1/auth/google/callback
```

### 4. Copy the credentials into the app

After the client is created, copy:

- Client ID -> `GOOGLE_CLIENT_ID`
- Client secret -> `GOOGLE_CLIENT_SECRET`

Put them in `apps/api/.env`.

### 5. Restart and retest

Restart FastAPI and try the login flow again.

## Notes

- The app should use `prompt=select_account consent` so Google shows the account chooser.
- If Google says `Missing required parameter: client_id`, the client ID is blank or missing in `.env`.
- If the app returns a local `503`, the API is intentionally blocking login until the OAuth credentials are configured.
