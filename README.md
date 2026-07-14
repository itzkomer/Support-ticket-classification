# AI Support Ticket Classifier

An n8n workflow that automatically classifies customer support tickets with an LLM, emails the right team, and saves the processed results to a CSV file.

## Features

- Process tickets from:
  - CSV file
  - Scheduled workflow
  - Webhook
- One AI request per ticket (classification + summary)
- Routes tickets to the correct department
- Sends notification emails
- Stores processed tickets in an output CSV
- Falls back to **Manual Review** if AI output is invalid

---

## Workflow

```
 CSV / Schedule / Webhook
            │
            ▼
     Normalize Ticket
            │
            ▼
      AI Classification
            │
            ▼
   Validate AI Response
            │
            ▼
      Route by Department
            │
            ▼
        Send Email
            │
            ▼
     Save to Output CSV
```

---

## Project Structure

```
.
├── workflow.json
├── README.md
├── DESIGN.md
├── .env.example
├── sample_tickets.csv
├── example_output.csv
└── prompts/
    ├── system_prompt.md
    └── user_prompt.md
```

---

## Requirements

- n8n (v1.50+ recommended)
- OpenAI API key
- SMTP credentials
- Read/write access to the input and output CSV files

---

## Setup

Clone the repository:

```bash
git clone https://github.com/itzkomer/Support-ticket-classification
cd Support-ticket-classification
```

Create your environment file:

```bash
cp .env.example .env
```

Fill in your API key, SMTP settings, and file paths.

Start n8n with your environment variables loaded.

---

## Environment Variables

Some important variables:

| Variable | Description |
|----------|-------------|
| OPENAI_API_KEY | OpenAI API key |
| LLM_MODEL | AI model (default: gpt-4o-mini) |
| INPUT_CSV_PATH | Input tickets CSV |
| OUTPUT_CSV_PATH | Output CSV |
| WEBHOOK_PATH | Webhook endpoint |
| CRON_SCHEDULE | Schedule trigger |
| SUPPORT_EMAIL | Support mailbox |
| FINANCE_EMAIL | Finance mailbox |
| ENGINEERING_EMAIL | Engineering mailbox |

See `.env.example` for the full list.

---

## Importing the Workflow

1. Open n8n.
2. Import `workflow.json`.
3. Select your SMTP credential in the email node.
4. Save and activate the workflow.

---

## Running

### Manual

Click **Test Workflow** in n8n.

### Schedule

Activate the workflow and it will process the CSV based on the configured cron schedule.

### Webhook

Example request:

```bash
curl -X POST http://localhost:5678/webhook/tickets \
-H "Content-Type: application/json" \
-d '{
  "ticket_id":"20001",
  "subject":"Charged twice",
  "message":"I was billed twice this month.",
  "customer_email":"customer@test.com"
}'
```

---

## Output

Each processed ticket includes:

- Category
- Department
- AI summary
- Confidence score
- Email destination
- Processing timestamp
- Status

Example:

```csv
ticket_id,category,department,confidence,status
10001,Login Issue,Support,0.94,Processed
```

---

## Notes

- The workflow processes every row in the input CSV each time it runs.
- Invalid AI responses are sent to **Manual Review** instead of failing the workflow.
- The output CSV grows over time and isn't automatically rotated.
- The workflow is designed for self-hosted n8n because it writes directly to the filesystem.

---

## Main Files

| File | Purpose |
|------|---------|
| workflow.json | n8n workflow |
| DESIGN.md | Design decisions |
| .env.example | Configuration template |
| sample_tickets.csv | Example input |
| example_output.csv | Example processed output |
