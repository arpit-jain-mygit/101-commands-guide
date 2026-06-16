# YouTube API v3 — Best Practices

Production-grade practices for secure, efficient YouTube API usage.

## 🔒 Security Best Practices

### 1. Protect Your API Key

**❌ DON'T:**
```python
# Never hardcode API key
API_KEY = "AIzaSyDxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Never commit to Git
git add credentials.py
git push  # EXPOSED FOREVER!

# Never share in chats/emails/screenshots
"Hey, my API key is: AIzaSyD..."

# Never print in logs
print(f"Connecting with key: {API_KEY}")
```

**✅ DO:**
```python
# Use environment variables
import os
from dotenv import load_dotenv

load_dotenv()
API_KEY = os.getenv('YOUTUBE_API_KEY')

# Or use Google Cloud secret manager
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
secret = client.access_secret_version(
    request={"name": "projects/123/secrets/youtube-api-key/versions/latest"}
)
API_KEY = secret.payload.data.decode('UTF-8')
```

**🛡️ Secure Your Repository:**
```bash
# .gitignore
.env
.env.local
.env.*.local
secrets/
credentials.json
*.key
*.pem
config/secrets.yaml
```

```bash
# Verify nothing sensitive is tracked
git log --all --full-history -- ".env"
# Should return nothing

# Check what would be committed
git diff --cached
# Should not contain API keys
```

### 2. Rotate Keys Regularly

```python
# Keep track of key creation dates
# In Google Cloud Console: Credentials page shows "Created" date

# Rotation schedule:
# - Rotate every 90 days (minimum)
# - Rotate immediately if exposed
# - Rotate when team member leaves

# Steps to rotate:
# 1. Create a new API key
# 2. Update all applications to use new key
# 3. Wait 24 hours
# 4. Delete old key
# 5. Monitor logs for any failures
```

### 3. Restrict Key Usage

In Google Cloud Console:

```
APIs & Services → Credentials → [Your API Key] → Edit
    ↓
Application Restrictions
├─ Option 1: HTTP referrers (for web apps)
│  └─ Referrer: example.com/*
├─ Option 2: IP addresses (for servers)
│  └─ IP: 192.168.1.100
└─ Option 3: None (not recommended)

API Restrictions
└─ Select: YouTube Data API v3 only
   (Don't grant access to other APIs)
```

---

## ⚡ Efficiency Best Practices

### 1. Minimize API Calls

**Quota costs:**
| Operation | Cost | Example |
|-----------|------|---------|
| Search | 100 units | Expensive! |
| Channel list | 1 unit | Cheap |
| Video list | 1 unit | Cheap |
| Playlist items | 1 unit | Cheap |

**✅ DO - Use efficient methods:**
```python
# Good: Use videos.list for multiple videos (1 unit)
video_ids = ['id1', 'id2', 'id3', 'id4', 'id5']
request = youtube.videos().list(
    part='snippet',
    id=','.join(video_ids)  # Max 50 videos at once
)
results = request.execute()  # 1 unit

# Bad: Use search for each video (500 units!)
for video_id in video_ids:
    request = youtube.search().list(
        part='snippet',
        q=video_id
    )
    results = request.execute()  # 100 units × 5 = 500 units!
```

**✅ DO - Cache results:**
```python
import json
from pathlib import Path
from datetime import datetime, timedelta

CACHE_FILE = 'youtube_cache.json'
CACHE_TTL = 3600  # 1 hour in seconds

def get_channel_info(channel_id):
    # Check cache first
    if Path(CACHE_FILE).exists():
        with open(CACHE_FILE) as f:
            cache = json.load(f)
            if channel_id in cache:
                cached_data = cache[channel_id]
                age = datetime.now().timestamp() - cached_data['cached_at']
                if age < CACHE_TTL:
                    return cached_data['data']  # Return cached data
    
    # Not in cache, fetch from API
    request = youtube.channels().list(
        part='contentDetails',
        id=channel_id
    )
    data = request.execute()
    
    # Update cache
    cache = {}
    if Path(CACHE_FILE).exists():
        with open(CACHE_FILE) as f:
            cache = json.load(f)
    
    cache[channel_id] = {
        'data': data,
        'cached_at': datetime.now().timestamp()
    }
    
    with open(CACHE_FILE, 'w') as f:
        json.dump(cache, f)
    
    return data
```

### 2. Batch Requests

```python
# For checking multiple channels efficiently
def check_channels_batch(channel_ids):
    # API allows up to 50 channels per request
    batch_size = 50
    all_channels = []
    
    for i in range(0, len(channel_ids), batch_size):
        batch = channel_ids[i:i+batch_size]
        request = youtube.channels().list(
            part='contentDetails,statistics',
            id=','.join(batch)
        )
        response = request.execute()
        all_channels.extend(response.get('items', []))
    
    return all_channels

# Instead of:
# 50 channels × 1 unit = 50 units (batched)
# OR
# 50 individual requests × 1 unit = 50 units (same cost, slower)
```

### 3. Smart Scheduling

