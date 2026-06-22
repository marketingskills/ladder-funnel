# LadderFunnel Pro API Build Guide

Use the LadderFunnel Pro tenant API only when the business is already hosted on the LadderFunnel Pro platform and you need to create, update, or sync assets programmatically.

**For a lighter alternative, see `blocks.md`.** The Funnel Blocks approach uses a portable JSON config, template partials, and your own Stripe keys. No platform dependency, no account management. If you don't need the Pro platform's course builder, AI features, or Meta campaign launch, start with blocks instead.

## Authentication Rules

- Resolve tenant context by the tenant's real host first.
- Send the tenant API key as either `x-api-key: lf_live_...` or `Authorization: Bearer lf_live_...`.
- Never send a self-tenant API key to a customer tenant host, and never assume tenant context from the key alone.

## Base Pattern

```bash
curl https://TENANT_HOST/api/v1/funnels \
  -H "x-api-key: lf_live_..." \
  -H "Content-Type: application/json"
```

## Stripe Connection

- `GET /api/v1/stripe/connection`
- `PUT /api/v1/stripe/connection`

Use this endpoint to save tenant-level Stripe credentials without opening the dashboard. Checkout uses the tenant `stripe_publishable_key` first, then falls back to the host-level `STRIPE_PUBLISHABLE_KEY` only if the tenant key is missing.

`PUT /api/v1/stripe/connection` accepts any subset of:

```json
{
  "stripe_secret_key": "sk_live_...",
  "stripe_publishable_key": "pk_live_...",
  "stripe_webhook_secret": "whsec_..."
}
```

Behavior:

- Omitted fields preserve the current saved value.
- Sending an empty string for a field clears that saved value.
- `GET` does not return raw secrets. It returns configured booleans plus masked previews.
- For checkout to work reliably on a tenant-owned Stripe account, save both `stripe_secret_key` and `stripe_publishable_key`.
- Use the products API to set each product row's `stripe_price_id`. The hosted Offers screen can still provision one-time prices for you, but price IDs themselves are already API-editable on products.

Example response:

```json
{
  "connection": {
    "secret_key_configured": true,
    "publishable_key_configured": true,
    "webhook_secret_configured": true,
    "secret_key_preview": "sk_liv...c9X2",
    "publishable_key_preview": "pk_liv...4Yq8",
    "webhook_secret_preview": "whsec_...8mP1"
  }
}
```

## Funnels

- `GET /api/v1/funnels`
- `POST /api/v1/funnels`
- `GET /api/v1/funnels/:id`
- `PUT /api/v1/funnels/:id`
- `DELETE /api/v1/funnels/:id`

Use `theme_config` as the flexible design payload. It can hold page copy, design tokens, section config, metadata, and any future style fields the renderer should preserve.

### Post-Purchase Delivery

Funnels also control how the paid content is delivered after checkout:

- `intro_vsl_url` — landing-page video URL for the customer intro page; can be an embed URL or a hosted MP4 URL
- `post_purchase_delivery_mode` — `full_course` | `single_video`
- `intro_video_url` — hosted MP4 URL for the simplified single-video flow

Rules:

- Use `intro_vsl_url` when the intro page should show a VSL or manual founder video. The customer intro page can render either an embed URL or a hosted `.mp4`.
- If you want LadderFunnel to host that landing-page MP4 in R2, upload it through `POST /api/v1/funnels/:id/video-upload` with `slot = intro_vsl` instead of pasting a third-party CDN URL manually.
- Hosted landing-page MP4 uploads also generate a still poster frame from about 2 seconds into the video so the intro page can show a better preview before play.
- Default to `full_course` unless the tenant explicitly wants a simplified one-video ladder funnel.
- Only set `intro_video_url` when you have a real hosted video URL. The hosted dashboard can upload this manually; the API can set the final URL directly.
- When `post_purchase_delivery_mode = "single_video"`, the customer upsell and thank-you pages will surface that video instead of relying purely on the classroom/course experience.
- Preserve unknown `theme_config` keys when changing this mode. Do not flatten unrelated design/copy fields just because you are only changing delivery.

