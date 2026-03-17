# Talk 2: Live AI Workshop — 5 Use Cases in 35 Minutes

> **Format:** Hands-on live demo. Ali walks through each use case in real time, pasting prompts and showing results on screen.
> **Total time:** ~35 minutes (5–8 min per use case, including transitions)
> **Prep required:** Files created, accounts logged in, tabs open BEFORE the talk starts.

---

## 🔧 Pre-Talk Setup Checklist

Before going on stage, Ali needs:

- [ ] **ChatGPT (GPT-5)** — logged in, fresh chat ready, file upload tested
- [ ] **Claude** — logged in, fresh chat ready
- [ ] **Perplexity Pro** — logged in
- [ ] **Gamma.app** — account created, logged in
- [ ] **Excel file** prepared (Use Case 1 — see Appendix A)
- [ ] **Engagement brief** in a .txt or .docx (Use Case 2 — see Appendix B)
- [ ] **This document** open on a second screen or phone for reference
- [ ] **Browser tabs** pre-opened: ChatGPT, Claude, Perplexity, Gamma
- [ ] **Timer** visible (phone or laptop widget) — keep yourself honest on pacing

---

## Use Case 1: Financial Analysis (Excel)

**⏱ Time: 7 minutes**

### Scenario
A messy spreadsheet of fictional company "GreenPulse Ltd" — 12 months of P&L data with inconsistent formatting, missing values, and mixed currencies. AI cleans it, calculates key ratios, spots trends, and writes an executive summary.

### Tool
**ChatGPT (GPT-5) with file upload** — best-in-class for spreadsheet analysis with visual chart generation. Claude is the backup if ChatGPT is slow.

### Prerequisites
- [ ] Excel file `greenpulse-financials-2025.xlsx` created from data in **Appendix A**
- [ ] File saved to Desktop for quick upload
- [ ] ChatGPT open with a fresh conversation

### Steps

**1. Set the scene (30 sec)**

> **AUDIENCE NOTE:** "Alright — let's start with something every business deals with: messy financials. I've got a spreadsheet here from a fictional company, GreenPulse. It's a mess — mixed currencies, missing data, inconsistent formatting. The kind of thing a junior analyst would spend half a day cleaning. Let's see what AI does with it."

**2. Upload the file and paste the primary prompt**

**Open ChatGPT → Click attachment → Upload `greenpulse-financials-2025.xlsx`**

Then paste:

```
I've uploaded a 12-month P&L spreadsheet for GreenPulse Ltd, an early-stage clean energy startup. The data is messy — there are inconsistent date formats, some values are in GBP and others in EUR (use an exchange rate of 1 EUR = 0.86 GBP to standardise everything to GBP), a few cells have missing values, and the formatting is all over the place.

Please do the following:

1. **Clean the data**: Standardise all values to GBP, fix formatting inconsistencies, and flag any missing values (interpolate where reasonable, note where you've made assumptions).

2. **Calculate key financial metrics** for each month and as annual totals:
   - Gross margin %
   - EBITDA and EBITDA margin %
   - Monthly burn rate
   - Revenue growth rate (month-over-month)
   - Runway estimate (based on current cash position of £420,000 and average burn rate)

3. **Identify the 3 most important trends** in this data — what story do these numbers tell?

4. **Produce a clean, formatted summary table** with all 12 months plus annual totals.

5. **Write a 200-word executive summary** suitable for a board meeting, highlighting performance, risks, and recommended actions.

Format everything clearly with headers and tables. Be specific with numbers — no vague statements.
```

**3. While AI processes (~30–60 sec), talk to audience:**

> **AUDIENCE NOTE:** "Notice what I did there — I didn't just say 'analyse this spreadsheet.' I gave it specific context: the company name, what it does, the exchange rate to use, exactly which metrics I want, and the output format. **Specificity is everything with AI.** A vague prompt gets a vague answer. A detailed prompt gets an analyst-quality output."

**4. Show the output, scroll through it, highlight the summary table and executive summary.**

> **AUDIENCE NOTE:** "Look at that — it's identified the missing values, converted currencies, and calculated every ratio I asked for. This would take a human analyst 2–3 hours. We did it in 45 seconds."

**5. Paste follow-up prompt #1:**