```python
import schedule
import time
from datetime import datetime

# Schedule checks at specific times (cheaper than continuous)
def monitor_channels():
    """Check channels at specific times"""
    # Morning check (6 AM IST = UTC 00:30)
    schedule.every().day.at("00:30").do(check_all_channels)
    
    # Evening check (6 PM IST = UTC 12:30)
    schedule.every().day.at("12:30").do(check_all_channels)
    
    while True:
        schedule.run_pending()
        time.sleep(60)

# With 50 channels checked twice daily:
# 50 units × 2 = 100 units/day (only 1% of free quota!)
```

### 4. Request Only What You Need

```python
# ✅ Good: Request only required fields
request = youtube.channels().list(
    part='contentDetails',  # Only upload playlist info
    id=channel_id,
    fields='items(contentDetails)'  # Filter response
)

# ❌ Bad: Request everything
request = youtube.channels().list(
    part='snippet,contentDetails,statistics,topicDetails'
    # Gets data you don't need
)

# The 'fields' parameter reduces bandwidth
```

---

## 🏗️ Architecture Best Practices

### 1. Use Environment Variables

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    YOUTUBE_API_KEY = os.getenv('YOUTUBE_API_KEY')
    GOOGLE_CLOUD_PROJECT = os.getenv('GOOGLE_CLOUD_PROJECT')
    DATABASE_URL = os.getenv('DATABASE_URL')
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
    
    # Validate required config
    @classmethod
    def validate(cls):
        if not cls.YOUTUBE_API_KEY:
            raise ValueError("YOUTUBE_API_KEY not set")
        if not cls.GOOGLE_CLOUD_PROJECT:
            raise ValueError("GOOGLE_CLOUD_PROJECT not set")

# In your app
Config.validate()
youtube = build('youtube', 'v3', developerKey=Config.YOUTUBE_API_KEY)
```

### 2. Implement Error Handling

```python
from googleapiclient.errors import HttpError
import logging
import time

logger = logging.getLogger(__name__)

def fetch_with_retry(request, max_retries=3):
    """Fetch with exponential backoff"""
    for attempt in range(max_retries):
        try:
            return request.execute()
        except HttpError as e:
            if e.resp.status == 429:  # Quota exceeded
                logger.warning(f"Quota exceeded, waiting...")
                time.sleep(2 ** attempt)  # Exponential backoff
            elif e.resp.status in [500, 503]:  # Server error
                logger.warning(f"Server error, retrying...")
                time.sleep(2 ** attempt)
            else:
                logger.error(f"API error {e.resp.status}: {e}")
                raise
    
    raise Exception(f"Failed after {max_retries} retries")

# Usage
try:
    request = youtube.search().list(
        part='snippet',
        q='jain',
        maxResults=10
    )
    results = fetch_with_retry(request)
except Exception as e:
    logger.error(f"Failed to search: {e}")
```

### 3. Logging Best Practices

```python
import logging
from logging.handlers import RotatingFileHandler

def setup_logging():
    logger = logging.getLogger('youtube_api')
    
    # Log to file with rotation
    handler = RotatingFileHandler(
        'api.log',
        maxBytes=10_000_000,  # 10 MB
        backupCount=5
    )
    
    # Log format
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.setLevel(logging.INFO)
    
    return logger

logger = setup_logging()

# Log important events
logger.info(f"Starting channel monitoring...")
logger.info(f"Found {new_videos} new videos")
logger.warning(f"API quota at 80%")
logger.error(f"Failed to fetch channel: {error}")

# Don't log sensitive data
logger.info(f"API call succeeded")  # ✅ Good
logger.info(f"API key: {api_key}")  # ❌ BAD!
```

### 4. Database for State Management

```python
import sqlite3
from datetime import datetime

