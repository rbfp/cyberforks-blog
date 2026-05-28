---
title: "I Automated My Bookkeeping With an AI and a Spreadsheet"
date: 2026-03-28
slug: bookkeeping-automation
tags: ["automation", "bookkeeping", "ai", "cyberforks"]
description: "How a simple command in Discord now handles receipt filing, PDF uploads, and spreadsheet entries automatically."
---

# I Automated My Bookkeeping With an AI and a Spreadsheet

Bookkeeping is not hard. It's just tedious in a way that compounds if you ignore it. An email comes in — a receipt, a bill, an invoice. You file it later. Later becomes a month. A month becomes tax season. Tax season becomes suffering.

I got tired of the suffering. So I automated it.

## The Problem

Doing security work means a constant drip of financial artifacts: software subscriptions, cloud bills, invoices, trip receipts. Each one needs to be:

1. Filed in cloud storage with a consistent name
2. Logged in a spreadsheet with amount, date, category, and card used
3. Not lost

None of that is intellectually interesting. All of it takes 3-5 minutes per item. At 10-15 items a month, that's an hour of my life spent copying numbers from PDFs into cells. An hour I'd rather spend on literally anything else.

## The Solution: "file N"

The entire workflow is triggered by two words: `file N` — where N is a number from the most recent email summary in my Discord.

That's it. I type `file 3` and the AI:

1. **Looks up email #3** from the last briefing list
2. **Downloads the attachment** (PDF receipt, invoice, or bill)
3. **Renames it** following a consistent scheme: `MM - Description.pdf`
4. **Uploads it** to the right folder in Google Drive
5. **Logs a row** in the bookkeeping spreadsheet — date, description, amount, card, category
6. **Confirms** with a summary of what it filed

Total time: about 15 seconds. Most of that is API latency.

## The Schema

Every transaction gets logged with the same fields:

| Field | Example |
|-------|---------|
| Date | 2026-03-15 |
| Description | AWS - EC2 Usage |
| Amount | -$47.23 |
| Card | Visa 4411 |
| Category | Cloud Infrastructure |
| Paid Off? | No |
| Notes | us-west-1, 8 instances |

Charges are negative by convention. The "Paid Off?" column tracks whether the credit card statement has been reconciled. Simple enough to build a monthly summary off of with a pivot table.

## File Naming

Consistency here matters more than cleverness:

```
03 - Bill - AWS.pdf
03 - Invoice - Vendor Name.pdf
03 - Trip - SFO Conference.pdf
```

Month prefix sorts everything chronologically. The prefix (`Bill /`, `Invoice /`, `Trip /`) groups by type. Descriptive enough to find without opening.

## What Makes This Work

The AI has two things: Google Drive access and Sheets access, both via OAuth. It knows the folder ID for the current year's receipts and the spreadsheet ID for the bookkeeping ledger. That's the whole integration — no custom backend, no webhook infrastructure, no scheduled jobs.

The intelligence is in the parsing: reading a PDF attachment, extracting the total, identifying the vendor, picking the right category, formatting the row. That's where a language model actually adds value over a dumb script.

## The Catch

It occasionally misreads totals on complex invoices — tax line items, partial payments, multi-service bills. I scan the confirmation message before closing the chat. Takes two seconds. Errors are rare enough that the time savings are still 90%+.

The other catch: you have to actually send the `file N` command. The system doesn't auto-file incoming receipts. I do a sweep once a week, 10 minutes, done. That's the right tradeoff — automated execution, human-triggered timing.

## What This Actually Cost to Build

About two hours. Most of that was getting the OAuth scopes right for Drive and Sheets. The actual filing logic was straightforward once auth was working.

The ROI is obvious: two hours of setup versus an hour of tedium every month, forever. That math closes in the second month.

---

*Cyberforks builds security workflows that don't suck. [cyberforks.com](https://cyberforks.com)*
