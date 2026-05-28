---
title: "Automating Instagram Without the Instagram API"
date: 2026-03-25
slug: instagram-no-api
tags: ["automation", "instagram", "python", "cyberforks"]
description: "Meta's API is a nightmare of Facebook Pages and OAuth scopes. So we automated it with a browser instead."
---

# Automating Instagram Without the Instagram API

The Instagram API exists. I would not recommend it.

To post a single photo programmatically, Meta requires: a Facebook Page, a connected Instagram Business account, a developer app, an approved `instagram_content_publish` permission, and an access token that expires. Miss any one of those and you get a 400 error with a reference code that links to documentation last updated in 2021.

We needed to automate social posting for [Cyberforks](https://cyberforks.com). We spent two hours on the API path before deciding life is short and browsers are programmable.

## The Approach

Browser automation via Playwright. Log in once, save the session, post via the web UI on a schedule.

The core insight: Instagram's web interface is stable. The API changes constantly and requires ongoing OAuth maintenance. The browser approach requires zero API keys, zero Facebook Pages, and zero permission reviews.

```python
from playwright.sync_api import sync_playwright

def post_to_instagram(image_path: str, caption: str, session_file: str):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        context = browser.new_context(storage_state=session_file)
        page = context.new_page()

        page.goto("https://www.instagram.com/")
        # Navigate to post creation, upload image, set caption, submit
        # ... (full implementation in the repo)

        context.storage_state(path=session_file)  # refresh session
        browser.close()
```

The session file handles authentication persistence. Log in manually once via `--setup` mode, and the saved cookies carry you through automated runs until they expire — usually weeks.

## The Scheduler

A cron job fires daily and pulls from a content queue — a simple JSON file with scheduled posts, captions, and image paths. The AI generates captions and queues them; the cron job handles delivery.

```json
{
  "queue": [
    {
      "image": "posts/2026-03-25-ttp-vault.png",
      "caption": "Built a pentest TTP vault in Obsidian...",
      "scheduled_for": "2026-03-25T09:00:00"
    }
  ]
}
```

## What Can Go Wrong

**Session expiry.** Instagram will eventually invalidate the saved session. When that happens, the job fails and you get an alert. Re-run setup mode, re-authenticate, done. Happens maybe once a month.

**UI changes.** Instagram occasionally redesigns the post flow. Playwright selectors break. This has happened once in six months of running this. Fix takes about 20 minutes.

**Rate limiting.** Post too frequently and Instagram starts adding friction. We post once per day maximum. Never had an issue.

## Is This Against the ToS?

Technically, automated posting without the official API violates Instagram's Terms of Service. So does every third-party scheduling tool you've ever used — Buffer, Later, Hootsuite — until Meta decides to partner with them.

We're posting our own content to our own account. The risk is account restriction, not legal exposure. Evaluate accordingly.

## Why This Is Worth Knowing

The broader pattern — browser automation as an API alternative — applies everywhere Meta has made their API deliberately difficult. WhatsApp Business API has the same friction. So does LinkedIn. So does half the web.

When an API exists to gatekeep rather than enable, the browser is the honest path forward.

---

*Cyberforks builds security workflows that don't suck. [cyberforks.com](https://cyberforks.com)*
