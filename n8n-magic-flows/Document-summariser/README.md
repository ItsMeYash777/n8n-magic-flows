# 🤖 n8n PDF-to-Telegram Automation Setup Guide

## 📋 Overview

This n8n workflow automatically processes PDF files uploaded to a specific Google Drive folder, analyzes them using Google Gemini AI, and sends the analysis to Telegram.

**Workflow Process:**
1. **Google Drive Trigger** → Monitors folder for new PDF uploads
2. **PDF Filter** → Validates file type
3. **File Download** → Downloads PDF content
4. **Gemini AI Analysis** → Processes and summarizes the PDF
5. **Telegram Notification** → Sends results to your Telegram chat

---

## 🛠️ Prerequisites

- n8n instance (cloud or self-hosted)
- Google account with Drive access
- Google Cloud Console access
- Telegram account
- Google AI Studio access

---

## 🔧 Setup Instructions

### 1. 📁 Google Drive API Setup

#### Step 1.1: Enable Google Drive API
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Navigate to **APIs & Services** → **Library**
4. Search for "Google Drive API"
5. Click **Enable**

#### Step 1.2: Create OAuth2 Credentials
1. Go to **APIs & Services** → **Credentials**
2. Click **+ CREATE CREDENTIALS** → **OAuth client ID**
3. If prompted, configure the OAuth consent screen first:
   - Choose **External** user type
   - Fill in required fields:
     - App name: "n8n PDF Automation"
     - User support email: Your email
     - Developer contact: Your email
   - Add scopes: `https://www.googleapis.com/auth/drive.readonly`
   - Add test users (your email)

4. Create OAuth client ID:
   - Application type: **Web application**
   - Name: "n8n Google Drive"
   - Authorized redirect URIs: `YOUR_N8N_URL/rest/oauth2-credential/callback`
     - For n8n Cloud: `https://app.n8n.cloud/rest/oauth2-credential/callback`
     - For self-hosted: `https://your-n8n-domain.com/rest/oauth2-credential/callback`

5. **Download the JSON** and note:
   - **Client ID**: `xxxxx.apps.googleusercontent.com`
   - **Client Secret**: `GOCSPX-xxxxx`

#### Step 1.3: Get Google Drive Folder ID
1. Open Google Drive in browser
2. Navigate to your target folder
3. Copy the folder ID from URL:
   ```
   https://drive.google.com/drive/folders/1uwYa1-
                                          ↑
                                    This is your Folder ID
   ```

---

### 2. 🧠 Google Gemini API Setup

