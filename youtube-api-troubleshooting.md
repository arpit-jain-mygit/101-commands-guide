# YouTube API v3 — Troubleshooting Guide

Solutions to common YouTube API problems.

## 🔍 Diagnostic Flowchart

```
Something not working?
│
├─ API error in code?
│  ├─ 401/403 error → See "Authentication Issues"
│  ├─ 400 error → See "Request Format Issues"
│  ├─ 429 error → See "Quota Issues"
│  └─ Other error → See "API Error Codes"
│
├─ Can't find API key?
│  └─ See "API Key Issues"
│
├─ Can't enable API?
│  └─ See "API Enablement Issues"
│
└─ Not sure?
   └─ Run diagnostic script below
```

---

## 🧪 Diagnostic Script

Run this Python script to check your setup:

```python
#!/usr/bin/env python3
"""Diagnostic script for YouTube API setup"""

import os
import sys
from pathlib import Path

print("🔍 YouTube API Diagnostic Tool\n")

# Check 1: Python version
print("1️⃣ Checking Python version...")
if sys.version_info >= (3, 8):
    print(f"   ✅ Python {sys.version_info.major}.{sys.version_info.minor} is supported")
else:
    print(f"   ❌ Python {sys.version_info.major}.{sys.version_info.minor} is too old (need 3.8+)")

# Check 2: .env file
print("\n2️⃣ Checking .env file...")
if Path('.env').exists():
    print("   ✅ .env file exists")
    
    with open('.env') as f:
        content = f.read()
        if 'YOUTUBE_API_KEY=' in content:
            print("   ✅ YOUTUBE_API_KEY found in .env")
            if 'YOUTUBE_API_KEY=' in content and content.split('YOUTUBE_API_KEY=')[1].strip():
                print("   ✅ API key value is set")
            else:
                print("   ❌ API key value is empty!")
        else:
            print("   ❌ YOUTUBE_API_KEY not found in .env")
else:
    print("   ❌ .env file not found (create it: cp .env.example .env)")

# Check 3: Environment variable
print("\n3️⃣ Checking environment variable...")
api_key = os.getenv('YOUTUBE_API_KEY')
if api_key:
    print(f"   ✅ YOUTUBE_API_KEY loaded: {api_key[:10]}...")
else:
    print("   ❌ YOUTUBE_API_KEY not found in environment")
    print("      Try: source .env or load_dotenv()")

# Check 4: Python packages
print("\n4️⃣ Checking Python packages...")
packages_to_check = [
    'google.auth',
    'googleapiclient',
    'dotenv'
]

for package in packages_to_check:
    try:
        __import__(package.split('.')[0])
        print(f"   ✅ {package} is installed")
    except ImportError:
        print(f"   ❌ {package} not found")
        print(f"      Install: pip install {package.split('.')[0]}")

# Check 5: API key format
print("\n5️⃣ Checking API key format...")
if api_key:
    if api_key.startswith('AIzaSy'):
        print(f"   ✅ API key format looks valid (starts with 'AIzaSy')")
    else:
        print(f"   ❌ API key format suspicious (doesn't start with 'AIzaSy')")
        print(f"      Got: {api_key[:20]}...")

# Check 6: Test API call
print("\n6️⃣ Testing API call...")
if api_key:
    try:
        from googleapiclient.discovery import build
        youtube = build('youtube', 'v3', developerKey=api_key)
        request = youtube.search().list(
            part='snippet',
            q='test',
            maxResults=1
        )
        response = request.execute()
        print(f"   ✅ API call successful!")
        print(f"      Got {len(response.get('items', []))} result(s)")
    except Exception as e:
        print(f"   ❌ API call failed: {str(e)}")
        print(f"      Error type: {type(e).__name__}")
else:
    print("   ⏭️ Skipped (API key not loaded)")

print("\n" + "="*50)
print("Diagnostic complete!")
```

Save as `diagnose.py` and run:
```bash
python diagnose.py
```

---

## Authentication Issues

### Error: `401 Unauthorized - API key not valid`

**What it means:** Your API key is invalid or incorrect.

**Solutions:**

1. **Check if API key is correct**
   ```python
   # Print what you're using
   print(f"API Key: {api_key}")
   print(f"First 10 chars: {api_key[:10]}")
   print(f"Length: {len(api_key)}")
   ```

2. **Verify API key in Google Cloud Console**
   - Go to https://console.cloud.google.com/
   - Select your project
   - APIs & Services → Credentials
   - Find your API Key and copy it again
   - Make sure there are no extra spaces

3. **Check in .env file**
   ```bash
   cat .env | grep YOUTUBE_API_KEY
   # Should show: YOUTUBE_API_KEY=AIzaSy...
   ```

4. **Reload environment variables**
   ```python
   # If using .env file
   import os
   from dotenv import load_dotenv, find_dotenv
   
   load_dotenv(find_dotenv())  # Force reload
   api_key = os.getenv('YOUTUBE_API_KEY')
   print(f"Loaded: {api_key}")
   ```