### Design Tokens

Control the visual style of the customer intro page:

- `design_style_preset` — `clean_dark` | `editorial_light` | `bold_contrast`
- `design_font_pair` — `modern_sans` | `editorial_serif` | `condensed_display`
- `design_density` — `comfortable` | `compact` | `airy`
- `design_corner_style` — `rounded` | `pill` | `sharp`
- `design_button_style` — `solid` | `outline` | `elevated`

### Section Ordering

The intro page uses a section-based architecture. Control which sections appear and in what order via `intro_section_order`. Available sections:

- `hero` (always first, cannot be removed)
- `pain_agitation` — emotional "if this, then what" copy from prospect's perspective
- `transformation` — paints the dream scenario after implementing
- `what_you_get` — deliverables framed as transformation tools
- `social_proof` — testimonials and order stats
- `video` — landing-page VSL/manual founder video (skipped if no `intro_vsl_url`)
- `pricing_cta` (always present, appended if missing)
- `faq` — objection-handling accordion

Default order: `["hero", "pain_agitation", "transformation", "what_you_get", "social_proof", "video", "pricing_cta", "faq"]`

### Section Content

Each section has customizable content via `theme_config`:

- **Pain**: `problems` array of `{title, description}`, `pain_section_headline`, `pain_section_subheadline`
- **Transformation**: `transformation_blocks` array of `{title, description}`, `transformation_section_headline`, `transformation_section_subheadline`
- **What You Get**: `features` array of `{title, description}` (falls back to `intro_bullets`), `what_you_get_section_headline`, `what_you_get_section_subheadline`
- **Social Proof**: `testimonials` array of `{quote, name, title, initials}`, `social_proof_section_headline`, `social_proof_section_subheadline`
- **FAQ**: `faqs` array of `{question, answer}`
- **Pricing**: `pricing_eyebrow`, `pricing_title`, `pricing_description`

All section headlines have sensible defaults if omitted.

### Checkout Copy Fields

The checkout page copy is API-editable.

Use these keys:

- Top-level funnel field: `order_bump_copy`
- `theme_config.checkout_headline`
- `theme_config.checkout_subheadline`
- `theme_config.checkout_urgency`
- `theme_config.order_bump_title`
- `theme_config.order_bump_price`

When editing checkout copy:

- Preserve unrelated `theme_config` keys.
- Treat `order_bump_copy` as a top-level funnel field, not a `theme_config` field.
- Keep checkout copy tenant-specific. Do not copy self-tenant checkout language into customer tenants by default.

### Copywriting Principles

All intro page copy must follow the **Transformation Framework** — focus on pain, transformation, and bridge. Never describe features or mechanisms. Use the "if this, then what?" framework to draw out emotional pain points. See `copy/pages.md` for formulas.

Example:

```json
{
  "name": "Activation Sprint Funnel",
  "slug": "activation-sprint",
  "status": "active",
  "is_default": true,
  "intro_headline": "Turn trial traffic into paying buyers on day one.",
  "intro_vsl_url": "https://cdn.example.com/funnels/activation-sprint/landing-vsl.mp4",
  "post_purchase_delivery_mode": "single_video",
  "intro_video_url": "https://cdn.example.com/funnels/activation-sprint/intro-video.mp4",
  "upsell_copy": "Add the implementation sprint and ship this faster.",
  "theme_config": {
    "design_style_preset": "clean_dark",
    "design_font_pair": "modern_sans",
    "design_density": "compact",
    "design_corner_style": "rounded",
    "design_button_style": "solid",
    "intro_section_order": ["hero", "pain_agitation", "what_you_get", "social_proof", "transformation", "pricing_cta", "faq"],
    "pain_section_headline": "This Is What's Really Happening",
    "problems": [
      {"title": "You're paying to acquire freeloaders", "description": "Free trial signups cost you money and deliver zero intent. The same person is on their 5th burner email."},
      {"title": "Your calendar is empty on Monday", "description": "You check your pipeline and there's nothing. You want to be busy but the leads aren't there."}
    ],
    "transformation_blocks": [
      {"title": "Wake up to a full pipeline", "description": "Your calendar is booked. Clients come to you. You stop chasing."},
      {"title": "Spend your time on delivery", "description": "The system works while you focus on what you're good at."}
    ],
    "faqs": [
      {"question": "Is this right for me?", "answer": "If you've been struggling with unpredictable revenue, this was built for you."}
    ]
  }
}
```