```
Now create a line chart showing monthly revenue vs total expenses over the 12-month period. Add a secondary line for EBITDA. Use clear colours and label everything. Title it "GreenPulse Ltd — 2025 Financial Performance".
```

> **AUDIENCE NOTE:** "And now we're iterating. This is the real power — you don't get it perfect first time, you refine. Just like working with a human analyst, except this one doesn't need a coffee break."

**6. Paste follow-up prompt #2:**

```
Based on this financial data, what are the three biggest risks facing GreenPulse right now? For each risk, rate it High/Medium/Low on likelihood and impact, and suggest one specific mitigation action. Present this as a risk matrix table.
```

**7. Paste follow-up prompt #3 (if time permits):**

```
Write a one-page board summary in formal tone, structured as: (1) Performance Highlights, (2) Key Concerns, (3) Recommended Actions. Use bullet points, include specific numbers, and keep it under 400 words. This should be ready to email to a non-executive director.
```

> **AUDIENCE NOTE:** "Three prompts, three minutes, and we've gone from a messy spreadsheet to a board-ready report with charts and risk analysis. That's the workflow — upload, analyse, iterate, deliver."

### Expected Output
- Clean formatted table of all 12 months with standardised GBP values
- Calculated metrics: gross margin, EBITDA, burn rate, runway
- Line chart (revenue vs expenses vs EBITDA)
- Risk matrix with likelihood/impact ratings
- Polished board summary ready to copy into an email

---

## Use Case 2: Contract Drafting

**⏱ Time: 7 minutes**

### Scenario
Draft a professional services agreement for a consulting engagement from a simple brief. Show how AI can produce a legally structured document, then refine specific clauses through iteration.

### Tool
**Claude** — stronger at long-form structured writing and legal language. Use the latest Claude model available.

### Prerequisites
- [ ] Engagement brief prepared (see **Appendix B**) — either typed out or in a .txt file ready to paste
- [ ] Claude open with a fresh conversation
- [ ] Google Docs or Word open to paste the final output

### Steps

**1. Set the scene (30 sec)**

> **AUDIENCE NOTE:** "Next up — contracts. Every consultancy, every freelancer, every business deals with contracts. Getting a first draft from a lawyer can take days and cost hundreds of pounds. Let's draft a professional services agreement from a one-paragraph brief."

**2. Paste the primary prompt:**

```
I need you to draft a Professional Services Agreement based on the following engagement brief:

---
**Engagement Brief**

**Service Provider:** Meridian Consulting Ltd (Company No. 12345678), registered at 45 King Street, London EC2V 8QA
**Client:** NovaTech Solutions Ltd (Company No. 87654321), registered at 12 Innovation Park, Manchester M1 3BB
**Scope:** Strategic digital transformation advisory — including current-state assessment, technology roadmap development, vendor selection support, and implementation oversight for the client's transition from legacy on-premise systems to cloud-based infrastructure.
**Duration:** 6 months, starting 1 April 2026, with option to extend by mutual written agreement.
**Fee:** £75,000 total, payable in 6 equal monthly instalments of £12,500 + VAT, due within 14 days of invoice.
**Key Personnel:** The service provider will assign a Lead Consultant (minimum 10 years experience) and a Senior Analyst. Named personnel cannot be substituted without client approval.
**Special Terms:** All work to be performed remotely with monthly on-site visits to Manchester. Client to provide access to all relevant systems and documentation within 5 business days of signing. Either party may terminate with 30 days written notice.
---

Draft a complete, professional contract that includes:
1. Definitions and Interpretation
2. Scope of Services (detailed, based on the brief)
3. Duration and Extension
4. Fees and Payment Terms (including late payment provisions)
5. Key Personnel and Substitution
6. Client Obligations
7. Confidentiality
8. Intellectual Property
9. Limitation of Liability
10. Termination
11. Dispute Resolution
12. General Provisions (notices, force majeure, entire agreement, governing law)

Use formal legal English. Include standard boilerplate where appropriate. Make it comprehensive but not unnecessarily verbose. This should be ready for a solicitor to review, not a first-year law student's homework.
```

**3. While AI processes (~45–90 sec), talk to audience:**

