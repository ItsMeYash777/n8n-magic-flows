# 📧 Gmail → Slack Automation

## Overview
Automatically forward important unread emails from Gmail to Slack channel.

---

## Quick Setup

### 1. Gmail API Setup
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Enable Gmail API
3. Create OAuth2 credentials
4. Add redirect URI: `https://app.n8n.cloud/rest/oauth2-credential/callback`

### 2. Slack App Setup
1. Go to [Slack API](https://api.slack.com/apps)
2. Create new app
3. Add bot scopes: `channels:read`, `chat:write`
4. Install to workspace
5. Copy bot token (starts with `xoxb-`)

---

## n8n Workflow

### Node 1: Gmail Trigger
```
Type: Gmail Trigger
Event: Email Received
Filters:
  - Label: INBOX
  - Unread: true
  - Important: true
Poll: Every 2 minutes
```

### Node 2: Slack Message
```
Type: Slack → Send Message
Channel: #important-emails
```

**Message Template:**
```
🚨 *Important Email*

*From:* {{$json.payload.headers.find(h => h.name === 'From').value}}
*Subject:* {{$json.payload.headers.find(h => h.name === 'Subject').value}}
*Preview:* {{$json.snippet}}

📧 <https://mail.google.com/mail/u/0/#inbox/{{$json.id}}|Open in Gmail>
```

### Node 3: Mark as Read (Optional)
```
Type: Gmail → Modify Message
Message ID: {{$json.id}}
Action: Remove label "UNREAD"
```

---

## Testing
1. Send yourself an important email
2. Mark it as important in Gmail
3. Check your Slack channel for notification
4. Adjust polling interval if needed

**Setup Time:** ~15 minutes  
**Value:** Never miss important emails in team chat!
