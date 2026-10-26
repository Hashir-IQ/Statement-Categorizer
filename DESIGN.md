# Ledger Lens design and decision system

## Use case

Ledger Lens is for a freelance designer cleaning up a business credit-card statement before tax time. They need to distinguish ordinary business expenses such as Adobe Creative Cloud, client travel, meals, office supplies, and advertising from transfers, card payments, refunds, and money coming in. A false positive is more costly than an unanswered row, so the app deliberately routes weak matches to Needs review.

The included statement is synthetic. It contains no account numbers, addresses, real people, or real bank documents.

## Product flow

1. Upload a CSV or open the synthetic demo statement.
2. Map the statement headers to date, description, amount, and optional transaction type.
3. Normalize every row, detect missing fields and duplicate fingerprints, then classify it.
4. Review the category totals and the Needs review queue.
5. Correct or approve rows individually or in a batch.
6. Export a CSV that retains the original fields plus the categorizer’s decision trail.

## Brain: deterministic, code-owned rules

The canonical implementation is `backend/categorizer.py`; the browser demo uses the same ordered rule table in `client/src/lib/categorizer.ts` so it works without a hosted service. No GPT, Claude, Gemini, embedding search, or probabilistic model is used for a category decision.

Rules run in this order:

1. **Data integrity:** missing date, description, or parseable amount becomes Needs review with a data-quality note. An exact duplicate fingerprint of date + normalized description + signed amount becomes Needs review instead of silently double-counting.
2. **Transfers and card payments:** transfer, payment, autopay, cash advance, and savings terms become Transfer / non-spend. These do not count toward business spend.
3. **Refunds and credits:** explicit refund, return, reversal, or credit-memo terms become Refund. A refund is money back, not income.
4. **Income:** positive inflows with client-payment, invoice, payout, or deposit terms become Income. A generic positive inflow without a recognized source stays Needs review.
5. **Fees:** bank fee, monthly service fee, overdraft, and foreign-transaction fee terms become Bank fees.
6. **Software:** Adobe, Figma, GitHub, Notion, Dropbox, and Google Workspace terms become Software.
7. **Travel:** Uber, Lyft, airline, hotel, Airbnb, Amtrak, parking, and rideshare terms become Travel.
8. **Meals:** recognizable coffee, restaurant, DoorDash, Uber Eats, Grubhub, and catering terms become Meals. A generic café name such as `JOES CAFE 9921` stays in Needs review rather than becoming a confident guess.
9. **Office supplies:** Staples, Office Depot, printer, paper, and desk-supply terms become Office supplies.
10. **Equipment:** camera, monitor, keyboard, B&H, Best Buy, Apple Store, Dell, and Logitech terms become Equipment.
11. **Advertising:** Meta Ads, Google Ads, campaign, sponsored, and promoted terms become Advertising.
12. **Safe fallback:** anything unmatched, conflicting, or too generic becomes Needs review. The UI never disguises uncertainty as a confident guess.

For an automatic result, the UI shows the exact matching condition, for example `Merchant text contains “ADOBE”` or `Direction is credit and description contains “CLIENT PAYMENT”`. Deterministic confidence is a communication aid, not a statistical probability: high means one strong, specific rule matched; medium means a broader but still explicit rule matched; review means the engine did not earn confidence.

## Visual direction

Ledger Lens uses a **quiet editorial finance studio** aesthetic: warm paper surfaces, ink-black type, oxidized teal for trusted system decisions, and a single clay-orange accent for human review. It is calm rather than flashy, with a left rail for orientation and a wide ledger canvas for the work.

- **Core principles:** make the rule trail visible; use color to separate system certainty from human judgment; keep the dense table scannable; celebrate completion without gamification.
- **Color philosophy:** parchment makes the app feel like a working tax notebook, ink keeps financial data serious, teal signals “the rule is known,” and clay orange marks the work only the user can finish.
- **Layout paradigm:** a narrow, persistent navigation rail opens into a full-width statement workspace; summary cards sit above a split ledger/review composition rather than a centered marketing grid.
- **Signature elements:** the “LL” ledger-mark monogram, a teal vertical rule beside auto-categorized rows, and a clay “needs your eyes” queue.
- **Interaction philosophy:** direct manipulation beats modal ceremony. A row can be corrected in place, selected for batch approval, filtered by status, and exported without leaving the statement.
- **Animation:** use quick 160–220ms fades and small vertical lifts for upload/completion states; never animate numbers continuously or make the ledger feel unstable.
- **Typography system:** system sans for tables and controls paired with a restrained Georgia-like serif for editorial headings; uppercase eyebrow labels carry the tool’s tax-workbench voice.
- **Brand essence:** “A transparent second pair of eyes for a freelancer’s statement.” Personality: precise, candid, reassuring.
- **Brand voice:** headlines are concrete (“Make this statement tax-ready.”); microcopy is candid (“A wrong guess is worse than a question.”).
- **Wordmark & logo:** a compact `LL` monogram built from two offset ledger strokes, paired with the Ledger Lens name.
- **Signature brand color:** oxidized teal `#0f766e`, used for a known rule rather than generic decoration.