> **AUDIENCE NOTE:** "This is a detailed brief — parties, scope, fees, special terms. The more context you give, the better the output. I've also specified the exact structure I want — 12 sections. Without that, the AI might skip important clauses or over-index on some areas. You're the architect, the AI is the builder."

**4. Scroll through the output — it'll be long. Highlight the structure, the definitions section, and the termination clause.**

> **AUDIENCE NOTE:** "Look — proper defined terms, cross-references between clauses, standard legal boilerplate. Is this ready to sign? No. You'd still want a solicitor to review it. But as a first draft that captures 80% of what you need? This saves days of back-and-forth. And the iteration is where it gets powerful..."

**5. Paste follow-up prompt #1:**

```
Add a detailed Intellectual Property clause that specifies: (a) all pre-existing IP remains with its original owner, (b) any IP created specifically for this engagement ("Deliverable IP") transfers to the client upon full payment, (c) the service provider retains the right to use general methodologies, frameworks, and know-how developed during the engagement, and (d) include a licence-back provision so the service provider can reference the work in anonymised case studies.
```

**6. Paste follow-up prompt #2:**

```
Revise the Termination clause to be more favourable to the service provider. Specifically: (a) if the client terminates for convenience, they must pay all fees due for the remainder of the current month plus a termination fee equal to 2 months' fees, (b) the service provider can terminate immediately if invoices remain unpaid for more than 30 days, and (c) add a "kill fee" of 50% of remaining contract value if the client terminates within the first 60 days.
```

> **AUDIENCE NOTE:** "See what just happened? I shifted the balance of the contract with one prompt. This is exactly what you'd do in a negotiation — except instead of waiting three days for your lawyer to send a redline, you've got a revised clause in 20 seconds."

**7. Paste follow-up prompt #3:**

```
Now simplify the entire contract's language. Rewrite it so that a non-lawyer business owner can read and understand every clause without needing legal interpretation. Keep the legal enforceability but remove unnecessary jargon. Use plain English. Where a legal term is essential, add a brief parenthetical explanation.
```

> **AUDIENCE NOTE:** "And that's the magic trick — we went from a brief, to a full contract, to refined clauses, to plain-English simplification. Four prompts, four minutes. The AI doesn't replace your lawyer — but it gets you to the 80% draft in minutes instead of days."

### Expected Output
- Full 12-section professional services agreement (2,000–3,000 words)
- Refined IP clause with licence-back provisions
- Service-provider-favourable termination clause
- Plain-English simplified version

---

## Use Case 3: Brand Identity (Image Generation)

**⏱ Time: 6 minutes**

### Scenario
Create a logo and basic brand identity for a fictional startup called "Aether" — an AI-powered air quality monitoring platform. Show how AI generates visual assets from text descriptions.

### Tool
**ChatGPT with DALL-E 3/4** — integrated, fast, and good at iterating. Alternatively, **Gemini** with image generation if DALL-E is slow.

### Prerequisites
- [ ] Startup description ready (see **Appendix C**)
- [ ] ChatGPT open with a fresh conversation
- [ ] Backup: Gemini open in another tab

### Steps

**1. Set the scene (30 sec)**

> **AUDIENCE NOTE:** "Now let's do something visual. Every new business needs a brand identity — logo, colours, the works. Designers charge thousands for this. Let's see if we can get a solid starting point in five minutes. We're creating a brand for 'Aether' — an AI-powered air quality monitoring startup."

**2. Paste the primary prompt:**

```
I'm building a brand identity for a startup called "Aether" — an AI-powered air quality monitoring platform for smart cities. The target audience is city councils, property developers, and health-conscious urban professionals. The brand should feel: clean, scientific, trustworthy, and slightly futuristic — but not cold or robotic. Think "calm technology."

Generate a logo for Aether. It should be:
- A modern logomark (icon + wordmark)
- The icon should abstractly represent air/atmosphere/breath — think flowing lines, gradients, or a stylised "A"
- Colour palette: deep teal/navy as primary, with a fresh green or sky blue accent
- Clean sans-serif typeface for the wordmark
- Suitable for use on white and dark backgrounds
- Simple enough to work at small sizes (favicon, app icon)

Style: minimal, geometric, professional. Not cartoonish, not overly complex. Think Stripe, Linear, or Notion's brand aesthetic.
```

