---
name: link-building-outreach
description: Use this skill when the user wants to automate link building outreach, send outreach emails for backlinks, find contact information for article authors, generate personalized outreach emails, process a list of link opportunities, or run an outreach campaign. Trigger this skill whenever the user mentions "outreach", "link building", "邮件外链", "contact author", "backlink campaign", "find email for", or asks to process a spreadsheet of outreach targets. Also trigger when the user says "run outreach", "setup outreach", or wants to scale their link building efforts.
metadata:
  author: GEO-SEO
  version: "1.0.4"
  homepage: https://github.com/GEO-SEO/seo-outreach-skill
  primaryEnv: SERPAPI_API_KEY
  requires:
    env:
      - SERPAPI_API_KEY
      - GOOGLE_SHEETS_TRACKER_URL
      - GMAIL_ADDRESS
      - GMAIL_APP_PASSWORD
    config:
      - credentials.json
      - token.json
    bins:
      - python3
---

# Link Building Outreach Skill

Automates the full link building outreach pipeline: reads your opportunity list, finds author contacts, deeply analyzes target articles, and generates personalized outreach emails — ready to send.

## Overview

Use this skill to turn outreach opportunities into a structured workflow: article analysis, contact discovery, personalized email drafting, and optional follow-up handling.

## Best For

- SEO teams running link-building campaigns that need better reply quality
- SaaS teams trying to earn placements in listicles, comparisons, and resource pages
- agencies that need repeatable outreach without mass-template language
- operators who want contact research, relevance analysis, and draft writing in one system

## Start With

```text
Setup outreach — my product is https://yourproduct.com, my audience is SEO teams, my name is Jane, my title is Founder
```

```text
Run outreach
```

```text
Run outreach on https://example.com/best-seo-tools
```

## External Access And Minimum Credentials

This workflow can touch external services. Use the minimum credentials needed for the stage you are running:

- `GOOGLE_SHEETS_TRACKER_URL`: optional source for opportunity rows; prefer read-only or exported CSV
- `SERPAPI_API_KEY`: recommended for article discovery and search-driven contact research
- `GMAIL_ADDRESS` and `GMAIL_APP_PASSWORD`: only needed for actual sending or inbox polling
- `credentials.json` and `token.json`: OAuth alternative for Gmail / Workspace setups

If these are unavailable:

- accept pasted tables or CSV files instead of assuming sheet access
- stop at draft generation instead of pretending the skill can send mail
- do not claim private inbox or spreadsheet access without explicit credentials

## Access Policy

Safe default: this skill should stop at research, contact discovery, and draft generation unless sending is explicitly enabled.

- spreadsheet input is optional; pasted tables and CSV exports are valid inputs
- email sending is optional and must be explicitly enabled by the user
- inbox polling is optional and must be explicitly enabled by the user
- do not assume Gmail, Workspace, inbox, or sheet access by default
- if credentials are missing, produce drafts and a send-ready checklist instead of pretending delivery happened

**Core principle:** The only reason outreach emails don't get replies is that authors can tell in one second it's a mass template. Every step in this pipeline exists to produce one thing: an email that could only have been written for that specific article.

## When to Use

- Before or during a link building campaign
- Processing a batch of outreach opportunities from a spreadsheet
- Finding contact info for article authors
- Generating personalized emails at scale
- Single-opportunity quick outreach

## Quick Start

**First time:** Tell Claude:
> "Setup outreach — my product is [URL], my audience is [description], my name is [name], my title is [title]"

**Every time after:**
> "Run outreach" or "Run outreach from this CSV / sheet export"

---

## Setup (One-Time Only)

Ask the user for these four things if not already provided:

| Input | Description | Example |
|-------|-------------|---------|
| **Product URL** | Main product/service page | `https://yourproduct.com` |
| **Target Audience** | Who uses the product | `SEO professionals, growth hackers` |
| **Your Name** | For email signature | `Jane Smith` |
| **Your Title** | Job title | `Founder at Acme SEO` |

