# Reddit Upvoter

Automated Reddit upvoting bot with intelligent random timing to help maintain your Reddit engagement streak. Built with Python and Docker, featuring multi-architecture support and automatic 2FA handling.

[![Docker Hub](https://img.shields.io/docker/pulls/zenjabba/reddit-upvoter?style=flat-square)](https://hub.docker.com/r/zenjabba/reddit-upvoter)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Features

- **Smart Random Timing** - Upvotes at unpredictable intervals (configurable 4-12 hours by default)
- **Automatic 2FA** - Built-in TOTP support for seamless authentication
- **Multi-Architecture Docker** - Supports both AMD64 and ARM64 (Raspberry Pi, Apple Silicon)
- **Time Windows** - Restrict upvoting to specific hours (e.g., only during waking hours)
- **Smart Post Selection** - Configurable filters for score, age, NSFW content, and subreddits
- **Comprehensive Logging** - Track all activity with detailed logs
- **State Persistence** - Remembers schedule across restarts
- **Secure** - Credentials protected, rate limiting, retry logic

## Quick Start

```bash
# Clone the repository
git clone https://github.com/zenjabba/reddit-upvoter.git
cd reddit-upvoter

# Create your config
cp config.yaml.example config/config.yaml
nano config/config.yaml

# Start the bot
docker-compose up -d
```

## Configuration

### 1. Get Reddit API Credentials

1. Go to https://www.reddit.com/prefs/apps
2. Click "create another app" or "are you a developer? create an app"
3. Fill in:
   - **name**: Any name (e.g., "Personal Upvoter")
   - **App type**: **script**
   - **redirect uri**: `http://localhost:8080`
4. Save and note your:
   - **client_id** (under "personal use script")
   - **client_secret** (the "secret" field)

### 2. Configure the Bot

Create `config/config.yaml`:

```yaml
reddit:
  client_id: "YOUR_CLIENT_ID"
  client_secret: "YOUR_CLIENT_SECRET"
  username: "YOUR_USERNAME"
  password: "YOUR_PASSWORD"
  totp_secret: ""  # For 2FA users - see 2FA Setup below
  user_agent: "RedditUpvoter/1.0"

timing:
  min_hours: 4          # Minimum hours between upvotes
  max_hours: 12         # Maximum hours between upvotes
  jitter_minutes: 30    # Random variation (±30 min)
  time_windows:         # Optional: Only upvote during these hours
    - start: "08:00"
      end: "23:00"

behavior:
  subreddits:
    - "popular"         # Use r/popular (or specific subreddits)
  sort_by: "hot"        # hot, new, rising, top
  consider_top_n: 25    # Select from top N posts
  min_score: 10         # Minimum post score
  max_age_hours: 24     # Skip posts older than 24h
  skip_nsfw: true       # Skip NSFW content
  skip_already_upvoted: true

logging:
  log_file: "/app/logs/reddit_upvoter.log"  # For Docker
  # log_file: "reddit_upvoter.log"          # For local Python
  log_level: "INFO"
  log_retention_days: 30

safety:
  max_retries: 3
  retry_delay: 60
  exit_on_auth_failure: true
```

### 3. Setup 2FA (If Enabled on Reddit)

If you use two-factor authentication, you need to provide your TOTP secret. **See [2FA-SETUP.md](2FA-SETUP.md) for detailed instructions** on how to extract your secret from various authenticator apps.

Quick summary:
1. Get your TOTP secret from your authenticator app or by resetting 2FA
2. Add it to `config.yaml`:
   ```yaml
   totp_secret: "JBSWY3DPEHPK3PXP"
   ```
3. The bot will automatically generate 2FA codes!

**Full 2FA Setup Guide**: [2FA-SETUP.md](2FA-SETUP.md)

## Documentation

- **[2FA-SETUP.md](2FA-SETUP.md)** - Detailed guide for extracting TOTP secrets from various apps

### Docker Commands

```bash
# View logs
docker logs -f reddit-upvoter

# Restart
docker restart reddit-upvoter

# Stop
docker stop reddit-upvoter

# Check status
docker ps | grep reddit-upvoter
```

## How It Works

1. **Authentication** - Connects to Reddit API using your credentials (with automatic 2FA if configured)
2. **Scheduling** - Calculates a random time for the next upvote (between min/max hours + jitter)
3. **Post Selection** - Fetches posts from configured subreddits and filters based on your criteria
4. **Upvoting** - Randomly selects and upvotes one post
5. **Repeat** - Saves the schedule and waits for the next upvote time

## Security & Safety

- **Credentials Protected** - Never committed to git (included in `.gitignore`)
- **Rate Limiting** - Respects Reddit's API limits with automatic retry logic
- **No Vote Manipulation** - Designed for genuine engagement, not manipulation
- **Transparent** - All actions logged for review
- **Open Source** - Review the code yourself

### Important Notes

- This tool is for **personal use** to maintain genuine engagement
- Do **NOT** use for vote manipulation, brigading, or spam
- Do **NOT** run multiple accounts in coordination
- Violating Reddit's rules may result in account suspension
- Use at your own risk

## Troubleshooting

### Authentication Failed

- Double-check your `client_id`, `client_secret`, `username`, and `password`
- Ensure you created a "script" type app (not "web app")
- For 2FA users: Make sure `totp_secret` is correct (see [2FA-SETUP.md](2FA-SETUP.md))

### No Valid Posts Found

- Lower the `min_score` threshold
- Increase `max_age_hours`
- Try different subreddits or use `popular`
- Try different `sort_by` methods

### Container Won't Start

- Check logs: `docker logs reddit-upvoter`
- Ensure config directory exists and contains `config.yaml`
- Verify file permissions (config should be readable)
- Make sure Docker has access to mount the volumes

### Time Sync Issues (2FA)

If using 2FA and getting authentication errors:
```bash
# Linux: Sync system time
sudo timedatectl set-ntp true
ntpdate -q pool.ntp.org
```

## Example Log Output

```
2025-10-15 10:20:51 - RedditUpvoter - INFO - Successfully authenticated as u/zenjabba
2025-10-15 10:20:51 - RedditUpvoter - INFO - Next upvote scheduled for: 2025-10-15 17:32:20
2025-10-15 10:20:51 - RedditUpvoter - INFO - (approximately 7.6 hours from now)
...
2025-10-15 17:32:20 - RedditUpvoter - INFO - Starting upvote action...
2025-10-15 17:32:21 - RedditUpvoter - INFO - Selecting post from r/popular (sort: hot)
2025-10-15 17:32:22 - RedditUpvoter - INFO - Selected post: 'Interesting title here...' by u/someuser
2025-10-15 17:32:23 - RedditUpvoter - INFO - ✓ Successfully upvoted post: abc123
2025-10-15 17:32:23 - RedditUpvoter - INFO - Upvote action completed successfully!
```

## Architecture

- **Language**: Python 3.11+
- **Key Libraries**:
  - PRAW (Python Reddit API Wrapper)
  - PyOTP (TOTP 2FA support)
  - PyYAML (Configuration)
- **Docker**: Multi-stage build with slim Python base
- **Platforms**: Linux (amd64, arm64), macOS, Windows

## Docker Image

- **Repository**: [zenjabba/reddit-upvoter](https://hub.docker.com/r/zenjabba/reddit-upvoter)
- **Architectures**: `linux/amd64`, `linux/arm64`
- **Base Image**: `python:3.11-slim`
- **Size**: ~150MB