**3. While image generates (~15–30 sec), talk to audience:**

> **AUDIENCE NOTE:** "With image generation, the description is everything. I've given it a mood — 'calm technology.' I've named reference brands — Stripe, Linear, Notion. I've specified what I don't want — not cartoonish, not complex. The more you constrain the creative space, the better the output."

**4. Show the generated logo. React to it honestly.**

> **AUDIENCE NOTE:** "Now, is this going to win a design award? Probably not. But as a starting point — something to show a designer and say 'this direction' — it's genuinely useful. And watch what happens when we iterate..."

**5. Paste follow-up prompt #1:**

```
I like the direction but I want it more minimal. Strip it back to just a clean icon mark — no text, no wordmark. The icon should work as a standalone app icon. Make it geometric, using only 2-3 colours. Think of how Airbnb's logo works as a simple, recognisable symbol. Keep the teal/blue palette.
```

**6. Paste follow-up prompt #2:**

```
Now, based on this logo design, generate a complete brand colour palette and typography recommendation. Give me:
- Primary colour (hex code)
- Secondary colour (hex code)
- Accent colour (hex code)
- Neutral/text colour (hex code)
- Background colour (hex code)
- Recommended heading font (from Google Fonts)
- Recommended body font (from Google Fonts)

Present this as a clean brand style guide section with colour swatches.
```

> **AUDIENCE NOTE:** "Notice we've gone from 'generate a logo' to building out a whole brand system. Each prompt builds on the last. The AI remembers context — it knows we're working on Aether, it knows the teal palette, it knows the minimal direction. That conversational memory is what makes this workflow possible."

**7. Paste follow-up prompt #3:**

```
Create a professional business card mockup for Aether using the logo and brand colours. The card should show:
- Front: Logo, company name "Aether", tagline "Breathe smarter."
- Back: Name "Sarah Chen", title "Chief Technology Officer", email sarah@aether.io, phone +44 20 7946 0958, website aether.io
- Clean, minimal design consistent with the brand identity we've built.
```

### Expected Output
- Initial logo (icon + wordmark)
- Refined minimal icon mark
- Brand colour palette with hex codes and typography recommendations
- Business card mockup

### Talking Point for Transition

> **AUDIENCE NOTE:** "Four prompts, four minutes — we've gone from a text description to a logo, colour palette, and business card. Is it finished brand identity work? No. But it's a brief you can hand to a designer that saves hours of back-and-forth on direction. And for an early-stage startup that can't afford a design agency yet? This gets you to 'good enough' to launch."

---

## Use Case 4: Market Research Report

**⏱ Time: 8 minutes**

### Scenario
Deep research on the UK sustainable packaging market — structured report with market sizing, competitor analysis, key trends, and strategic recommendations. All from a single prompt, then refined.

### Tool
**Perplexity Pro** — uses real-time web search with cited sources. This is critical for research credibility. If Perplexity is unavailable, use ChatGPT with browsing enabled.

### Prerequisites
- [ ] Perplexity Pro logged in, fresh thread
- [ ] No files needed — researching from scratch

### Steps

**1. Set the scene (30 sec)**

> **AUDIENCE NOTE:** "Now for something that used to take a research team weeks — market research. We're going to research the UK sustainable packaging market from scratch. No files, no data — just a prompt and an AI with access to the internet. The key difference with this tool, Perplexity, is that it cites its sources. Every claim links back to a real article, report, or data source. That's what makes it usable for real business decisions."

**2. Paste the primary prompt in Perplexity Pro:**

```
Conduct a comprehensive market research analysis of the UK sustainable packaging industry as of 2026. I need a structured report covering:

1. **Market Overview**: Current market size (£), growth rate (CAGR), and projected size by 2030. Break down by segment: biodegradable plastics, paper/cardboard alternatives, reusable packaging systems, and edible/dissolvable packaging.

2. **Key Market Drivers**: What's fuelling growth? Cover regulatory changes (UK Plastic Packaging Tax, EPR regulations), consumer demand shifts, retailer commitments (name specific companies), and technological advances.

3. **Competitive Landscape**: Identify the top 10 companies operating in this space in the UK — both established players and high-growth startups. For each, note their focus area, approximate revenue or funding, and key differentiator.

4. **Challenges and Barriers**: What's holding the market back? Cover cost premiums, performance limitations, supply chain constraints, and greenwashing concerns.

5. **Key Trends for 2026-2028**: What's coming next? Emerging materials, AI/automation in packaging, circular economy models, direct-to-consumer shifts.

Cite all statistics and claims with sources. Use the most recent data available. Format with clear headers and bullet points. Write in a professional tone suitable for a strategy presentation to a board of directors.
```