### During Setup, Claude Will:

1. **Fetch product page** — Extract product name, features, USPs, tone/voice
2. **Identify differentiators** — What makes this product stand out vs. alternatives
3. **Generate base email templates** — One per article type (Best/Top, How-to, Review, Resource)
4. **Confirm with user** — Show a summary before proceeding

> ⚠️ **Note:** Competitor analysis requires either a manually provided competitor list or a connected SEO API. Claude will ask if none is available.

Store all setup output in memory for the session. If context is lost, re-run setup.

---

## Outreach Pipeline (6 Steps)

### Step 1 — Read Opportunity List

**Source:** A connected tracker if explicitly configured, or a CSV/table pasted directly.

**Required columns:**

| Column | Description | Example |
|--------|-------------|---------|
| `URL` | Target article to get a link from | `https://example.com/best-seo-tools` |
| `Target Page` | Your page to link to | `https://yourproduct.com` |
| `Status` | Processing flag | `pending` |

**Auto-detected optional columns:** `Article Type`, `Anchor Text`, `Notes`

If a tracker is explicitly configured, filter to rows where `Status = pending`. Update `Status` to `processed` after each row completes. If no tracker is configured, ask the user for a pasted list or CSV export instead.

---

### Step 2 — Detect Article Type

Fetch each target URL and classify based on title keywords and page structure:

| Type | Detection Signals | Author's Core Interest |
|------|-------------------|----------------------|
| **Best / Top List** | Title: `best`, `top`, `N tools`, `review of` | Content coverage — did I miss anything important? |
| **How-to / Tutorial** | Title: `how to`, `guide`, `tutorial`, `step-by-step` | Reader success — can readers actually complete each step? |
| **Review / Comparison** | Title: `review`, `vs`, `comparison`, `alternatives` | Helping readers choose — did I cover the right options? |
| **Resource Page** | Many outbound links, aggregator structure | Resource completeness — is this list genuinely useful? |
| **Other** | Cannot classify | Flag for manual review |

**Why this matters:** The same product needs a completely different pitch depending on article type. For a Top List, the angle is "your list is missing this tool." For a How-to, the angle is "this tool removes the hardest manual step in your tutorial." Same product, entirely different email. Getting the type wrong means the entire email will feel off.

If user pre-filled `Article Type`, skip detection and use their value.

---

### Step 3 — Find Contact Information

**Priority principle:** Finding the right person matters more than finding any email. Sending to the actual author or site owner gets 3-5x higher reply rates than sending to a generic contact address.

**Never guess an email address.** A guessed email that bounces damages your sending domain's reputation and flags your domain as a spam source. An honest "Not Found" is always better than an unreliable guess.

Search in this priority order. Stop as soon as a high-confidence result is found:

```
1. Article byline
   → Extract author full name
   → Search "author name + site domain" to find their contact

2. Website Contact / About page
   → Visit /contact, /about, /team, /advertise
   → Extract any email directly listed — this is the most reliable source

3. Author personal website
   → If byline links to author's own site, visit it
   → Check their About or Contact page

4. Twitter/X bio
   → If author has a linked Twitter profile, visit it
   → Check bio text and pinned tweet for email

5. Not found
   → Mark as Not Found
   → Recommend Twitter DM or LinkedIn message instead
   → Do NOT attempt to guess or construct an email address
```

> ⚠️ **LinkedIn:** LinkedIn prohibits automated data scraping. If LinkedIn is the only available source, flag the entry as `confidence: Low` and explicitly recommend manual DM. Do not attempt to extract data from LinkedIn profiles.

**Output per entry:**
```
email: john@example.com
source: contact page / byline search / Twitter bio / not found
confidence: High / Medium / Low / Not Found
```

**Confidence levels and recommended actions:**

