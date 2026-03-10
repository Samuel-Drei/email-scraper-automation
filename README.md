# 📧 Email Scraper Automation

#### Video Demo: [TODO: Add URL]

## Description

**Email Scraper Automation** is a full pipeline that automatically collects business emails from websites found on Google Maps and sends personalized cold emails at scale — without any manual copy-pasting.

The project was built to solve a real problem at **Ibrand Agency**: reaching out to real estate businesses whose contact emails are buried inside their websites or domain registration records. Instead of spending hours doing this manually, this system automates the entire process from domain → email → outreach.

---

## How It Works

The pipeline has two main parts:

### Part 1 — Email Scraping (n8n Workflow)

A [Map Scraper]([https://www.stevesie.com/apps/google-maps-scraper](https://chromewebstore.google.com/detail/maps-scraper-business-lea/ofbhiclojhjkggpnapcnjjjdjlgcikdf?hl=pt-BR&utm_source=ext_sidebar)) browser extension is used to export business website URLs from Google Maps into a spreadsheet. From there, the n8n automation workflow takes over:

1. **Loop Over Items** — iterates over each domain in the spreadsheet one by one
2. **Normalização (Normalization)** — a Python script that cleans the raw domain string, stripping `https://`, `http://`, `www.` prefixes and trailing paths, producing a clean base domain (e.g. `example.com.br`)
3. **Wait2** — rate-limit pause to avoid overloading external APIs
4. **Two parallel scraping strategies run simultaneously:**
   - **HTTP Request2 → Scrapper1 (HTML Scraper):** fetches the website's raw HTML and runs a JavaScript extractor that uses regex to find emails, including obfuscated formats like `[at]`, `(dot)`, and `&#64;`. It filters out noreply, support, admin addresses and validates TLDs (`.com`, `.com.br`, `.br`, etc.)
   - **HTTP Request → RDAP_Protocol1 (RDAP Lookup):** queries the RDAP protocol (a public WHOIS replacement) for the domain's registration data, extracting contact emails from the `vcardArray` of registered entities
5. **Merge** — combines results from both scrapers into a single item
6. **Finalizer** — a JavaScript node that deduplicates all emails using a `Set`, merges source labels (`rdap`, `scraper`), and outputs a clean newline-separated email list alongside the domain
7. **Append or update row in sheet1** — writes the results back to the Google Sheet
8. **Wait3** — final pause before the next loop iteration

### Part 2 — Automated Email Sending (Google Apps Script)

Once the sheet is populated with emails, a Google Apps Script runs directly inside Google Sheets:

- Reads all emails from the `Email` column (supports multiple emails per cell, separated by newline, comma, or semicolon)
- Uses a **Gmail Draft** as the email template (identified by subject line), supporting `{{variable}}` placeholders mapped to spreadsheet columns
- Sends emails in **batches of 95/day** to respect Gmail's sending limits, with an 800ms sleep between each send to avoid spam flagging
- Uses **LockService** to prevent duplicate sends from concurrent trigger executions
- Tracks which emails were already sent inside the `Email Sent` column (logs address + timestamp per send)
- Schedules the next batch automatically via a **time-based trigger** (~24h interval)
- Sends log emails to the operator after each batch with sent count and next scheduled run

---

## Files

| File | Description |
|------|-------------|
| `scripts/normalization.py` | Python node in n8n — cleans raw domain strings |
| `scripts/rdap_extractor.py` | Python node in n8n — parses RDAP API response, extracts emails from vCard |
| `scripts/html_scraper.js` | JavaScript node in n8n — fetches and parses raw HTML, extracts and filters business emails |
| `scripts/finalizer.js` | JavaScript node in n8n — deduplicates emails and merges source labels across both scrapers |
| `scripts/mail_merge.gs` | Google Apps Script — batch email sender with template engine, scheduling, and status tracking |
| `workflow/email_scraper.json` | Exported n8n workflow JSON (importable directly into any n8n instance) |

---

## Design Decisions

**Why two scraping methods?**  
No single method is reliable for every domain. RDAP returns structured registration data but only contains emails if the registrant didn't opt for privacy. HTML scraping catches contact page emails but can miss obfuscated ones. Running both in parallel maximizes coverage.

**Why n8n instead of a pure Python script?**  
n8n provides visual orchestration, built-in retry logic, easy rate-limit controls via Wait nodes, and native Google Sheets integration — all without needing to deploy a server.

**Why Google Apps Script for sending?**  
The emails are already in Google Sheets. Using Apps Script keeps the entire workflow inside Google's ecosystem, avoids third-party SMTP costs, and allows the team to monitor sends directly in the spreadsheet.

**Why a batch limit of 95/day?**  
Gmail's personal account limit is 500 emails/day; Google Workspace is 2000/day. The 95-email cap was set conservatively to avoid triggering spam filters, given that cold outreach to multiple unknown domains in a single burst is a common spam pattern.

**AI Acknowledgement**  
The initial versions of `html_scraper.js`, `rdap_extractor.py`, and `finalizer.js` were generated with the assistance of AI tools (Gemini). The `mail_merge.gs` sender was substantially refactored from an AI-generated base to add proper batch control, LockService, inline image support, and multi-email-per-cell handling. All code was reviewed, tested, and modified by the author.

---

## Setup

### n8n Workflow
1. Import `workflow/email_scraper.json` into your n8n instance
2. Configure Google Sheets credentials in n8n
3. Feed a list of domains into the input spreadsheet
4. Run the workflow

### Mail Merge (Google Sheets)
1. Open your Google Sheet containing the scraped emails
2. Go to **Extensions → Apps Script** and paste the contents of `mail_merge.gs`
3. Create a Gmail Draft with subject: `Por que alguns anúncios imobiliários performam mais`
4. Use the custom menu **📧 Automação Pro → 🚀 Iniciar Automação** to start sending

---

## Tech Stack

- **n8n** — workflow automation
- **Python** — domain normalization and RDAP parsing (n8n Code nodes)
- **JavaScript** — HTML email extraction and deduplication (n8n Code nodes)
- **Google Apps Script** — automated email sending via Gmail
- **RDAP Protocol** — public domain registration data lookup
- **Google Sheets** — data storage and status tracking
