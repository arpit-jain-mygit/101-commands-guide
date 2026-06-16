# YouTube Data API v3 — Complete 101 Guide

Learn how to enable YouTube Data API v3 and create an API key from scratch. This guide takes you from zero to working API key in 15 minutes.

## Table of Contents

1. [What You'll Need](#what-youll-need)
2. [Step 1: Create a Google Cloud Project](#step-1-create-a-google-cloud-project)
3. [Step 2: Enable YouTube Data API v3](#step-2-enable-youtube-data-api-v3)
4. [Step 3: Create Credentials (API Key)](#step-3-create-credentials-api-key)
5. [Step 4: Verify Your API Key](#step-4-verify-your-api-key)
6. [Step 5: Use Your API Key](#step-5-use-your-api-key)
7. [Understanding Quotas & Limits](#understanding-quotas--limits)
8. [Security Best Practices](#security-best-practices)
9. [Troubleshooting](#troubleshooting)

---

## What You'll Need

Before you start, make sure you have:

- ✅ **Google Account** (Gmail or Google Workspace)
- ✅ **Web Browser** (Chrome, Firefox, Safari, Edge)
- ✅ **Internet Connection**
- ✅ **Phone Number** (for account verification if needed)
- ✅ **5-10 minutes** of time

**Note:** YouTube Data API access is free! Google offers 10,000 quota units per day at no cost.

---

## Step 1: Create a Google Cloud Project

### 1.1 Go to Google Cloud Console

1. Open your browser and go to: **https://console.cloud.google.com/**
2. Sign in with your Google account
   - If prompted, verify your identity
   - Accept terms and conditions

### 1.2 Create a New Project

**What You'll See:**
```
Google Cloud Console
├── Top Left: "Google Cloud" logo
├── Next to it: "Select a Project" dropdown
└── Click it to see options
```

**Steps:**

1. Click the **"Select a Project"** dropdown (top left, next to Google Cloud logo)
   
2. Click **"NEW PROJECT"** button (top right of the popup)

3. Fill in the form:
   ```
   Project Name: "YouTube Transcript Pipeline" (or any name you prefer)
   Organization: (leave blank if you don't have one)
   Location: (leave as default)
   ```

4. Click **"CREATE"** button

5. **Wait 30-60 seconds** for the project to be created

**You'll see:**
```
Creating project... [progress bar]
```

Once done, you'll be redirected to your new project dashboard.

### 1.3 Verify Project Creation

1. Look at the top left
2. You should now see your project name instead of "Select a Project"
3. Example: 
   ```
   YouTube Transcript Pipeline
   ```

**Congratulations!** You've created your first Google Cloud project. ✅

---

## Step 2: Enable YouTube Data API v3

Now you need to enable the YouTube API on your project.

### 2.1 Open the APIs & Services Library

1. In the left sidebar, look for **"APIs & Services"**
2. If you don't see it, click the **☰ (menu)** icon (top left)
3. Click **"APIs & Services"**

**What You'll See:**
```
APIs & Services
├── Overview
├── Credentials
├── OAuth Consent Screen
├── Library          ← Click this
├── Enabled APIs & Services
└── ...
```

### 2.2 Search for YouTube Data API

1. Click **"Library"** in the left sidebar

2. You'll see a search bar at the top:
   ```
   🔍 Search for APIs and Services
   ```

3. Type: `youtube data api v3`

4. Press Enter or wait for search results

**What You'll See:**
```
Search Results:
├── YouTube Data API v3 ← This is what you want
├── YouTube Reporting API
├── YouTube Analytics API
└── ...
```

### 2.3 Enable the API

1. Click on **"YouTube Data API v3"**

2. You'll see a page with:
   - API name and description
   - A large blue **"ENABLE"** button

3. Click **"ENABLE"**

4. **Wait 10-30 seconds** while it enables

**You'll see:**
```
Enabling API... [progress]
API Enabled! ✓
```

**Congratulations!** YouTube Data API v3 is now enabled. ✅

---

## Step 3: Create Credentials (API Key)

Now you need to create an API key to use the API.

### 3.1 Go to Credentials Page

1. In the left sidebar, click **"Credentials"**

2. You'll see:
   ```
   Credentials
   ├── Create Credentials (large button at top)
   ├── API Keys
   ├── OAuth 2.0 Client IDs
   └── Service Accounts
   ```

### 3.2 Create an API Key

1. Click the **"+ CREATE CREDENTIALS"** button (top left)

2. From the dropdown menu, select **"API Key"**

**What You'll See:**
```
Create API Key

Your API key has been created:

AIzaSyDxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Keep it secret! This key is sensitive.
```

3. **Copy your API key** to a safe place (password manager, notes, etc.)

4. Click **"CLOSE"** or the **X** button

### 3.3 Find Your API Key Later

If you need to find your API key again:

1. Go to **APIs & Services** → **Credentials**

2. Under **"API Keys"** section, you'll see your key listed:
   ```
   API Keys
   ├── Default API Key  [Copy button] [Edit] [Delete]
   ```

3. Click the **key name** or the copy icon to copy it

---

## Step 4: Verify Your API Key

Let's test that your API key works!

### 4.1 Using a Web Browser (Easiest Method)

1. Open a new browser tab

2. Paste this URL (replace `YOUR_API_KEY` with your actual key):
   ```
   https://www.googleapis.com/youtube/v3/search?part=snippet&q=jain&key=YOUR_API_KEY&maxResults=5
   ```

3. Press Enter

**If it works, you'll see:**
```json
{
  "kind": "youtube#searchListResponse",
  "etag": "...",
  "items": [
    {
      "kind": "youtube#searchResult",
      "etag": "...",
      "id": {
        "kind": "youtube#video",
        "videoId": "..."
      },
      "snippet": {
        "publishedAt": "...",
        "title": "...",
        "description": "...",
        ...
      }
    }
  ]
}
```

This is JSON data! Your API key is working! ✅

**If you see an error:**
```json
{
  "error": {
    "code": 403,
    "message": "The request did not include an API key...",
    "errors": [...]
  }
}
```

See [Troubleshooting](#troubleshooting) section below.

### 4.2 Using Python (Advanced Method)

If you want to test with Python:

```bash
pip install google-api-python-client
```

Create a test script `test_youtube_api.py`:

```python
from googleapiclient.discovery import build

API_KEY = "YOUR_API_KEY_HERE"

youtube = build('youtube', 'v3', developerKey=API_KEY)

request = youtube.search().list(
    part='snippet',
    q='jain',
    maxResults=5
)
response = request.execute()

print(f"Found {len(response['items'])} videos")
for item in response['items']:
    print(f"  - {item['snippet']['title']}")
```

Run it:
```bash
python test_youtube_api.py
```

**Expected output:**
```
Found 5 videos
  - Video Title 1
  - Video Title 2
  - Video Title 3
  - Video Title 4
  - Video Title 5
```

---

## Step 5: Use Your API Key

Now you can use your API key in your applications!

### 5.1 In Python Applications

```python
from googleapiclient.discovery import build

API_KEY = "YOUR_API_KEY"
youtube = build('youtube', 'v3', developerKey=API_KEY)

# Search for videos
request = youtube.search().list(
    part='snippet',
    q='jain teachings',
    maxResults=10
)
results = request.execute()

# Get channel details
request = youtube.channels().list(
    part='contentDetails',
    id='UCxxxxxxxxxxxxxx'
)
channel = request.execute()

# Get video transcripts
from youtube_transcript_api import YouTubeTranscriptApi
transcript = YouTubeTranscriptApi.get_transcript('dQw4w9WgXcQ')
```

### 5.2 In Node.js Applications

```javascript
const {google} = require('googleapis');

const youtube = google.youtube({
  version: 'v3',
  auth: 'YOUR_API_KEY'
});

// Search for videos
youtube.search.list({
  part: 'snippet',
  q: 'jain teachings',
  maxResults: 10
}, (err, res) => {
  if (err) console.error(err);
  console.log(res.data.items);
});
```

### 5.3 In Environment Variables

**Best Practice:** Don't hardcode your API key!

Create `.env` file:
```
YOUTUBE_API_KEY=YOUR_API_KEY_HERE
```

Load it in Python:
```python
import os
from dotenv import load_dotenv

load_dotenv()
API_KEY = os.getenv('YOUTUBE_API_KEY')
```

Or in Node.js:
```javascript
require('dotenv').config();
const API_KEY = process.env.YOUTUBE_API_KEY;
```

---

## Understanding Quotas & Limits

### 5.1 Free Quota

**You get 10,000 quota units per day for free.**

Common operations and their costs:

| Operation | Quota Cost | Example |
|-----------|-----------|---------|
| Search | 100 units | Search for videos |
| Channel list | 1 unit | Get channel info |
| Video list | 1 unit | Get video details |
| Transcript download | 1 unit | Get captions |
| Playlist items | 1 unit | List playlist videos |

### 5.2 Calculate Daily Usage

For **Vidyavahak** (monitoring 50 channels twice daily):

```
Morning check:
  50 channels × 1 unit per channel = 50 units

Evening check:
  50 channels × 1 unit per channel = 50 units

Total per day: 100 units
(Well under 10,000 limit!)
```

### 5.3 Check Your Usage

1. Go to **APIs & Services** → **Library**
2. Click **YouTube Data API v3**
3. Click **"Quotas"** tab
4. See your daily usage graph

---

## Security Best Practices

### ⚠️ DO NOT

❌ **Commit API key to Git**
```bash
# Bad - never do this!
git add .env
git commit -m "Add API key"
git push  # API key is now public forever!
```

❌ **Share API key in messages**
```
"My YouTube API key is: AIzaSy..."  ← NEVER!
```

❌ **Use API key from browser console**
```javascript
// Bad - key visible in network requests
fetch('https://www.googleapis.com/youtube/v3/search?key=AIzaSy...')
```

### ✅ DO

✅ **Use `.gitignore` to exclude `.env`**
```bash
# .gitignore
.env
.env.local
secrets/
```

✅ **Use environment variables**
```python
import os
API_KEY = os.getenv('YOUTUBE_API_KEY')
```

✅ **Rotate keys regularly**
- Delete old keys if compromised
- Create new keys for different projects

✅ **Restrict key usage (Optional but recommended)**

In Google Cloud Console:
1. **APIs & Services** → **Credentials**
2. Click your API Key
3. Under **"Application Restrictions"**
   - Select **"IP addresses"** if using server
   - Select **"HTTP referrers"** if using web app
4. Under **"API Restrictions"**
   - Select **"Restrict key"**
   - Choose **"YouTube Data API v3"**

---

## Troubleshooting

### Problem 1: "403 Forbidden - YouTube Data API v3 is not enabled"

**Cause:** API is not enabled on your project

**Solution:**
1. Go to **APIs & Services** → **Library**
2. Search for "YouTube Data API v3"
3. Click on it and press **"ENABLE"**
4. Wait 30 seconds and try again

---

### Problem 2: "401 Unauthorized - API key not valid"

**Cause:** Invalid or expired API key

**Solution:**
1. Copy your API key again from **Credentials** page
2. Paste it carefully (no spaces!)
3. If still failing, delete the old key:
   - Go to **Credentials**
   - Find your key
   - Click the delete icon (🗑️)
4. Create a new API key

---

### Problem 3: "429 Too Many Requests"

**Cause:** You've exceeded your daily quota (10,000 units)

**Solution:**
1. Check your quota usage:
   - **APIs & Services** → **YouTube Data API v3** → **Quotas**
2. Options:
   - Wait until next day (quota resets at midnight UTC)
   - Reduce your API calls
   - Upgrade to paid plan for higher quotas

---

### Problem 4: "400 Bad Request - Invalid parameter"

**Cause:** Wrong API parameter or malformed request

**Solution:**
1. Check your request format matches documentation
2. Verify all required parameters are included
3. Make sure parameter names are spelled correctly
4. Example:
   ```python
   # Correct
   youtube.search().list(
       part='snippet',      # ✅ Correct
       q='query',           # ✅ Correct
       maxResults=10        # ✅ Correct
   )

   # Wrong
   youtube.search().list(
       part='snipet',       # ❌ Typo
       query='q',           # ❌ Should be 'q'
       max_results=10       # ❌ Should be camelCase
   )
   ```

---

### Problem 5: "Cannot find YouTube API in Library"

**Cause:** Search is not returning the correct API

**Solution:**
1. Clear search box (click X)
2. Type the exact name: `youtube data api v3`
3. It should appear in results
4. Click the one that shows:
   ```
   YouTube Data API v3
   (Official Google Product)
   ```

---

### Problem 6: "API Key works in browser but not in code"

**Cause:** Environment variable not set or code is looking in wrong place

**Solution:**
1. Print the API key to see what it is:
   ```python
   import os
   print(f"API Key: {os.getenv('YOUTUBE_API_KEY')}")
   ```

2. If it prints `None`, then:
   - Check `.env` file exists
   - Check it has: `YOUTUBE_API_KEY=YOUR_KEY`
   - Restart your application
   - Try adding `.env` file path explicitly:
     ```python
     from dotenv import load_dotenv
     load_dotenv('/path/to/.env')
     ```

---

## FAQ

**Q: Do I need a credit card?**  
A: No! YouTube Data API is free (10,000 units/day). You won't be charged unless you upgrade to a paid plan for higher quotas.

**Q: Can I use one API key for multiple projects?**  
A: Yes, but it's better to create separate keys for each project for security.

**Q: How often does the quota reset?**  
A: Daily at midnight UTC. You can check the reset time in your quotas dashboard.

**Q: What if I lose my API key?**  
A: You can always create a new one. Old ones will stop working automatically. Just delete the old one and create fresh.

**Q: Can I request higher quotas?**  
A: Yes! Go to **APIs & Services** → **YouTube Data API v3** → **Quotas**, then click "REQUEST QUOTA INCREASE" (you'll need to provide usage justification).

**Q: Is there a rate limit per request?**  
A: No, but there's a daily quota limit (10,000 units). Spread your requests throughout the day.

**Q: Do I need OAuth 2.0 or just an API Key?**  
A: For public data (search, channel info, transcripts), API Key is enough. For user accounts (subscriptions, uploads), you need OAuth 2.0.

---

## Next Steps

### 🎉 You've Successfully Created a YouTube API Key!

Now you can:

1. **Use it in your application** - Add to `.env` and start making requests
2. **Explore the API** - Check [YouTube API Documentation](https://developers.google.com/youtube/v3)
3. **Monitor your usage** - Check quotas dashboard regularly
4. **Scale responsibly** - Keep your key secret and track usage

### 📚 Learn More

- [YouTube API Best Practices](youtube-api-best-practices.md)
- [YouTube API Troubleshooting](youtube-api-troubleshooting.md)
- [YouTube API Quick Reference](youtube-api-quick-reference.md)

### 🚀 Ready to Build?

If you're building a YouTube transcript pipeline:
- Check out [Vidyavahak](https://github.com/arpit-jain-mygit/Vidyavahak)
- Add your API key to `.env`
- Start the pipeline!

---

**Questions?** Create an issue or reach out!  
**Found a bug in this guide?** Submit a pull request!  
**Last Updated:** 2026-06-16