| Confidence | Meaning | Action |
|------------|---------|--------|
| **High** | Directly from Contact page or official source | Send immediately |
| **Medium** | From social bio or indirect page | Quick verify on Hunter.io before sending |
| **Low** | Single unverified source | Must verify on Hunter.io before sending |
| **Not Found** | All channels exhausted | Switch to Twitter DM or LinkedIn message |

---

### Step 4 — Deeply Analyze Target Article

**This step determines email quality.** Generic outreach tools stop at Step 3. The difference between a reply and an ignored email is almost entirely decided here.

Fetch and read the full article content. Extract four things:

---

**① Find a specific, quotable reference to use in the email**

Goal: Make the author know in the first sentence that you genuinely read their article — not a bot, not a template.

Find one specific thing from the article: a concrete claim, a data point, a recommendation, or a position the author took. This will be the opening of the email.

❌ Without this analysis:
> "I really enjoyed your article on SEO tools. It was very insightful!"
> *(Could apply to any article. Author ignores it.)*

✅ With this analysis:
> "Read your SEO tools comparison — you pointed out that most tools have a 24-48hr backlink data lag. That's something we hear constantly from users too."
> *(References something specific. Author thinks: "This person actually read it.")*

---

**② Identify the content gap — your entry point**

Find what the article does NOT cover: a tool category, a use case, a type of user, or a feature area that's missing.

This gap is the reason your product belongs in the article. It's not "our product is great" — it's "your article covers X and Y but not Z, and Z is what [Your Product] does."

Example: An article reviewing 10 SEO tools covers keyword research, technical audit, and backlink analysis — but says nothing about AI Overview tracking. If your product tracks AI Overviews, that's the gap. The pitch writes itself.

---

**③ Assess the author's writing style and audience**

Determine: Is this author data-driven or experience-driven? Are they writing for beginners or experts? Is the tone formal or casual?

This directly controls the email's tone and vocabulary. A mismatch in tone — too formal for a casual blogger, too casual for a professional publication — will make even good content feel wrong.

| Author Style | Email Approach |
|-------------|----------------|
| Data-driven, cites research | Lead with numbers or a specific metric |
| Experience-based, practical | Lead with a relatable scenario or problem |
| Beginner-focused | Plain language, explain what your product does simply |
| Expert/technical | Skip basics, reference specific technical differentiation |
| Casual, conversational | Short sentences, informal language, no corporate speak |
| Formal, professional | Complete sentences, structured, no slang |

---

**④ Define the one-sentence relevance statement**

Combine the above three into a single sentence: what your product does for THIS article's readers in THIS specific context.

This sentence is the core of the email body. Everything else supports it.

**Produce this analysis card before writing the email:**
```
article_summary: [one sentence on what the article covers and its main conclusion]
key_reference: [the specific quote, claim, or section to open the email with]
gap_identified: [what topic/tool/use case is missing that your product covers]
author_style: [data-driven / experience / beginner / expert + formal / casual]
relevance_statement: [one sentence: your product helps this author's readers do X]
```

---

### Step 5 — Generate Personalized Email

Using setup data + the Step 4 analysis card, generate one email per opportunity.

**Five non-negotiable rules:**

---

**Rule 1: Under 150 words — strictly enforced**

Authors are busy. Every word over 150 reduces reply probability. The email must be completable in 30 seconds. If you cannot say it in 150 words, cut — don't expand.

---

**Rule 2: First sentence must prove you read the article**

Never open with:
- "Hope this finds you well"
- "I came across your article"
- "My name is X and I work at Y"
- Any sentence that could apply to any article

Open with the specific reference identified in Step 4①. Make the author's first reaction be "this person actually read my piece."

---

**Rule 3: Reference ONE specific piece of content — never generic praise**

One precise reference > five vague compliments.

❌ Generic: "Your article was really comprehensive and well-researched."
✅ Specific: "Your point about backlink data lag (24-48hrs for most tools) is something our users bring up constantly."

---

**Rule 4: One ask only — make it impossible to misunderstand**