#### Step 2.1: Access Google AI Studio
1. Go to [Google AI Studio](https://aistudio.google.com/)
2. Sign in with your Google account
3. Accept terms and conditions

#### Step 2.2: Generate API Key
1. Click on **Get API key** in the left sidebar
2. Click **Create API key**
3. Select your Google Cloud project (same as Drive API)
4. Click **Create API key**
5. **Copy and save** the API key: `AIzaSyxxxxx`

#### Step 2.3: Enable Billing (If Required)
1. Some Gemini models require billing enabled
2. Go to [Google Cloud Billing](https://console.cloud.google.com/billing)
3. Link a payment method to your project

---

### 3. 📱 Telegram Bot Setup

#### Step 3.1: Create Telegram Bot
1. Open Telegram and search for **@BotFather**
2. Start a chat with BotFather
3. Send `/newbot` command
4. Follow the prompts:
   - **Bot name**: "PDF Analyzer Bot" (display name)
   - **Bot username**: "your_pdf_bot" (must end with 'bot')

5. **Save the Bot Token**: `8453009597:XXXXXX_lXuvdU0`

#### Step 3.2: Get Your Chat ID
1. Start a conversation with your bot
2. Send any message to the bot (e.g., "/start" or "hello")
3. Open this URL in your browser (replace TOKEN with your bot token):
   ```
   https://api.telegram.org/bot8453009597:XXXXX/getUpdates
   ```

4. Look for the `chat` object in the response:
   ```json
   {
     "result": [
       {
         "message": {
           "chat": {
             "id": 13971211313250e13202,  ← This is your Chat ID
             "type": "private"
           }
         }
       }
     ]
   }
   ```

5. **Save the Chat ID**: `1397013522423402`

---

## 🔐 n8n Credentials Configuration

### 1. Google Drive OAuth2 API
1. In n8n, go to **Settings** → **Credentials**
2. Click **Add Credential**
3. Select **Google Drive OAuth2 API**
4. Fill in:
   - **Client ID**: Your Google OAuth2 Client ID
   - **Client Secret**: Your Google OAuth2 Client Secret
5. Click **Connect my account** and authorize
6. **Save** the credential

### 2. Google Gemini (PaLM) API
1. Add new credential
2. Select **Google PaLM API** or **Google Gemini API**
3. Fill in:
   - **API Key**: Your Google AI Studio API key
4. **Save** the credential

### 3. Telegram API
1. Add new credential
2. Select **Telegram API**
3. Fill in:
   - **Access Token**: Your Telegram bot token
4. **Save** the credential

---

## 📥 Importing the Workflow

### Option 1: Import JSON File
1. Copy the workflow JSON from your file
2. In n8n, click **Import from file** or **Import from clipboard**
3. Paste the JSON content
4. Click **Import**

### Option 2: Manual Setup
Use the workflow structure provided in the JSON file to recreate nodes manually.

---

## ⚙️ Workflow Configuration

### 1. Google Drive Trigger Node
```yaml
Parameters:
- Poll Time: Every Minute
- Trigger On: Specific Folder
- Folder ID: 1uwYa1-XHdwZF7OGRfxhrLsERHl6FqyPF
- Event: File Created
- File Type: application/pdf

Credentials: Google Drive OAuth2 API
```

### 2. PDF Filter (If Node)
```yaml
Condition: 
- Left Value: {{ $json["mimeType"] }}
- Operation: equals
- Right Value: application/pdf
```

### 3. Download File Node
```yaml
Operation: Download
File ID: {{ $json.id }}
Credentials: Google Drive OAuth2 API
```

### 4. Gemini Analysis Node
```yaml
Model: gemini-2.5-flash
Input Type: Binary
Prompt: [Configured for PDF analysis]
Credentials: Google Gemini API
```

### 5. Telegram Node
```yaml
Chat ID: 1397013202
Message: {{ $json.content.parts[0].text }}
Credentials: Telegram API
```

---

## 🚀 Testing the Workflow

### 1. Test Individual Nodes
1. **Execute** each node manually
2. Check outputs and fix any errors
3. Verify credentials are working

### 2. End-to-End Test
1. **Activate** the workflow
2. Upload a PDF to your monitored Google Drive folder
3. Wait for the trigger (up to 1 minute)
4. Check Telegram for the analysis message

### 3. Troubleshooting Common Issues

#### Google Drive Issues
- **Error**: "Insufficient permissions"
- **Solution**: Ensure OAuth2 scope includes `drive.readonly`

#### Gemini API Issues  
- **Error**: "API key invalid"
- **Solution**: Regenerate API key and update credentials

#### Telegram Issues
- **Error**: "Chat not found"
- **Solution**: Send a message to the bot first, then get fresh Chat ID

---

## 📊 Monitoring & Logs

### Enable Workflow Logs
1. Go to **Executions** tab
2. Monitor successful and failed runs
3. Check error messages for debugging

### Google Drive Folder Monitoring
- The workflow checks every minute for new files
- Only PDF files trigger the workflow
- Files are downloaded as binary data for processing

---

## 🔒 Security Best Practices

### API Key Security
- **Never commit** API keys to version control
- Use **environment variables** in production
- **Rotate keys** regularly
- **Restrict API key** permissions in Google Cloud Console

### OAuth2 Security
- Use **HTTPS** for redirect URIs
- **Revoke access** for unused credentials
- **Monitor** OAuth2 usage in Google Cloud Console

### Telegram Security
- **Keep bot token** private
- **Restrict bot** to specific chats if needed
- **Monitor** bot usage through BotFather

---

## 📈 Workflow Optimization

### Performance Tips
- **Adjust polling interval** based on usage
- **Filter file types** early in the workflow
- **Monitor API quotas** and limits

### Cost Optimization
- **Choose appropriate** Gemini model (Flash vs Pro)
- **Limit prompt length** to reduce token usage
- **Cache results** for similar documents

## 📝 Configuration Checklist

- [ ] Google Cloud Project created
- [ ] Google Drive API enabled
- [ ] OAuth2 credentials configured
- [ ] Google Drive folder ID obtained
- [ ] Gemini API key generated
- [ ] Telegram bot created via BotFather
- [ ] Telegram bot token saved
- [ ] Chat ID retrieved from getUpdates
- [ ] All n8n credentials configured
- [ ] Workflow imported successfully
- [ ] Test PDF processed successfully
- [ ] Workflow activated and monitoring

---

## 🎉 You're All Set!

Your n8n automation is now ready to:
✅ Monitor Google Drive for PDF uploads  
✅ Analyze documents with AI  
✅ Send intelligent summaries to Telegram  

**Happy Automating!** 🚀

---

*Last updated: August 2025*
