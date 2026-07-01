# The House of Mouth — Shopify Theme

Storefront theme for **The House of Mouth**, a Gold Coast–based B2B / wholesale
oral-care supplier. Built on Shopify's Horizon-family **Savor** theme with a set
of bespoke, self-contained "THOM" sections layered on top for the trade
experience.

This repository holds the theme as plain, version-controlled source so it can be
reviewed, diffed, and deployed through the
[Shopify GitHub integration](https://shopify.dev/docs/storefronts/themes/tools/github)
— rather than shipped around as a `.zip` export.

## Structure

Standard [Shopify theme layout](https://shopify.dev/docs/storefronts/themes/architecture):

| Folder       | Contents                                                        |
| ------------ | --------------------------------------------------------------- |
| `assets/`    | CSS, JS, SVG icons and other static files                       |
| `blocks/`    | Reusable theme blocks                                           |
| `config/`    | `settings_schema.json` and saved `settings_data.json`           |
| `layout/`    | `theme.liquid` (and `password.liquid`) page shells              |
| `locales/`   | Storefront + schema translation strings                         |
| `sections/`  | Section templates, including the custom THOM sections           |
| `snippets/`  | Reusable Liquid partials                                        |
| `templates/` | Page templates (JSON + Liquid), including `page.b2b.json`        |

## Custom sections

These are bespoke to The House of Mouth and are the primary surface for
storefront work here:

- **`sections/thom-b2b-home.liquid`** — the B2B homepage: trade utility bar,
  hero, value props, shop-by-category, a popular-products rail, a feature band
  and an "open a wholesale account" CTA. Reads live collections/products; all
  content is editable from the theme editor.
- **`sections/thom-b2b-mini-nav.liquid`** — a compact circular-icon category
  nav strip with configurable icons/images, colours and layout.

The homepage (`templates/index.json`) and `templates/page.b2b.json` render the
B2B home section.

## Local development

Use the [Shopify CLI](https://shopify.dev/docs/api/shopify-cli):

```bash
# Preview against a development store with hot reload
shopify theme dev --store your-store.myshopify.com

# Lint the theme
shopify theme check

# Push to an unpublished theme for review
shopify theme push --unpublished
```

## Conventions

- Brand palette: deep navy `#37456E`, lime accent `#9CCB3B`, soft surfaces
  `#F5F7FB` / `#E3E7F0`, on the `Work Sans` typeface.
- Custom sections are prefixed `thom-` and namespaced with a `.thb` / `.thom-`
  CSS scope so they stay isolated from the base theme.
- Images use Shopify's `image_url` + `image_tag` filters so every image ships a
  responsive `srcset`, intrinsic dimensions and lazy/eager loading.