### Funnel Video Uploads

- `POST /api/v1/funnels/:id/video-upload`

Use this endpoint when the tenant wants LadderFunnel to host a funnel video in R2 (or the app asset store in local/dev). The upload endpoint persists the funnel field for you, so you do not need a second `PUT /api/v1/funnels/:id` call afterward.

Multipart fields:

- `video` — required MP4 file
- `slot` — optional; `intro_vsl` for the landing-page video, `post_purchase` for the simplified single-video delivery path

Behavior:

- `slot = intro_vsl` uploads the MP4, stores it, and saves the resulting URL into `intro_vsl_url`
- `slot = intro_vsl` also stores `theme_config.intro_vsl_poster_url` and returns `poster_url` so the intro page uses a still frame preview instead of a black first frame
- `slot = post_purchase` uploads the MP4, stores it, and saves the resulting URL into `intro_video_url`
- The response includes `funnel_id`, `slot`, `field`, `url`, `object_key`, and `poster_url`

Example:

```bash
curl -X POST https://TENANT_HOST/api/v1/funnels/FUNNEL_ID/video-upload \
  -H "x-api-key: lf_live_..." \
  -F "slot=intro_vsl" \
  -F "video=@founder-vsl.mp4;type=video/mp4"
```

Example response:

```json
{
  "funnel_id": "11111111-1111-1111-1111-111111111111",
  "slot": "intro_vsl",
  "field": "intro_vsl_url",
  "url": "https://cdn.example.com/tenants/.../landing-videos/abcd.mp4",
  "object_key": "tenants/.../landing-videos/abcd.mp4",
  "poster_url": "https://cdn.example.com/tenants/.../landing-videos/abcd-poster.jpg"
}
```

Checkout-focused example:

```json
{
  "order_bump_copy": "Add the implementation checklist pack so your team can launch this without guessing.",
  "theme_config": {
    "checkout_headline": "Complete Your Order",
    "checkout_subheadline": "Lock this in while the plan is fresh.",
    "checkout_urgency": "You are one step away from getting this live.",
    "order_bump_title": "Add The Checklist Pack",
    "order_bump_price": "$27"
  }
}
```

## Courses and Curriculum

- `GET /api/v1/courses`
- `POST /api/v1/courses`
- `GET /api/v1/courses/:id`
- `PUT /api/v1/courses/:id`
- `DELETE /api/v1/courses/:id`
- `POST /api/v1/courses/:course_id/modules`
- `PUT /api/v1/modules/:id`
- `DELETE /api/v1/modules/:id`
- `POST /api/v1/modules/:module_id/lessons`
- `PUT /api/v1/lessons/:id`
- `DELETE /api/v1/lessons/:id`

Use this to build the paid intro product curriculum, onboarding material, or implementation course without manual dashboard work.

## Course Builder Automation

The tenant API now exposes the lesson production workflow on top of normal course/module/lesson CRUD.

### 1. Sync a Course into the Builder Layer

- `POST /api/v1/courses/:course_id/course-builder/sync`

This creates or updates the builder job, outline, and `generated_lesson_content` rows for the current live curriculum so the automation layer matches the actual course tree.

### 2. Inspect Lesson Builder State

