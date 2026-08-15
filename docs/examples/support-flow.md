# Example: Support Flow

Route support requests to different paths based on issue type.

## Overview

When users need support, ask for the issue type and route them appropriately.

## Flow Configuration

### Basic Settings

```
Flow Name: Support Request
Enabled: ✓
Trigger Keywords: support, help, issue, problem
Initial Message: I'm here to help! Let me route you to the right team.
```

### Steps

Steps are ordered by drag-and-drop. Use **Conditional Next** and **Next Step** to control flow between branches.

#### Step 1: Ask Issue Type

| Field | Value |
|-------|-------|
| Step Name | ask_issue_type |
| Message | What type of issue do you have? |
| Message Type | Text |
| Input Type | Select |
| Options | Billing\|Technical\|Order Issue\|General Question |
| Store As | issue_type |
| Conditional Next | (see below) |

**Conditional Next:**
```json
{
    "billing": "billing_info",
    "technical": "tech_details",
    "order issue": "order_lookup",
    "general question": "general_query"
}
```

---

#### Step 2a: Billing Info

| Field | Value |
|-------|-------|
| Step Name | billing_info |
| Message | For billing inquiries:\n\n📧 Email: billing@company.com\n📞 Phone: 1800-123-4567\n⏰ Mon-Fri, 9 AM - 6 PM\n\nOr describe your billing issue and I'll create a ticket: |
| Message Type | Text |
| Input Type | Text |
| Store As | billing_issue |
| Next Step | create_ticket |

---

#### Step 2b: Technical Details

| Field | Value |
|-------|-------|
| Step Name | tech_details |
| Message | Please describe your technical issue in detail: |
| Message Type | Text |
| Input Type | Text |
| Store As | issue_description |
| Next Step | tech_urgency |

#### Step 2b-2: Technical Urgency

| Field | Value |
|-------|-------|
| Step Name | tech_urgency |
| Message | How urgent is this issue? |
| Message Type | Text |
| Input Type | Select |
| Options | Critical - System Down\|High - Major Feature Broken\|Medium - Minor Issue\|Low - Question |
| Store As | urgency |
| Next Step | create_ticket |

---

#### Step 2c: Order Lookup

| Field | Value |
|-------|-------|
| Step Name | order_lookup |
| Message | Please enter your Order ID: |
| Message Type | Text |
| Input Type | Text |
| Store As | order_id |
| Next Step | order_issue_desc |

#### Step 2c-2: Order Issue Description

| Field | Value |
|-------|-------|
| Step Name | order_issue_desc |
| Message | What's the issue with order {order_id}? |
| Message Type | Text |
| Input Type | Text |
| Store As | issue_description |
| Next Step | create_ticket |

---

#### Step 2d: General Query

| Field | Value |
|-------|-------|
| Step Name | general_query |
| Message | Please describe your question: |
| Message Type | Text |
| Input Type | Text |
| Store As | issue_description |
| Next Step | create_ticket |

---

#### Step 3: Create Ticket (Script)

| Field | Value |
|-------|-------|
| Step Name | create_ticket |
| Message | Creating ticket... |
| Message Type | Script |
| Input Type | None |

**Response Script:**

```python
issue_type = data.get('issue_type', 'General')
description = data.get('issue_description') or data.get('billing_issue', '')
urgency = data.get('urgency', 'Medium')
order_id = data.get('order_id', '')

# Map urgency to priority
priority_map = {
    'critical - system down': 'Urgent',
    'high - major feature broken': 'High',
    'medium - minor issue': 'Medium',
    'low - question': 'Low'
}
priority = priority_map.get(urgency.lower(), 'Medium')

# Build subject
subject = f"WhatsApp: {issue_type}"
if order_id:
    subject += f" - Order {order_id}"

# Create Issue document
try:
    ticket = frappe.get_doc({
        'doctype': 'Issue',
        'subject': subject,
        'description': description,
        'priority': priority,
        'raised_by': phone_number,
        'issue_type': issue_type
    })
    ticket.insert(ignore_permissions=True)
    frappe.db.commit()

    response = f"""✅ *Support Ticket Created*

Ticket ID: {ticket.name}
Type: {issue_type}
Priority: {priority}

Our team will respond within:
• Critical: 1 hour
• High: 4 hours
• Medium: 24 hours
• Low: 48 hours

You'll receive updates on WhatsApp."""

except Exception as e:
    frappe.log_error(f"Ticket creation error: {str(e)}", "WhatsApp Chatbot")
    response = """⚠️ Sorry, I couldn't create a ticket.

Please contact us directly:
📧 support@company.com
📞 1800-123-4567"""
```

### Completion Settings

```
Completion Message: Is there anything else I can help with?
```

## Flow Diagram

```
                        ┌─────────────────┐
                        │ ask_issue_type  │
                        └────────┬────────┘
                                 │
    ┌────────────────┬───────────┼───────────┬────────────────┐
    ▼                ▼           ▼           ▼                ▼
┌─────────┐   ┌─────────────┐ ┌───────────┐ ┌──────────────┐
│ billing │   │ tech_details│ │order_lookup│ │general_query │
│ _info   │   └──────┬──────┘ └─────┬─────┘ └──────┬───────┘
└────┬────┘          │              │              │
     │         ┌─────▼──────┐ ┌─────▼──────┐       │
     │         │tech_urgency│ │order_issue │       │
     │         └─────┬──────┘ │   _desc    │       │
     │               │        └─────┬──────┘       │
     │               │              │              │
     └───────────────┴──────────────┴──────────────┘
                                 │
                        ┌────────▼────────┐
                        │  create_ticket  │
                        │    (Script)     │
                        └─────────────────┘
```

## Note on Step Ordering

With branching flows like this, the row order in the UI doesn't matter as much because:
- The first step is always executed first
- **Conditional Next** and **Next Step** control which step runs next
- Steps without explicit next step go to the next row

You can drag steps to organize them logically in the UI.

## Testing Scenarios

1. **Billing Issue:**
   - Type "support" → Select "Billing" → Describe issue → Ticket created

2. **Critical Technical Issue:**
   - Type "support" → Select "Technical" → Describe → Select "Critical" → Ticket created with Urgent priority

3. **Order Problem:**
   - Type "support" → Select "Order Issue" → Enter order ID → Describe → Ticket created with order reference