The author must finish reading and know exactly what you want them to do.

❌ Vague: "Would love to explore some collaboration opportunities or a potential partnership."
✅ Clear: "Would it make sense to add [Product] to your list? Happy to set up a free trial account."

One action. Clear outcome. Low effort to say yes.

---

**Rule 5: Match tone to author style**

Use the author style assessment from Step 4③. Casual author = casual email. Technical author = technical email. This is not optional — tone mismatch is one of the most common reasons good pitches get ignored.

---

**Full before/after example:**

Target article: "Top 10 SEO Tools in 2025"
Author style: Data-driven, professional, expert audience

❌ Email without Skill:
```
Subject: Partnership Opportunity

Hi,

Hope this email finds you well. I came across your article about SEO
tools and found it very helpful and insightful.

I wanted to reach out because we have a great SEO tool called [Product]
that I think would be a great fit for your article. It has many amazing
features including keyword tracking, backlink analysis, site audit,
and much more.

Would love to discuss a potential partnership or collaboration.
Please let me know if you're interested.

Best regards,
[Name]
```

✅ Email with Skill:
```
Subject: One gap in your SEO tools roundup

Hi [Name],

Read your SEO tools comparison — your point about backlink data lag
(24-48hrs for most tools) is something we hear constantly from users.

One category your roundup doesn't cover: AI Overview tracking.
It's becoming a significant organic traffic source, and [Product]
focuses on exactly this — showing which queries trigger AI Overviews
and whether your content appears in them.

Happy to set up a free trial if you're considering adding a new category.

[Name] · [Title]
```

**Why the second email works:**

| Dimension | ❌ Without Skill | ✅ With Skill |
|-----------|----------------|--------------|
| Opening | Generic filler | References specific article content |
| Product description | "Amazing, many features" | One sentence, specific use case |
| Relevance | None | Tied directly to an identified gap |
| Ask | Vague (partnership) | Clear (free trial for new category) |
| Author reaction | "Another mass email" | "This person read my article" |

---

**Templates by article type** (use as base, always customize with Step 4 analysis):

**Best/Top List:**
> Hi [Name], read your [article title] — [specific reference from Step 4①]. One tool missing from your list: [Your Product]. [One-sentence relevance from Step 4④]. Happy to set up a free trial if you're considering an update. — [Your Name], [Title]

**How-to / Tutorial:**
> Hi [Name], your guide on [topic] is one of the cleaner ones I've found — especially [specific section from Step 4①]. One addition that might help your readers: [Your Product] handles [the hard manual step] automatically. Worth a mention if it fits. — [Your Name], [Title]

**Review / Comparison:**
> Hi [Name], solid breakdown of [tools compared] — [specific reference from Step 4①]. One tool that didn't make the list: [Your Product]. Key difference from [competitor they mentioned]: [differentiator from gap analysis]. Happy to share more if you're considering an update. — [Your Name], [Title]

**Resource Page:**
> Hi [Name], found your [page title] while researching [topic]. [Your Product] might be worth adding — [one-sentence relevance]. Let me know if you'd like to take a look. — [Your Name], [Title]

### Step 6 — Review, Send, and Close the Loop

This is where drafts become a fully automated pipeline. Step 6 has three parts: review, send, and loop.

---

#### Part 1: Review the report

Output all processed entries in a table:

| # | Target URL | Article Type | Email | Source | Confidence | Subject | Body | Status |
|---|-----------|--------------|-------|--------|------------|---------|------|--------|
| 1 | [url] | Best/Top | john@x.com | Contact page | High | [subject] | [body] | processed |
| 2 | [url] | How-to | not found | — | Not Found | [subject] | [body] | processed |

User reviews each row. Confidence-based actions before sending:

| Confidence | Action |
|------------|--------|
| **High** | Ready to send |
| **Medium** | Quick verify on Hunter.io, then send |
| **Low** | Must verify on Hunter.io before sending |
| **Not Found** | Switch to Twitter DM or LinkedIn message — remove from send queue |