- `GET /api/v1/lessons/:id/course-builder`

Returns:

- course/module/lesson identity
- lesson position within the course
- generated content record
- latest audio recording, if any
- latest render status, if any

### 3. Generate Lesson Content

- `POST /api/v1/lessons/:id/course-builder/generate-content`

Optional JSON body:

```json
{
  "key_points": ["Screen spam calls", "Route buyer enquiries", "Escalate urgent calls"],
  "duration_minutes": 8,
  "tone": "conversational"
}
```

This generates:

- `written_content`
- `slide_scripts`

and also updates the lesson's `content_html`.

### 4. Update Generated Lesson Content Manually

- `PUT /api/v1/lessons/:id/course-builder/content`

Example:

```json
{
  "written_content": "<p>Updated lesson content</p>",
  "slide_scripts": [
    {
      "title": "What The Receptionist Handles",
      "bullets": ["Inbound sales calls", "Rental questions", "Routine admin filtering"],
      "speaker_notes": "Explain the three main buckets clearly.",
      "layout_hint": "split"
    }
  ],
  "status": "draft"
}
```

If you replace `slide_scripts`, existing rendered slide HTML is cleared unless you also send `slides_html`.

### 5. Generate Slide HTML

- `POST /api/v1/lessons/:id/course-builder/slides`

This renders branded HTML slides from `slide_scripts` and saves them onto the generated content record.

### 6. Generate AI Voiceover

- `POST /api/v1/lessons/:id/course-builder/voiceover`

Optional JSON body:

```json
{
  "voice": "ash",
  "model": "tts-1-hd"
}
```

This:

- generates a continuous narration script from the lesson slides
- generates TTS audio
- stores/updates the tenant audio recording row

### 7. Start and Check a Render

- `POST /api/v1/lessons/:id/course-builder/render`
- `GET /api/v1/lessons/:id/course-builder/render`

The render route uses the saved slide HTML plus the latest audio recording and queues a Remotion job. The `GET` route returns the latest status, progress, video URL, and any error.

### Recommended Sequence

1. `POST /api/v1/courses/:course_id/course-builder/sync`
2. `POST /api/v1/lessons/:id/course-builder/generate-content`
3. Optional manual `PUT /api/v1/lessons/:id/course-builder/content`
4. `POST /api/v1/lessons/:id/course-builder/slides`
5. `POST /api/v1/lessons/:id/course-builder/voiceover`
6. `POST /api/v1/lessons/:id/course-builder/render`
7. Poll `GET /api/v1/lessons/:id/course-builder/render`

## Products and Course Links

- `GET /api/v1/products`
- `POST /api/v1/products`
- `GET /api/v1/products/:id`
- `PUT /api/v1/products/:id`
- `DELETE /api/v1/products/:id`
- `PUT /api/v1/products/:id/course-links`

Use `product_type` to distinguish `intro`, `bump`, `upsell`, `downsell`, `recurring`, or custom product rows. Use `stripe_price_id` for the Stripe object the product should bill against. Use `course-links` to control which courses unlock after purchase.

Stripe note:

- Save tenant Stripe credentials first via `PUT /api/v1/stripe/connection`.
- Then set each product's `stripe_price_id` through the products API.
- If you need LadderFunnel to auto-create missing one-time Stripe prices inside the tenant's Stripe account, that provisioning flow still lives in the hosted Offers UI.

For simplified funnels, a tenant can still keep normal product rows while setting `post_purchase_delivery_mode = "single_video"` on the funnel. Use `full_course` when the purchase should route into the classroom experience.

## Ad Creatives

### Read Existing Ad Creatives

- `GET /api/v1/ads`
- `GET /api/v1/ads/:id`
- `GET /api/v1/ads/:id/image`
- `PUT /api/v1/ads/:id`

Use these IDs as `ad_ids` when launching a Meta campaign through the API.

Response notes:

