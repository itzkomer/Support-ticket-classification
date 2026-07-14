# Design Notes

This document explains the main design decisions behind the workflow.

---

## Overall Approach

The goal was to keep the workflow simple, easy to follow, and easy to extend.

Instead of building different logic for each trigger, every input follows the same processing pipeline:

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
 Validate Response
        │
        ▼
 Route Department
        │
        ▼
    Send Email
        │
        ▼
 Save Result to CSV
```

The only thing that changes between triggers is how the ticket enters the workflow.

---

## Ticket Normalization

CSV rows and webhook requests don't have exactly the same structure, so the first step converts them into one common format.

Example:

```json
{
  "ticket_id": "...",
  "subject": "...",
  "message": "...",
  "customer_email": "...",
  "source": "csv"
}
```

After this point, every node works with the same data structure.

This also makes adding another source (database, queue, API, etc.) straightforward.

---

## One AI Call

Each ticket is sent to the LLM only once.

The model returns:

- category
- confidence
- summary

Doing everything in one request reduces API cost, keeps the workflow faster, and avoids inconsistent results between multiple AI calls.

---

## Validation

AI responses should never be trusted blindly.

After the HTTP request, the workflow checks that:

- required fields exist
- confidence is valid
- category is one of the supported values

If something looks wrong, the ticket is sent to **Manual Review** instead of stopping the workflow.

---

## Routing

Routing is handled with a single Switch node.

Supported departments are:

- Support
- Finance
- Engineering
- Product
- Sales
- General Support
- Manual Review

Every branch eventually reaches the same email node, so the email template only exists once.

---

## Why Two Code Nodes?

Most of the workflow uses native n8n nodes.

Only two Code nodes were added.

### Validate & Enrich

Responsible for:

- parsing the AI response
- validating the JSON
- mapping categories to departments
- setting the email recipient
- handling invalid responses

This logic would require many IF and Set nodes, making the workflow much harder to read.

### Append Output CSV

Appends processed tickets to the output file.

Using JavaScript with the filesystem is much simpler than rebuilding the CSV with native nodes every time a ticket is processed.

---

## Error Handling

The workflow is designed to continue processing even if something fails.

Examples:

- AI request fails → Manual Review
- Invalid AI response → Manual Review
- Email fails → ticket is still written to the CSV
- Empty ticket → Manual Review

The goal is to avoid stopping an entire batch because of a single bad ticket.

---

## Configuration

Most settings come from environment variables instead of being hardcoded.

That includes:

- OpenAI model
- API key
- email addresses
- webhook path
- cron schedule
- CSV locations

This makes the workflow easier to move between environments.

---

## Current Limitations

A few things were intentionally left simple for the assignment:

- No duplicate ticket detection
- Output CSV grows indefinitely
- Filesystem access assumes a self-hosted n8n instance
- Tickets are processed one at a time

For a production system, I'd replace the CSV with a database or object storage and add idempotency using the ticket ID.

---

## Future Improvements

Some obvious next steps would be:

- Skip duplicate tickets
- Store failed tickets separately
- Add logging and monitoring
- Use a queue for higher throughput
- Rotate output files automatically
- Add PII masking before sending data to the LLM
- Support multiple languages
- Add a simple review interface for Manual Review tickets

---