5. **Create a new API key**
   - If still not working, delete the old key and create a new one
   - Go to **Credentials** → Find your key → Click delete (🗑️)
   - Click **"+ CREATE CREDENTIALS"** → **"API Key"**
   - Copy the new key immediately

---

### Error: `403 Forbidden - YouTube Data API v3 is not enabled`

**What it means:** The API is not enabled on your project.

**Solutions:**

1. **Verify API is enabled**
   - Go to https://console.cloud.google.com/
   - Select correct project (check top left)
   - APIs & Services → Enabled APIs & Services
   - Look for "YouTube Data API v3"

2. **Enable the API if missing**
   - APIs & Services → Library
   - Search: `youtube data api v3`
   - Click on "YouTube Data API v3"
   - Click **"ENABLE"**
   - Wait 30-60 seconds

3. **Check project selection**
   Make sure you're in the correct project:
   ```python
   # Get your project ID
   # Go to: https://console.cloud.google.com/
   # Top left shows "Project Name"
   # This should match where you enabled the API
   ```

4. **Try a fresh project**
   If still failing:
   - Create a completely new project
   - Enable the API on the new project
   - Create an API key on the new project
   - Test with the new key

---

### Error: `401 Unauthorized - API key has not been created`

**What it means:** You don't have a valid API key configured.

**Solutions:**

1. **Create an API key**
   - Go to **APIs & Services** → **Credentials**
   - Click **"+ CREATE CREDENTIALS"** → **"API Key"**
   - Copy the key

2. **Add to code**
   ```python
   API_KEY = "AIzaSy..."  # Your actual key
   youtube = build('youtube', 'v3', developerKey=API_KEY)
   ```

3. **Or use environment variable**
   ```python
   import os
   from dotenv import load_dotenv
   
   load_dotenv()
   API_KEY = os.getenv('YOUTUBE_API_KEY')
   ```

---

## Request Format Issues

### Error: `400 Bad Request - Invalid parameter`

**What it means:** Your API request has a syntax error.

**Solutions:**

1. **Check parameter names** (Python uses snake_case in code, but API uses camelCase)
   ```python
   # ✅ Correct
   youtube.search().list(
       part='snippet',      # ✅
       q='jain',           # ✅
       maxResults=10       # ✅
   )
   
   # ❌ Wrong
   youtube.search().list(
       parts='snippet',           # ❌ 'parts' should be 'part'
       query='jain',             # ❌ 'query' should be 'q'
       max_results=10            # ❌ Should be 'maxResults'
   )
   ```

2. **Check for typos**
   ```python
   # ❌ Wrong
   part='snipet'          # Typo in 'snippet'
   
   # ✅ Correct
   part='snippet'
   ```

3. **Check parameter values**
   ```python
   # ❌ Wrong - maxResults should be integer, not string
   maxResults='10'
   
   # ✅ Correct
   maxResults=10
   ```

4. **Check required parameters are included**
   ```python
   # ❌ Wrong - 'part' is required!
   youtube.search().list(q='jain')
   
   # ✅ Correct
   youtube.search().list(part='snippet', q='jain')
   ```

---

### Error: `400 Bad Request - Invalid video ID`

**What it means:** The video ID format is wrong.

**Solutions:**

1. **Check video ID format**
   ```python
   # Video ID should be 11 characters, alphanumeric
   # From URL: https://youtube.com/watch?v=dQw4w9WgXcQ
   # Video ID: dQw4w9WgXcQ (11 chars)
   
   # ✅ Correct
   youtube.videos().list(part='snippet', id='dQw4w9WgXcQ')
   
   # ❌ Wrong - full URL instead of ID
   youtube.videos().list(part='snippet', id='https://youtube.com/watch?v=dQw4w9WgXcQ')
   ```

2. **Extract video ID correctly**
   ```python
   from urllib.parse import urlparse, parse_qs
   
   url = 'https://youtube.com/watch?v=dQw4w9WgXcQ'
   video_id = parse_qs(urlparse(url).query)['v'][0]
   print(video_id)  # dQw4w9WgXcQ
   ```

---

## Quota Issues

### Error: `429 Too Many Requests - Quota exceeded`

**What it means:** You've exceeded your daily quota (10,000 units).

**Solutions:**

1. **Check your current usage**
   - Go to **APIs & Services** → **YouTube Data API v3**
   - Click **"Quotas"** tab
   - See your daily usage graph

2. **Wait for quota reset**
   - Quota resets daily at **midnight UTC**
   - No way to speed this up
   - Plan your API calls more efficiently tomorrow

3. **Optimize your requests**
   ```python
   # ❌ Inefficient - 100 units per search
   for video_id in video_ids:
       request = youtube.search().list(part='snippet', q=video_id)
       # Uses 100 units per call = 10,000 units for 100 videos
   
   # ✅ Efficient - 1 unit per videos list call
   request = youtube.videos().list(part='snippet', id=','.join(video_ids))
   # Uses only 1 unit for all 50 videos at once (max 50 per call)
   ```

