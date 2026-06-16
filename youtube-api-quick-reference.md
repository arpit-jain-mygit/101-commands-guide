# YouTube API v3 — Quick Reference Checklist

Fast checklist for setting up YouTube Data API v3 in 5 minutes.

## 📋 Quick Setup Checklist

### ✅ Create Google Cloud Project
- [ ] Go to https://console.cloud.google.com/
- [ ] Sign in with Google account
- [ ] Click **"Select a Project"** → **"NEW PROJECT"**
- [ ] Enter project name (e.g., "YouTube Transcript Pipeline")
- [ ] Click **"CREATE"**
- [ ] Wait for project to be created (30-60 seconds)

### ✅ Enable YouTube Data API v3
- [ ] Click **☰** (menu) → **APIs & Services** → **Library**
- [ ] Search: `youtube data api v3`
- [ ] Click **"YouTube Data API v3"**
- [ ] Click blue **"ENABLE"** button
- [ ] Wait for API to be enabled (10-30 seconds)

### ✅ Create API Key
- [ ] Go to **APIs & Services** → **Credentials**
- [ ] Click **"+ CREATE CREDENTIALS"** → **"API Key"**
- [ ] Copy your key: `AIzaSy...`
- [ ] Click **"CLOSE"**

### ✅ Test Your API Key
- [ ] Open browser and paste this URL:
  ```
  https://www.googleapis.com/youtube/v3/search?part=snippet&q=jain&key=YOUR_API_KEY&maxResults=5
  ```
- [ ] Replace `YOUR_API_KEY` with your actual key
- [ ] Press Enter
- [ ] If you see JSON data, it works! ✅

### ✅ Secure Your Key
- [ ] Create `.env` file:
  ```
  YOUTUBE_API_KEY=YOUR_API_KEY_HERE
  ```
- [ ] Add to `.gitignore`:
  ```
  .env
  .env.local
  ```
- [ ] Never commit `.env` to Git

### ✅ Use in Your Code
- [ ] Install dependencies:
  ```bash
  pip install google-api-python-client python-dotenv
  ```
- [ ] In your Python code:
  ```python
  import os
  from dotenv import load_dotenv
  from googleapiclient.discovery import build

  load_dotenv()
  API_KEY = os.getenv('YOUTUBE_API_KEY')
  youtube = build('youtube', 'v3', developerKey=API_KEY)
  ```

---

## 🔑 Common API Calls

### Search Videos
```python
request = youtube.search().list(
    part='snippet',
    q='jain teachings',
    maxResults=10
)
results = request.execute()
```

### Get Channel Details
```python
request = youtube.channels().list(
    part='contentDetails',
    id='UCxxxxxxxxxxxxxx'  # Channel ID
)
channel = request.execute()
```

### Get Uploads Playlist
```python
channel_id = 'UCxxxxxxxxxxxxxx'
request = youtube.channels().list(
    part='contentDetails',
    id=channel_id
)
response = request.execute()
uploads_playlist_id = response['items'][0]['contentDetails']['relatedPlaylists']['uploads']
```

### Get Playlist Items
```python
request = youtube.playlistItems().list(
    part='snippet',
    playlistId=uploads_playlist_id,
    maxResults=50
)
items = request.execute()
```

---

## 📊 Quota Information

**Free Daily Limit:** 10,000 units/day

**Common Operations:**
| Operation | Units |
|-----------|-------|
| Search | 100 |
| Channel list | 1 |
| Video list | 1 |
| Playlist items | 1 |
| Transcript download | 1 |

**Check Usage:** APIs & Services → YouTube Data API v3 → Quotas

---

## 🚀 Copy-Paste Setup Script

### For macOS/Linux:
```bash
# 1. Create .env file
cat > .env << EOF
YOUTUBE_API_KEY=YOUR_API_KEY_HERE
EOF

# 2. Add to .gitignore
echo ".env" >> .gitignore
echo ".env.local" >> .gitignore

# 3. Install Python dependencies
pip install google-api-python-client python-dotenv

# 4. Create test script
cat > test_youtube.py << 'EOTEST'
import os
from dotenv import load_dotenv
from googleapiclient.discovery import build

load_dotenv()
API_KEY = os.getenv('YOUTUBE_API_KEY')

if not API_KEY:
    print("❌ API_KEY not found in .env")
else:
    try:
        youtube = build('youtube', 'v3', developerKey=API_KEY)
        request = youtube.search().list(
            part='snippet',
            q='jain',
            maxResults=5
        )
        response = request.execute()
        print(f"✅ API Key works! Found {len(response['items'])} videos")
    except Exception as e:
        print(f"❌ Error: {e}")
EOTEST

# 5. Test it
python test_youtube.py
```

### For Windows (PowerShell):
```powershell
# 1. Create .env file
@"
YOUTUBE_API_KEY=YOUR_API_KEY_HERE
"@ | Out-File -Encoding UTF8 .env

# 2. Add to .gitignore
Add-Content -Path .gitignore ".env"

# 3. Install Python dependencies
pip install google-api-python-client python-dotenv

# 4. Test
python -c "import os; from dotenv import load_dotenv; load_dotenv(); print(os.getenv('YOUTUBE_API_KEY'))"
```

---

## ⚠️ Security Checklist

- [ ] API Key is in `.env` file (not in code)
- [ ] `.env` is in `.gitignore`
- [ ] `.env` is NOT committed to Git
- [ ] API Key not shared in messages/chats
- [ ] API Key not visible in browser history
- [ ] No API Key in screenshots/documentation
- [ ] Consider restricting key to IP/domain if available

---

## 🆘 Quick Troubleshooting

| Error | Solution |
|-------|----------|
| `403 Forbidden` | Enable YouTube Data API v3 in Google Cloud Console |
| `401 Unauthorized` | Check your API key is correct (copy again) |
| `400 Bad Request` | Check parameter names and values |
| `429 Too Many Requests` | You've exceeded daily quota (10,000 units), retry tomorrow |
| `API_KEY not found` | Create `.env` file with `YOUTUBE_API_KEY=...` |
| `ModuleNotFoundError: google` | Run `pip install google-api-python-client` |

---

## 🎯 Next Steps

1. **Get your API key** - Follow steps above
2. **Test it** - Use browser test URL
3. **Secure it** - Add to `.env` and `.gitignore`
4. **Build your app** - Start coding with YouTube API!

---

**Need detailed instructions?** See [YouTube API 101 Guide](youtube-api-101.md)  
**Having issues?** Check [Troubleshooting Guide](youtube-api-troubleshooting.md)  
**Production use?** Review [Best Practices](youtube-api-best-practices.md)
