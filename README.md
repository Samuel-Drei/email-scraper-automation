# Email Scraper & Outreach Automation

Automation built with n8n to collect, normalize and deduplicate public emails from websites.

## What this project does
- Scrapes public emails from websites (HTML + RDAP)
- Normalizes and removes duplicates
- Merges multiple data sources
- Stores data in Google Sheets
- Sends personalized emails in controlled batches

## Why I built this

I built this project to automate the process of collecting business emails from public websites, which was previously a slow and repetitive manual task.
With this system, the same task is executed automatically, reducing the manual workload by roughly **30%–40%**, while also improving consistency and accuracy.
Beyond time savings, the automation eliminates human error, prioritizes higher-quality emails, and integrates directly with a controlled email-sending pipeline. This allowed the company to scale outreach without increasing operational effort.

## Tech stack
- n8n
- JavaScript and Python (Function nodes)
- HTTP Requests
- RDAP (Registration Data Access Protocol)
- Google Sheets sender

## How to run
1. Import the workflow JSON into n8n
2. Set the Appscripts automation on sheets (change the draft's name in the code)
3. Configure credentials
4. Run the workflow
5. Run the Sheets Automation