- `GET /api/v1/ads` and `GET /api/v1/ads/:id` include `has_image` plus `image_url`
- Newly generated or API-updated ad images are mirrored into LadderFunnel-hosted storage (R2 when configured, local asset storage in dev/fallback)
- When `has_image = true`, you can use the returned `image_url` directly, or fetch the image through `GET /api/v1/ads/:id/image` with the same tenant API key

### Update Existing Ad Creatives

Use `PUT /api/v1/ads/:id` when you need to revise copy on an already-saved ad without logging into the dashboard.

`PUT /api/v1/ads/:id` accepts any subset of:

```json
{
  "name": "Character Spotlight",
  "top_line": "FOR PROPERTY MANAGERS",
  "headline": "Your AI assistant answers after hours",
  "support": "Stop losing leads to voicemail and missed calls",
  "price_callout": "$297/mo",
  "cta": "Book Demo",
  "bullets": "Never miss a call\nQualify every lead\nSync to CRM",
  "layout": "facebook_square",
  "image_url": "https://example.com/new-creative.png",
  "image_data": "iVBORw0KGgoAAAANSUhEUgAA..."
}
```

Rules:

- Fields are partial-update: omit any field you do not want to change
- `image_url` may be a public `http(s)` URL or an existing `/assets/...` path
- Remote `image_url` values are mirrored into LadderFunnel-hosted storage automatically
- `image_data` should be base64 PNG/JPEG payload; if both `image_url` and `image_data` are present, `image_data` wins
- Send `""` for `image_url` or `image_data` to clear the saved image

### Generate Ad Creatives

- `POST /api/v1/ads/generate`
- `GET /api/v1/ads/generate/:job_id`

Ad generation is asynchronous. The POST returns a `job_id`; poll the GET endpoint until `status` is `"complete"`.

`POST /api/v1/ads/generate` accepts:

```json
{
  "offer_name": "The Growth Blueprint",
  "offer_price": "$7",
  "offer_description": "Step-by-step system to build a funnel that pays for your ads in 24 hours",
  "intro_headline": "Build a Funnel That Pays for Your Ads on Day 1",
  "intro_subheadline": "The exact system used by 200+ SaaS founders",
  "intro_bullets": [
    "The 4-page funnel structure that self-liquidates",
    "How to price your intro product for maximum take rate",
    "The email sequence that converts $7 buyers into $97/mo subscribers"
  ],
  "intro_cta": "Get the Blueprint for $7",
  "target_audience": "B2B SaaS founders spending $2k+/mo on ads with <2% trial conversion",
  "transformation_from": "Burning cash on free trials that never convert",
  "transformation_to": "Acquiring customers at breakeven on Day 1",
  "unique_mechanism": "The value-ladder funnel that replaces free trials",
  "high_ticket_offer": "$2,500 Done-For-You Funnel Build",
  "pain_points": [
    "Free trial signups cost money and deliver zero intent",
    "Pipeline is empty every Monday morning",
    "Spending on ads with no measurable ROI"
  ],
  "desires": [
    "Wake up to new paying customers every day",
    "Ads that pay for themselves immediately",
    "A predictable acquisition system"
  ],
  "objections": [
    "I've tried funnels before and they didn't work",
    "My market is too sophisticated for a $7 offer"
  ],
  "proof": [
    "200+ SaaS founders using this system",
    "Average 2.3x ROAS on Day 1"
  ],
  "generate_images": true,
  "image_size": "1080x1080",
  "image_quality": "high"
}
```

Required fields: `offer_name`, `offer_price`, `offer_description`, `intro_headline`, `intro_subheadline`, `intro_cta`. All other fields are optional but improve output quality significantly.

**`generate_images`** (default `true`): Set to `false` for copy-only generation (faster, no image API cost).

**`image_size`** options:
- Platform-style sizes: `"1080x1080"` (square, default), `"1080x1350"` (portrait/feed), `"1200x628"` (landscape/link)
- Direct GPT Image sizes: `"1024x1024"`, `"1024x1536"`, `"1536x1024"`, or `"auto"`

