# Funnel Blocks

A LadderFunnel built with Funnel Blocks is a simple JSON config that describes 4 pages as ordered arrays of typed content blocks. The config drives rendering — you can host it on the lightweight Funnel Blocks platform, or render the same blocks inside your own app.

## Philosophy

- **Convenience, not lock-in.** The blocks format is an open spec. Your funnel config is portable JSON. Host it wherever you want.
- **No framework required.** A block is just a `type` + `props`. How you render it is your choice — template partials, server-rendered HTML, static generation, whatever fits your stack.
- **No overhead.** No user accounts, no course builder, no AI onboarding, no Meta campaign management. Just: blocks → pages → Stripe → delivery.

## Funnel Config Structure

```json
{
  "stripe": {
    "secret_key": "sk_live_...",
    "publishable_key": "pk_live_...",
    "webhook_secret": "whsec_..."
  },
  "delivery": {
    "mode": "single_video",
    "video_url": "https://cdn.example.com/intro-video.mp4",
    "poster_url": "https://cdn.example.com/intro-video-poster.jpg"
  },
  "meta_pixel_id": "9876543210",
  "pages": {
    "intro": {
      "blocks": [
        { "type": "hero", "props": { ... } },
        { "type": "pain", "props": { ... } },
        { "type": "pricing_cta", "props": { ... } }
      ]
    },
    "checkout": {
      "blocks": [
        { "type": "checkout", "props": { ... } }
      ]
    },
    "upsell": {
      "blocks": [
        { "type": "upsell", "props": { ... } }
      ]
    },
    "thank_you": {
      "blocks": [
        { "type": "thank_you", "props": { ... } }
      ]
    }
  }
}
```

## Delivery Modes

- `single_video` — Post-purchase page shows one hosted video. Simplest path.
- `download` — Post-purchase page shows a download link (PDF, zip, etc.).
- `custom_url` — Redirect to a URL you control after purchase.

No course management layer. If you need structured modules and lessons, use LadderFunnel Pro instead.

## Block Types

### Hero Block
Shown above the fold on the intro page. Must include a price and CTA.

```json
{
  "type": "hero",
  "props": {
    "badge_text": "FOR B2B SAAS FOUNDERS",
    "headline": "Install A Self-Funding Acquisition Pipeline In 24 Hours",
    "subheadline": "Without wasting money on spray-and-pray ads.",
    "price": "$7",
    "cta_text": "Get The Blueprint Now",
    "media_type": "image",
    "media_url": "https://cdn.example.com/hero-visual.jpg"
  }
}
```

`media_type` can be `image`, `video_embed` (YouTube/Vimeo iframe URL), or omitted.

### Pain Block
Emotional pain agitation. Each item is one pain point.

```json
{
  "type": "pain",
  "props": {
    "headline": "Sound Familiar?",
    "subheadline": "Here's what's really going on.",
    "items": [
      {
        "title": "Your calendar is empty on Monday",
        "description": "You wake up, check your pipeline, and there's nothing. Another week of hoping."
      },
      {
        "title": "You're paying for freeloaders",
        "description": "Free trial signups cost you API fees and deliver zero intent."
      }
    ]
  }
}
```

### Transformation Block
The dream scenario after implementing the intro product.

```json
{
  "type": "transformation",
  "props": {
    "headline": "Imagine This Instead",
    "items": [
      {
        "title": "Wake up to a full pipeline",
        "description": "Your calendar is booked. Clients come to you."
      },
      {
        "title": "Ads that pay for themselves",
        "description": "Every dollar in ad spend is liquidated on Day 1 by the front-end offer."
      }
    ]
  }
}
```

### Features Block
What the buyer gets, framed as transformation tools.

```json
{
  "type": "features",
  "props": {
    "headline": "Here's Everything You Get",
    "items": [
      {
        "title": "The 4-Page Funnel Blueprint",
        "description": "The exact page structure that converts cold traffic into $7 buyers."
      },
      {
        "title": "The Email Ascension Sequence",
        "description": "30 days of pre-written emails that turn $7 buyers into $97/mo subscribers."
      }
    ]
  }
}
```

### Testimonials Block
Social proof with real quotes.

```json
{
  "type": "testimonials",
  "props": {
    "headline": "What Founders Are Saying",
    "items": [
      {
        "quote": "I replaced my free trial and cut my CAC by 60% in the first month.",
        "name": "Sarah Chen",
        "title": "Founder, GrowthKit"
      }
    ]
  }
}
```

### FAQ Block
Objection-handling accordion.

```json
{
  "type": "faq",
  "props": {
    "headline": "Frequently Asked Questions",
    "items": [
      {
        "question": "Is this right for my industry?",
        "answer": "If you have a B2B SaaS with a free trial problem, this was built for you."
      }
    ]
  }
}
```

### Pricing CTA Block
Final call to action before checkout.

```json
{
  "type": "pricing_cta",
  "props": {
    "eyebrow": "GET STARTED TODAY",
    "title": "The Growth Blueprint",
    "description": "We price this at $7 because we know once you see how powerful this strategy is, you'll want the software that automates it.",
    "price": "$7",
    "cta_text": "Get The Blueprint Now"
  }
}
```

