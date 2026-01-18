# Email Scraper & Outreach Automation

Automation built with n8n to collect, normalize and deduplicate public emails from websites.

## What this project does
- Scrapes public emails from websites (HTML + RDAP)
- Merges multiple data sources
  
- Normalizes and removes duplicates
- Stores data in Google Sheets
- Sends emails in controlled batches

## Why I built this
I wanted to learn how to design real-world automations involving
data extraction, cleaning, merging and delivery.

## Tech stack
- n8n
- JavaScript (Function nodes)
- HTTP Requests
- RDAP
- Google Sheets

## How to run
1. Import the workflow JSON into n8n
2. Configure credentials
3. Run the workflow
