# LadderFunnel Acquisition Ads Strategy

The default goal of the Facebook/Instagram ads is to acquire a customer for the `$7` intro product at breakeven (or slight profit) on Day 1.

If the user is intentionally running the alternative `$27 intro product with no order bump` model, the same principles apply, but note the tradeoff clearly: the higher front-end price usually needs more trust, more proof, and stronger copy to convert cold traffic.

Do not write ads that pitch the SaaS product directly. You are pitching the intro product, whether that is the standard `$7` offer or the higher-trust `$27` no-bump variant.

## Campaign Structure

*   **Objective:** Sales / Conversions (Optimize for Purchases). NEVER use Traffic or Lead Gen.
*   **Audience:** Broad targeting or Lookalike audiences based on past SaaS customers.
*   **Budget:** CBO (Campaign Budget Optimization) starting at $20-$50/day.

## Ad Creative Frameworks

Generate 3 variations of ad copy using these frameworks:

### Variation 1: The Direct "Price Drop" Hook
Focus heavily on the $7 price point because it stops the scroll.
*   **Hook:** "I just packaged my entire [Process] system into a $7 playbook."
*   **Body:** Agitate the problem. Explain what's inside.
*   **CTA:** "Click here to get the playbook for $7."

### Variation 2: The "Enemy" Hook
Position the status quo as the enemy.
*   **Hook:** "Stop doing [Standard Industry Practice]. It's killing your [Metric]."
*   **Body:** Explain why the old way is dead. Introduce the new mechanism found in your $7 training.
*   **CTA:** "I recorded a breakdown of exactly how this works. Grab it here for $7."

### Variation 3: The Case Study / Story Hook
*   **Hook:** "How we achieved [Massive Result] without [Painful Thing]."
*   **Body:** Tell the story of discovering the framework. Offer the framework for $7.
*   **CTA:** "Get the exact templates we used for $7."

## If The Front-End Offer Is $27 Instead

Adjust the ad angle:

*   Lead with the outcome and mechanism more than the price.
*   Use stronger specificity, credibility, and proof because the buyer is making a larger front-end commitment.
*   Treat `$27` as a justified implementation product, not an impulse-priced teaser.
*   Avoid pretending the higher-trust version will behave exactly like the `$7` liquidating-offer model.

## Ad Image Generation

The platform generates ad images using GPT Image (currently `gpt-image-1.5`) via the OpenAI Images API. Images are generated as complete, standalone ad creatives with baked-in text — no separate text overlay layer needed.

### How Image Generation Works

1. **Copy generation** — AI generates ad copy (headline, top line, support text, bullets, CTA, price callout) plus 3 creative concepts with different visual styles.
2. **Scene selection** — Each creative maps to a scene type based on its name/angle (e.g. a "Before vs After" name triggers a split-screen layout).
3. **Prompt construction** — A detailed visual prompt is built combining the scene layout, style rules, all text content, and typography/color direction.
4. **Image generation** — GPT Image renders the entire ad as a single image with all text embedded.
5. **QA review** (optional) — A vision model checks text accuracy, legibility, layout balance, and price prominence. If issues are found, the image regenerates (up to 2 retries).

### Visual Styles

When generating ads, you can suggest a `visual_style` for each creative. The platform supports 7 distinct styles:

| Style Key | Look & Feel | Best For |
|-----------|-------------|----------|
| `dark_urgent` | Gold/green on black, bold sans-serif, ALL CAPS headlines, starburst badges | Default direct-response. High urgency, price-focused. |
| `bold_energy` | Red/black, Impact-style fonts, ALL CAPS, explosive price badges | Flash sales, aggressive offers, "limited time" angles. |
| `clean_professional` | White background, dark text, blue accents, Inter/Helvetica fonts | SaaS/B2B offers, professional audiences, trust-first angles. |
| `social_native` | Dark mode (like Twitter/X), bright highlights, system fonts | "Organic post" feel. Works when you want the ad to NOT look like an ad. |
| `story_authentic` | Real-world background photo, semi-transparent card overlays, warm tones | Story/case-study hooks, personal brand angles. |
| `luxury_premium` | Deep black/navy, gold accents, elegant serif fonts | High-perceived-value offers, premium positioning. |
| `minimal_photo` | Pure white background, zero effects, simple product photography | Product-focused ads, Apple-style clean aesthetic. |