**3. While AI researches (~60–90 sec — this takes longer because it's searching the web), talk to audience:**

> **AUDIENCE NOTE:** "This is going to take a bit longer than the others because it's actually searching the internet in real time — pulling from news articles, industry reports, company websites. Watch the sources it pulls in. A junior analyst would spend a week gathering this. A research firm would charge five to ten thousand pounds. We're about to get a solid first pass in 90 seconds. Is it as deep as a McKinsey report? No. But is it 70-80% of the way there? Absolutely."

**4. Show the output — scroll through, highlight the cited sources.**

> **AUDIENCE NOTE:** "Look at those citations — real sources, real data. This isn't hallucinated. You can click through and verify every number. That's the game-changer for research — AI with sources you can actually trust and check."

**5. Paste follow-up prompt #1:**

```
Now do a SWOT analysis for a hypothetical new entrant in the UK sustainable packaging market — a startup that has developed a seaweed-based alternative to plastic film for food packaging. Consider their position against the established competitors you've identified. Format as a clear SWOT grid with 4-5 points in each quadrant.
```

**6. Paste follow-up prompt #2:**

```
From the competitors you identified, pick the top 5 most relevant to a seaweed-based packaging startup. For each competitor, identify:
- Their primary weakness or vulnerability
- Where a seaweed-based alternative could outcompete them
- Their likely response to a new entrant

Present this as a competitive threat assessment table.
```

> **AUDIENCE NOTE:** "We've gone from general market research to specific competitive intelligence. Each prompt narrows the focus. This is how you use AI for research — start broad, then drill down into exactly what matters for your specific business case."

**7. Paste follow-up prompt #3:**

```
Size the addressable market for seaweed-based food packaging film in the UK:
- **TAM** (Total Addressable Market): The entire UK food packaging film market
- **SAM** (Serviceable Addressable Market): The portion that could realistically switch to seaweed-based alternatives given current technology
- **SOM** (Serviceable Obtainable Market): What a well-funded startup could realistically capture in years 1-3

Show your reasoning and assumptions for each figure. Cite sources for the base market data.
```

**8. Paste follow-up prompt #4 (if time permits):**

```
Write a 250-word executive summary of this entire research, suitable for including in an investor memo. It should cover: the market opportunity, why now, the competitive gap, and the recommended next steps. Make it compelling but grounded in the data we've gathered.
```

### Expected Output
- Structured market research report with 5 sections, all with cited sources
- SWOT analysis for the hypothetical new entrant
- Competitive threat assessment table (top 5 competitors)
- TAM/SAM/SOM market sizing with reasoning
- Investor-ready executive summary

---

## Use Case 5: Investor Pitch Deck

**⏱ Time: 7 minutes**

### Scenario
Take a one-paragraph business idea and produce a complete 12-slide pitch deck with content, speaker notes, and actual designed slides — using AI for content and Gamma.app for design.

### Tool
**Claude** for pitch deck content (strong at structured, persuasive writing) → then **Gamma.app** to turn it into designed slides in one click.

### Prerequisites
- [ ] Business idea paragraph ready (see **Appendix D**)
- [ ] Claude open with a fresh conversation
- [ ] Gamma.app logged in and ready (https://gamma.app)

### Steps

**1. Set the scene (30 sec)**

> **AUDIENCE NOTE:** "Last one — and this is the big finale. We're going to take a one-paragraph business idea and turn it into a complete investor pitch deck. Not just bullet points — actual designed slides. Two AI tools working together: Claude for the content, then Gamma to design it. Let's go."

**2. Paste the primary prompt in Claude:**

```
I need you to create a complete investor pitch deck for the following startup:

---
**Aether** is a B2B SaaS platform that uses networks of low-cost IoT sensors combined with AI-powered analytics to provide real-time, hyperlocal air quality monitoring and forecasting for smart cities. City councils use Aether to identify pollution hotspots, optimise traffic flow to reduce emissions, and provide citizens with real-time air quality data via a public dashboard and mobile app. The platform also serves property developers (air quality scores for new developments) and health services (asthma/respiratory alert systems). Founded in London in 2025, Aether has 3 pilot cities, £1.2M in pre-seed funding, and a founding team of 4 (ex-Dyson engineering lead, ex-DeepMind ML researcher, ex-Capita smart cities PM, and a commercial lead from Octopus Energy).
---

Create content for a 12-slide investor pitch deck (Seed round, raising £4M):

**Slide 1: Title Slide** — Company name, tagline, one-line description
**Slide 2: The Problem** — Air pollution kills 36,000 people per year in the UK. Current monitoring is sparse, outdated, and not actionable. Make this emotional and urgent.
**Slide 3: The Solution** — What Aether does, in simple terms. Include a visual description.
**Slide 4: How It Works** — Technical overview in 3 simple steps. Keep it accessible.
**Slide 5: Market Opportunity** — Market size, growth, why now.
**Slide 6: Traction** — 3 pilot cities, key metrics, testimonials (create plausible ones).
**Slide 7: Business Model** — How you make money. Pricing tiers.
**Slide 8: Competitive Landscape** — 2x2 matrix positioning. Who else is here, why you're different.
**Slide 9: The Team** — Founders with their relevant credentials.
**Slide 10: Go-to-Market Strategy** — How you'll acquire customers in the next 12 months.
**Slide 11: Financial Projections** — 3-year revenue forecast with key assumptions.
**Slide 12: The Ask** — Raising £4M, how you'll use it, what milestones it unlocks.

For each slide, provide:
- **Headline** (bold, attention-grabbing)
- **Body content** (2-4 bullet points or short paragraphs — deck-appropriate, not essay-length)
- **Visual suggestion** (what graphic, chart, or image should accompany this slide)

Write in a confident, investor-facing tone. Every slide should tell a story and build towards the ask.
```

**3. While AI processes (~60 sec), talk to audience:**

> **AUDIENCE NOTE:** "This is the most complex prompt we've done tonight — I'm asking for 12 slides, each with a headline, content, and visual suggestion. The key here is structure. I've told it exactly what each slide should be, what tone to use, and even the round details — raising £4M seed. The more you specify, the less editing you do later."

**4. Show the output — scroll through a few slides, highlight the problem slide and the financial projections.**

> **AUDIENCE NOTE:** "Look at the problem slide — it's not just statistics, it's framed emotionally. And the financial projections are structured with assumptions. This is content you could genuinely take into a pitch meeting as a starting point."

**5. Paste follow-up prompt #1:**

```
Rewrite Slide 2 (The Problem) to be more emotionally compelling. Start with a specific story: a mother in London checking her phone before deciding whether it's safe for her asthmatic daughter to walk to school. Then zoom out to the macro statistics. Make the audience feel the problem before they see the numbers.
```

> **AUDIENCE NOTE:** "This is a storytelling technique — start with one person, then zoom out to the millions. AI can do this if you ask it explicitly. The first draft was good; this one will land with investors."

**6. Paste follow-up prompt #2:**

```
Create a detailed Financial Projections slide with a 3-year forecast. Show:
- Year 1: 8 cities, £480K ARR
- Year 2: 25 cities + 40 property developers, £2.1M ARR
- Year 3: 60 cities + 120 developers + 15 NHS trusts, £6.8M ARR

Include key assumptions (average contract value, churn rate, sales cycle length). Present as a table with a note on path to profitability.
```

**7. Paste follow-up prompt #3:**

```
Create the Competitive Landscape slide as a 2x2 matrix. The axes should be:
- X-axis: "Data Granularity" (low to high)
- Y-axis: "Actionable Insights" (low to high)

Position these competitors: traditional government monitoring (DEFRA), BreezoMeter, Plume Labs, IQAir, and Aether. Aether should be in the top-right quadrant. Explain why for each positioning in a brief note below the matrix.
```

**8. Now bring it to life — switch to Gamma.app**

> **AUDIENCE NOTE:** "Now here's the magic moment. We've got all this content from Claude. Let's turn it into actual designed slides in about 30 seconds."

**Open Gamma.app → Click "New" → "Paste in text" → Paste the full Claude output → Click Generate**

> **AUDIENCE NOTE:** "Gamma takes structured text and turns it into designed slides automatically. It picks colours, layouts, adds stock imagery — all from the content we generated. Two AI tools, working in sequence: Claude for brains, Gamma for beauty. In under 7 minutes, we've gone from a one-paragraph idea to a designed pitch deck."

### Expected Output
- Full 12-slide pitch deck content with headlines, bullets, and visual suggestions
- Emotionally rewritten problem slide with storytelling
- Detailed 3-year financial projections table
- 2x2 competitive landscape matrix
- Designed slides via Gamma.app (if time allows during live demo)

---

## Transition to Wrap-Up

> **AUDIENCE NOTE:** "So let's step back. In about 35 minutes, we've done: financial analysis, contract drafting, brand identity, market research, and a pitch deck. Five things that would traditionally take a team of people days or weeks. The AI didn't do any of them perfectly — but it got us 70-80% of the way there in minutes. That last 20% is where your expertise, your judgement, your human skills come in. AI is the accelerator. You're still the driver."

---

---

# APPENDICES — Sample Data & Input Files

_Ali: prepare these BEFORE the talk. Copy-paste into the relevant format._

---

## Appendix A: Financial Data (Use Case 1)

**File name:** `greenpulse-financials-2025.xlsx`

Copy the following data into an Excel spreadsheet. **Deliberately keep it messy** — this is the point of the demo. The inconsistencies are intentional:

| Month | Revenue | COGS | Staff Costs | Marketing | Rent & Utilities | Software & Tools | Other Opex | Notes |
|-------|---------|------|-------------|-----------|------------------|------------------|------------|-------|
| Jan-25 | £42,000 | £18,500 | £28,000 | £5,200 | £3,400 | £1,800 | £2,100 | First month of new pricing |
| Feb-25 | £38,500 | £17,200 | £28,000 | £4,800 | £3,400 | £1,800 | £1,950 | Lost 2 clients |
| Mar 2025 | 44000 | 19800 | 28000 | 6500 | 3400 | 1800 | 2200 | New enterprise client signed |
| April 25 | £51,200 | €24,500 | £28,000 | £7,800 | €3,950 | £1,800 | £2,400 | COGS and rent paid to EU supplier in EUR |
| May-25 | £48,900 | £22,100 | £32,000 | £6,200 | £3,400 | £2,100 | £1,800 | Hired new developer |
| Jun-25 | £55,300 | £24,800 | £32,000 | £8,500 | £3,400 | £2,100 | | Missing other opex data |
| Jul-25 | £52,100 | £23,400 | £32,000 | £5,900 | £3,400 | £2,100 | £2,000 | Summer slowdown |
| Aug-25 | | £21,000 | £32,000 | £4,200 | £3,400 | £2,100 | £1,900 | Revenue figure missing — estimate ~£47k |
| Sep-25 | £61,800 | £27,200 | £32,000 | £9,200 | £3,400 | £2,100 | £2,300 | Q4 pipeline building |
| Oct-25 | £68,400 | £30,100 | £36,000 | £11,500 | £3,400 | £2,500 | £2,600 | Hired sales rep, big month |
| Nov 25 | 72500 | 31900 | 36000 | 10200 | 3400 | 2500 | 2800 | Record month |
| Dec-25 | £64,200 | €33,800 | £36,000 | £6,800 | €3,950 | £2,500 | £3,100 | Some EU invoices in EUR again |

**Intentional messiness to include:**
- Mixed date formats: "Jan-25", "Mar 2025", "April 25", "Nov 25"
- Some rows have £ symbols, others are plain numbers
- EUR values in April and December (COGS and Rent)
- Missing revenue in August (with a note)
- Missing Other Opex in June
- Inconsistent "Notes" column

**When pasting into Excel:** Put each column in a separate column. Don't fix anything — leave it messy. That's the demo.

---

## Appendix B: Engagement Brief (Use Case 2)

**File name:** `engagement-brief.txt`

This is embedded in the prompt itself, but if Ali wants a separate file to show the audience:

```
ENGAGEMENT BRIEF — CONFIDENTIAL

Client: NovaTech Solutions Ltd
Provider: Meridian Consulting Ltd
Engagement: Strategic Digital Transformation Advisory

Background:
NovaTech is a mid-sized manufacturing company (£45M turnover, 280 employees) based in
Manchester. They're running critical business systems on legacy on-premise infrastructure
that's approaching end-of-life. The board has approved a digital transformation programme
with a £2M budget and wants external advisory support.

Scope of Work:
1. Current-state assessment of all IT systems and infrastructure
2. Development of a 3-year technology roadmap
3. Vendor selection support for cloud migration (AWS/Azure/GCP evaluation)
4. Implementation oversight and programme governance

Duration: 6 months (1 April 2026 — 30 September 2026)
Fee: £75,000 + VAT, paid monthly
Key Personnel: Lead Consultant + Senior Analyst (named, no substitution without approval)
Working Arrangement: Remote-first, with monthly on-site visits to Manchester
Special Conditions: Client must provide system access within 5 business days of signing

Contact: James Wright, CTO, NovaTech Solutions — james.wright@novatech.example.com
```

---

## Appendix C: Startup Description (Use Case 3)

**The description is embedded in the prompt, but here's the standalone version Ali can reference:**

```
AETHER — Brand Brief

Company: Aether
Industry: CleanTech / Smart Cities / Environmental Monitoring
Product: AI-powered air quality monitoring platform using IoT sensors

What we do: Provide real-time, hyperlocal air quality data and forecasting for cities,
property developers, and health services.

Target audience:
- City councils and local authorities
- Property developers (sustainability compliance)
- Health-conscious urban professionals

Brand personality:
- Clean, scientific, trustworthy
- Slightly futuristic but warm — "calm technology"
- NOT cold, robotic, or clinical

Visual inspiration: Stripe, Linear, Notion, Headspace
Colour direction: Deep teal/navy primary, fresh green or sky blue accent
Logo style: Modern, geometric, minimal — should work at favicon size
Tagline: "Breathe smarter."
```

---

## Appendix D: Business Idea Paragraph (Use Case 5)

**This is the same as the Aether description used in the pitch deck prompt. Here it is standalone for reference:**

```
Aether is a B2B SaaS platform that uses networks of low-cost IoT sensors combined with
AI-powered analytics to provide real-time, hyperlocal air quality monitoring and forecasting
for smart cities. City councils use Aether to identify pollution hotspots, optimise traffic
flow to reduce emissions, and provide citizens with real-time air quality data via a public
dashboard and mobile app. The platform also serves property developers (air quality scores
for new developments) and health services (asthma/respiratory alert systems). Founded in
London in 2025, Aether has 3 pilot cities, £1.2M in pre-seed funding, and a founding team
of 4 (ex-Dyson engineering lead, ex-DeepMind ML researcher, ex-Capita smart cities PM,
and a commercial lead from Octopus Energy).
```

---

## Quick Reference: Time Budget

| Use Case | Topic | Tool | Time |
|----------|-------|------|------|
| 1 | Financial Analysis | ChatGPT + Excel | 7 min |
| 2 | Contract Drafting | Claude | 7 min |
| 3 | Brand Identity | ChatGPT + DALL-E | 6 min |
| 4 | Market Research | Perplexity Pro | 8 min |
| 5 | Pitch Deck | Claude + Gamma | 7 min |
| — | **Total** | — | **~35 min** |

---

## ⚠️ Contingency Plan

Things that might go wrong live:

| Problem | Fix |
|---------|-----|
| ChatGPT is slow/down | Switch to Claude (have it open in another tab) |
| DALL-E fails to generate | Have 2-3 pre-generated images saved as backup |
| Perplexity is slow | Use ChatGPT with browsing, or have key stats noted |
| Gamma.app doesn't load | Show the Claude markdown output and say "imagine this as slides" |
| Any tool is completely down | Skip to the next use case, come back if time allows |
| Internet goes down | Have all prompts saved locally. Show screenshots of expected outputs. |

**Golden rule:** If something breaks, don't panic. Say "This is live tech — it happens. Let me show you what the output looks like..." and move on. The audience will respect the honesty.

---

_Prepared by Ark Medicea, COO — Enriched Solutions_
_17 March 2026_