Normalization:
- `"1080x1080"` maps to `"1024x1024"`
- `"1080x1350"` maps to `"1024x1536"`
- `"1200x628"` maps to `"1536x1024"`
- `"1080x1920"` and `"story"` also map to `"1024x1536"`

**`image_quality`**: Always normalized to `"high"` internally.

Response (202 Accepted):

```json
{
  "job_id": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "status": "pending",
  "poll_url": "/api/v1/ads/generate/aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
}
```

Poll `GET /api/v1/ads/generate/:job_id` until `status` is `"complete"` or `"error"`:

```json
{
  "job_id": "...",
  "status": "complete",
  "progress": [
    "Queued ad generation.",
    "Generating ad plan (copy + concepts).",
    "Rendering creative 1/3.",
    "Rendering creative 2/3.",
    "Rendering creative 3/3.",
    "Saved 3 creatives to Ads Library."
  ],
  "result": {
    "angle": "Price contrast — $4,000/mo agency retainer vs $7 one-time",
    "hook": "I packaged my entire growth system into a $7 playbook",
    "audience_interests": ["SaaS", "digital marketing", "growth hacking"],
    "creatives": [
      {
        "name": "Price Contrast — Agency vs Blueprint",
        "top_line": "FROM THE FOUNDER",
        "headline": "$4,000/mo Agency Retainer vs $7 Once",
        "support": "Everything you need to build a self-liquidating funnel",
        "price_callout": "Just $7",
        "bullets": ["4-page funnel template", "Email sequence included", "Stripe checkout setup"],
        "cta": "Get It Now — $7",
        "has_image": true,
        "image_error": null,
        "qa_status": "ok",
        "qa_notes": []
      }
    ],
    "copies": [
      {
        "framework": "Price Drop Hook",
        "primary_text": "I just packaged my entire growth system into a $7 playbook...",
        "headline": "The Growth Blueprint — $7",
        "description": "Build a funnel that pays for your ads on Day 1."
      }
    ],
    "ad_ids": [
      "11111111-1111-1111-1111-111111111111",
      "22222222-2222-2222-2222-222222222222",
      "33333333-3333-3333-3333-333333333333"
    ],
    "saved_ads": [
      {
        "id": "11111111-1111-1111-1111-111111111111",
        "name": "Price Contrast — Agency vs Blueprint",
        "has_image": true,
        "image_url": "/api/v1/ads/11111111-1111-1111-1111-111111111111/image"
      }
    ]
  }
}
```

The `ad_ids` array contains the saved ad IDs — pass these directly to `POST /api/v1/meta/campaigns` to launch. `saved_ads` gives you the same IDs plus the retrievable `image_url` values.

### End-to-End: Generate Ads → Launch Campaign

1. `POST /api/v1/ads/generate` — generate creatives with images
2. Poll `GET /api/v1/ads/generate/:job_id` — wait for completion, collect `ad_ids`
3. Verify Meta connection: `GET /api/v1/meta/connection` — ensure token + ad account + page + pixel are set
4. `POST /api/v1/meta/campaigns` — pass the `ad_ids` from step 2

## Meta Ads and Facebook Campaign Launch

The tenant API exposes the same Meta connection and campaign-launch path the hosted dashboard uses.

### Meta Connection

- `GET /api/v1/meta/connection`
- `PUT /api/v1/meta/connection`
- `GET /api/v1/meta/ad-accounts`
- `GET /api/v1/meta/pages`
- `GET /api/v1/meta/pixels?ad_account_id=act_123...`
- `GET /api/v1/meta/interests?q=property%20management`

`PUT /api/v1/meta/connection` accepts:

```json
{
  "system_user_token": "EAAB...",
  "ad_account_id": "act_1234567890",
  "page_id": "1234567890",
  "pixel_id": "9876543210"
}
```

