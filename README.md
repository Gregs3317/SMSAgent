# RingCentral AI SMS Agent

A production-ready Node.js application that uses RingCentral for SMS and Claude AI to automatically handle lead conversations and book appointments for the IRS Fresh Start Program.

## 🎯 What This Does

- **Receives real SMS messages** from leads via RingCentral webhooks
- **AI-powered responses** using Claude to answer questions about IRS Fresh Start
- **Automatic appointment booking** when leads are ready
- **Real-time dashboard** to monitor all conversations
- **Manual intervention** option to take over conversations
- **Complete conversation history** for each lead

## 🏗️ Architecture

```
Lead sends SMS → RingCentral → Webhook → Node.js Server → Claude AI → Response → RingCentral → Lead
```

The application runs 24/7 and automatically responds to incoming messages using AI.

## 📋 Prerequisites

1. **RingCentral Developer Account**
   - Sign up at https://developers.ringcentral.com
   - Create an app with SMS permissions
   - Get: Client ID, Client Secret, JWT Token
   - Get a RingCentral phone number

2. **Anthropic API Key**
   - Sign up at https://console.anthropic.com
   - Create an API key

3. **Public URL for Webhooks**
   - For production: A server with a public domain/IP
   - For local development: Use ngrok (https://ngrok.com)

4. **Node.js** (v16 or higher)

## 🚀 Installation

### Step 1: Clone and Install Dependencies

```bash
cd ringcentral-ai-agent
npm install
```

### Step 2: Configure Environment Variables

Copy the example environment file:

```bash
cp .env.example .env
```

Edit `.env` and fill in your credentials:

```env
# RingCentral Configuration
RINGCENTRAL_CLIENT_ID=your_client_id_here
RINGCENTRAL_CLIENT_SECRET=your_client_secret_here
RINGCENTRAL_JWT_TOKEN=your_jwt_token_here
RINGCENTRAL_SERVER_URL=https://platform.devtest.ringcentral.com
RINGCENTRAL_FROM_NUMBER=+1234567890

# Anthropic Claude API
ANTHROPIC_API_KEY=sk-ant-xxxxx

# Server Configuration
PORT=3000
WEBHOOK_URL=https://your-server.com/webhook/sms

# Agent Configuration
AGENT_NAME=Sarah Johnson
```

### Step 3: Set Up Webhook URL

#### For Local Development (using ngrok):

1. Install ngrok: https://ngrok.com/download
2. Start ngrok:
   ```bash
   ngrok http 3000
   ```
3. Copy the HTTPS URL (e.g., `https://abc123.ngrok.io`)
4. Update `.env`:
   ```env
   WEBHOOK_URL=https://abc123.ngrok.io/webhook/sms
   ```

#### For Production:

Use your server's public URL:
```env
WEBHOOK_URL=https://yourdomain.com/webhook/sms
```

### Step 4: Start the Application

```bash
npm start
```

For development with auto-reload:
```bash
npm run dev
```

You should see:

```
🚀 Starting RingCentral AI SMS Agent...
✅ Authenticated with RingCentral
✅ Webhook subscription created
📞 Listening for SMS at: https://your-url.com/webhook/sms
✅ Server running on port 3000
📊 Dashboard: http://localhost:3000
```

## 📱 Using the Application

### Adding Leads via Dashboard

1. Open http://localhost:3000 in your browser
2. Enter the lead's name and phone number
3. Click "Add Lead & Send Message"
4. The initial message is sent automatically
5. Watch the conversation in real-time!

### Adding Leads via API

```bash
curl -X POST http://localhost:3000/api/leads \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Smith",
    "phone": "+1234567890"
  }'
```

### How It Works

1. **Initial Contact**: When you add a lead, they receive your configured initial message
2. **Lead Responds**: They reply via SMS to your RingCentral number
3. **Webhook Triggered**: RingCentral sends the message to your server
4. **AI Responds**: Claude AI generates an intelligent response
5. **SMS Sent**: Response is sent back via RingCentral
6. **Continuous Loop**: Conversation continues until appointment is booked

### Monitoring Conversations

- **Dashboard**: Real-time view of all conversations at http://localhost:3000
- **Stats**: Total leads, active conversations, booked appointments
- **Message History**: Full conversation log for each lead
- **Manual Override**: Send manual messages anytime

## 🤖 AI Behavior

The Claude AI agent:

- Builds rapport with prospects
- Answers questions about IRS Fresh Start Program
- Qualifies leads by understanding their tax situation
- Suggests specific appointment times
- Confirms bookings automatically
- Keeps messages brief and SMS-friendly (1-3 sentences)

### Key Topics AI Handles:

- **Offer in Compromise**: Settling debt for less
- **Installment Agreements**: Payment plans
- **Penalty Abatement**: Reducing penalties
- **Eligibility Requirements**: Financial hardship assessment

## 🔧 API Endpoints

### POST /api/leads
Add a new lead and send initial message

```json
{
  "name": "John Smith",
  "phone": "+1234567890"
}
```

### GET /api/conversations
Get all conversations

### GET /api/conversations/:phone
Get specific conversation details

### GET /api/stats
Get system statistics

### POST /api/send
Send manual message

```json
{
  "phone": "+1234567890",
  "message": "Can you provide more details?"
}
```

### POST /webhook/sms
Webhook endpoint for RingCentral (automatically configured)

### GET /health
Health check endpoint

## 🎛️ Configuration

### Customize Initial Message

Edit in `.env`:

```env
INITIAL_MESSAGE=Hi {name}! This is {agent} following up on your IRS Fresh Start Program inquiry. I'd love to help you explore your options for tax relief. Do you have 15 minutes this week to discuss your situation?
```

Use `{name}` for lead name and `{agent}` for agent name.

### Change Agent Personality

Edit the system prompt in `server.js` (line ~120) to adjust:
- Tone and style
- Qualification questions
- Booking approach
- Response length

## 🚨 Troubleshooting

### Webhook Not Receiving Messages

1. Check WEBHOOK_URL is publicly accessible
2. Test with: `curl https://your-url.com/webhook/sms`
3. Verify RingCentral webhook subscription in developer portal
4. Check server logs for errors

### Authentication Failed

1. Verify credentials in `.env`
2. Check JWT token hasn't expired
3. Ensure app has SMS permissions
4. Try regenerating JWT token

### Messages Not Sending

1. Check RingCentral phone number format: `+1234567890`
2. Verify account has SMS credits (sandbox has limits)
3. Check from number is registered with RingCentral
4. Review server logs for errors

### AI Not Responding

1. Verify ANTHROPIC_API_KEY is valid
2. Check API quota/limits
3. Review server logs for Claude API errors

## 📊 Production Deployment

### Recommended Setup

1. **Use a VPS or Cloud Service** (AWS, DigitalOcean, etc.)
2. **Set up a domain** with SSL certificate
3. **Use PM2** for process management:
   ```bash
   npm install -g pm2
   pm2 start server.js --name "sms-agent"
   pm2 startup
   pm2 save
   ```
4. **Use a database** (PostgreSQL/MongoDB) instead of in-memory storage
5. **Add authentication** to the dashboard
6. **Set up monitoring** (logs, alerts)
7. **Use environment-specific configs**

### Database Integration

Replace the `conversations` Map with a real database. Example with MongoDB:

```javascript
const mongoose = require('mongoose');

const ConversationSchema = new mongoose.Schema({
  phone: { type: String, unique: true, required: true },
  name: String,
  messages: [{
    text: String,
    direction: String,
    timestamp: Date
  }],
  status: String,
  booked: Boolean,
  createdAt: { type: Date, default: Date.now }
});

const Conversation = mongoose.model('Conversation', ConversationSchema);
```

## 🔐 Security Notes

- Never commit `.env` file to version control
- Use strong JWT tokens
- Restrict API access with authentication
- Validate all webhook requests from RingCentral
- Rate limit API endpoints
- Use HTTPS in production
- Sanitize user inputs

## 📝 License

MIT

## 🤝 Support

For RingCentral API issues: https://developers.ringcentral.com/support
For Anthropic API issues: https://support.anthropic.com

## 🎉 Features Coming Soon

- [ ] Multi-language support
- [ ] Calendar integration for appointment scheduling
- [ ] CRM integration
- [ ] Analytics dashboard
- [ ] A/B testing for messages
- [ ] Conversation templates
- [ ] Lead scoring
- [ ] Email notifications

---

**Made with ❤️ for tax relief professionals**