> **Always review emails before sending.** This Skill generates drafts — you approve. Speed is the AI's job. Quality is yours.

---

#### Part 2: Gmail setup + bulk send

Once the user confirms the report, immediately ask:

> "Ready to send. Which Gmail auth method do you want to use?
> - **App Password** — personal accounts, setup in 5 minutes
> - **OAuth2** — Google Workspace or shared team accounts"

Based on their choice, refer to the **Gmail Auth Setup** section and guide them through configuration. Then generate a send script on the spot that includes:

- Personalized variable substitution from Step 5
- Staggered sending (default 30s delay, adjustable)
- Daily send cap (see **Sending Volume Guidelines**)
- Send log written to `sent_log.json` — records Message-ID per email for thread matching
- Skips already-sent contacts to prevent duplicates

Generate install and run commands:

```bash
# App Password
pip install python-dotenv
python send_emails.py

# OAuth2
pip install google-auth-oauthlib google-auth-httplib2 google-api-python-client python-dotenv
python send_emails.py
```

The loop starts the moment emails are sent.

---

#### Part 3: Closed loop — fetch, analyze, follow up

```
[Emails sent] → fetch replies → AI classify → auto follow-up
                      ↑_____________↓
               repeat until every contact reaches a final state
```

**Final states** (loop stops):
- ✅ Positive — flagged for manual action (send trial, pitch topics, etc.)
- ❌ Rejected — closed
- 🚫 Max follow-ups reached — closed (default: 2 attempts)

**Fetching replies** — trigger with "check for replies" or "who replied":

Generate a fetch script that matches replies using (in priority order):
1. Message-ID — most accurate, from `sent_log.json`
2. Sender email match
3. Sender domain match

Filtering: auto-detect OOO / no-reply responses; skip self-sent emails. Output: `replies.json`. Contacts with no reply are marked `no_reply`.

**AI classification** — trigger with "analyze replies":

| Status | Signal |
|--------|--------|
| ✅ Positive | Interested, wants to collaborate, asks for more info |
| 🔄 Follow up | "Will look later", "reach out next month", etc. |
| ❌ Rejected | Explicitly declined |
| 📭 No reply | No response after specified number of days |
| 🤖 Auto-reply | OOO or system-generated message |

Three outputs: ① summary table with next action per contact ② grouped status counts ③ follow-up task list with dates and priority flags.

**Auto follow-up** — trigger with "send follow-ups":

| Classification | Action | Timing |
|----------------|--------|--------|
| ✅ Positive | Do NOT auto follow-up — flag for manual action | Immediate |
| 🔄 Follow up | Send short "checking in" | +3 days after reply |
| 📭 No reply (first) | Send follow-up #1 | +7 days after original send |
| 📭 No reply (second) | Send follow-up #2 (final) | +7 days after #1 |
| 📭 No reply (third+) | Mark closed — do not contact again | — |
| ❌ Rejected | Mark closed — do not contact again | — |
| 🤖 Auto-reply | Re-queue as no-reply, follow up after OOO window | +5 days |

---

#### Follow-up email rules

**Keep it short — 3 sentences max.** The goal is a gentle nudge, not a new pitch.

- First follow-up: reference the original email briefly, restate the value in one line, single CTA
- Second (final) follow-up: acknowledge it's the last one, keep the door open

**Follow-up templates** (Claude generates these on the spot based on the original email):

First follow-up:
```
Subject: Re: [original subject]

Hi [Name], just circling back on my previous note in case it got buried.

[One-line restatement of the gap/value from the original email.]

Worth a quick look? — [Your Name]
```

Final follow-up:
```
Subject: Re: [original subject]

Hi [Name], last follow-up from me — I don't want to clog your inbox.

If [Your Product] ever becomes relevant for [site], feel free to reach out.

Either way, keep up the great work. — [Your Name]
```

#### Contact lifecycle tracking

