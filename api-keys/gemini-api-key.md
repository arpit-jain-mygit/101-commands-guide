# Create a Gemini API key for a backend app

## Problem

An app needs to call Gemini from the backend, but the API key needs to be created safely and stored in the right place.

For example:

- Angular frontend
- FastAPI backend
- Render deployment

The key must not be stored in the Angular frontend because frontend environment variables and bundled code can be inspected by users.

## Solution

Create the Gemini API key in Google AI Studio, then store it as a backend environment variable.

## Steps

### 1. Open Gemini API keys

Go to:

```text
https://aistudio.google.com/app/apikey
```

Or open the official docs:

```text
https://ai.google.dev/gemini-api/docs/api-key
```

Then click:

```text
Get API key
```

or:

```text
Create or view a Gemini API Key
```

### 2. Sign in and open Google AI Studio

Sign in with your Google account.

If you are a new user, Google AI Studio can create a default Google Cloud project and API key after you accept the terms.

### 3. Create or select a project

Use the default project if Google AI Studio creates one for you.

If the Create API key button is disabled, it usually means one of these is true:

- You need to select a different Google Cloud project.
- You do not have permission to create keys in the selected project.
- An admin needs to grant the required Google Cloud permissions.

### 4. Copy the API key

Copy the generated key.

Store it only in the backend environment, for example:

```text
GEMINI_API_KEY=your_key_here
```

Do not put the key in:

```text
src/environments/environment.ts
src/environments/environment.prod.ts
Angular frontend code
GitHub commits
```

## Render Setup

For a Render-hosted FastAPI service:

1. Open the Render dashboard.
2. Open the FastAPI backend service.
3. Go to Environment.
4. Add this environment variable:

```text
GEMINI_API_KEY=your_key_here
```

5. Redeploy the backend service.

The FastAPI backend should read the key from the environment and call Gemini from the server.

## Python Backend Example

```python
import os
from google import genai

client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])
```

## Important Notes

- The free tier includes limited access to certain models and free input/output tokens.
- The free tier is meant for getting started and small projects.
- Do not expose the key in browser code.
- If a key is leaked, delete or rotate it immediately.
- New API keys created in Google AI Studio are created as auth keys by default.
- Google docs say unrestricted standard keys are rejected starting June 19, 2026, and standard keys are rejected starting September 2026. Prefer newly created AI Studio auth keys.

## Official Docs

- Using Gemini API keys: https://ai.google.dev/gemini-api/docs/api-key
- Gemini Developer API pricing: https://ai.google.dev/gemini-api/docs/pricing
