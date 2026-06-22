# LadderFunnel Copywriting Formulas

Use these strict formulas to draft copy for the 4-page LadderFunnel.

## 1. The Intro Page ($7 Product)

This page must sell the $7 educational product using the **Transformation Framework**. The page uses a section-based architecture — populate each section via `theme_config` (see `build/api.md`).

### Section-by-Section Copy Direction

**Hero** — Above the fold, centered layout:
*   **Pre-Headline (Eyebrow / `badge_text`):** Call out the specific audience (e.g., "FOR B2B SAAS FOUNDERS")
*   **Headline (`intro_headline`):** Focus on a massive, desirable outcome, minus the biggest pain point. (e.g., "Install A Self-Funding Acquisition Pipeline In 24 Hours... Without Wasting Money on Spray-and-Pray Ads.")
*   **Subheadline (`intro_subheadline`):** One sentence that bridges pain → transformation.
*   **Price + CTA:** Prominent price display and action button.

**Pain Agitation** (`problems` array) — Use the "if this, then what?" framework:
*   Each problem has a `title` (bold emotional hook) and `description` (descriptive paragraph from the prospect's perspective).
*   Write from the prospect's shoes. Not "You need leads" but "You wake up on Monday, check your calendar, and it's empty."
*   Push the knock-on effects: if no leads → empty calendar → worry while working → distracted → quality drops → clients leave.
*   `pain_section_headline`: e.g., "Sound Familiar?" or "Here's What's Really Going On"

**Transformation** (`transformation_blocks` array) — Paint the dream scenario:
*   Each block describes a specific aspect of life AFTER implementing.
*   Focus on feelings and outcomes, not mechanisms: "Wake up to a full pipeline" not "Our system generates leads."
*   `transformation_section_headline`: e.g., "Imagine This Instead"

**What You Get** (`features` array or `intro_bullets`) — Deliverables as transformation tools:
*   Frame each item as "You get X so you can Y" — not feature lists.
*   `what_you_get_section_headline`: e.g., "Here's Everything You Get"

**Social Proof** (`testimonials` array) — Real quotes, not hype:
*   Each testimonial has `quote`, `name`, `title`, `initials`.
*   Focus on transformation stories, not generic praise.

**FAQ** (`faqs` array) — Objection handling:
*   Each FAQ has `question` and `answer`.
*   Address the real objections: "Is this right for me?", "How quickly will I see results?", "What if it doesn't work for my industry?"

**Pricing CTA** — Final call to action:
*   `pricing_eyebrow`, `pricing_title`, `pricing_description`
*   Recap the transformation promise. Price prominent. Large CTA button.
*   **The Price Justification:** Explain *why* it's only $7 (e.g., "We price this at $7 because we know once you see how powerful this strategy is, you'll eventually want to use our software to automate it.").

### Core Copywriting Principles

*   Never mention features, mechanisms, platforms, or software. Only mention the transformation.
*   Use the "if this, then what?" framework to chain emotional pain points.
*   Focus on how the problem impacts the prospect's daily life, not their business metrics.
*   Every section sells the same transformation from a different angle — this is deliberate.
*   The prospect should feel understood, not sold to.

## 2. The Checkout Page & Order Bump ($27)

The checkout page needs a summary of what they get. The crucial piece is the Order Bump copy.

*   **Order Bump Headline:** A red flashing box that says: "ONE TIME OFFER - ONLY $27"
*   **Order Bump Copy:** "Want to implement this 10x faster? Add the [Asset Name] to your order. Normally $97, but yours today for just $27. Check the box above to add this to your order."

## 3. The Upsell Page (Core SaaS)

This page bridges the gap between the education they just bought and the software you sell.

*   **Headline:** "Wait! Your order is not complete..."
*   **Subheadline:** "You have the strategy. Now get the software that automates it."
*   **The Video Script / Letter:** 
    1. Validate their purchase: "Congratulations on grabbing the [Intro Product]."
    2. Introduce the hard reality: "But let me be honest. Implementing this manually takes hours."
    3. The Pivot: "That's why we built [SaaS Name]."
    4. The Offer: "As a new customer, you can get access to [SaaS Name] today for a special price."
*   **Buttons:** Two buttons only.
    *   YES: "Yes, Upgrade My Order (+$[Price]/mo)"
    *   NO: "No thanks, I will implement this manually for now."

## 4. The Thank You Page

*   **Headline:** "Your Order is Complete!"
*   **Instructions:** Tell them exactly how to log in to access their $7 digital product.
*   **Secondary CTA (Optional):** If they declined the SaaS upsell, put one last subtle link to book a demo or start a free trial of the software.