### Video Block
Landing-page VSL or founder video. Skips rendering when the URL is empty.

```json
{
  "type": "video",
  "props": {
    "embed_url": "https://www.youtube.com/embed/...",
    "poster_url": "https://cdn.example.com/video-poster.jpg",
    "headline": "Watch How It Works"
  }
}
```

### Checkout Block
The checkout page with Stripe Checkout and optional order bump.

```json
{
  "type": "checkout",
  "props": {
    "headline": "Complete Your Order",
    "subheadline": "You're one step away.",
    "product_name": "The Growth Blueprint",
    "product_price": 700,
    "product_currency": "usd",
    "success_url": "https://yourdomain.com/upsell?session_id={CHECKOUT_SESSION_ID}",
    "cancel_url": "https://yourdomain.com/intro",
    "bump": {
      "title": "ONE TIME OFFER — ADD THE CHECKLIST PACK",
      "price": 2700,
      "description": "Add the implementation checklist pack so you can launch this without guessing. Normally $97, yours today for just $27."
    }
  }
}
```

Price values are in cents (Stripe convention). The bump is optional — omit the `bump` object for a clean checkout with no order bump.

### Upsell Block
The SaaS upsell after purchase, before the thank-you page.

```json
{
  "type": "upsell",
  "props": {
    "headline": "Wait! Your order is not complete...",
    "subheadline": "You have the strategy. Now get the software that automates it.",
    "body": "You just bought the playbook. But implementing manually takes hours. That's why we built [SaaS Name] — it automates the entire process in one click.",
    "offer": "Get your first month for just $47 (normally $97/mo).",
    "yes_cta": "Yes, Upgrade My Order (+$47/mo)",
    "no_cta": "No thanks, I'll implement this manually.",
    "accept_url": "https://yourdomain.com/upsell/accept?session_id={CHECKOUT_SESSION_ID}",
    "decline_url": "https://yourdomain.com/thank-you?session_id={CHECKOUT_SESSION_ID}"
  }
}
```

### Thank You Block
Post-purchase delivery page.

```json
{
  "type": "thank_you",
  "props": {
    "headline": "Your Order is Complete!",
    "body": "Here is your access link. Check your email for a receipt.",
    "delivery_headline": "Access Your Product",
    "secondary_cta": "Book a Demo",
    "secondary_url": "https://calendly.com/..."
  }
}
```

## Section Ordering

The `intro` page blocks render in the order listed. Typical ordering:

```
["hero", "video", "pain", "transformation", "features", "testimonials", "faq", "pricing_cta"]
```

Any block type can appear at any position. Omitted blocks are skipped. Duplicate blocks are allowed (e.g., two testimonial sections).

## Self-Hosting the Blocks

The blocks format is intentionally simple. To render blocks inside your own app:

1. Parse the funnel config JSON
2. For each block in the page's `blocks` array, look up the `type` to find the matching template partial
3. Pass the `props` as template variables
4. Render the partial into the page layout

Each block type maps to a template. The template receives the `props` object directly. No special context, no framework helpers needed. A block renderer can be ~50 lines of code in any language:

```
match block.type:
  "hero"           → render("hero.html",           block.props)
  "pain"           → render("pain.html",           block.props)
  "transformation" → render("transformation.html", block.props)
  "features"       → render("features.html",       block.props)
  "testimonials"   → render("testimonials.html",   block.props)
  "faq"            → render("faq.html",            block.props)
  "pricing_cta"    → render("pricing_cta.html",    block.props)
  "video"          → render("video.html",          block.props)
  default          → skip (unknown type)
```

The checkout, upsell, and thank-you pages are special — they need Stripe Session creation, webhook handling, and session-based access. Self-hosting these requires implementing:

- `POST /create-checkout-session` — Creates a Stripe Checkout Session with the product price and optional bump, returns the session URL for redirect
- `POST /stripe-webhook` — Handles `checkout.session.completed`, records the purchase, determines if the bump was accepted, redirects to upsell or thank-you
- `POST /accept-upsell` — Creates a new Stripe Checkout Session for the recurring upsell price
- Session-based access control for the thank-you page (one-time access via session ID or token)

For the simplest self-hosted path, skip the order bump and upsell — the intro block links directly to a Stripe payment link, and the thank-you page is a static URL with the download/video link.

## Comparison: Blocks vs LadderFunnel Pro

| Concern | Funnel Blocks | LadderFunnel Pro |
|---------|---------------|------------------|
| Page rendering | Template partials | Hosted pages |
| Stripe integration | Your own Stripe keys | Dashboard or API |
| Delivery | Direct link, download, or redirect | Full course builder with AI content generation |
| Email | Your own email service (Resend, SendGrid, etc.) | Built-in automation |
| Ad generation | Copy-only output, render elsewhere | AI image generation + Meta campaign launch |
| User accounts | None | Customer records, classroom access |
| AI features | None (intentionally) | AI onboarding chat, course builder, ad gen |
| Lock-in | None — portable JSON config | Platform-managed, no export |
| Complexity | Minimal | Full SaaS |
