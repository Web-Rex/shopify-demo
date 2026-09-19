# ModernShop — Shopify Theme

A Shopify theme built from scratch (Online Store 2.0 structure), using Liquid templates, JSON-less sections with schema-driven settings, and Shopify's native cart/checkout/contact-form systems. No build tooling is required — all CSS and JS are plain files or CDN-loaded.

## Folder structure

```
layout/       Wraps every page (theme.liquid) — nav, footer, cart badge
templates/    One file per page type (index, product, collection, cart, page, gift_card)
sections/     Reusable, schema-editable blocks rendered inside templates
assets/       CSS files (flat structure — Shopify doesn't support subfolders here)
config/       Global theme settings schema (config/settings_schema.json)
```

## Templates

| File | Route | Purpose |
|---|---|---|
| `templates/index.liquid` | `/` | Homepage — renders `home-hero` and `home-products` sections |
| `templates/collection.liquid` | `/collections/<handle>` | Product listing for any collection — real filters, sorting, and pagination via `collection.filters` / `collection.sort_options` / `{% paginate %}` |
| `templates/product.liquid` | `/products/<handle>` | Single product page — image gallery, variant add-to-cart form, specs/rating via metafields, related products |
| `templates/page.liquid` | `/pages/<handle>` | Branches on `page.handle` (`about` / `contact`) to render different section sets |
| `templates/cart.liquid` | `/cart` | Real cart contents via the `cart` global — quantity updates, discount code redirect, checkout |
| `templates/gift_card.liquid` | issued gift card links | Required system template Shopify won't let you omit — shows balance/code/QR |

## Sections

Each section below has its own `{% schema %}`, so its text/settings are editable from the Shopify Theme Editor (Online Store → Themes → Customize) without touching code.

- **`home-hero`** — homepage hero heading, subheading, button text/link
- **`home-products`** — "Featured Products" heading, product count slider (2–12), rendered as a Swiper.js carousel (loaded via CDN)
- **`collection-header`** — collection banner; title is always `collection.title` (dynamic per collection), subheading falls back to an editable default when a collection has no description set
- **`product-related`** — "You Might Also Like" heading + related-products grid (excludes the current product)
- **`about-hero`**, **`about-details`**, **`about-stats`**, **`about-teams`** — About page content blocks; `about-teams` uses repeatable blocks (name/role/photo URL) instead of hardcoded team members
- **`contact-header`**, **`contact-content`**, **`contact-form`**, **`contact-map`** — Contact page content blocks; `contact-content` uses repeatable blocks for each info card (address/email/phone/etc.), `contact-form` submits through Shopify's real `{% form 'contact' %}`

## Key functionality

- **Cart & checkout**: `product.liquid`'s add-to-cart button is a real `{% form 'product', product %}` posting to Shopify's cart. `cart.liquid` reads/writes the actual `cart` object (`cart.items`, `cart.total_price`, quantity updates via `updates[]`, checkout via a `name="checkout"` submit button). No custom backend involved — Shopify's hosted checkout takes over from there.
- **Contact form**: uses `{% form 'contact', ... %}`, so submissions email whichever address is set as the store's **Customer email** (Admin → Settings → General).
- **Collection filtering/sorting/pagination**: fully dynamic via `collection.filters`, `collection.sort_options`, and `{% paginate %}` — no hardcoded category lists; filters only ever show options that actually apply to products in that collection.
- **Product specs & rating**: sourced from **metafields** (see Setup below), not hardcoded — falls back to an honest empty state ("No specifications available." / rating block hidden) when unset, rather than showing fake data.
- **Reviews**: intentionally left as "No reviews yet." — no reviews app is installed. Wiring real reviews requires installing one (e.g. Judge.me) and swapping in its snippet/metafields.

## Required setup for full functionality (not just code)

These are admin-side steps this repo can't perform for you:

1. **Collections**: a collection with handle `head-phone` and Shopify's default `all` collection (used by the nav and homepage carousel).
2. **Pages**: create Pages with handles `about` and `contact` in Admin → Online Store → Pages (routes `page.liquid`'s branching logic).
3. **Metafields** (Admin → Settings → Custom data → Products):
   - `custom.rating` (Decimal) and `custom.review_count` (Integer) — powers the product rating display
   - `custom.specs` (List of single-line text) — powers the Specifications tab
4. **Store contact email** (Admin → Settings → General) — where contact form submissions are delivered.

## Development workflow

```
shopify theme dev              # local live-reload preview
shopify theme push             # push to a theme on the store (prompts to pick/create one)
shopify theme push --theme <id>  # push to a specific theme directly
```

`gitignore`d: `.DS_Store`, `node_modules/`, `.env`, `.shopify/`. Everything under `assets/`, `sections/`, `templates/`, `layout/`, `config/` is committed as-is — there's no compiled/build output in this project currently.

## Known limitations

- Tax and shipping are shown as "Calculated at checkout" on the cart page rather than estimated numbers, since Shopify only knows those once a shipping address is entered.
- Discount codes apply via redirect to `/discount/<code>`, Shopify's real mechanism — invalid codes are silently ignored by Shopify (no error message is possible client-side).
- No variant picker UI yet — add-to-cart uses `product.selected_or_first_available_variant`.