Notes:

- The token is stored on the tenant's `meta_connections` row with `connection_method = "system_user"`.
- `ad_account_id`, `page_id`, and `pixel_id` can be set in separate calls; omitted fields preserve the current saved value.
- Use the discovery endpoints above to fetch valid account/page/pixel/interest IDs before launching.
- The saved `pixel_id` is also used for the browser Meta Pixel on the hosted intro, checkout, upsell, downsell, and thank-you pages.
- Customer upsell and thank-you pages fire browser-side `Purchase` events automatically when `pixel_id` is configured and the page has purchase context.

### Meta Campaigns

- `GET /api/v1/meta/campaigns`
- `POST /api/v1/meta/campaigns`
- `GET /api/v1/meta/campaigns/:id`

Campaign creation requires:

- a saved Meta connection with a valid token
- `ad_account_id`
- `page_id`
- `pixel_id`
- one or more existing `ad_ids`
- one or more interests, either directly via `interest_ids` or indirectly via `interest_query`

Example:

```json
{
  "name": "AI Receptionist - Property Manager Test",
  "destination_url": "https://go.example.com/intro",
  "ad_ids": [
    "11111111-1111-1111-1111-111111111111"
  ],
  "interest_query": "property management, vacation rentals",
  "countries": "ES,GB",
  "daily_budget": 30,
  "age_min": 28,
  "age_max": 60
}
```

Behavior:

- The API creates the tenant-scoped campaign, ad sets, and Meta ad rows first.
- It then starts the same background publisher the hosted dashboard uses.
- The create response includes `publish_job_id` plus the new campaign payload.
- Poll `GET /api/v1/meta/campaigns/:id` to inspect campaign/ad-set/ad status as publishing progresses.

## Agent Behavior

- Prefer API updates over “please click in the dashboard” instructions when the tenant API is available.
- When a tenant needs Stripe checkout to work on their own account, save `stripe_secret_key`, `stripe_publishable_key`, and `stripe_webhook_secret` through `PUT /api/v1/stripe/connection` before debugging checkout.
- Preserve unknown `theme_config` keys instead of flattening them away.
- When editing a funnel, fetch the current record first and merge carefully instead of overwriting fields blindly.
- If the tenant has multiple funnels, only mark one as `is_default: true` unless the user explicitly wants a staging-only draft.
- For customer tenants, edit the customer funnel design system through `theme_config` or the funnel agent. Do not copy self-tenant copy/layout decisions into customer hosts by default.
- When a tenant wants a landing-page founder video or VSL hosted inside LadderFunnel, prefer `POST /api/v1/funnels/:id/video-upload` with `slot = intro_vsl` so the returned URL is already wired into `intro_vsl_url`.
- When a tenant wants the simplified one-video flow, set `post_purchase_delivery_mode` to `single_video` and confirm `intro_video_url` points at the hosted MP4 that should appear on the upsell and thank-you pages.
- When driving Meta campaign launch through the API, verify the tenant has a saved system-user token plus `ad_account_id`, `page_id`, and `pixel_id` before attempting `POST /api/v1/meta/campaigns`.
- When a tenant says “the Meta pixel isn’t on the page,” check `GET /api/v1/meta/connection` first. If `pixel_id` is blank, the browser pixel will not render on the hosted funnel pages.
- When driving the course-builder API, sync the course first so lesson builder records line up with the current curriculum before generating slides, voiceover, or video.
- When generating ads via the API, always provide `target_audience`, `transformation_from`/`transformation_to`, `pain_points`, and `desires` — these directly feed the visual layouts (Before/After scenes, bullet points) and dramatically improve image quality.
- After ad generation completes, use the returned `ad_ids` to immediately launch a Meta campaign if the user wants to go live. Do not ask the user to copy IDs manually.
- The ad generation endpoint always generates images by default. Set `generate_images: false` only when the user explicitly wants copy-only output (e.g. for review before rendering).