4. **Request higher quota** (optional)
   - Go to **APIs & Services** → **YouTube Data API v3** → **Quotas**
   - Look for "REQUEST QUOTA INCREASE" button
   - Note: Requires approval and business justification

5. **Batch your requests**
   ```python
   # Instead of checking channels every minute,
   # check them once every 5-10 minutes
   ```

---

## API Key Issues

### Problem: Can't find my API key

**Solutions:**

1. **Go to Google Cloud Console**
   - https://console.cloud.google.com/

2. **Select your project**
   - Click "Select a Project" dropdown (top left)
   - Find and click your project

3. **Find your API Key**
   - APIs & Services → Credentials
   - Under "API Keys" section
   - You'll see your key listed

4. **Copy it**
   - Click the key name or the copy icon
   - Paste into your code or `.env` file

---

### Problem: Lost my API key (deleted the message)

**Solutions:**

1. **Create a new API key**
   - Go to **APIs & Services** → **Credentials**
   - Click **"+ CREATE CREDENTIALS"** → **"API Key"**
   - Copy the new key immediately

2. **Delete the old key** (if you still see it)
   - Click the key name in the list
   - Click **"Delete"** (🗑️ icon)

---

### Problem: API key showing in Git/GitHub

**URGENT!** Your API key is compromised!

**Solutions:**

1. **Immediately delete the exposed key**
   - Go to **APIs & Services** → **Credentials**
   - Find the exposed key
   - Click **"Delete"** (🗑️ icon)

2. **Create a new API key**
   - Click **"+ CREATE CREDENTIALS"** → **"API Key"**

3. **Clean your Git history**
   ```bash
   # This is complex - here's the simplest approach:
   
   # Option 1: Create a new repo from scratch
   git init new-repo
   git add .  # (excluding .env)
   git commit -m "Initial commit"
   git remote add origin https://github.com/you/repo.git
   git push -u origin main
   
   # Option 2: Use git-filter-repo (advanced)
   # See: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository
   ```

4. **Add to .gitignore**
   ```bash
   echo ".env" >> .gitignore
   git add .gitignore
   git commit -m "Add .env to gitignore"
   git push
   ```

5. **Notify GitHub** (they may have already flagged it)
   - The key is now invalid anyway
   - GitHub usually notifies you

---

## API Enablement Issues

### Problem: "YouTube Data API v3" not showing in search results

**Solutions:**

1. **Clear the search box**
   - Click the X button to clear search
   - Type again: `youtube data api v3`

2. **Search variations**
   - Try: `youtube`
   - Try: `youtube data`
   - Try: `youtube v3`

3. **Scroll through results**
   - Might be on page 2-3 of results
   - Look for one that says "YouTube Data API v3"
   - Official Google product (has checkmark)

4. **Check your project**
   - Verify you're in the right project (top left)
   - APIs might be enabled on wrong project

---

### Problem: "Enable" button is grayed out

**Solutions:**

1. **Wait for page to load**
   - Sometimes Google Cloud Console loads slowly
   - Wait 10 seconds and refresh (F5)

2. **Check if already enabled**
   - Go to **APIs & Services** → **Enabled APIs & Services**
   - Search for "YouTube Data API v3"
   - If listed, it's already enabled ✅

3. **Try a different browser**
   - Try Chrome, Firefox, or Safari
   - Clear browser cache first

4. **Check project permissions**
   - You need "Editor" role or higher
   - Contact your Google Workspace admin if needed

---

## Common Error Codes

| Code | Error | Meaning | Solution |
|------|-------|---------|----------|
| 400 | Bad Request | Invalid parameter | Check parameter names and values |
| 401 | Unauthorized | No auth or bad key | Create/check API key |
| 403 | Forbidden | API not enabled | Enable YouTube Data API v3 |
| 404 | Not Found | Resource doesn't exist | Check video ID, channel ID, etc. |
| 429 | Too Many Requests | Quota exceeded | Wait for daily reset (midnight UTC) |
| 500 | Internal Server Error | Google's problem | Retry in a few minutes |
| 503 | Service Unavailable | Google's API is down | Check https://status.cloud.google.com |

---

## Still Stuck?

1. **Check the full error message** - It usually tells you what's wrong
2. **Read the official documentation** - https://developers.google.com/youtube/v3
3. **Search Google** - Your error message is probably documented
4. **Ask on Stack Overflow** - Tag: `youtube-api`, `google-api-python-client`
5. **Check Google Cloud Status** - https://status.cloud.google.com

---

**Need help?** Run the diagnostic script above!  
**Ready for production?** Check [Best Practices](youtube-api-best-practices.md)  
**Back to basics?** See [101 Guide](youtube-api-101.md)
