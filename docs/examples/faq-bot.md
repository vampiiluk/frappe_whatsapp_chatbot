# Example: FAQ Bot

A simple FAQ bot using keyword replies.

## Overview

Create automatic responses for frequently asked questions.

## Setup

### 1. Business Hours

```
Title: Business Hours
Keywords: hours, timing, open, close, when
Match Type: Contains
Response Type: Text
Response Text:
We're open Monday to Friday, 9 AM to 6 PM IST.
Saturday: 10 AM to 2 PM
Sunday: Closed
```

### 2. Location

```
Title: Location
Keywords: address, location, where, directions
Match Type: Contains
Response Type: Text
Response Text:
📍 Our Office:
123 Main Street, 4th Floor
Mumbai, Maharashtra 400001

Google Maps: https://maps.google.com/...
```

### 3. Contact Info

```
Title: Contact
Keywords: contact, phone, email, call
Match Type: Contains
Response Type: Text
Response Text:
📞 Phone: +91 22 1234 5678
📧 Email: support@company.com
💬 WhatsApp: You're already here!
```

### 4. Pricing

```
Title: Pricing
Keywords: price, cost, pricing, rates, fees
Match Type: Contains
Response Type: Text
Response Text:
Visit our pricing page: https://company.com/pricing

Or type "sales" to speak with our team!
```

### 5. Help Menu

```
Title: Help
Keywords: help, menu, options, commands
Match Type: Exact
Priority: 20
Response Type: Text
Response Text:
🤖 How can I help you?

Type any of these:
• "hours" - Business hours
• "location" - Office address
• "contact" - Contact info
• "pricing" - See our prices
• "sales" - Talk to sales team
• "support" - Get help with issues
```

## Tips

1. **Set higher priority for Help** - So "help" doesn't match other rules with "Contains"

2. **Use Contains for flexibility** - Matches "what are your hours?" and "business hours"

3. **Add emojis sparingly** - Makes messages more readable

4. **Include next steps** - Guide users to other options

## Testing

Send these messages:
- "hello" → Default response (or greeting if configured)
- "help" → Help menu
- "what are your business hours?" → Business hours response
- "where are you located" → Location response