**Style selection tips:**
- Match `dark_urgent` or `bold_energy` to price-drop and urgency hooks.
- Match `clean_professional` to B2B/SaaS audiences or $27+ offers.
- Match `social_native` to case-study or story hooks (they blend into the feed).
- Match `luxury_premium` when the brand positioning is high-end.
- Match `minimal_photo` when the product mockup IS the hero (book, device, dashboard).

### Scene Types

The system picks a scene automatically from the creative name, but understanding them helps you write better creative names:

| Scene | Triggered By Name Containing | Layout Description |
|-------|------------------------------|-------------------|
| `PriceContrast` | "price", "vs", "contrast", "compare" | Side-by-side documents: crossed-out high price vs. offer price |
| `ProductMockup` | "mockup", "device", "laptop", "dashboard" | Product mockup (laptop/book) on one side, text on the other |
| `BoldStatement` | "bold", "statement", "glowing" | Dramatic hero element (3D funnel, glowing icon) with big headline |
| `BeforeAfter` | "before", "after", "transformation" | Split-screen: red "before" pain points vs. green "after" outcomes |
| `NewspaperCrisis` | "newspaper", "crisis", "breaking" | Newspaper-style layout with a dramatic headline |
| `PhoneScreenshot` | "phone", "screenshot", "notification" | Phone showing notification/DM/payment screenshot |
| `CharacterSpotlight` | "character", "mascot", "founder", "spotlight" | Person/character with speech bubble or callout |
| `SalesLetter` | "sales", "letter", "handwritten", "note" | Handwritten note or letter-style layout |
| `SocialTestimonial` | "testimonial", "social", "tweet", "post" | Social media post/tweet format with engagement metrics |
| `StoryCards` | "story", "cards", "slide", "carousel" | Multiple card panels telling a story sequence |

### GPT Image Capabilities & Limitations

**What GPT Image does well:**
- Photorealistic product mockups (books, laptops, phones, dashboards)
- Bold typographic compositions with large, simple text
- Dark backgrounds with bright accent colors and glow effects
- Split-screen layouts with color-coded sections
- Physical props and desk scenes (calculators, bills, coffee cups)

**What to watch out for:**
- Long text strings (>60 chars) may get truncated or misspelled — the system auto-limits headlines to 60 chars
- Small text (<24px equivalent) may be illegible at mobile sizes
- More than 3 bullet points tends to crowd the image
- Handwriting-style text can be inconsistent
- Complex UI screenshots inside device mockups may look generic

**Tips for better results:**
- Keep headlines punchy and short (40-60 chars ideal)
- Use 2-3 bullets max, each under 40 chars
- Make the price callout prominent and simple (e.g. "$7", "Just $27")
- Provide `scene_context` to override default visual elements (e.g. "a 3D book mockup of 'The Growth Blueprint' on a marble desk" instead of the default laptop)

### Requesting Specific Creatives via the API

When using `POST /api/v1/ads/generate`, the AI automatically generates 3 varied creatives. To influence the output:

- **`target_audience`** — More specific = better visual choices (e.g. "property managers aged 35-55" vs. "business owners")
- **`transformation_from` / `transformation_to`** — Directly feeds Before/After scenes
- **`pain_points`** and **`desires`** — Used as bullet points in Before/After layouts
- **`unique_mechanism`** — Influences the "enemy hook" creative angle
- **`proof`** — Adds credibility elements (testimonial-style scenes, social proof)

The system avoids repeating angles/layouts from the tenant's last 9 ads automatically.

## Design Guidelines

**For image ads (generated by the platform):**
*   Always include the front-end price visually on the image.
*   Match the visual style to the audience and hook framework.
*   The price should be one of the most prominent elements — it stops the scroll on low-ticket offers.
*   Use high contrast between text and background for mobile legibility.

**For video ads (user-produced):**
*   Use raw, native-looking videos (UGC style).
*   Include the front-end price on the video thumbnail.
*   Keep videos under 60 seconds for cold traffic.