class APIStateManager:
    """Track API usage and state"""
    
    def __init__(self, db_path='api_state.db'):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        with sqlite3.connect(self.db_path) as conn:
            conn.execute('''
                CREATE TABLE IF NOT EXISTS api_calls (
                    id INTEGER PRIMARY KEY,
                    timestamp DATETIME,
                    endpoint TEXT,
                    quota_used INTEGER,
                    status_code INTEGER
                )
            ''')
            conn.execute('''
                CREATE TABLE IF NOT EXISTS quota_tracking (
                    date DATE PRIMARY KEY,
                    quota_used INTEGER DEFAULT 0,
                    quota_limit INTEGER DEFAULT 10000
                )
            ''')
    
    def log_call(self, endpoint, quota_used, status_code):
        """Log API call for tracking"""
        with sqlite3.connect(self.db_path) as conn:
            conn.execute(
                'INSERT INTO api_calls VALUES (NULL, ?, ?, ?, ?)',
                (datetime.now(), endpoint, quota_used, status_code)
            )
    
    def get_today_quota(self):
        """Get today's quota usage"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                'SELECT quota_used FROM quota_tracking WHERE date = ?',
                (datetime.now().date(),)
            )
            result = cursor.fetchone()
            return result[0] if result else 0
```

---

## 🚀 Production Deployment

### 1. Environment-Specific Config

```python
import os

ENV = os.getenv('ENVIRONMENT', 'development')

if ENV == 'production':
    YOUTUBE_API_KEY = fetch_from_secret_manager('youtube-api-key')
    LOG_LEVEL = 'ERROR'
    CACHE_TTL = 3600
elif ENV == 'staging':
    YOUTUBE_API_KEY = os.getenv('YOUTUBE_API_KEY')
    LOG_LEVEL = 'INFO'
    CACHE_TTL = 600
else:  # development
    load_dotenv()
    YOUTUBE_API_KEY = os.getenv('YOUTUBE_API_KEY')
    LOG_LEVEL = 'DEBUG'
    CACHE_TTL = 60
```

### 2. Monitoring & Alerting

```python
import logging

class QuotaAlertHandler(logging.Handler):
    """Alert when quota is running low"""
    
    def emit(self, record):
        if 'quota' in record.getMessage().lower():
            if record.levelno >= logging.WARNING:
                self.send_alert(record.getMessage())
    
    def send_alert(self, message):
        # Send to Slack, email, etc.
        # Example: send_slack_message(f"⚠️ YouTube API: {message}")
        pass

# Add to logger
alert_handler = QuotaAlertHandler()
logger.addHandler(alert_handler)

# Alert when reaching 80% quota
quota_used = get_today_quota()
if quota_used > 8000:
    logger.warning(f"Quota usage at 80%: {quota_used}/10000 units")
```

### 3. Health Checks

```python
def health_check():
    """Verify API connectivity"""
    try:
        request = youtube.search().list(
            part='snippet',
            q='test',
            maxResults=1
        )
        response = request.execute()
        return True, "YouTube API is healthy"
    except Exception as e:
        return False, f"YouTube API error: {str(e)}"

# Use in monitoring
is_healthy, message = health_check()
if not is_healthy:
    logger.error(f"Health check failed: {message}")
    # Alert ops team
```

---

## 📊 Monitoring & Quotas

### Daily Quota Tracking

```python
def track_daily_quota():
    """Monitor daily quota usage"""
    used = get_today_quota()
    limit = 10000
    percentage = (used / limit) * 100
    
    print(f"""
    Daily Quota Report
    ├─ Used: {used:,} units
    ├─ Limit: {limit:,} units
    ├─ Remaining: {limit - used:,} units
    └─ Usage: {percentage:.1f}%
    """)
    
    # Alert thresholds
    if percentage > 90:
        logger.critical("Quota usage above 90%!")
    elif percentage > 80:
        logger.warning("Quota usage above 80%!")
    elif percentage > 70:
        logger.info("Quota usage above 70%!")
```

### Performance Monitoring

```python
import time

def api_call_with_metrics(func):
    """Decorator to measure API call performance"""
    def wrapper(*args, **kwargs):
        start = time.time()
        try:
            result = func(*args, **kwargs)
            duration = time.time() - start
            logger.info(f"{func.__name__} completed in {duration:.2f}s")
            return result
        except Exception as e:
            duration = time.time() - start
            logger.error(f"{func.__name__} failed after {duration:.2f}s: {e}")
            raise
    return wrapper

@api_call_with_metrics
def check_channel(channel_id):
    request = youtube.channels().list(
        part='contentDetails',
        id=channel_id
    )
    return request.execute()
```

---

## 🧪 Testing

### Unit Tests with Mocking

```python
import unittest
from unittest.mock import patch, MagicMock

class TestYouTubeAPI(unittest.TestCase):
    
    @patch('googleapiclient.discovery.build')
    def test_search_videos(self, mock_build):
        # Mock the API response
        mock_youtube = MagicMock()
        mock_build.return_value = mock_youtube
        
        mock_response = {
            'items': [
                {'id': {'videoId': 'vid1'}, 'snippet': {'title': 'Video 1'}}
            ]
        }
        mock_youtube.search().list().execute.return_value = mock_response
        
        # Test your code
        from my_app import search_videos
        results = search_videos('jain')
        
        # Assertions
        self.assertEqual(len(results), 1)
        self.assertEqual(results[0]['id']['videoId'], 'vid1')
```

---

## 📋 Production Checklist

- [ ] API key is in environment variable (not hardcoded)
- [ ] `.env` file is in `.gitignore`
- [ ] Sensitive data never appears in logs
- [ ] Error handling implemented for all API calls
- [ ] Retry logic with exponential backoff
- [ ] Quota monitoring and alerting
- [ ] Health checks implemented
- [ ] Database for state management
- [ ] Rate limiting per API endpoint
- [ ] Request signing/authentication verified
- [ ] HTTPS only for API calls (automatic with SDK)
- [ ] API key restricted to specific IPs/domains
- [ ] Logging configured (not too verbose)
- [ ] Unit tests with mocked API calls
- [ ] Documentation for API key rotation
- [ ] Monitoring dashboard set up
- [ ] Incident response plan ready

---

**Ready for production?** Check this checklist before deploying.  
**Want optimization tips?** Review the Efficiency section.  
**Need security hardening?** Start with Security Best Practices.
