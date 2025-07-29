# 72‑Hour Review & LTV Maximizer (Shopify + n8n)

## What this repo contains
- `workflow.json`: export from n8n Cloud (review + upsell)
- `/email-templates`: HTML emails with unsubscribe footer
- This README: setup, testing, and handoff

## Prereqs
- Shopify **Development store** (Bogus Gateway on)
- n8n **Cloud** (free tier is fine)
- SendGrid (free) or Gmail SMTP
- Google Sheets (logging)
- Slack incoming webhook (optional logs)

## Environment / Credentials (set in n8n)
| Name | Where used | Note |
|---|---|---|
| SHOPIFY_STORE_URL | Shopify Creds | `your-store.myshopify.com` |
| SHOPIFY_ADMIN_TOKEN | Shopify Creds | Admin API access token |
| SENDGRID_API_KEY or SMTP creds | Email node | Single Sender OK |
| GOOGLE_SHEETS_OAUTH | Sheets node | OAuth in n8n |
| SLACK_WEBHOOK_URL | Slack node | Channel like `#automation-logs` |

## Build the flow (n8n Cloud)
1) **Trigger:** Shopify *Order paid* (poll 1m)  
2) **Wait (Review):** **TEST=5m**, PROD=7d  
3) **If:** has `customer.email` AND `accepts_marketing == true`  
4) **Send Email (Review):** load `email-templates/review.html`  
5) **Google Sheets:** append (`order_id`, `customer_email`, `timestamp`, `event_type=review_requested`, `branch=-`)  
6) **Slack:** “✅ Review email sent for Order #{{ $json.id }}”  
7) **Wait (Upsell):** **TEST=10m**, PROD=21d  
8) **If (product_type):** `Beans` → Branch A, else Branch B  
9) **Send Email (Upsell):** load `upsell-beans.html` / `upsell-gear.html`  
10) **Sheets:** append (`upsell_sent`, `Beans`/`Other`)  
11) **Slack:** “💡 Upsell email sent for Order #{{ $json.id }}”

## Compliance
- Unsubscribe footer present in all templates
- Opt‑in gate: check `accepts_marketing` (or your custom flag)

## Testing (no external users)
1) Create 2 products: `Beans` and `Equipment`  
2) Place 3 **Bogus** paid orders (one without email to test skip)  
3) Watch Executions; after waits hit: verify **SendGrid**, **Sheet rows**, **Slack**  
4) Test both branches (Beans/Other)  
5) Export **workflow.json** and commit to `/n8n/workflow.json`  
6) Switch waits to **7d/21d** for production

## Handoff
- Provide `workflow.json`, this README, template files
- 3‑day post‑launch support window