After Stage D runs, every contact has a tracked state in `sent_log.json`:

```json
{
  "to_email": "john@example.com",
  "site": "example.com",
  "status": "positive | follow_up | no_reply | rejected | closed",
  "follow_up_count": 1,
  "last_contact_date": "2025-01-15",
  "final_state": false
}
```

Loop continues until `final_state: true` for every contact.

---

## Gmail Auth Setup

### Option 1: App Password (personal accounts)

**Best for:** Personal Gmail accounts, quick setup, single-machine use.

> ⚠️ Requires two-factor authentication (2FA) to be enabled on the account.

**1. Enable 2-Step Verification**

Go to [Google Account Security](https://myaccount.google.com/security) and enable 2-Step Verification.

**2. Generate an App Password**

1. Go to [App Passwords](https://myaccount.google.com/apppasswords)
2. Select app → `Mail`, Select device → `Other` → enter `outreach-tool`
3. Click Generate and copy the **16-character password**

**3. Enable IMAP (required for receiving)**

Gmail → Settings → See all settings → Forwarding and POP/IMAP → **Enable IMAP** → Save Changes

**4. Configure .env file**

```
GMAIL_ADDRESS=your.email@gmail.com
GMAIL_APP_PASSWORD=xxxx xxxx xxxx xxxx
SENDER_NAME=Your Name
SENDER_SITE=yoursite.com
```

**5. Usage in script**

```python
import smtplib, imaplib, os
from dotenv import load_dotenv
load_dotenv()

GMAIL = os.getenv("GMAIL_ADDRESS")
PASSWORD = os.getenv("GMAIL_APP_PASSWORD").replace(" ", "")

# Send
smtp = smtplib.SMTP_SSL('smtp.gmail.com', 465)
smtp.login(GMAIL, PASSWORD)

# Receive
imap = imaplib.IMAP4_SSL('imap.gmail.com')
imap.login(GMAIL, PASSWORD)
```

---

### Option 2: OAuth2 (Google Workspace / team accounts)

**Best for:** Google Workspace, shared team use, compliance requirements.

**1. Create a Google Cloud project**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click the project dropdown → New Project → name it `email-outreach` → Create

**2. Enable the Gmail API**

APIs & Services → Library → search `Gmail API` → Enable

**3. Configure OAuth consent screen**

1. APIs & Services → OAuth consent screen
2. User Type: External (personal) or Internal (Workspace)
3. Add Scopes:
   - `https://www.googleapis.com/auth/gmail.send`
   - `https://www.googleapis.com/auth/gmail.readonly`
4. Add your Gmail address as a test user

**4. Create OAuth2 credentials**

APIs & Services → Credentials → Create Credentials → OAuth client ID → Desktop app → Download JSON → rename to `credentials.json` and place in project root

**5. First-time authorization (run once only)**

```python
# auth_setup.py
from google_auth_oauthlib.flow import InstalledAppFlow

SCOPES = [
    'https://www.googleapis.com/auth/gmail.send',
    'https://www.googleapis.com/auth/gmail.readonly'
]

flow = InstalledAppFlow.from_client_secrets_file('credentials.json', SCOPES)
creds = flow.run_local_server(port=0)

with open('token.json', 'w') as f:
    f.write(creds.to_json())

print("Authorization complete. token.json saved.")
```

A browser window will open automatically. After login, `token.json` is saved and reused for all future runs — no repeat authorization needed.

**6. Usage in script**

```python
from google.oauth2.credentials import Credentials
from google.auth.transport.requests import Request
from googleapiclient.discovery import build

def get_gmail_service():
    creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    if creds.expired and creds.refresh_token:
        creds.refresh(Request())
        with open('token.json', 'w') as f:
            f.write(creds.to_json())
    return build('gmail', 'v1', credentials=creds)
```

> ⚠️ Both `credentials.json` and `token.json` contain sensitive credentials. Add both to `.gitignore` — never commit them to Git.
