# House of Mouth — Theme Improvement Backlog

Machine-generated audit of every non-vendored block, section and snippet (239 files) for CRO, customization, accessibility, performance, bugs and SEO. Produced by a 48-agent triage sweep. Vendored app files (`bss-*`, `locksmith*`) were excluded.

**Totals:** 239 files · 432 opportunities — 91 cro, 74 bug, 51 performance, 138 accessibility, 66 customization, 12 seo

Legend: 💸 CRO · 🐛 bug · ⚡ perf · ♿ a11y · 🎛️ customization · 🔎 SEO — each tagged `[impact | safety | file-risk]`. `safety=additive` means backward-compatible / default-off.

---

## Blocks

### `blocks/_accordion-row.liquid` — Renders a collapsible details/summary accordion row with an optional icon/image, heading, and nested theme/app blocks, wired to a custom accordion web component.
- 💸 **Expose SEO/JSON-LD-friendly default FAQ preset for wholesale** `[low | additive | medium]`  
  The preset (lines 335-352) defaults heading to 'when_will_order_arrive'. For a B2B/wholesale FAQ (MOQ, case-pack, lead time, freight), add an optional checkbox setting like 'is_faq' (default false) that, when on, lets the merchant flag rows as FAQ content for future FAQPage schema. Default-off keeps current output identical.
- ♿ **Icon caret/plus SVGs need aria-hidden** `[low | additive | medium]`  
  The two decorative <span class="svg-wrapper icon-caret"> and <span class="svg-wrapper icon-plus"> that inline icon-caret.svg / icon-plus.svg (lines 20-25) have no aria-hidden. Add aria-hidden="true" to each span so screen readers do not announce the toggle glyph twice alongside the native <summary> state.

### `blocks/_announcement.liquid` — Renders a single announcement-bar slideshow slide with rich text, optional link, and full typography controls, wired to the slideshow-slide web component.
- 💸 **Announcement bar is prime free-shipping/MOQ banner but link only wraps whole slide** `[medium | additive | medium]`  
  For wholesale, this bar is the natural place for a free-freight threshold or trade-account CTA. Add an optional 'cta_label' text setting that renders a visible inline link at the end of the announcement when set (default '' so nothing changes), giving a clear tappable CTA instead of only an invisible full-slide overlay anchor.
- 🎛️ **No color control for announcement text/background** `[medium | additive | medium]`  
  Typography header (lines 132-310) exposes font/size/weight/spacing/case but no text color. Add an optional select or color setting 'text_color' (default '' = inherit) applied only when non-empty in the style block. Default-empty preserves current rendering while letting the merchant match the navy #37456E / lime #9CCB3B brand palette per announcement.
- ♿ **Full-slide overlay link has no accessible discrete label / hides visible text** `[medium | additive | medium]`  
  When block_settings.link is set (lines 31-40) an <a class="announcement-bar__link"> overlays the slide with only a .visually-hidden copy of the text; the visible <p class="announcement-bar__text"> is not itself the link. Screen-reader users get the label but the anchor has no aria-label fallback if text is empty after strip_html. Add aria-label using plain_text (already computed line 2) on the anchor to guarantee an accessible name.

### `blocks/_blog-post-card.liquid` — Renders a blog post card (image, heading link, info text, truncated description) with configurable text alignment and horizontal/vertical layout.
- ♿ **Card title link has no accessible context beyond article.title** `[low | additive | low]`  
  The <a href="{{ article.url }}" data-testid="blog-post-link"> (lines 12-17) wraps only the heading block. It is fine for the title, but the separate 'read more' link rendered via _blog-post-description (show_read_more: true, line 26) points to the same URL with generic text, creating two same-destination links per card. Consider adding aria-label to the read-more link (in the description block) referencing article.title, or mark it aria-hidden since the title link already covers navigation.

### `blocks/_blog-post-description.liquid` — Renders a truncated article excerpt/content block with full typography/padding controls and an optional 'read more' link.
- 🐛 **truncatewords logic inverted vs intent for image cards** `[low | moderate | low]`  
  Lines 52-55 set truncatewords to 30, then bump to 90 only 'unless should_truncate or article.image'. So a card WITH an image keeps the short 30-word cut and a card WITHOUT an image and without should_truncate gets 90 words. In _blog-post-card (show_read_more:true, no should_truncate) an imageless post shows 90 words, which can overflow the card. Verify the condition matches design intent; if long imageless cards are unwanted, invert so imageless cards also truncate to 30.
- ♿ **Generic 'read more' link lacks article context** `[low | additive | low]`  
  The read-more anchor (line 67) outputs only the translated 'content.read_more' text with no reference to the article, so a screen-reader link list shows many identical 'Read more' entries. Add aria-label combining the read_more string with article.title (article is already in scope) to make each link uniquely identifiable.

### `blocks/_blog-post-featured-image.liquid` — Renders the article's featured (hero) image on the blog post page with ratio, width, height, and border controls.
- 🐛 **aspect_ratio 'square' has no case branch** `[low | additive | low]`  
  The schema offers image_ratio option 'square' but the --ratio {% case block_settings.image_ratio %} block only handles 'landscape', 'portrait', 'adapt' and falls through to {% else %}1. It currently renders 1 (correct value) by luck, but adding an explicit {% when 'square' %}1 / 1 makes intent clear and guards against the else default changing. Additive.
- ♿ **Featured image alt falls back to filename-less empty** `[low | additive | low]`  
  image_tag uses alt: image.alt with no fallback; when an article image has no alt set, the hero image gets an empty alt. Add a fallback like alt: image.alt | default: closest.article.title so the LCP hero always has descriptive text for screen readers. Additive (only affects the empty-alt case).

### `blocks/_blog-post-image.liquid` — Renders a blog post card's linked thumbnail image (used in article listings/blog cards) with height and border settings.
- ♿ **Card image link has no accessible name when alt is empty** `[medium | additive | low]`  
  The <a href="{{ article.url }}"> wraps only the <img>; alt="{{ image.alt | escape }}" is empty when the article image has no alt, leaving the link with no discernible name for screen readers. Add aria-label="{{ article.title | escape }}" to the <a>, or fall back the alt to article.title. Additive.
- ⚡ **Every blog card thumbnail is eager + high priority** `[medium | additive | low]`  
  The <img> hardcodes loading="eager" and fetchpriority="high". In a blog/article listing these are below-the-fold card thumbnails, so eager-loading and high-priority hinting all of them competes with the real LCP element and wastes bandwidth. Add a block setting (e.g. eager_load default false) or pass loading based on forloop.index/context so only the first card is eager; default the rest to lazy. Keeping current output as the default keeps it additive.

### `blocks/_blog-post-info-text.liquid` — Renders blog post metadata (published date and author) with typography preset and alignment controls.
- ♿ **Date/author separator span is read as content by screen readers** `[low | additive | low]`  
  The separator between date and author is output as a plain <span>{{ 'content.blog_details_separator' | t }}</span> in the flex row. Wrapping it with aria-hidden="true" prevents the punctuation from being announced and reduces noise for assistive tech, without changing visual output. Additive.

### `blocks/_card.liquid` — Generic layout container card (theme block) that composes child blocks with background media, overlay, aspect-ratio, color scheme, and a full-card link.
- 💸 **No new-tab affordance / rel hygiene note for link** `[low | additive | medium]`  
  When open_in_new_tab is set, the anchor already gets target=_blank rel=noopener; consider adding a visually-hidden 'opens in new tab' hint tied to the same setting for accessibility/trust. Additive and default-off.
- ♿ **Full-card overlay link has no accessible name** `[medium | additive | medium]`  
  The .card__link <a href="{{ block_settings.link }}"> is an empty anchor covering the whole card; when the card's visible content is media only (no heading/text child), the link has no discernible name. Add an optional link_aria_label schema text setting and emit aria-label when present, so merchants can label media-only linked cards. Additive (only outputs when the new setting is filled).

### `blocks/_carousel-content.liquid` — Content carousel block that renders its child _card blocks inside the shared slideshow component with column, gap, arrows, border and spacing controls.
- 🎛️ **Autoplay/looping not exposed to merchant** `[low | additive | medium]`  
  infinite: false and there is no autoplay control passed to the 'slideshow' render; for a B2B storefront showcasing product/trust cards, an opt-in autoplay + loop toggle would let the merchant animate the carousel. Add default-off checkboxes (e.g. auto_rotate default false, infinite default false) and pass them into the slideshow render so current behavior is unchanged unless enabled. Additive.
- ♿ **Carousel region lacks an accessible label** `[low | additive | medium]`  
  The wrapper .force-full-width div and the rendered slideshow expose no aria-label/aria-roledescription identifying it as a carousel of content cards. Adding an optional accessibility_label schema text setting passed through to the slideshow (or an aria-label on the wrapper) helps screen-reader users understand the region. Additive and default-empty.

### `blocks/_cart-products.liquid` — Cart page block that renders the line-item list via the cart-products snippet, with settings for gap, image ratio, dividers, vendor and padding.
- 💸 **Expose vendor toggle default on for B2B recognition** `[medium | additive | medium]`  
  The 'vendor' checkbox (id: vendor) defaults to false. For a wholesale/dental audience that reorders by brand, showing the vendor per line item aids scanning/reordering; leave default false but this is the natural place to add a new default-off 'show_sku' or 'show_variant_barcode' setting so trade buyers can verify exact SKUs in-cart before checkout.
- 🎛️ **Add opt-in per-unit price display setting** `[medium | additive | medium]`  
  Add a new default-off checkbox (e.g. 'show_unit_price') that, when enabled, surfaces each line's per-unit/case price alongside line total in the cart-products snippet. B2B buyers purchasing case packs benefit from confirming unit economics; default off keeps current rendering unchanged.

### `blocks/_cart-summary.liquid` — Cart page summary block that renders the cart-summary snippet (subtotal, note, discount, checkout) with extend/color-scheme/border settings and sticky positioning.
- 💸 **Add opt-in free-shipping / bulk-order threshold progress bar** `[high | additive | medium]`  
  The block renders {% render 'cart-summary' %} which shows subtotal and checkout but has no free-shipping/minimum-order threshold indicator (no free-ship snippet exists in /snippets). Add a new default-off setting group (e.g. 'show_shipping_progress' + 'shipping_threshold' amount + message text) rendered above the subtotal. For a Gold Coast wholesale supplier with free-freight-over-X or MOQ rules, a live 'Add $X to unlock free freight' bar is a high-impact AOV lever; default off means no change for merchants who don't configure it.
- 💸 **Add opt-in trust-signal / reassurance richtext under checkout** `[medium | additive | medium]`  
  The sticky .cart-summary__inner ends at the checkout button with no trust reassurance. Add a new default-off inline_richtext setting (e.g. 'reassurance_text') rendered beneath the summary for wholesale trust cues (net terms, ABN invoicing, dispatch times, bulk contact). Empty default keeps current output identical.
- ♿ **Decorative section-background div lacks aria-hidden** `[low | additive | medium]`  
  When extend_summary is true the block outputs <div class="section-background ..."></div> purely for visual backing. Add aria-hidden="true" to this empty presentational div so assistive tech skips it; purely additive markup change with no visual effect.

### `blocks/_cart-title.liquid` — Cart page heading block that renders the cart title (or empty-cart message) as an h1 with optional item-count bubble, typography preset, alignment and padding.
- ♿ **Count bubble adds unlabeled number to the h1 accessible name** `[low | additive | low]`  
  When show_count is true, {% render 'cart-bubble' %} injects the item count inside the <h1> alongside block_settings.title, so screen readers announce a bare number appended to the heading (e.g. 'Cart 5') with no unit. Add visually-hidden context (e.g. an aria-label / visually-hidden 'items' string) on the bubble so the h1 reads meaningfully; additive markup only.

### `blocks/_collection-card-image.liquid` — Renders a collection's image inside a collection card via the resource-image snippet, with aspect ratio, overlay (solid/gradient) and border settings.
- 🎛️ **Expose overlay opacity/text-contrast control for on-image labels** `[low | additive | medium]`  
  Overlay is configured via toggle_overlay + overlay_color (default #00000026, ~15% black) + overlay_style. That default is very light, so white collection titles placed on_image (via _collection-card placement) can fail contrast over pale product shots. Keep the default but add an info note or a default-off 'overlay_darken_on_hover' style aid; the alpha color is already merchant-controllable so guidance is the main gap.

### `blocks/_collection-card.liquid` — Collection card block that assembles the card image plus child text/button/title blocks and renders them through the collection-card snippet.
- 💸 **Add opt-in product-count badge for collection cards** `[medium | additive | medium]`  
  The card composes card_image + children (text/button/collection-title) but never surfaces collection.products_count. Add a new default-off checkbox (e.g. 'show_product_count') that renders '{{ collection.products_count }} products' on the card. For a wholesale catalog with large ranges (toothpaste, brushes, whitening), showing range size aids buyer navigation; default off preserves current output.
- ♿ **placement select has no default, risking blank layout value** `[medium | moderate | medium]`  
  The 'placement' select (on_image / below_image) defines no "default", so a freshly added card can render with an empty placement until the merchant picks one, and downstream vertical_alignment is gated on placement == 'on_image'. Add "default": "below_image" so cards render predictably out of the box.

### `blocks/_collection-image.liquid` — Renders a collection's featured image with configurable aspect ratio and width inside collection layout sections.
- 🐛 **custom ratio produces invalid aspect-ratio when height is 0** `[medium | moderate | low]`  
  When image_ratio == 'custom', line 23 sets `ratio = image_width | append: ' / ' | append: image_height`. The `collection_image_height` range has min 0, so a merchant setting height to 0 yields `aspect-ratio: 50 / 0`, which is invalid and collapses/breaks the image. Also `image_width` is only assigned when ratio=='custom', so `--image-width` is empty for non-custom ratios (line 30 emits `--image-width: %`), leaving `.collection-image { width: var(--image-width) }` invalid and falling back to full width. Clamp height min to 1 (or guard for 0) and default `--image-width` to 100% for non-custom ratios.
- ⚡ **featured image is preloaded unconditionally, hurting LCP budget on lower placements** `[low | additive | low]`  
  Line 42 sets `preload: true` on every collection image regardless of whether it is above the fold. If this block is placed lower in a section group, preloading competes with the true LCP image. Add a schema checkbox (default true to preserve current behavior) like `preload_image` and pass `preload: block_settings.preload_image` so merchants can opt out for non-hero placements.
- ⚡ **sizes math hardcodes a 50vw desktop assumption via divided_by: 2** `[low | additive | low]`  
  Lines 33-35 compute `media_width_desktop = 100 | divided_by: 2 | append: 'vw'` (fixed 50vw) even though `collection_image_width` can be as low as 0% in custom mode. When the merchant narrows the image, the browser still downloads a 50vw-sized candidate, wasting bytes. Derive the desktop sizes value from `image_width` when ratio=='custom' (e.g. `image_width | divided_by: 2`) for a tighter, more accurate sizes attribute.

### `blocks/_collection-info.liquid` — Wrapper block that holds collection slideshow header content and renders the slideshow prev/next controls.
- 🎛️ **placement select has no default so unset value can yield an undefined modifier class** `[low | moderate | medium]`  
  The `placement` select (lines 30-43) has no `default`; the wrapper class uses `collection-info--{{ block_settings.placement }}` (line 3). A newly added block with no placement chosen renders `collection-info--` with no layout modifier. Add `"default": "above-carousel"` to the placement setting to guarantee a defined layout without changing intended behavior.
- ♿ **controls container has no accessible label distinguishing carousel navigation** `[low | additive | medium]`  
  The `.collection-info__controls` div (line 11) renders slideshow arrows via the `slideshow-controls` snippet but the wrapper has no `role`/`aria-label`. If the underlying snippet emits bare buttons, screen-reader users get unlabeled 'previous/next' with no group context. Add an opt-in schema text setting (e.g. `controls_aria_label`, default empty) and, when set, apply it as `aria-label` on the controls container so merchants running multiple carousels can disambiguate them.

### `blocks/_collection-link.liquid` — Renders a single collection link (title + hover/spotlight image) inside the collection-links component, with lazy loading and onboarding placeholders.
- 🎛️ **count uses raw all_products_count with no thousands formatting or cap** `[low | additive | high]`  
  Line 21 assigns `closest.collection.all_products_count` directly; large wholesale catalogs render as e.g. `1204` with no grouping. Offer an additive schema option to format the count (or a max like `99+`) so dense trade collections read cleanly in the spotlight/text layouts.
- ♿ **product count sup is announced without a unit, giving ambiguous screen-reader output** `[low | moderate | high]`  
  Lines 22-24 wrap the count in `<sup class="collection-links__count">{{ count }}</sup>` appended to the link title. A screen reader reads the link as e.g. 'Whitening 42' with no context that 42 is a product count. Add a visually-hidden label (e.g. `<span class="visually-hidden">products</span>`) inside/after the sup, or an `aria-label` on the count, so it reads 'Whitening, 42 products' - useful for B2B buyers scanning catalog breadth.

### `blocks/_content.liquid` — Full-featured layout container that renders child blocks via the 'group' snippet with alignment, gap, color scheme, border radius, and padding controls.
- 🎛️ **no border color/width control despite exposing border_radius** `[medium | additive | medium]`  
  The schema exposes `border_radius` (lines 99-106) and color_scheme but no border width/color, so a merchant can round corners yet cannot draw a visible border to frame a content group (useful for boxing B2B trust/spec callouts in navy #37456E or lime #9CCB3B). Add additive settings `border_width` (range, default 0 = no visual change) and `border_color` (color, default transparent) passed into the 'group' render so bordered cards become possible without altering existing groups.

### `blocks/_divider.liquid` — Renders a horizontal divider via the 'divider' snippet with settings for thickness, corner radius, width, and padding.
- ♿ **Expose decorative vs semantic role** `[low | additive | low]`  
  The block delegates entirely to render 'divider' and outputs no role/aria hint. Add an opt-in select setting 'semantic_role' (default 'decorative' -> aria-hidden/role=presentation, alt 'separator' -> role=separator) passed into the divider snippet so screen readers can treat section breaks correctly on long B2B category pages. Default matches current behavior.

### `blocks/_featured-blog-posts-card.liquid` — Renders a single featured blog post card (image, title, author/date, description) with alignment, color scheme, border and padding controls.
- 🎛️ **No control over which meta fields (author/date) show** `[low | additive | medium]`  
  display_author and display_date are always passed to _blog-post-info-text with no toggle. Add opt-in checkboxes 'show_author' / 'show_date' (both default true) in the schema and gate the details capture, giving the merchant control for a trade-blog where author bylines are often irrelevant. Defaults preserve current output.
- ♿ **Whole-card overlay link lacks accessible date/author context and uses h4 unconditionally** `[medium | additive | medium]`  
  The overlay <a class="featured-blog-posts-card__link"> only contains a visually-hidden span with display_title; the article date/author in {{ details }} sit outside the link so assistive tech gets no 'published on' context. Also the card heading is hard-coded <h4> around {{ title }} (line 95) which can break heading order on pages where the section is the primary content. Add an opt-in schema select 'heading_level' (default 'h4' = current) to let the merchant fix document outline without changing defaults.

### `blocks/_featured-blog-posts-image.liquid` — Renders a blog article image inside a featured-posts card via the shared 'resource-image' snippet with aspect-ratio and border controls.
- ⚡ **No loading/priority control for the resource image** `[low | additive | medium]`  
  The block passes image_source/block_settings to render 'resource-image' but exposes no loading strategy. Add an opt-in checkbox 'preload_first_image' (default false) or a 'loading' select passed through to resource-image so the merchant can eager-load above-the-fold cards and lazy-load the rest, reducing LCP on the homepage featured-posts row. Default-off preserves current lazy behavior.

### `blocks/_featured-blog-posts-title.liquid` — Renders the featured blog section title (from closest.blog, section blog setting, or placeholder) with full typography/width/color controls via the 'text' snippet.
- ♿ **Heading level hard-coded to h3 regardless of typography preset** `[low | additive | low]`  
  blog_title is always wrapped in <h3> (line 28) in blog_title_text, while the type_preset setting offers h1-h6 visual styles that only change appearance, not the semantic tag. Add an opt-in 'heading_tag' select (default 'h3' = current) and use it in the capture so merchants can align the semantic level with page structure without altering the default render.

### `blocks/_featured-product-gallery.liquid` — Renders a product card's media gallery with an optional quick-add button, delegating to 'card-gallery' and 'quick-add' snippets.
- 🐛 **quick-add gated on section settings but rendered inside a card that may lack an add form context** `[medium | moderate | high]`  
  The block reads settings.quick_add / settings.mobile_quick_add (section scope) but renders quick-add for product = closest.product without verifying the product is available/purchasable; on sold-out or draft products this can surface a non-functional add button. Guard with an availability check (e.g. unless product.available) or pass availability to the quick-add snippet so the CTA reflects stock state, important for accurate wholesale stock signalling.
- 💸 **Quick-add is only wired for single add; no bulk/case-pack affordance for wholesale buyers** `[medium | moderate | high]`  
  children only renders 'quick-add' (single-unit add) when settings.quick_add/mobile_quick_add is true. For a B2B/wholesale audience, add an opt-in checkbox 'enable_qty_quick_add' (default false) that, when set, passes a quantity flag into the quick-add snippet so trade buyers can add case packs from the card grid without opening the PDP. Default-off keeps current behavior and avoids touching the quick-add web-component contract for existing merchants.

### `blocks/_featured-product-information-carousel.liquid` — Thin wrapper block that renders the product media gallery as a carousel via the product-media-gallery-content snippet, exposing aspect-ratio, zoom, pagination, thumbnail and padding settings.
- 💸 **Default pagination to thumbnails for B2B media scanning** `[medium | moderate | medium]`  
  slideshow_controls_style (line 126) has no default, so desktop pagination renders with no dots/counter/thumbnails selected. Set a default of 'thumbnails' (or 'dots') so trade buyers can scan multiple product angles/spec shots without guessing there are more images. Purely a schema default change.
- 🎛️ **Expose alt/caption visibility toggle is absent** `[low | additive | medium]`  
  The block delegates everything to product-media-gallery-content; no merchant-facing control exists to force 'contain' fit for whitening/spec imagery on light backgrounds. media_fit (line 47) is only visible_if aspect_ratio=='adapt' AND constrain_to_viewport. Add an unconditional opt-in 'media_fit' fallback or a new default-off setting so wholesale product shots (often on white) aren't cropped.

### `blocks/_featured-product-price.liquid` — Featured-product price block that shows a 'From {price}' string for varying-price products with non-swatch options, otherwise delegates to the price snippet with unit price.
- 💸 **Surface unit / per-case price for wholesale clarity** `[high | moderate | medium]`  
  The varying-price branch (lines 37-38) renders only display_price and skips the price snippet, so show_unit_price:true is lost for variable products. For a case-pack B2B catalog, buyers need per-unit pricing on the 'From' state too. Pass unit-price context into the display_price branch (or always render the price snippet and let it format the 'from') so per-unit/per-case pricing shows on variable SKUs.
- 🎛️ **Add opt-in label prefix for trade pricing** `[medium | additive | medium]`  
  display_price uses the fixed 'content.price_from' translation (line 12). Add a default-off schema text setting (e.g. 'price_prefix') so the merchant can prepend 'Trade price from' / 'ex GST' — important for AU B2B where prices are commonly shown ex-GST — without touching defaults.

### `blocks/_featured-product.liquid` — Featured-product composite block assembling title, price, gallery, and swatches (when product options carry swatches) into a product-card render.
- 💸 **Add opt-in quick-add / reorder CTA slot for wholesale** `[high | additive | medium]`  
  product_card_children (lines 12-24) composes only title, price, gallery and swatches — there is no add-to-cart / quick-reorder affordance. For trade buyers who reorder known SKUs, add an optional 'buy-buttons' content_for slot gated behind a new default-off schema toggle so merchants can enable one-click add without changing the current card layout.
- ♿ **featured-product-content-top uses align-items:baseline with variable presets** `[low | moderate | medium]`  
  The .featured-product-content-top flex row (lines 29-34) aligns title and price on 'baseline'; if the price block's type_preset differs from the title size, baselines diverge and the price visually misaligns. Consider align-items:flex-end or a documented shared preset; low risk but improves visual polish. Optional.

### `blocks/_footer-social-icons.liquid` — Footer social-icons container that flex-wraps child _social-link blocks, centered on mobile and left-aligned on desktop.
- ♿ **Wrap social icons in a labelled nav landmark** `[low | additive | low]`  
  The container is a bare div.social-icons__wrapper (lines 1-6) with no landmark or label. Wrap the {% content_for 'blocks' %} in <nav aria-label> (translated 'Social media') or add role/aria-label so screen-reader users can identify and skip the social link cluster. Additive markup, no visual change.

### `blocks/_header-logo.liquid` — Header logo block rendering the standard logo (plus an inverse variant for transparent-header inverse color schemes), with home-page hide option and desktop padding controls.
- 🐛 **logo_width math breaks when settings.logo is blank** `[medium | moderate | high]`  
  logo_width = settings.logo_height | times: settings.logo.aspect_ratio (line 44) assumes settings.logo exists; if no logo is uploaded the {% render 'image' %} falls back to shop.name text but the container still sets --header-logo-image-width via aspect_ratio of a blank image (0/nil), yielding a 0px or malformed width. Guard the aspect_ratio math with an 'if settings.logo != blank' so the text fallback sizes correctly.
- 🎛️ **Expose mobile padding controls to match desktop** `[low | additive | high]`  
  Schema only offers desktop padding-block-start/end under 't:content.padding_desktop' (lines 163-184); the CSS zeroes padding at max-width:749px (lines 108-110). Add optional mobile padding range settings (default 0) so merchants can fine-tune logo spacing on mobile headers without altering current output.

### `blocks/_header-menu.liquid` — Renders the header navigation (desktop mega-menu, mobile drawer, and mobile navigation-bar variants) with an overflow-list web component and configurable typography/color schemes.
- 🎛️ **Add opt-in 'Shop by category' promo/CTA slot in mega-menu** `[medium | additive | high]`  
  For a B2B/wholesale catalog, a merchant-controlled promo (e.g. 'Request a wholesale quote' or 'Download price list' link) inside the mega-menu column area would drive trade conversions. Add a new default-off block/setting (e.g. a url + label pair surfaced via the existing mega-menu-list render on line 123) so it appears only when the merchant fills it in, leaving current menus unchanged.
- ♿ **Submenu toggle state not synced to aria-expanded** `[medium | risky | high]`  
  The top-level <a class="menu-list__link"> for items with children is rendered with a static aria-expanded="false" (line 101) but submenus open on CSS :hover/:focus-within (lines 413-422) with no JS updating aria-expanded. Screen-reader users are told the submenu is always collapsed. Have header-menu.js toggle aria-expanded on activate/deactivate, or drive the CSS visibility off the same attribute so state and markup agree.

### `blocks/_heading.liquid` — Thin wrapper block that renders a rich-text heading via the shared 'text' snippet with extensive typography presets (font, size, line-height, case, wrap, color, padding).
- ♿ **No semantic-level guard when preset is 'rte'/'custom'** `[low | additive | low]`  
  type_preset offers h1-h6 plus 'rte' and 'custom' (lines 17-56); the actual tag rendering is delegated to the 'text' snippet. There is no info/warning helping merchants avoid multiple h1s or skipped heading levels per section, a common a11y/SEO regression on B2B landing pages. Add a 't:info' note on type_preset advising one h1 per page and sequential levels; purely informational, no output change.

### `blocks/_hotspot-product.liquid` — Renders an interactive image hotspot that opens a dialog with a product's image, title, price and quick-add (or sold-out badge).
- 💸 **Dialog omits case-pack / wholesale unit context** `[medium | additive | medium]`  
  The dialog shows only title and price via {% render 'price' %} (line 58). For a wholesale audience, surfacing the unit/case size (e.g. product.metafields case pack) next to the price would reduce friction before quick-add. Add an optional block setting to display a metafield-backed 'per case' line above quick-add; default-off so existing hotspots are unchanged.
- ♿ **Hotspot color/bullseye have no contrast safeguard** `[medium | moderate | medium]`  
  The trigger colors come from section.settings.hotspot_color / bullseye_color injected as --hotspot-bg and --hotspot-bullseye (lines 13-14) with no minimum contrast or focus-visible styling shown. Ensure the hotspot-trigger button has a visible focus outline and consider a subtle ring so low-contrast merchant color choices (e.g. lime on light imagery) remain perceivable for keyboard users.

### `blocks/_image.liquid` — Renders a responsive image (from a resolved collection/product featured image) inside an <image-block> with configurable ratio, height and border-radius, falling back to a placeholder SVG.
- ♿ **Alt falls back to empty when no resource resolves** `[low | additive | medium]`  
  alt is derived from closest.collection.title or closest.product.title (line 14); when neither resolves at render time the img gets an empty alt while still showing a real image (not the placeholder branch). Add a merchant-facing optional alt/text setting, or fall back to the block's own context, so informative images are not silently un-described.
- ⚡ **No fetchpriority/eager hint and oversized default srcset for a resource card image** `[medium | moderate | medium]`  
  The image_tag (lines 32-36) passes loading through but the sizes string is hardcoded to 100vw (line 21) and widths go up to 2560 (line 22) even though this is described as a collection card image; when many cards render, 100vw sizes force the browser to download near-full-width candidates. Expose an optional 'sizes' override or narrow the default for card contexts to cut bytes/CLS. Additive if implemented as a new optional param with the current value as default.

### `blocks/_inline-collection-title.liquid` — Renders the closest collection's title (or a placeholder in the editor) as a styled inline text-block with typography controls and an optional suffix.
- 🔎 **Collection title rendered as generic <span>, not a heading** `[medium | additive | low]`  
  The title is output inside <span class="text-block ..."><span>{{ collection_title }}</span></span> (lines 16-27) with no heading semantics. On a collection page this is the primary page title; wrapping it in a semantic heading (or offering a merchant tag setting like h1/h2) would improve SEO and document outline. Add an optional heading-level setting defaulting to the current span so no existing layout changes.

### `blocks/_inline-text.liquid` — Renders an inline <span> text block with custom typography (font, weight, line-height, letter-spacing, case) and an optional suffix, kept alive in design mode for live editing.
- 🎛️ **Expose font_size + color settings** `[low | additive | low]`  
  The wrapper already applies 'custom-font-size' when block_settings.font_size is set and passes settings to 'typography-style', but the schema only exposes font/weight/line_height/letter_spacing/case. Add opt-in default-blank 'font_size' (range or select) and a 'text_color' color setting so merchants can size/tint inline callouts (e.g. lime #9CCB3B trade-price emphasis) without touching CSS; blank defaults keep current output identical.
- ♿ **Inner <span> adds no semantics** `[low | additive | low]`  
  Line 22 wraps text in a redundant inner <span>{{ block.settings.text }}</span> with no role/label. If this block is used for standalone labels, consider allowing an optional tag/aria via schema; currently purely decorative markup with no a11y hook. Minor.

### `blocks/_layered-slide.liquid` — Renders one panel of a layered slideshow (tabpanel with image or video background, optional overlay, and nested content blocks) with proper tab/inert/aria wiring.
- 🎛️ **Placeholder SVGs are apparel-themed** `[low | moderate | medium]`  
  Lines 67-71 hard-code placeholder_name = 'hero-apparel-' + variant, so empty slides show clothing imagery irrelevant to a Gold Coast oral-care wholesaler. Swap the placeholder base to a neutral product/abstract placeholder (e.g. 'collection-apparel' -> 'lifestyle-2'/'product-N') or expose it as a setting so the editor preview matches the store vertical.
- ♿ **Background image alt is empty** `[medium | additive | medium]`  
  On lines 36-40 image_1 is rendered via image_tag without an alt: parameter, so the alt falls back to the asset default (often empty). Add alt: block.settings.image_1.alt (with a sensible fallback) and a matching alt for the video poster on lines 43-52 so screen-reader users get context for hero slides.
- ⚡ **First slide image loads lazily** `[medium | moderate | medium]`  
  Line 32 sets loading = 'lazy' for every slide including block_index 0, which is the visible above-the-fold panel and a likely LCP element. Gate loading/fetchpriority on block_index: eager + fetchpriority:'high' for the first panel, lazy for the rest, to cut LCP on hero/slideshow sections.

### `blocks/_marquee.liquid` — Renders a scrolling <marquee-component> web component that repeats child text/icon/divider blocks with configurable speed, direction, gap, color scheme and reduced-motion handling.
- 🎛️ **Speed is hard-coded** `[low | additive | medium]`  
  data-speed-factor="25" is hard-coded on line 22 and --marquee-speed drives the animation on line 93. Expose a 'speed' range setting (default 25) so merchants can slow a trust/announcement marquee (e.g. 'Free freight on orders over $X to dental practices') for readability without editing the block.
- ♿ **Scrolling region lacks aria-hidden/label** `[medium | moderate | medium]`  
  The <marquee-component> (lines 10-24) exposes duplicated, endlessly-repeated content to the accessibility tree and provides no aria-label or aria-hidden on the decorative repeated copy. Since the preset category is 'decorative', add aria-hidden to the duplicated marquee__repeated-items copies (or aria-label on the component) so screen readers do not read the scrolling text multiple times.

### `blocks/_media-without-appearance.liquid` — Thin wrapper that renders the shared 'media' snippet, passing unset_image_tag=true only when image_position is 'contain'.
- 🐛 **'contain' option value has trailing space, breaking contain mode** `[medium | moderate | medium]`  
  Schema line 74 defines the image_position 'contain' option value as "contain " (trailing space), but the Liquid check on line 3 compares block.settings.image_position == 'contain' (no space). The strings never match, so selecting Contain never sets unset_image_tag=true and the image still renders as cover. Fix the option value to "contain" (or trim in the comparison).

### `blocks/_product-card-gallery.liquid` — Product-card media block: renders sold-out/sale badges, optional quick-add, the card gallery, and a title shown in the zoomed-out grid view.
- 💸 **No low-stock / bulk-availability badge for B2B buyers** `[high | additive | medium]`  
  Lines 12-25 only emit Sold out or Sale badges. For a wholesale/dental audience, add an opt-in low-stock badge driven by a schema-configurable threshold using product.selected_or_first_available_variant.inventory_quantity (badge like 'Low stock' or 'X in stock'), rendered in the existing product-badges container. Default the setting off so current cards are unchanged. Drives urgency on case-pack reorders.
- 💸 **Sale badge shows no savings amount** `[medium | additive | medium]`  
  The sale branch (lines 21-23) outputs a generic 'content.product_badge_sale' label with no magnitude. Add an opt-in variant that shows the percent or amount saved (compare_at_price minus price) so trade buyers instantly see bulk savings on the collection grid; keep the plain-text label as the default.
- ♿ **Grid-view title uses h3 styled as h4 with no landmark link** `[low | moderate | medium]`  
  Lines 36-42 render the zoom-out product title inside an <h3 class="h4"> that is not a link, so keyboard users in the zoomed-out grid cannot navigate to the product from this title. Wrap it in an <a href="{{ product.url }}"> (with the placeholder branch left as plain text) to add a reachable target and improve heading-to-link semantics.

### `blocks/_product-card-group.liquid` — Nestable layout group block for product cards; captures child blocks and delegates rendering to the shared 'group' snippet with full layout/appearance/border/overlay/link/padding controls.
- ♿ **Announce new-tab behaviour on group link** `[low | additive | medium]`  
  The schema exposes a 'link' url + 'open_in_new_tab' checkbox but this file only forwards settings to 'group'. When the underlying group renders target=_blank there is no visually-hidden 'opens in new window' cue. Add an optional 'link_aria_label' text setting (default blank, unused when empty) so the merchant can label the whole-card link for screen readers without changing current output.

### `blocks/_product-card.liquid` — Root product-card block: resolves closest.product, captures child blocks scoped to that product, and renders via the 'product-card' snippet plus buy-buttons/quick-add/badge style partials.
- 💸 **Expose opt-in per-unit / case-pack price hint on card** `[medium | additive | high]`  
  For a B2B/wholesale catalogue the card renders price via the child 'price' block only; buyers cannot see per-unit or case-pack economics at the grid level. Add a default-off checkbox setting (e.g. 'show_unit_price_hint') that, when enabled, lets the merchant surface product.selected_or_first_available_variant.unit_price_measurement under the price. Leave it off by default so existing cards are unchanged.
- 💸 **Optional bulk quick-reorder / quick-add affordance** `[medium | additive | high]`  
  The block already pulls in quick-add-styles and quick-add-modal-styles. Add a default-off 'enable_quick_add' toggle in the schema so trade buyers can add to cart directly from the collection grid (reorder friction is a major wholesale CRO lever). Because the styles are already rendered, wiring is additive and off by default.

### `blocks/_product-details.liquid` — Product-information details container on PDP: renders a hidden view-product-title link plus child blocks (price, variant-picker, buy-buttons, trust badges, accordion, recommendations) through the 'group' snippet, with sticky/full-height desktop options.
- 💸 **Wholesale-oriented default trust copy instead of apparel defaults** `[medium | additive | high]`  
  The preset ships apparel-centric defaults: accordion rows 'materials'/'care_instructions'/'fit' and a 'styled_with' complementary block, plus 'free_returns'. For a dental/trade supplier these mislead. Add (opt-in, not replacing defaults) preset-independent trust content guidance or a new default-off setting to swap in B2B trust rows (e.g. bulk lead time, ABN/GST invoicing, minimum order) so merchants get relevant signals without breaking the existing preset.
- 💸 **Make free-shipping threshold text dynamic** `[medium | additive | high]`  
  The preset hardcodes a 'free_shipping_over' text block (group_Hrq6NU). Expose a merchant-editable threshold setting or wire it to the cart free-shipping goal so the PDP promise matches the actual cart threshold; mismatched thresholds erode trust for repeat trade buyers. Keep current static text as the default.
- ♿ **Sticky details max-height can trap content on short viewports** `[low | moderate | high]`  
  In {% stylesheet %}, .full-height--desktop sets max-height: calc(100vh - var(--header-group-height,0)) with sticky enabled; on short desktop windows a long variant list/accordion can overflow without scroll. Add overflow-y:auto to .full-height--desktop so keyboard/scroll users can reach clipped buy-buttons. Low risk, desktop-only.

### `blocks/_product-list-button.liquid` — Renders a 'View all' button on collection/product-list sections only when closest.collection.products_count exceeds section.settings.max_products.
- 🐛 **Button label/style rely on ambient block context, not passed explicitly** `[low | additive | medium]`  
  The render call {% render 'button', link: button_url %} passes only 'link'; button.liquid reads block.settings.label/style_class/open_in_new_tab from the ambient block. This works because 'render' inherits the current block here, but it is fragile: if this include is ever reused where 'block' differs, label/target break silently. Passing block: block explicitly documents the contract with no behaviour change.
- 💸 **Show remaining product count in the View-all label** `[medium | additive | medium]`  
  The block only conditionally renders the button using products_count > section.settings.max_products but the label is static ('view_all_button_label'). Add an optional 'show_count' checkbox (default off) that appends closest.collection.products_count (e.g. 'View all 128 products') to signal catalogue depth to trade buyers. Off by default preserves current label.

### `blocks/_product-list-text.liquid` — Renders a text/collection-title block on collection cards via the shared 'text' snippet, with rich typography, background, and padding controls.
- 🎛️ **Add opt-in link target for the collection title** `[low | additive | low]`  
  The block only outputs static richtext (via {% render 'text' %}) with no way to make the collection title clickable to its collection page. Add a new default-off checkbox setting (e.g. 'link_to_collection', default false) that, when enabled, wraps the rendered title in an <a href="{{ closest.collection.url }}">; leaving it off preserves current non-linked output. Improves navigation for B2B buyers scanning collection grids.

### `blocks/_product-media-gallery.liquid` — Product page media gallery block that delegates rendering to the 'product-media-gallery-content' snippet, exposing grid/carousel layout, thumbnails, aspect ratio, and zoom settings.
- 💸 **Default enable zoom is good but expose thumbnail default on desktop for trade product inspection** `[medium | moderate | medium]`  
  slideshow_controls_style (desktop pagination) has no default, so desktop carousels fall back to no explicit pagination. For a wholesale catalog where buyers scrutinise pack/branding detail, consider adding "default": "thumbnails" to slideshow_controls_style so multi-image products expose thumbnail navigation by default. Note: this changes default output, so gate as merchant choice — mark moderate.

### `blocks/_search-input.liquid` — Search page input block: a role=search form posting to routes.search_url with a hidden type=product, icon, clear button, no-results message, and its own web component search-page-input-component.
- 🎛️ **Add optional placeholder/label override settings** `[medium | additive | medium]`  
  Placeholder and label are hardcoded to translation keys 'content.search_input_placeholder' / 'content.search_input_label' (lines 27, 45). Add optional text settings (e.g. 'custom_placeholder') that, when non-blank, override the default; empty keeps the current i18n string. Lets the merchant tailor prompts like 'Search products or SKU' for trade buyers without editing locales.
- ♿ **Give the no-results <p> an aria-live region so screen readers announce it** `[low | additive | medium]`  
  The search-results__no-results block (lines 61-67) renders 'no results, check spelling' text but has no role="status" or aria-live. For keyboard/screen-reader trade buyers, add role="status" (or aria-live="polite") to the wrapping div so the empty-result state is announced. Purely additive attribute.

### `blocks/_slide.liquid` — Slideshow slide block rendering an image/video background plus nested content blocks via slideshow-slide, with responsive image_tag, eager/lazy + fetchpriority logic, overlay, and color-scheme controls.
- ♿ **Image alt is not surfaced or overridable, and video has no descriptive label** `[medium | additive | medium]`  
  The image_tag call (lines 59-63) passes no alt argument, so slides rely on the asset's default alt (often blank), and the video_tag (lines 79-82) has no accessible label. Add an optional 'image_alt' text setting passed as alt: block_settings.image_alt (falling back to image.alt) so merchants can describe hero/promo slides; additive since blank keeps current behavior.
- ⚡ **First-slide LCP: eager loading is capped at section_index <= 3 but fetchpriority high only when section_index == 1** `[medium | moderate | medium]`  
  block_index == 0 gets loading:'eager' when section_index <= 3, but fetchpriority:'high' only when section_index == 1 (lines 47-56). A hero slideshow placed as section 2 or 3 still loads eagerly yet without high fetchpriority, hurting LCP. Consider extending fetchpriority 'high' to block_index == 0 and section_index <= 3 to match the eager window. Changes emitted attribute, so moderate.

### `blocks/_social-link.liquid` — Footer social link block that derives the platform name from the URL domain and renders the matching icon, hiding on storefront unless a valid profile path exists.
- 🐛 **has_profile can be undefined when link is blank, and platform is unset for unknown icons** `[low | additive | low]`  
  has_profile is only assigned inside the 'if block_settings.link != blank' block (lines 4-15); when link is blank on the storefront, has_profile is nil which the guard on line 23 handles, but the {% render 'icon', icon: platform %} on line 48 receives a nil/empty platform for unrecognised domains, producing an empty <svg> (the :has(path) CSS then hides the wrapper). Consider initialising has_profile=false before the link check and skipping render when the icon partial has no match to avoid empty wrappers occupying layout space.
- ♿ **aria-label derived from domain can be inaccurate/generic for platforms on multi-word domains** `[low | additive | low]`  
  platform is derived by splitting the domain (lines 5, 32, 40) so a link like business.facebook.com or a link with a subdomain yields an odd aria-label/label. Add an optional 'label' text setting used for aria-label and the visually-hidden .social-icons__icon-label when present, falling back to the current capitalized platform. Additive and improves screen-reader clarity.

### `blocks/accelerated-checkout.liquid` — Renders the dynamic accelerated/express payment button (payment_button) inside the product form, hidden when the product cannot be added to cart.
- 💸 **Opt-in toggle to hide express checkout for wholesale/B2B** `[medium | additive | medium]`  
  For a trade/wholesale audience, dynamic express-pay buttons (Shop Pay/PayPal one-click via form_obj | payment_button on line 21) often bypass the quantity/case-pack review flow and add checkout noise. Add a new default-on checkbox setting (e.g. show_accelerated_checkout) in the empty schema (lines 43-48) and wrap the render at line 25 with it, so the merchant can suppress express-pay on wholesale templates without editing Liquid. Default-on preserves current behavior.

### `blocks/accordion.liquid` — Collapsible accordion container that renders child _accordion-row blocks with configurable icon, dividers, heading type preset, color scheme, borders and padding.
- 🎛️ **Expose an 'all rows open by default' convenience toggle** `[low | additive | medium]`  
  Open state is controlled per-row via the _accordion-row open_by_default setting (see preset row-1 at lines 257-262). For long spec/FAQ lists a merchant currently must toggle each row. A default-off container-level setting that passes a hint to child rows would let trade buyers see all shipping/spec content expanded for scanning without touching each block. Default-off keeps current collapsed behavior.
- ♿ **Add optional aria-label/landmark to accordion container** `[low | additive | medium]`  
  The wrapper div (lines 2-15) has no accessible name, so an FAQ/spec accordion is announced only as a generic group. Add an optional text setting (e.g. accessible_label) to the schema and emit it as aria-label on the container when present; leave it blank by default so nothing changes for existing merchants. Helps screen-reader users distinguish multiple accordions (shipping, returns, product specs) on a B2B product page.

### `blocks/add-to-cart.liquid` — Renders the standalone Add to Cart submit button via the add-to-cart-button snippet, styled per the style_class setting.
- 💸 **Primary Add to Cart is de-emphasized (button-secondary default)** `[high | moderate | medium]`  
  The style_class setting defaults to 'button-secondary' (schema line 47), and the buy-buttons preset also forces button-secondary (buy-buttons.liquid lines 453-459), so the main purchase CTA renders as a secondary/outline button while the accelerated-checkout button is visually dominant. For a B2B store where trade buyers add cases to cart (not one-click express pay), the Add to Cart should be the primary action. Changing the default option order or the preset to 'button' would strengthen the CTA — flagged as moderate because it changes rendered default styling.

### `blocks/button.liquid` — Generic link/button block that renders the shared button snippet using the block's link, label, style and width settings.
- 💸 **No aria-label option for icon/short-label CTAs** `[low | additive | low]`  
  The button renders only block_settings.label as its accessible name (snippets/button.liquid line 32). For trade CTAs like 'Request a quote' or 'Download price list' that a merchant might shorten to 'Quote', an optional aria_label setting would let them provide a fuller accessible name without changing the visible label. Default-blank so nothing changes for existing buttons.
- ♿ **New-tab links give no warning to assistive tech / users** `[low | additive | low]`  
  open_in_new_tab (schema lines 20-24) makes the snippet emit target="_blank" rel="noopener noreferrer" (snippets/button.liquid lines 26-29) but no visible or screen-reader cue that the link opens a new window. Add an optional default-off setting to append visually-hidden 'opens in new window' text (or aria-label suffix) when open_in_new_tab is on. Additive and only active when the merchant already opted into new-tab behavior.

### `blocks/buy-buttons.liquid` — Core product purchase block: builds the product form, quantity rules, volume/quantity-break pricing table, add-to-cart, accelerated checkout, and local pickup availability.
- 💸 **Surface a low-stock urgency cue for trade buyers** `[high | additive | high]`  
  variant.inventory_quantity, inventory_policy and inventory_managed are already computed (lines 9-13) and drive can_add_to_cart, but a low-stock signal is never shown. Add a default-off 'show low stock' checkbox plus a threshold range setting, and render a 'Only {{ inventory_quantity }} left' line near the add-to-cart block (around line 219) when inventory_managed and quantity is below threshold. High CRO value for reorder-driven wholesale buyers; default-off keeps current output.
- 💸 **Show per-unit savings % in the volume pricing table** `[high | additive | high]`  
  The volume-pricing table (lines 137-217) lists each price break as 'qty+  $price /each' but never quantifies the saving versus the base variant.price. Adding an optional default-off setting to append a computed 'save X%' (price_break.price vs variant.price) per row would make case-pack value explicit for trade buyers comparing tiers. Purely additive markup behind a toggle; does not alter the existing volume-pricing web component contract or refs.
- 🎛️ **Expose the volume-pricing collapse limit (hard-coded limit:2 / offset:2)** `[medium | moderate | high]`  
  The number of always-visible price-break rows is hard-coded via 'limit: 2' (line 155) and 'offset: 2' (line 179), so merchants with many case-pack tiers cannot control how many show before the show-more toggle. Add an integer/range setting (default 2) and substitute it into these filters. Defaulting to 2 preserves current behavior; medium risk because it touches the collapsible logic feeding the volume-pricing component.

### `blocks/collection-card.liquid` — Wrapper block that renders a collection card (image, title, child text/button/group blocks) via the 'collection-card' snippet with placement/alignment/border settings.
- 💸 **Optional hover overlay CTA text** `[medium | additive | medium]`  
  Add a default-empty text setting 'card_cta_text' (e.g. 'Shop wholesale range') passed to the collection-card snippet as an on-hover overlay label to sharpen the click affordance on category tiles; empty default keeps current output unchanged.
- 🎛️ **Add opt-in product-count badge setting** `[low | additive | medium]`  
  For a wholesale catalog it helps buyers to see catalog depth. Add a default-off checkbox 'show_product_count' and render {{ collection.products_count }} in the 'collection-card' snippet caption when enabled; leave off by default so nothing changes for existing merchants.

### `blocks/collection-title.liquid` — Renders the closest collection's title (or a placeholder) wrapped in a <p> and delegated to the shared 'text' snippet, with full typography/width/padding schema.
- ♿ **Placeholder title lacks context** `[low | moderate | medium]`  
  When closest.collection is blank the block outputs the generic 'placeholders.collection_title' string; in a live wholesale storefront an empty collection binding renders meaningless text. Consider hiding the block entirely (render nothing) when collection is blank outside the theme editor to avoid exposing placeholder copy to shoppers.
- 🔎 **Wrap collection title in a heading rather than a paragraph** `[medium | additive | medium]`  
  Both branches build '<p>' | append: title | append: '</p>' before passing fallback_text to 'text'. On a collection page the collection name is the primary heading; emitting it as <p> weakens document outline/SEO. Add an opt-in schema select 'semantic_tag' (default 'p' to preserve current behavior, option 'h2') and use it in the append so merchants can promote it to a real heading without changing defaults.

### `blocks/comparison-slider.liquid` — Before/after image comparison web component (comparison-slider-component) with a range-input handle, optional text labels, ratio/width/height, color scheme and border/padding controls.
- 🐛 **'square' ratio option has no case handler** `[low | additive | high]`  
  The schema image_ratio offers a 'square' option, but the {% case block.settings.image_ratio %} only handles landscape/portrait/adapt; 'square' falls through leaving ratio=1, which happens to be square, so it works by coincidence. Add an explicit `when 'square'` assigning ratio='1 / 1' for clarity and to avoid breakage if the default fallback changes.
- ♿ **Before/after images fall back to empty alt** `[medium | additive | high]`  
  Both image_tag calls set alt: block.settings.before_image.alt / after_image.alt with no fallback, so images uploaded without alt text render alt="" and screen readers get nothing meaningful. Add a default via '| default: block.settings.before_text' (or a localized 'Before'/'After' string) so the comparison is described. Purely additive - only affects images that currently have empty alt.
- ⚡ **Both comparison images request width 3840 and load lazy** `[medium | additive | high]`  
  before_image/after_image use image_url: width: 3840 with loading: 'lazy' and sizes: 'auto, 100vw'. On a block that is often above the fold in a section, forcing a 3840px source plus lazy can hurt LCP and waste bandwidth on trade buyers on mobile. Consider an opt-in 'eager_load' checkbox (default off) mapping to loading:'eager'/fetchpriority to let the merchant prioritize a hero comparison without changing the current lazy default.

### `blocks/contact-form-submit-button.liquid` — Submit button block for the contact form; renders a <button type="submit"> with configurable label, primary/secondary style, and desktop/mobile width via size-style.
- 💸 **No submitting/loading state on submit** `[low | additive | medium]`  
  The button renders a static <button type="submit"> with no busy indicator; on a slow wholesale-inquiry submission buyers may double-click. Consider adding an opt-in 'show_loading_spinner' setting that toggles an aria-busy/disabled state via the form's JS on submit, default off to preserve current behavior.

### `blocks/contact-form.liquid` — Contact form block rendering Shopify {% form 'contact' %} with name/email/phone/comment fields, success/error states, and an embedded submit-button block.
- 🐛 **Textarea preserves leading/trailing whitespace from template indentation** `[low | moderate | medium]`  
  The <textarea> contains newlines/indentation around {{- form.body -}}; although the whitespace-control dashes trim form.body, the literal indentation between the opening tag and the output can still inject whitespace into the field value on error re-render. Collapse the textarea to a single line (<textarea ...>{{- form.body -}}</textarea>) to guarantee a clean value.
- 💸 **No B2B qualifying fields for wholesale inquiries** `[medium | additive | medium]`  
  The form only collects name/email/phone/comment; a wholesale supplier typically needs practice/business name and ABN to route trade inquiries. Add opt-in default-off checkboxes (e.g. 'show_company_field') that render an extra contact[company] input, keeping the current form unchanged when disabled.
- ♿ **Email error container is not linked to the input** `[medium | moderate | medium]`  
  The email input sets aria-describedby="ContactForm-email-error" when form.errors contains 'email', but the error markup (.contact-form__error) has no id="ContactForm-email-error" - it only carries tabindex/autofocus. Add id="{{ form_id }}-email-error" (matching the describedby, which should be scoped to form_id) so assistive tech announces the message. Low-risk markup fix.

### `blocks/custom-liquid.liquid` — Renders merchant-authored custom Liquid/HTML from a single 'custom_liquid' setting inside a wrapper div.
- 🎛️ **Add opt-in wrapper padding settings** `[low | additive | low]`  
  The block outputs a bare <div> with no spacing/size hooks. Add default-0 'padding-block-start/end' and 'padding-inline-start/end' range settings and render them via 'spacing-style' (as other blocks like follow-on-shop do) so merchants can space embedded custom HTML without editing Liquid. Defaults of 0 keep current output identical.

### `blocks/email-signup.liquid` — Newsletter/email capture block using the customer form with configurable border/button/color styling.
- 🐛 **Duplicated style_class token in button class list** `[low | moderate | medium]`  
  Line 42 emits '{{ block_settings.style_class }} {{ block_settings.style_class }}--{{ block.id }}' but repeats '{{ block_settings.style_class }}' twice ('... {{ block_settings.style_class }} {{ block_settings.style_class }}'), producing a duplicate class token (e.g. 'button button--id button'). Harmless visually but is a copy-paste defect; remove the redundant token.
- 💸 **Add optional trade/wholesale consent + benefit subtext** `[medium | additive | medium]`  
  The block only renders 'heading' + input. For a B2B audience, add an optional 'subtext' text setting rendered under the heading (e.g. 'Get trade pricing & case-pack deals') and an optional 'consent_text' rich-text/paragraph below the form. Both default blank so nothing renders unless set, improving signup relevance for dental/trade buyers.
- ♿ **Success/error message region is not a live region** `[medium | additive | medium]`  
  The success div (#Email-signup__message-success) and error div use tabindex="-1" but no aria-live/role, and are only focused via aria-describedby on the input. Add role="status" (success) and role="alert" (error) so screen readers announce the post-submit outcome when the form re-renders, since the input's aria-describedby alone does not trigger announcement on non-focus flows.

### `blocks/featured-collection.liquid` — Container block that binds a chosen collection and renders nested theme/app blocks via content_for 'blocks'.
- ♿ **Wrapper has no landmark/label for the featured collection region** `[low | additive | low]`  
  The .featured-collection-block div is unlabeled. Add an optional aria-label/aria-labelledby (or role/region) tied to the selected 'collection.title' so assistive tech can identify this product region; keep it opt-in via a setting to avoid altering current output.

### `blocks/filters.liquid` — Renders the storefront facet filtering/sorting/grid-density UI (horizontal, vertical, and mobile drawer) driven by search/collection results.
- 💸 **Product count uses generic item_count with no wholesale context** `[medium | additive | high]`  
  Both products-count-wrapper blocks render 'content.item_count' only. Add an optional block setting (default off) to append trade context (e.g. show 'X products' plus a merchant-defined suffix like 'in stock for trade') so B2B buyers scanning large catalogues get reassurance; gate behind a new checkbox so default rendering is unchanged.
- ♿ **Filter count bubble ref is mislabeled 'cartBubbleText'** `[low | risky | high]`  
  In the toggle button the count span uses ref="cartBubbleText" (line ~308) though this is a filter-count bubble, not a cart bubble. If any facets.js logic targets this ref it is misleading/incorrect; rename to a filter-specific ref (e.g. filterBubbleText) after confirming no JS depends on the current name. Verify against facets.js before changing since this touches the web-component contract.
- ⚡ **Filters computed twice (pre-count loop then per-render loop)** `[low | moderate | high]`  
  total_active_values is computed in the first for-loop (lines 39-53) and then reset and recomputed again inside rendered_filters (lines 164-201) and the drawer loop. For large facet sets this doubles Liquid iteration. Consider computing active counts once and reusing, reducing server render time on filtered collection pages. Behavior-preserving but touches shared render logic, so validate horizontal/vertical/drawer parity.

### `blocks/footer-copyright.liquid` — Renders the footer copyright line (year, shop name link, optional 'powered by Shopify') with custom typography settings.
- 💸 **Add optional trust/ABN line setting** `[low | additive | low]`  
  Add a new default-empty 'text' setting (e.g. id 'legal_line') rendered after the copyright span so House of Mouth can display its ABN / registered business name / trade licensing beside the copyright. B2B/dental buyers look for a registered ABN as a legitimacy signal; leaving it blank preserves current output.
- 🎛️ **Expose case setting that is defined but unused** `[low | moderate | low]`  
  The schema defines a 'case' select (none/uppercase, lines 63-78) but the markup only renders 'custom-font-size' and typography-style; the 'case' value is never applied to .footer-utilities__text. Wire the selected case into the typography-style render or a text-transform inline var so the merchant control actually works.

### `blocks/footer-policy-list.liquid` — Renders a hover/click popover listing the shop's store policies (terms, privacy, refund, etc.) in the footer.
- 🎛️ **'case' select defined but never applied** `[low | moderate | medium]`  
  The schema exposes a 'case' select (lines 180-195) but nothing sets --text-transform; the CSS reads var(--text-transform, none) (lines 50, 65) which never receives the merchant's choice. Emit --text-transform from the selected 'case' (via typography-style or inline style) so the uppercase option functions.
- ♿ **Popover trigger button lacks aria-expanded/haspopup** `[medium | moderate | medium]`  
  The .policy-list-trigger button (lines 5-14) uses the native popovertarget API but exposes no aria-haspopup or aria-expanded state to assistive tech. Add aria-haspopup="true" and toggle aria-expanded via the anchored-popover-component so screen-reader users know a list will open; keep the visual behaviour identical.

### `blocks/icon.liquid` — Renders a selectable built-in icon or uploaded image (optionally linked) via the icon-or-image snippet with width/link settings.
- ♿ **Linked icon has no accessible label** `[medium | additive | medium]`  
  When block_settings.link is set, the <a class="text-inherit"> (lines 14-23) wraps only a decorative icon/SVG with no text and no aria-label, producing an unlabelled link (common for social icons). Add an optional 'aria_label' text setting rendered as aria-label on the anchor so linked social/utility icons are announced; default empty keeps current markup.

### `blocks/image.liquid` — Renders a responsive image block (or placeholder) with configurable aspect ratio, width/height, borders and padding, optionally wrapped in a link.
- 🐛 **'square' ratio option falls through to fallback aspect ratio** `[low | moderate | medium]`  
  The image_ratio schema offers a 'square' option (lines 145-148) but the case statement (lines 5-12) has no 'square' branch, so it relies on the ratio==1 default at lines 14-16. This works today only because ratio initialises to 1, but any change to that default would silently break square; add an explicit `when 'square' assign ratio = '1 / 1'` for correctness and clarity.
- ♿ **No alt-text override for the image** `[low | additive | medium]`  
  image_tag (lines 43-53) emits only the image's stored alt; there is no schema field to set descriptive alt per placement. Add an optional 'alt' text setting passed as `alt:` to image_tag so merchants can describe product/trade imagery; empty default falls back to the current stored alt.
- ⚡ **No loading/fetchpriority control on image_tag** `[medium | additive | medium]`  
  The image_tag call (lines 46-52) sets widths/sizes but no loading attribute, so it inherits the default (typically lazy). For an above-the-fold hero image block this hurts LCP. Add an opt-in schema select (e.g. 'loading_priority': auto/eager) that passes loading:'eager' and fetchpriority:'high' when chosen; default 'auto' preserves current lazy behaviour.

### `blocks/jumbo-text.liquid` — Decorative oversized text block that delegates rendering to the 'jumbo-text' snippet and exposes font/alignment/case/animation controls.
- 🎛️ **Add opt-in link/CTA setting** `[medium | additive | low]`  
  The block renders text only via {% render 'jumbo-text' %} with no link. Add an optional default-empty 'link' (type url) + 'link_label' setting so merchants can turn a jumbo headline into a clickable CTA (e.g. 'Open a Trade Account') without editing markup. Default empty preserves current non-linked behavior.
- ♿ **Expose heading level for semantics** `[medium | additive | low]`  
  The block offers a 'font' select (body/subheading/heading/accent) but no semantic tag control, so the underlying snippet likely renders a non-heading element regardless of visual prominence. Add an optional 'heading_tag' select (div default, h1-h6) passed to the snippet so a page's main headline can be a real heading for screen readers/SEO; keeping div as default leaves existing pages unchanged.

### `blocks/logo.liquid` — Renders the store logo (or inverse logo, with text fallback) as a responsive image with configurable pixel/percent sizing and mobile overrides.
- ♿ **Logo alt is always shop.name, ignoring inverse/context** `[low | additive | medium]`  
  image_tag sets alt: shop.name for both the primary and inverse logo. For a B2B supplier this is acceptable but generic; add an optional 'logo_alt' text setting (default blank -> falls back to shop.name) so the merchant can supply descriptive alt like 'House of Mouth wholesale oral care' without changing default output.
- ⚡ **Header logo should load eagerly / high priority** `[medium | moderate | medium]`  
  logo-block__image is generated with image_tag but no loading/fetchpriority hint; as an above-the-fold LCP-adjacent header element it should not be lazy. Add fetchpriority: 'high' (and explicit loading: 'eager') to the image_tag call to reduce LCP on landing pages. Since the logo is near the top of every page this is a safe, targeted change but alters emitted attributes, hence moderate.

### `blocks/menu.liquid` — Renders a linklist menu (e.g. footer/header column) with optional accordion behavior, dividers, color scheme, and typography presets, via the accordion-custom-component web component.
- 🎛️ **Add optional external-link new-tab / rel control** `[low | additive | medium]`  
  Menu links render as a plain <a href="{{ link.url }}"> with no rel or target handling. For a B2B site linking to supplier portals/PDFs, add an opt-in 'open_external_new_tab' checkbox (default off) that adds target=_blank + rel="noopener" for off-site link.url hosts, leaving current behavior unchanged when off.
- ♿ **Accordion nav lacks aria-label when heading present** `[medium | moderate | medium]`  
  The <nav class="details-content"> only gets aria-label when heading == blank. When a heading exists the <nav> has no accessible name and is not programmatically tied to the <summary> text; add aria-labelledby pointing to the summary/heading (or always set aria-label to heading|default:menu.title) so multiple menu landmarks are distinguishable to screen readers.

### `blocks/page-content.liquid` — Outputs the current page's RTE content wrapped in an <rte-formatter> web component for the page template.
- 🐛 **No guard when page.content is blank** `[low | moderate | medium]`  
  The block unconditionally renders <rte-formatter>{{ page.content }}</rte-formatter>. On a page with empty content (or when the block is used outside a page context) it emits an empty rte-formatter element. Wrap in {% if page.content != blank %} to avoid an empty stray web component in the DOM. Low-risk but touches the web-component contract, so moderate.

### `blocks/page.liquid` — Embeds a selected page's title and RTE content anywhere (e.g. section on homepage) with color-scheme, background, and padding controls plus placeholder fallback.
- 🎛️ **Add 'show title' toggle** `[low | additive | low]`  
  block_settings.page.title is rendered whenever present with no way to suppress it. Add an opt-in 'show_page_title' checkbox (default true) so a merchant embedding trade/wholesale policy copy can hide the redundant page title without losing the content; defaulting true keeps existing behavior.
- ♿ **Hardcoded h2 for embedded page title** `[low | additive | low]`  
  The title always renders as <h2 class="page-title">. When this block is placed on a page that already has its own h1/h2 structure this can break heading order. Add an optional 'heading_tag' select (default h2, options h2-h6) so merchants can maintain a correct document outline; default h2 preserves current output.

### `blocks/payment-icons.liquid` — Renders the shop's enabled payment method SVG icons as a list in the footer with alignment/gap/padding controls.
- 🎛️ **Optional trust-line text above icons** `[low | additive | low]`  
  Add a default-off `checkbox` (e.g. `show_secure_note`) plus a `text` setting rendered as a `<span>` before the `.payment-icons__list`. For a B2B/wholesale dental buyer, a 'Secure checkout - Visa, Mastercard, Amex accepted' line reinforces payment trust. Guard with `{% if block_settings.show_secure_note %}` so default output is unchanged.
- ♿ **Icons lack per-type accessible names** `[low | additive | low]`  
  `type | payment_type_svg_tag` outputs decorative SVGs; the only label is a single visually-hidden `{{ 'blocks.payment_methods' | t }}` span. Add `aria-hidden="true"` to `.payment-icons__list` (since the summary span already names the group) to prevent screen readers announcing each unlabeled SVG as a list item.

### `blocks/popup-link.liquid` — A button that opens a dialog/drawer (via dialog.js web component) containing nested theme/app blocks, used for links like return-policy popovers.
- ♿ **External-link icon is unlabeled and may mislead** `[low | moderate | medium]`  
  The trigger button appends `icon-external.svg` after `{{ block_settings.heading }}`, implying an external navigation, but it opens an in-page dialog. Add an `aria-haspopup="dialog"` attribute to the `on:click="/showDialog"` button and consider a schema toggle to hide the external icon, so assistive tech and sighted users get the correct affordance.
- 🔎 **heading text not escaped in visible button/dialog heading** `[low | additive | medium]`  
  `{{ block_settings.heading }}` is output raw at lines 17 and 31 (the h2). It is a merchant `text` setting so low-risk, but piping through `| escape` on the button label and the `#popup-dialog-heading` prevents accidental markup breakage from stray characters.

### `blocks/price.liquid` — Product-price theme block: renders the price snippet plus optional tax/duties note and installment (payment_terms) form inside a <product-price> web component.
- 💸 **Surface unit / case-pack price prominently for wholesale** `[high | additive | medium]`  
  `{% render 'price', show_unit_price: true ... %}` already passes unit price, but there is no control to emphasize it. Add a default-off `checkbox` (e.g. `emphasize_unit_price`) that adds a class toggling the unit-price styling to bold/larger. For dental trade buyers comparing case packs, a clear per-item unit price is the key purchase-decision signal.
- 💸 **Add optional volume/free-shipping threshold note under price** `[high | additive | medium]`  
  The `show_tax_info` block already renders a `.tax-note` div. Add a parallel default-off `checkbox` + `text` setting (e.g. `show_bulk_note`) rendering a merchant string like 'Free freight on orders over $X' below the price. This is high-value urgency/threshold messaging for B2B and is fully additive when left off.

### `blocks/product-card.liquid` — Standalone product-card block that resolves a product, gathers nested blocks, and delegates rendering to the 'product-card' snippet.
- 🐛 **Card renders nothing / broken when product setting is empty** `[medium | moderate | medium]`  
  `assign product = block.settings.product` with no fallback to `closest.product`; if the merchant leaves the `product` setting blank the card and its `content_for 'blocks', closest.product: product` receive a blank product. Consider `block.settings.product | default: closest.product` so the card degrades gracefully in product-context templates.
- 💸 **No quick-add / reorder affordance in card preset** `[high | additive | medium]`  
  The block allows nested blocks (text, image, price, swatches, review, sku) but the default preset (`block_order` gallery/title/price) has no add-to-cart. For wholesale quick-reorder, expose an opt-in nested buy/quick-add block type in the schema `blocks` array so merchants can drop an add-to-cart into cards without editing the snippet.

### `blocks/product-custom-property.liquid` — Line-item custom property input (text/textarea/checkbox) with live character counter, attached to the product form for buyer-supplied data like practice name or PO number.
- 🐛 **property_key collision when heading contains bracket/special chars** `[medium | moderate | medium]`  
  `property_name` is built via `'properties[custom-property]' | replace: 'custom-property', property_key` where `property_key` defaults to `property_heading`. A heading containing `]` or `[` would produce a malformed line-item property name. Sanitize/handleize `property_key` (e.g. `| handleize`) before injecting into the name attribute.
- 🎛️ **Text validation pattern for structured B2B inputs** `[medium | additive | medium]`  
  Add an opt-in `text` schema setting (e.g. `validation_pattern`) applied as the input's `pattern` attribute, default empty. Wholesale buyers often enter PO numbers / ABN / practice codes; a merchant-configurable pattern reduces bad-data friction. Empty default keeps current behavior unchanged.
- ♿ **Inputs have no visible/programmatic label** `[medium | moderate | medium]`  
  The text/textarea `field__input` elements have only a `placeholder` and no `<label for>` (the only label is `.__character-label` wrapping the counter). The `.__heading` `<p>` is not associated. Add `aria-label` from `block_settings.property_heading` (or wire the heading via `aria-labelledby`) so screen-reader users know what the field is, e.g. a PO-number field for trade orders.

### `blocks/product-description.liquid` — Renders the product description (or any richtext) via the shared 'text' snippet with typography, width, background and padding controls.
- 🐛 **Preview product mismatch when block.settings.product is set** `[low | moderate | low]`  
  In the {% liquid %} block, `product` is only assigned from `block.settings.product` and `text` from `product.description` inside the `request.visual_preview_mode and product == blank` branch. If a merchant binds a specific product to `block.settings.product`, the editor preview path still works, but on the storefront the block relies entirely on the preset `text: '{{ closest.product.description }}'`; the top-level `assign product = block.settings.product` is dead outside preview. Guard the description fallback so a bound `block.settings.product` also populates `text` (e.g. `if text == blank and product != blank; assign text = product.description; endif`), preventing an empty description when a product override is used.
- 🎛️ **Add opt-in read-more / truncate toggle for long B2B spec descriptions** `[low | additive | low]`  
  Wholesale oral-care listings often carry long spec/ingredient copy. Add a default-off `checkbox` setting `enable_truncate` plus a `range` `truncate_lines` that, when enabled, wraps the rendered 'text' output in a clamped container (CSS line-clamp) with a show-more control. Default off preserves current full-render behavior.

### `blocks/product-inventory.liquid` — Shows stock status (in stock / low / out of stock) with a colored icon and optional exact quantity, driven by inventory_management and a low-stock threshold.
- 🐛 **data-product-id uses undefined `product` variable, always emitting empty** `[medium | moderate | medium]`  
  Line 50 sets `data-product-id="{{ product.id }}"` but this block never assigns `product`; the inventory logic reads from `closest.product`. So `data-product-id` renders empty, breaking any JS/analytics that keys off it. Change to `data-product-id="{{ closest.product.id }}"`.
- 💸 **Add opt-in urgency threshold styling for wholesale reorder pressure** `[medium | additive | medium]`  
  The block already computes `status == 'low'` and can show the count via `content.inventory_low_stock_show_count`. For a B2B reorder audience, add a default-off `checkbox` `emphasize_low_stock` that, when on, adds a modifier class to `.product-inventory__text` (e.g. bold/lowstock color) so low-stock cases read as an urgency cue. Default off keeps current rendering identical.
- ♿ **aria-label overrides the actual stock text for screen readers** `[medium | moderate | medium]`  
  The `.product-inventory__text` span has `role="status"` plus a static `aria-label="{{ 'accessibility.inventory_status' | t }}"`. A non-empty aria-label overrides the element's inner text, so screen-reader users hear only the generic label (e.g. 'Inventory status') and never the meaningful 'In stock' / 'Low stock, N left'. Drop the aria-label (let the visible text be the accessible name) or move it to a wrapping element, keeping role=status for live announcements.

### `blocks/product-recommendations.liquid` — Fetches and renders related/complementary product recommendations as a grid or carousel via the Section Rendering API, with skeleton loading and product-card child blocks.
- 🐛 **icons_shape visible_if has unparenthesized and/or precedence bug** `[low | additive | high]`  
  The `icons_shape` setting's `visible_if` is `{{ block.settings.icons_style != 'none' and block.settings.layout_type == 'carousel' or block.settings.carousel_on_mobile == true }}`. With and/or evaluated left-to-right, this reduces to '(icons_style!=none AND layout==carousel) OR carousel_on_mobile', so the icon-background control shows whenever carousel_on_mobile is true even if icons_style is 'none'. Wrap the intended grouping in parentheses to match the header's `carousel or carousel_on_mobile` logic while still respecting icons_style != 'none'.
- 💸 **Complementary fallback renders nothing when no recs found** `[medium | additive | high]`  
  In the no-recommendations fallback (lines 75-82), `related` falls back to `collections.all.products` but `complementary` sets `products = null`, producing an empty section with a heading and no cards. For a B2B store this wastes prime cross-sell space (case packs, refills). Add an opt-in fallback: a `checkbox` `complementary_fallback_to_related` (default false) that reuses the related-catalog fallback so the block always shows relevant cross-sells.

### `blocks/product-title.liquid` — Renders the product title as a linked heading (via the shared 'text' snippet) using closest.product, with a placeholder when no product is bound.
- ♿ **Title link has no accessible/visual distinction and uses non-semantic p[role=heading]** `[low | additive | medium]`  
  The title is wrapped in `<a class="contents user-select-text" ref="productTitleLink">` around a `<p role="heading" aria-level="3">`. The `contents` class removes the link box and there is no aria-label, so on product cards the link's accessible name is just the title with no context ('view product'). Add an opt-in (default-off) setting to append an `aria-label` like `{{ closest.product.title }} - view details` on the anchor for clearer screen-reader link purpose without changing visuals.
- 🔎 **Product title is not a real heading element** `[low | additive | medium]`  
  Both branches emit `<p role="heading" aria-level="3">` rather than an `<h*>`. On the main product template the product name should ideally be an actual heading for SEO/document outline. Since this block is reused on cards (where h-levels would be wrong), add an opt-in `select` `heading_tag` (default 'p' = current behavior) letting the merchant promote it to a semantic tag on the product page only.

### `blocks/quantity.liquid` — Thin wrapper block that renders the shared 'quantity-selector' snippet for closest.product.
- 💸 **No schema settings expose the snippet's volume/case-pack capabilities** `[medium | additive | medium]`  
  The block is a bare passthrough (`{% render 'quantity-selector', product: closest.product %}`) with an empty schema (only name/tag). The underlying snippet already supports quantity_rule min/increment and price-per-item volume pricing, but the merchant has zero control here. Add opt-in settings (default-off/blank) such as a `text` `bulk_hint` rendered near the selector (e.g. 'Sold in cases of 12') to communicate case-pack/MOQ to trade buyers, passed through to the snippet. Purely additive.

### `blocks/review.liquid` — Renders product review star rating (from reviews metafields) with optional numeric rating/review count, in outline/shaded/single styles.
- 🐛 **Unclosed rgb() in half-star gradient stop-color** `[medium | moderate | medium]`  
  Line 49 stop-color for the non-outline half fill is `rgb(var(--star-fill-color-rgb) / var(--opacity-20)` with the closing paren missing before `{% endif %}`, producing invalid CSS for shaded/single styles so the empty half of a half-star may not render its intended 20% tint. Add the missing `)`.
- 💸 **Make review count a clickable anchor to reviews** `[medium | additive | medium]`  
  The rating_count text in the `.rating-count` <p> is static. Wrapping the '{{ rating_count }} {{ 'content.reviews' | t }}' output in an <a href='#reviews'> (behind a new default-off 'link_to_reviews' checkbox) lets trade buyers jump straight to the review app section, a proven social-proof CRO lever for wholesale trust.
- ♿ **aria-label omits review count and scale** `[medium | additive | medium]`  
  Line 60 aria-label uses `accessibility.rating` with only `rating:`, so screen readers hear e.g. '4.5' with no max or count. Extend the label to include scale_max and rating_count (e.g. 'rating: rating, count: rating_count') so assistive-tech users get the full social-proof signal that sighted users see.

### `blocks/sku.liquid` — Renders the selected variant's SKU inside a <product-sku-component> web component with typography/width/padding controls.
- 💸 **Optional SKU label prefix for trade reorder clarity** `[medium | additive | medium]`  
  The block renders only the raw SKU via {% render 'sku' %}. For B2B buyers who reorder by code, add a default-off 'label_text' text setting (e.g. 'SKU:' / 'Product code:') rendered before the SKU so wholesale purchasers can scan and quote codes faster. Keep empty default so current output is unchanged.
- ♿ **SKU hidden purely via inline display:none is not announced** `[low | additive | medium]`  
  When sku is blank the component gets inline `display: none` (lines 42-46) but no aria treatment; when populated there is no semantic label. Consider wrapping the value with an sr-only prefix ('Product code') so screen-reader users understand the standalone alphanumeric string, which otherwise reads as gibberish.

### `blocks/social-links.liquid` — Renders a configurable row of social media icon links (Facebook, Instagram, X, LinkedIn, etc.) with editor disabled-state handling.
- 🐛 **custom_url with a bare domain renders an empty, iconless link** `[medium | moderate | low]`  
  For 'custom_url' the code sets render_icon=false (line 49) then forces has_profile=true (lines 61-63), so a link with no recognizable icon renders an <a> whose only content is the domain-derived platform text via `.social-icons__icon-label`, which CSS hides only when an icon path exists (lines 139-149). A custom link therefore shows a visible domain word or an empty clickable box. Guard the custom branch so a label is shown intentionally or the link is skipped.
- ♿ **aria-label uses domain guess for custom links** `[low | additive | low]`  
  For custom_url the aria-label (line 81) becomes the capitalized first domain segment (e.g. 'Linktr' for linktr.ee), which is misleading to screen readers. Add an optional 'custom_link_label' text setting and use it for both the visible label and aria-label when provided.

### `blocks/spacer.liquid` — Layout spacer block that adds pixel/percent vertical or horizontal space with optional separate mobile sizing.
- 🐛 **Percent desktop spacer emits unitless flex value while mobile emits %** `[medium | moderate | low]`  
  Line 13 sets `--spacer-size: {{ percent_size | divided_by: 100.00 }}` (e.g. 1) used as `flex: var(--spacer-size)` (line 42), but the mobile percent branch (line 20) sets `--spacer-size-mobile: {{ percent_size_mobile }}%` (e.g. 100%) also consumed by `flex:` (lines 58,65). A `flex: 100%` is not equivalent to `flex: 1`, so mobile percent spacers can size very differently from desktop. Normalize mobile percent to `divided_by: 100.00` for consistent flex growth.

### `blocks/swatches.liquid` — Renders variant color/option swatches inside a <product-swatches> web component when the product has swatch-enabled options.
- 🐛 **hide_padding setting is permanently hidden and unused for control** `[low | additive | high]`  
  The 'hide_padding' checkbox has `visible_if: {{ false }}` (line 102) so merchants can never toggle it, yet every padding range and the padding header are gated on `hide_padding == false`. This dead setting adds confusion; either remove it or make it visible so merchants can actually collapse swatch padding.
- 🎛️ **Expose swatch overflow behavior / max rows as a setting** `[low | additive | high]`  
  The <product-swatches> host has `overflow: hidden` (stylesheet line 17) and relies on --overflow-list-alignment JS, but there is no merchant control over wrap vs. scroll. For products with many trade variants (e.g. flavour/size cases), add an opt-in select ('wrap' vs 'overflow') mapped to an existing modifier class so merchants control how long swatch lists display without editing code.

### `blocks/text.liquid` — Renders a rich-text/heading block via the 'text' snippet with extensive typography, width, background, and padding schema settings.
- 🎛️ **No max-width option for 'wide' content** `[low | additive | low]`  
  The 'max_width' select only offers narrow/normal/none. For long B2B spec/policy copy a merchant may want a 'wide' step between 'normal' and 'none'. Add a new 'wide' option to the existing max_width select; existing blocks keep 'normal' default so no visual change.
- ♿ **Background color alpha default risks low contrast** `[low | additive | low]`  
  The 'background' checkbox pairs with 'background_color' defaulting to '#00000026' (15% black). On dark or busy backgrounds this near-transparent overlay gives text a low-contrast surface. Add an 'info' note on the background_color setting warning to verify WCAG contrast, or add an optional 'text_contrast_boost' checkbox that applies a solid backdrop; keep it default-off so nothing changes.

### `blocks/variant-picker.liquid` — Resolves the closest product (with visual-preview fallback) and delegates to the 'variant-main-picker' snippet; schema exposes dropdown/button style, swatches, alignment, padding.
- 💸 **No per-variant availability/stock hint at block level** `[medium | moderate | high]`  
  This block drives variant selection but exposes no setting to surface case-pack/stock status. Add an opt-in 'show_variant_availability' checkbox (default false) passed through to variant-main-picker so trade buyers see in-stock/backorder per option; keep default off to avoid altering the current rendered picker.

### `blocks/video.liquid` — Renders an uploaded or external (YouTube/Vimeo) video via the 'video' snippet, building size/aspect-ratio/border CSS custom properties from block settings.
- 🐛 **video_autoplay setting not passed to render, relies on snippet fallback** `[low | additive | medium]`  
  The schema defines 'video_autoplay' but neither {% render 'video' %} call passes it; it only works because snippets/video.liquid falls back to block_settings.video_autoplay. Passing video_autoplay: block_settings.video_autoplay explicitly on both render calls makes the contract robust against future snippet refactors that drop the fallback.
- ♿ **Alt text only available for external URL source** `[medium | moderate | medium]`  
  The 'alt' text setting is gated with visible_if source == 'url', and only the URL branch passes video_alt to the snippet. Uploaded videos get no author-provided alt/label. Add the 'alt' field visibility for the uploaded source too and pass video_alt in the uploaded {% render 'video' %} call so screen-reader users get a description regardless of source.


## Sections

### `sections/_blocks.liquid` — Generic layout section wrapper that captures child theme/app/_divider blocks and renders them through the 'section' snippet with full layout, size, background media, overlay, and border controls.
- ♿ **Background image picker has no alt handling** `[low | additive | medium]`  
  The 'background_image' image_picker feeds decorative section backgrounds but there is no alt/labeling path visible here. Confirm the downstream 'section' snippet outputs role/aria appropriately (decorative bg should have empty alt); if it uses image alt from the asset, add an optional 'background_image_alt' text setting so merchants can label meaningful background imagery.

### `sections/carousel.liquid` — Storytelling carousel section rendering a static header group plus a static _carousel-content block of image/heading/text cards, with columns, mobile columns, gap, nav icon, and color-scheme controls.
- 🎛️ **Autoplay/loop and drag not exposed for the carousel** `[medium | additive | medium]`  
  Schema offers columns, mobile_columns, gap, icons_style/icons_shape but no autoplay or loop toggle. Add opt-in 'autoplay' checkbox (default false) and 'autoplay_interval' range so merchants can auto-advance product/story carousels; default-off keeps current static behavior. Wire only if the underlying _carousel-content component supports it.
- ⚡ **gap-style hardcodes value 12, ignoring merchant columns_gap** `[medium | moderate | medium]`  
  In the wrapper the inline style calls {% render 'gap-style', value: 12 %} with a literal 12, while the schema exposes a 'columns_gap' range (default 8). The horizontal gap the merchant sets is not applied to this render, so adjusting columns_gap has no effect on the section's own gap-style. Pass value: section.settings.columns_gap to honor the setting.
- 🔎 **Default card copy is generic furniture/design filler** `[low | additive | medium]`  
  Preset text defaults (t:html_defaults.discover_elevated_design, artistry_in_action, bold_style_recognizable, made_to_last) are generic design-brand filler irrelevant to a B2B oral-care/dental-supply store. Swap preset defaults for oral-care-relevant placeholders (e.g. bulk-savings, clinical-grade, fast-dispatch) so merchant-added carousels start on-brand; changes only the preset seed, not existing instances.

### `sections/collection-links.liquid` — Renders a set of linked collections in a spotlight (image + slideshow) or text layout using the <collection-links-component> web component.
- 💸 **Expose an optional heading/intro text block for the section** `[medium | additive | medium]`  
  The section jumps straight into collection links with no titling; for a B2B catalog with many categories (whitening, interdental, kids, etc.) a merchant-editable heading like 'Shop by category' improves scannability. Add an optional default-empty 'heading' text setting rendered above .collection-links__container only when populated, so unset installs render identically.
- 🎛️ **Add opt-in autoplay/interval control for spotlight slideshow** `[low | additive | medium]`  
  The 'spotlight' layout renders {% render 'slideshow' %} (line 87) with no merchant control over autoplay or timing. Add default-off schema settings (e.g. 'autoplay' checkbox default false, 'autoplay_interval' range) passed into the slideshow render, letting the merchant animate collection spotlights without changing current static behavior.

### `sections/collection-list.liquid` — Renders a merchant-chosen list of collections in grid/carousel/bento/editorial layouts via _collection-card blocks and the shared resource-list snippet.
- 🐛 **icons_shape visible_if has wrong operator precedence, showing the setting for grid layout** `[low | additive | medium]`  
  The icons_shape setting (line 292) uses visible_if "{{ section.settings.icons_style != 'none' and section.settings.layout_type == 'carousel' or section.settings.carousel_on_mobile == true }}". With Liquid left-to-right evaluation, the trailing 'or carousel_on_mobile == true' makes it visible whenever carousel_on_mobile is true even for grid, and the intended grouping (style != none AND (carousel OR mobile-carousel)) is broken. Wrap the OR in explicit grouping to match the sibling icons_style condition and avoid an orphan control.
- 💸 **Editorial/empty-state placeholder loop iterates without real collections, hiding merchant setup value** `[low | additive | medium]`  
  When section_collections is null the section builds placeholder cards (lines 37-43); for a B2B store the default 'Shop by collection' heading text (t:html_defaults.shop_by_collection) is generic. Consider an additive schema 'heading_text' default-empty override so merchants can label it 'Shop by product category' for trade buyers without editing the group block JSON.

### `sections/custom-liquid.liquid` — Outputs merchant-authored raw Liquid (section.settings.custom_liquid) inside a padded, color-scheme-aware section wrapper.
- 🎛️ **Add optional max content-width / horizontal alignment setting** `[low | additive | low]`  
  The wrapper only offers page-width vs full-width (section_width select) and vertical padding. Add a default-off 'content_alignment' or optional inline max-width setting so merchants embedding custom HTML (e.g. a wholesale enquiry form or trade-terms block) can constrain and center it without wrapping their own CSS; defaults keep current full-bleed output.

### `sections/divider.liquid` — Renders a horizontal divider line with configurable thickness, length, alignment, corner radius and color scheme via the shared 'divider' snippet.
- 🐛 **corner_radius visible_if uses references to width_percent with ambiguous precedence** `[low | additive | low]`  
  corner_radius visible_if (line 66) is "{{ section.settings.thickness > 1 and section.settings.width_percent != 100 or section.settings.section_width == \"page-width\" }}". Left-to-right Liquid evaluation means the 'or section_width == page-width' clause makes the control appear for page-width even when thickness <= 1, contradicting the '(thickness > 1)' intent. Re-group the boolean so corner radius only shows for a rounded-eligible divider.
- ♿ **Confirm divider is exposed as a semantic separator with aria** `[low | additive | low]`  
  The snippet is rendered via {% render 'divider' ... attributes: true %}; verify the underlying snippet emits role="separator" (or an <hr>) so screen readers announce the section break. If it emits a bare styled <div>, add role="separator" in the snippet — purely additive semantics with no visual change.

### `sections/featured-blog-posts.liquid` — Renders featured articles from a chosen blog in grid/carousel/editorial layouts using _featured-blog-posts-card blocks and the shared resource-list snippet.
- ⚡ **paginate wraps only the assign, defeating its limiting purpose and adding a redundant query** `[low | moderate | medium]`  
  Lines 23-26 open {% paginate section.settings.blog.articles by max_items %} but only assign section_articles inside it, then the actual card loop (line 36) iterates section_articles with its own 'limit: max_items' outside the paginate block. The paginate here adds no value (and can trigger an extra count query); the 'limit' on the for-loop already bounds output. Drop the paginate wrapper and assign the articles directly to simplify and avoid the redundant pagination overhead.
- 🔎 **Add optional 'View all posts' link to the blog for internal linking** `[medium | additive | medium]`  
  The section shows a title and N cards but no path to the full blog. Add a default-off 'show_view_all' checkbox that, when enabled, renders a link to section.settings.blog.url below the list. For a B2B content-marketing store this improves crawl depth and gives readers a next step; defaults keep current output.

### `sections/featured-product-information.liquid` — Renders a full product-information experience (media carousel, title, price, review stars, variant picker, buy buttons) for a single merchant-selected product via theme blocks.
- 💸 **Expose quantity/case-pack context on the buy-buttons block** `[medium | additive | medium]`  
  The preset wires buy_buttons with a static 'quantity' block but no volume/case-pack guidance. For B2B trade buyers, add an opt-in text setting (e.g. 'case_pack_note', default empty) rendered near the quantity block so merchants can state 'Sold per case of 12' without editing markup; keep it default-off so nothing changes for merchants who don't set it.
- 🎛️ **Add a toggle to show/hide the SKU near the title** `[medium | additive | medium]`  
  The header group preset only contains 'title' and 'price'. Trade buyers reorder by SKU; add an optional checkbox setting (default false) that, when enabled, surfaces product.selected_or_first_available_variant.sku beneath the product-title block. Default-off keeps current layout intact.
- ♿ **visually-hidden h2 uses a generic label** `[low | additive | medium]`  
  Line 24 outputs `<h2 class="visually-hidden">{{ 'accessibility.featured_product' | t }}</h2>` with no product context. Interpolate the selected product title (e.g. append `section.settings.product.title`) into the accessible heading so screen-reader users landing via the region know which product this is.

### `sections/featured-product.liquid` — Renders a promotional single-product feature (media + title/price/gallery/swatches) in a two-column full-width layout with aspect-ratio-driven sizing.
- 💸 **No add-to-cart or trust signal in the feature block** `[high | additive | medium]`  
  The preset's _featured-product block only contains title, price, gallery and swatches (lines 178-204) with no buy-buttons or 'View product' CTA rendered in this section itself, so the feature is non-transactional. Add an opt-in setting/block for a CTA button linking to the product (or a quick-add) so trade buyers can act without an extra click; default-off preserves current behavior.
- ♿ **Duplicate visually-hidden 'featured product' heading** `[low | additive | medium]`  
  Line 25 outputs `<h2 class="visually-hidden">{{ 'accessibility.featured_product' | t }}</h2>` identical to the featured-product-information section; add the product title to the accessible heading so multiple featured-product sections on one page are distinguishable to screen readers.
- ⚡ **gallery_aspect_ratio default of 1 can cause CLS** `[low | moderate | medium]`  
  Line 7 sets `gallery_aspect_ratio` from `product.featured_media.preview_image.aspect_ratio | default: 1` feeding `--gallery-aspect-ratio` (line 22). When a product has no featured media the forced 1:1 differs from the real first image ratio, shifting the layout. Guard by only emitting the custom property when a real ratio exists, letting `--media-preview-ratio` (already referenced line 93) take over.

### `sections/footer-group.json` — Section-group config that assembles the live footer: a team image, a 3-column menu group (Shop/Ask/Connect), an email signup, plus the utilities row (copyright, policy list, social links).
- 🐛 **Two footer menu columns point to empty menus** `[high | moderate | high]`  
  In group_EJMKU9, menu_CY8HUp ('Ask') and menu_n8zrAw ('Connect') both have `"menu":""` while only menu_k94JXn ('Shop') is bound to the `footer` menu. The 'Ask' and 'Connect' headings render with no links, giving customers dead columns. Assign real linklists (e.g. support/contact and social/about menus) to these two blocks.
- 💸 **Email signup lacks a B2B value proposition** `[medium | moderate | high]`  
  email_signup_t3aAfi has `heading:"We send tasty emails"` and `label:"Sign Up"` but no supporting copy about trade pricing, bulk offers, or new-product alerts relevant to dental practices. Add descriptive text (via the footer.liquid newsletter-group pattern) framing the list around wholesale deals/early access to lift signups from trade buyers.
- 🔎 **Footer team image has no descriptive context** `[low | additive | high]`  
  image_Tafkxb uses `oti-team-full.webp` with `link:""` and no alt provided in config; ensure the image asset carries meaningful alt text (team/brand) and consider linking it to an About/wholesale page so the prominent footer image is both accessible and a navigational asset.

### `sections/footer-utilities.liquid` — Renders the bottom footer utilities row (copyright, policy links, social icons) with responsive 1/2/3-column grid alignment driven by block count.
- 🐛 **text-wrap: nowrap can overflow on narrow viewports** `[medium | moderate | medium]`  
  Line 22 sets `text-wrap: nowrap` on `.utilities`, which prevents copyright/policy text from wrapping; long policy link labels or a long shop name can cause horizontal overflow on small phones. Scope nowrap to the social-icons row only, or allow wrapping below the 750px breakpoint.
- 🎛️ **No control over utilities row alignment** `[low | additive | medium]`  
  Alignment is hard-coded by block count (`.utilities--blocks-N` rules, lines 50-86). Add an opt-in select setting (default preserving current behavior) letting the merchant force left/center/spread alignment, useful when they add or remove blocks and dislike the auto layout.

### `sections/footer.liquid` — Main footer content section rendering merchant blocks (menus, image, email-signup, logo, etc.) in a responsive grid with orphan-item handling for 5-column layouts.
- 💸 **Default newsletter preset copy is generic B2C** `[low | additive | medium]`  
  The preset newsletter blocks (lines 224-241) use 'join_our_email_list' and 'get_exclusive_deals_and_early_access' generic copy. Since the live footer already overrides this, no default change is needed, but consider an opt-in setting or trade-focused default translation string so a fresh install nudges toward wholesale-relevant signup messaging.
- ⚡ **content-visibility: auto without contain-intrinsic-size** `[medium | moderate | medium]`  
  Lines 49-50 set `contain: content; content-visibility: auto` on `.footer-content` but no `contain-intrinsic-size`. When the footer scrolls into view its real height replaces the browser's default estimate, causing a scroll-position jump / CLS. Add a `contain-intrinsic-size` estimate (e.g. auto with a height fallback) so reserved space matches rendered height.

### `sections/header-announcements.liquid` — Renders the header announcement bar with rotating slides, optional autoplay/arrows, color scheme, divider and padding controls.
- 💸 **Add optional trust/USP announcement preset content for B2B** `[high | additive | medium]`  
  The section only ships a single empty '_announcement' block by default. For a wholesale audience, add an additive schema paragraph/info or default block text options is out of scope for markup, but a low-risk win is documenting via a new 'header' + default-off setting; more concretely, the _announcement block content is where free-shipping-threshold / 'Trade pricing on account' / 'Min order $X' messaging should live, and autoplay speed (section.settings.speed, default 5s) is fast for reading long trade messages — raise the max above 10 or leave default and let merchant slow it. This is the primary place to surface a free-shipping/case-pack threshold bar.
- ♿ **aria-live only set when autoplay is active** `[medium | additive | medium]`  
  aria-live="polite" is applied on announcement-bar-component only inside the autoplay==true branch (line 26). With a single announcement (autoplay false) that is acceptable, but when multiple slides rotate the polite region is fine; no defect, however the rotating region has no pause control beyond arrows — expose a merchant toggle to disable autoplay (set autoplay=false) so users who need to read long B2B messages can, improving WCAG 2.2.1 compliance without changing default.

### `sections/header-group.json` — Auto-generated header section group wiring the header section, its static logo/menu blocks, and all header layout/color/transparency settings.
- 💸 **No announcement bar section is present in the header group** `[high | moderate | high]`  
  The 'order' array contains only 'header_section'; there is no announcement-bar section instance in this header group. For a B2B store, adding an announcement bar above the header (free-shipping / trade-account / min-order messaging) is a high-value merchandising slot. This is an additive change (append a new section to sections + order) but because the file is auto-generated by the theme editor it should be done in the editor, not by hand-editing.

### `sections/header.liquid` — Core header section rendering logo/menu/search/localization/actions rows, sticky and transparent-header behavior, Organization JSON-LD, and the header web component.
- 💸 **No B2B account/wholesale entry point in header actions** `[medium | additive | high]`  
  Header actions are rendered via 'header-actions' with only customer_account_menu and display style; there is no dedicated 'Trade login / Apply for wholesale account' affordance. Add an optional schema setting (default blank) for a wholesale/quote-request link surfaced in the top row so trade buyers can reach account application quickly. Keep default empty so existing headers are unchanged.
- 🔎 **Organization JSON-LD omits contactPoint and social profiles** `[medium | additive | high]`  
  The Organization schema (lines 285-295) only outputs name, logo and url. For a Gold Coast B2B supplier, add optional 'contactPoint' (telephone/email, contactType 'sales') and 'sameAs' from settings.social_* links so trade buyers and Google surface phone/email. Gate behind existing global settings so it is additive and only emits when the merchant has set them.

### `sections/hero.liquid` — Full-width hero banner supporting one or two image/video media slots with custom mobile media, art-directed picture elements, blurred reflection, overlay, and content blocks.
- 💸 **Hero link wraps whole hero but has no accessible label** `[medium | additive | medium]`  
  When section.settings.link is set, an empty <a class="hero__link"> (lines 502-510) covers the hero with no text or aria-label, so screen readers and analytics see an unlabeled link. Add an optional 'link_aria_label' setting (default blank, falling back to first text block or shop name) so the merchant can name the destination (e.g. 'Shop trade catalogue') — additive and improves both a11y and click clarity.
- ♿ **Autoplay hero videos lack merchant-controlled captions/labels and always autoplay** `[medium | additive | medium]`  
  Both video slots (video_tag calls at lines 167-177, 269-281, and mobile variants) hardcode autoplay:true, loop:true, controls:false, muted:true with no accessible name. Since a decorative background video is muted this is mostly fine, but there is no merchant option to enable controls or a reduced-motion fallback poster; add an additive 'enable_video_controls' checkbox (default false) so B2B merchants using an explainer video in the hero can expose controls without breaking the current decorative default.
- ⚡ **Blurred-reflection duplicate images add extra decode/download weight** `[medium | moderate | medium]`  
  When blurred_reflection is on, media_blurred and mobile_media_blurred re-render the same image_1/image_2 at width 3840/1600 (lines 332-449) in addition to the visible media, doubling image bytes for the hero. These reflection images should request smaller widths (they are heavily blurred via blur(20px)) — cap image_url width to ~1200 and drop the large widths list for the blurred copies to cut LCP/transfer with no visible quality loss.

### `sections/layered-slideshow.liquid` — Accordion-style layered slideshow with draggable tab panels driven by a web component, tablist ARIA roles, and per-block image/video content.
- ♿ **Tab panels rely on external component for tab/panel wiring; tab buttons have no visible label or focus indicator by default** `[low | additive | high]`  
  Each tab button (lines 37-45) is aria-labelled with slide_status and controls a panel, but the buttons have opacity:0 and outline:none, only becoming visible on :focus-visible (line 124). Keyboard users get a focus ring, but there is no persistent visual indicator of the active tab for sighted mouse users beyond panel position. Consider an additive setting to show a subtle tab indicator; low risk since it is opt-in CSS driven by a new default-off setting.
- ⚡ **All panel media rendered eagerly via content_for blocks** `[low | additive | high]`  
  content_for 'blocks' (line 49) renders every slide's image/video up front; the CSS shows images use object-fit:cover at full panel size (lines 179-185). If slides carry videos, all autoplay/load simultaneously off-screen. A merchant-facing note or block-level lazy setting is out of this file's control, but the section could pass a 'first slide eager, rest lazy' hint; the safest additive change is verifying the _layered-slide block honors loading=lazy for non-first panels. No default change here beyond documentation.

### `sections/logo.liquid` — Renders the shop logo (image or jumbo text fallback) for the footer section group with sizing, alignment, and color controls.
- 🎛️ **No link option on the logo image** `[low | additive | low]`  
  The rendered logo (logo-section__image-wrapper) is a bare image with no anchor. Add an optional checkbox setting 'link_to_home' (default false) that wraps logo_image in `<a href="{{ routes.root_url }}" aria-label="{{ shop.name }}">`; keeps default output identical while letting the merchant make the footer logo clickable back to home.
- ♿ **Logo alt text is generic shop.name** `[low | additive | low]`  
  The image_tag uses alt: shop.name; for a footer logo this duplicates the header logo and gives screen-reader users redundant info. Add an optional schema text setting (e.g. id 'logo_alt', default blank) and use `alt: section_settings.logo_alt | default: shop.name` so the merchant can supply meaningful alt (or an empty decorative alt when a header logo already announces the brand). Default-off preserves current output.

### `sections/main-404.liquid` — Renders the 404 not-found page body as a flex layout panel that hosts theme/app blocks.
- 💸 **404 has no recovery path for lost B2B buyers** `[medium | additive | low]`  
  The section only renders `{% content_for 'blocks' %}` with no built-in navigation, so a trade buyer who mistypes a product URL hits a dead end. Add an optional schema block or default-off settings (e.g. checkbox 'show_search' + text 'help_text') that renders the predictive search input and a link to the catalog/collections, helping wholesale customers recover to a purchase path instead of bouncing.

### `sections/main-blog-post.liquid` — Renders a single blog article: title, meta, featured image, content, app blocks, and paginated comments with structured data.
- ♿ **Comments list uses div soup instead of semantic list** `[low | moderate | low]`  
  Each comment is a `<div class="blog-post-comment">` inside `.blog-post-comments`; screen readers get no item count/structure. Wrap comments in `<ul>`/`<li>` (or add role="list"/role="listitem") so assistive tech announces the number of comments. Low visual risk since styling is class-based, but it changes markup so verify CSS.
- 🔎 **Only Article structured_data emitted, no BreadcrumbList** `[low | additive | low]`  
  The <script type="application/ld+json"> outputs `article | structured_data` only. For a B2B content-marketing blog, add an optional default-off checkbox 'enable_breadcrumb_schema' that emits a BreadcrumbList JSON-LD (Home > Blog > Article) to improve SERP breadcrumb display. Additive and does not alter existing article schema.

### `sections/main-blog.liquid` — Renders the blog listing as a responsive 6-column grid with Liquid-computed hero/standard/compact card layout driven by article count, inside a blog-posts-list web component.
- ♿ **Blog grid container is not a semantic list/region** `[low | moderate | medium]`  
  The `.blog-posts-container` grid holds `.blog-post-item` divs with no list semantics or landmark; add role="list" on the container and role="listitem" on each item (or a `<ul>`) so screen readers announce article count on the listing page. Class-driven styling means minimal visual risk.
- ⚡ **Per-article {% style %} block duplicates CSS on every iteration** `[medium | moderate | medium]`  
  Inside the `for article in blog.articles` loop, a `{% style %}` tag emits a rule keyed on [data-blog-index] plus a mobile media query for each post, producing N inline <style> tags and CLS/parse overhead on large blogs. Since --col-span/--blog-post-card-scale are already deterministic, move them into the inline `style="--col-span:..."` attribute on the .blog-post-item div (mobile handled by a single stylesheet rule using an attribute selector), removing repeated style injection.

### `sections/main-cart.liquid` — Renders the full cart page (title, line items, summary, extra blocks) plus an empty-cart template, wired into the cart-items-component web component.
- 💸 **No free-shipping / order-minimum progress signal in cart** `[high | additive | high]`  
  The cart page renders title, _cart-products, and _cart-summary but has no threshold/urgency messaging, which is high-leverage for wholesale AOV. Add an optional default-off block/settings (e.g. richtext 'free_shipping_threshold_text' + a progress bar computed from cart.total_price) rendered above .cart-page__summary to nudge trade buyers toward a free-shipping or minimum-order case-pack threshold. Additive; nothing renders unless the merchant enables it.
- 💸 **Empty-cart state offers no reorder / continue-shopping path** `[medium | additive | high]`  
  The `#empty-cart-template` renders only the cart title and _cart-products empty state with more-blocks; for a B2B reorder audience add an optional default-off setting for a 'continue_shopping' button (link + label) or featured collection so an empty cart routes trade buyers back into the catalog instead of a dead end. Keep default output unchanged.

### `sections/main-collection-list.liquid` — Renders a list/grid/carousel/bento/editorial of collections using the shared resource-list and _collection-card blocks.
- 💸 **Expose product-count label per collection card** `[medium | additive | medium]`  
  The section drives collection cards via '_collection-card' but offers no schema toggle to surface each collection's products_count. For a B2B/wholesale catalog, adding a default-off checkbox setting (e.g. 'show_product_count') that the _collection-card block can read helps trade buyers gauge range depth (e.g. 'Toothbrushes 42 products'). Add as a new default:false setting so nothing changes for existing merchants.
- ⚡ **max_items hardcoded to 20 ignores editorial max_collections** `[medium | moderate | medium]`  
  Line 3 hardcodes 'assign max_items = 20' and passes it as slide_count and the for-loop limit, but the schema exposes 'max_collections' (min 1, max 16, default 4) only used in editorial layout. In grid/carousel the loop always iterates up to 20 collections regardless of any count intent, rendering more cards than needed. Wire max_items to section.settings.max_collections (falling back to 20) so merchants can cap rendered/queried cards and reduce DOM/image load.

### `sections/main-collection.liquid` — Main collection page container: renders filters, paginated product grid via results-list web component with optional infinite scroll.
- 💸 **Add opt-in free-shipping / wholesale trust banner above the grid** `[high | additive | high]`  
  The collection wrapper (div.collection-wrapper) jumps straight from filters to the product grid with no merchant-controlled trust/urgency strip. Add a default-empty richtext or text schema setting (e.g. 'collection_banner_html', default '') rendered just inside results-list before the grid, so House of Mouth can surface 'Free shipping over $X for trade accounts' or MOQ messaging on every collection without editing markup. Default-empty keeps current output identical.
- ♿ **Product grid <li> items lack a wrapping list semantic guarantee** `[medium | moderate | high]`  
  Lines 52-62 emit <li> elements with class product-grid__item, but the surrounding <ul>/<ol> is produced inside the 'product-grid' snippet; confirm and, if missing, ensure the list role is present. As an additive improvement, add an aria-label to results-list (e.g. via a translated string using collection.title and collection.products_count) so screen-reader users hear how many products are in the filtered result set announced on the region.
- ⚡ **products_per_page fixed at 24 when infinite scroll is on** `[medium | moderate | high]`  
  Line 43 hardcodes products_per_page = 24 and only overrides it from settings when infinite scroll is disabled (lines 45-47). With large wholesale catalogs, 24 image-heavy cards on first paint can hurt LCP/CLS on mobile. Consider an additive setting to let the merchant choose the first-page count even with infinite scroll enabled (default 24 preserves behavior), reducing initial payload on mobile.

### `sections/main-page.liquid` — Renders a standard page template's blocks inside a page-width flex column wrapper with spacing/color settings.
- 🐛 **content_direction select is unreachable and only lists 'column'** `[low | moderate | low]`  
  The 'content_direction' setting (lines 40-51) has only one option ('column') and is gated by visible_if '{{ section.settings.gap < 0 }}', but gap (lines 52-61) has min 0, so the condition can never be true and the control is permanently hidden. Either remove the dead setting or fix the visible_if so the control is actually usable; as-is it's confusing dead schema. Removing it is safe since it has a single default that matches current rendered output.

### `sections/marquee.liquid` — Scrolling marquee web component (marquee-component) that repeats text/icon/logo blocks with animation respecting prefers-reduced-motion.
- 🐛 **movement_direction option labels are swapped** `[medium | moderate | medium]`  
  In the schema (lines 116-125) value 'reverse' is labelled 't:options.forward' and value 'normal' is labelled 't:options.reverse', so the merchant-facing labels are inverted relative to the CSS animation direction fed to --marquee-direction. A merchant selecting 'Forward' gets reverse motion. Swap the label keys so the control matches actual scroll direction.
- 💸 **Add pause-on-hover / speed control for trust-logo marquees** `[medium | additive | medium]`  
  data-speed-factor is hardcoded to 25 (line 17) and there is no way to slow or pause the marquee. For a wholesale trust strip of certification/brand logos, add an additive 'animation_speed' range setting (mapping to data-speed-factor, default 25 to preserve behavior) and/or a default-off 'pause_on_hover' checkbox so buyers can read partner/accreditation logos. Default values keep current motion unchanged.

### `sections/media-with-content.liquid` — Editorial media + content split section with configurable media position/width/height, section width, and responsive grid layouts.
- 💸 **Presets point CTA to shopify://collections/all** `[medium | moderate | medium]`  
  The editorial preset button (lines 378-385) links to 'shopify://collections/all' with label 'shop_now_button_label'. For a B2B supplier this dumps buyers into an undifferentiated catalog. Consider updating the default preset link to a trade/best-sellers or wholesale-signup collection, or add a schema comment guiding merchants. Since this only affects newly-inserted presets (not existing placed sections), changing the preset link is low-risk but does alter default new-section output, so mark moderate.
- ⚡ **media_height fixed breakpoints may cause mobile CLS with 'auto'** `[low | additive | medium]`  
  The media height mapping (lines 21-32) sets --media-height-mobile: auto for the 'auto' option, which can produce layout shift as the media/image loads since no aspect ratio is reserved on mobile. An additive improvement: expose a mobile aspect-ratio hint or reserve space via a min-height custom property for the auto case so the _media block can avoid CLS. Keep default behavior when unset.

### `sections/password-footer.liquid` — Renders the storefront password-page footer with the 'powered by Shopify' mark, an 'enter using password' button, and the store-owner admin link.
- 🎛️ **Add opt-in wholesale contact/trade text setting** `[medium | additive | low]`  
  The footer only exposes a color_scheme setting. Add an optional rich-text setting (e.g. id 'trade_contact_html', default blank) rendered above .password-footer__links so House of Mouth can surface a 'Trade buyers: request wholesale access at sales@...' line to dental-practice visitors while the store is locked. Default-off keeps current output unchanged.

### `sections/password.liquid` — The main password-page section that renders theme/app blocks plus the shop.password_message inside a configurable layout/background/border wrapper.
- 🎛️ **Add opt-in B2B lead-capture headline/subtext block guidance** `[medium | additive | medium]`  
  Since this is a wholesale store behind a password wall, add an optional text setting (e.g. id 'wholesale_intro_html', default blank) rendered just before the {%- if shop.password_message != blank -%} block, letting the merchant show a 'Wholesale oral-care supplier — request trade access' message without editing shop.password_message. Default blank preserves current rendering.
- ♿ **Give the password-content region a semantic landmark** `[low | moderate | medium]`  
  The .password-content div (line 34) wraps merchant messaging with no landmark. Wrap it in a <section aria-label> or add role/aria so screen-reader users on the locked page get context; currently it is an anonymous centered div.

### `sections/predictive-search.liquid` — Renders the predictive-search dropdown: query suggestions, product/collection/page/article results, live-region status counts, no-results message, and single-result URL hint.
- 💸 **Surface a 'view all results' CTA and product count in the dropdown** `[medium | moderate | high]`  
  When search_results_count > 0 (line 62), the dropdown lists resources but offers no explicit link to the full search results page or a count summary. Adding an opt-in 'View all N results for {terms}' link (href to routes.search_url) after the product/collection blocks reduces friction for trade buyers scanning many SKUs; today only the hidden role=status announces the count.
- 💸 **Make the no-results state actionable for wholesale buyers** `[medium | moderate | high]`  
  The empty branch (line 101) only prints search_results_no_results text. For a B2B catalog, add an opt-in fallback CTA (e.g. 'Contact us / request this product') or link to the full collection so a dental buyer who mis-types a SKU is not dead-ended.

### `sections/product-hotspots.liquid` — 'Shop the look' style section overlaying clickable product hotspots on a background image, each opening a dialog/quick-add popover.
- 🎛️ **Expose hotspot trigger size as a merchant setting** `[low | additive | high]`  
  --hotspot-size is hardcoded to 36px in the inline style (line 33). Add an opt-in range setting (default 36) so the merchant can enlarge tap targets for trade-buyer product callouts; default 36 keeps current rendering identical.
- ♿ **Ensure hotspot dialogs are reachable on mobile where they are display:none** `[medium | moderate | high]`  
  The @media max-width:749px rule hides .hotspot .hotspot-dialog (line 93) and relies on the quick-add modal instead; verify the hotspot-trigger still exposes an accessible name/aria-label for the product on mobile, otherwise touch users get an unlabeled circular button. Add an aria-label fallback tied to the block's product title.
- ⚡ **Mark the hotspot background image as eager/high-priority to cut LCP** `[high | moderate | high]`  
  The background is rendered via {% render 'image', image: section.settings.image, class: 'hotspots__background-image' %} (line 50) with no loading/fetchpriority hint, so this large full-width hero-style image likely lazy-loads. Pass loading: 'eager' / fetchpriority: 'high' (opt-in) when the section is above the fold to reduce CLS/LCP; the placeholder_svg fallback needs no change.

### `sections/product-information.liquid` — Product page section that renders the media gallery, product details blocks, and an optional sticky add-to-cart bar via the sticky-add-to-cart web component.
- 💸 **Show case-pack/volume price hint in sticky bar** `[medium | additive | high]`  
  The sticky bar price (line 114, {% render 'price' %}) shows only the unit price. For B2B buyers add an opt-in setting (e.g. checkbox 'show_volume_note') that, when the selected variant has quantity_rule.min > 1 or quantity_price_breaks, renders a small 'from X / case' or 'min. qty N' note next to .sticky-add-to-cart__price, defaulting off so nothing changes for existing merchants.
- 🎛️ **Add sticky-bar color scheme setting** `[low | additive | high]`  
  The sticky bar hardcodes color-{{ settings.popover_color_scheme }} (line 41) while the section already defines a per-section 'color_scheme' setting (default scheme-1) that is unused by the bar. Add an opt-in 'sticky_color_scheme' select (default '' = inherit current popover behavior) so merchants can brand the sticky bar in navy/lime without touching global popover color.

### `sections/product-list.liquid` — Renders a configurable grid/carousel/editorial list of products from a chosen collection using _product-card static blocks and the resource-list snippet.
- 🐛 **Placeholder loop references undefined selected_product via injected BSS blocks** `[low | risky | high]`  
  In the empty-collection branch (lines 43-55) the vendor BSS 'Hide Price' snippets are injected referencing 'selected_product', which is never defined in this section (the loop variable is 'i'), so bss-lock-condition always evaluates against blank. This is vendor-injected markup; do not edit the BSS lines, but the merchant should be aware placeholder cards may render unlocked. Flag only, no theme change recommended.
- ⚡ **Expose image loading strategy for first row** `[low | additive | high]`  
  Cards are emitted via _product-card static blocks with image handling downstream, so the section itself has no eager/lazy control. Consider an additive setting to mark the first N cards' images eager (LCP) — but since loading is owned by _product-card-gallery, the safer change lives there, not here; leave this section unchanged.

### `sections/product-recommendations.liquid` — Renders JS-hydrated related/complementary product recommendations in a grid or carousel with a skeleton loading state.
- 🐛 **Carousel slide_count uses recommendations.products.size even on catalog fallback** `[medium | moderate | high]`  
  When recommendations return 0 and the code falls back to collections.all.products (lines 71-78), the built 'slides' array is derived from that fallback list, but the carousel is passed slide_count: recommendations.products.size (line 130 and 154), which is 0 in that case. This can make the carousel report zero slides while slides exist, breaking navigation/pagination. Change slide_count to slides.size (or products count) so it matches the rendered slides.
- ⚡ **Skeleton count should match max_products, not columns** `[low | moderate | high]`  
  The loading skeleton loops (1..section.settings.columns) at line 163, but the grid renders up to max_products items (line 85). When max_products (min 3) differs from columns the skeleton height mismatches the loaded grid, causing layout shift (CLS). Loop over max_products (capped by columns for the visible row) to better reserve space.

### `sections/quick-order-list.liquid` — B2B quick-order table letting buyers set quantities per variant with volume pricing, live totals, and bulk add/remove via the quick-order-list-component web component.
- 🐛 **Desktop non-discount price cell shows per-item formatting inconsistent with discount branch** `[medium | moderate | high]`  
  In the desktop price cell, the discounted branch outputs display_price_formatted (line 303) while the non-discount else branch outputs display_price_per_item_formatted (line 309). For variants without a discount this renders the 'per item' variant of the same price, which can differ in suffix/formatting from the discounted rows and the mobile branch (line 195 uses per_item consistently). Align both desktop branches to the same format-price type so the price column is consistent.
- 💸 **Add free-shipping / bulk-threshold progress note to totals bar** `[high | additive | high]`  
  The sticky totals bar (quick-order-list-total, lines 351-425) shows item count and product subtotal but no incentive. Add an opt-in setting (default off) to show a 'spend X more for free freight' or 'order N+ cases to unlock trade pricing' message near .quick-order-list-total__summary, driving larger wholesale orders without altering current output when disabled.
- 💸 **Enable image column by default consideration / SKU emphasis for trade buyers** `[medium | additive | high]`  
  show_image and show_sku both default false (schema lines 1130,1135). Trade buyers reorder by SKU; keep defaults but add a merchant hint, and optionally an additive 'show_barcode' or 'show_inventory_qty' toggle (default off) rendering variant.inventory_quantity next to the SKU so practices can gauge stock before bulk ordering.

### `sections/search-header.liquid` — Renders the search page header with a heading and the _search-input block, switching the heading between 'search' and 'search results'.
- 💸 **Show result count in the search heading** `[medium | additive | low]`  
  When search.performed is true the heading is a static 'search_results' string (lines 12-14). Add an opt-in setting to append search.results_count and search.terms (e.g. 'N results for "terms"'), giving buyers immediate feedback and improving perceived relevance; default off so current heading is unchanged.
- 🔎 **Heading hardcoded to h3 reduces page semantics** `[low | additive | low]`  
  The search page heading is captured as <h3> (line 17) inside the _heading block, so the primary search page has no h1/h2. Add an optional heading-level setting (default h3 to preserve current output) so merchants can promote it to h1 for SEO/accessibility on the dedicated search template.

### `sections/search-results.liquid` — Renders the storefront search results as a paginated product grid via the results-list web component, with an empty-state fallback collection.
- 💸 **Surface result count / query as a heading** `[medium | additive | medium]`  
  The template computes `search.results_count` (line 28) but only passes `title` to `product-grid` in the no-results branch; add an opt-in setting `show_results_count` that, when on, renders a visible '{{ search.results_count }} results for "{{ search.terms }}"' heading so B2B buyers confirm they searched the right SKU/brand. Default off keeps current output.
- 🎛️ **Expose products_per_page for infinite-scroll batch size** `[low | additive | medium]`  
  `products_per_page` is hardcoded to 24 (line 20) whenever `enable_infinite_scroll` is true, ignoring the `products_per_page` range setting which is only read when infinite scroll is off (line 23). Add an additive setting (e.g. `infinite_scroll_batch`) used on line 20 so the merchant can tune the initial/appended batch for large wholesale catalogs without changing the default 24.

### `sections/section-rendering-product-card.liquid` — Lightweight product-card markup rendered only by the Section Rendering API so variant-picker.js can morph price/SKU/swatches after client-side variant changes.
- ♿ **Give the morph-target card link an accessible name** `[low | moderate | high]`  
  The `<a href="{{ url }}" ref="productCardLink">` (lines 41-44) wraps only variant-picker JSON, price and SKU with no text or aria-label, so when this fragment is swapped in the link can be announced as unlabeled. Add `aria-label="{{ product.title | escape }}"` (and matching it to the full card) to preserve the accessible name after morphing. Verify against variant-picker.js expectations before shipping given this fragment is a JS morph contract.

### `sections/slideshow.liquid` — Full-frame or with-hints slideshow section driven by the slideshow web component and _slide blocks, with autoplay, arrows and pagination controls.
- 💸 **Allow autoplay in with-hints mode too** `[low | moderate | medium]`  
  `autoplay` is force-disabled unless `display_mode == 'full_frame'` (lines 2-6) and its setting is `visible_if` full_frame only (schema line 369); a wholesale promo carousel in with-hints layout can never rotate. Making autoplay available (opt-in, default still false) for with_hints would let the merchant rotate trade offers. Touches the render contract so treat as moderate.

### `sections/thom-b2b-home.liquid` — Self-contained B2B/wholesale homepage: utility bar, hero, value props, category grid, product rail, feature band, brand grid and account CTA, reading live collections/products.
- 💸 **Add price + case-pack signal to product rail cards** `[medium | additive | low]`  
  Rail cards (lines 85-90) show vendor, title and `product.price | money` but no stock or bulk cue; for a wholesale audience add an opt-in per-card badge using `product.metafields` or `product.available` (e.g. 'In stock' / 'Case pack') gated behind a `show_rail_stock` checkbox, reinforcing the 'Case-pack pricing' value prop already advertised on line 53.
- 🎛️ **Make the '$99' free-shipping threshold and utility-bar copy editable** `[medium | additive | low]`  
  The utility bar hardcodes 'Free shipping over $99', 'Dispatched from the Gold Coast' and 'Trade account login' (lines 14-18). Add schema text settings (e.g. `util_shipping`, `util_dispatch`, `util_account_label`) defaulting to the current strings so the merchant can change the threshold or wording without editing Liquid. Purely additive with matching defaults.
- ♿ **Decorative value-prop SVGs need aria-hidden** `[low | additive | low]`  
  The four inline value-prop icons (lines 40,44,48,52) are purely decorative but have no `aria-hidden="true"` / `role="img"`, so screen readers may announce raw SVG. Add `aria-hidden="true" focusable="false"` to each `.thb-ic` svg; the adjacent `.thb-vh`/`.thb-vt` text already conveys meaning.

### `sections/thom-b2b-mini-nav.liquid` — Custom compact circular-icon navigation strip for the B2B page, with per-link icon/image blocks and full color/spacing controls.
- 💸 **Optional per-link badge/caption for B2B signals** `[medium | additive | low]`  
  The link_item block only has label, link, icon, image. Add an optional default-blank 'badge' text setting (e.g. 'Case pack', 'Bulk', 'New') rendered as a small pill inside .thom-mininav__link when non-blank. Lets the merchant surface wholesale cues (volume/case-pack) directly on the nav circles without changing existing installs.
- ♿ **Custom nav images are hard-coded alt=""** `[low | additive | low]`  
  The .thom-mininav__img at line 58-65 always uses alt="". That is fine while the label span carries the accessible name, but if a merchant uses image-only tiles the label is still rendered, so it is acceptable; however add an optional block 'image_alt' setting (default blank -> falls back to label) so decorative-vs-informative can be controlled. Additive.
- ⚡ **Work Sans font loaded via render-blocking stylesheet link** `[low | additive | low]`  
  Lines 16-20 inject a blocking <link rel="stylesheet"> to Google Fonts when load_work_sans is on (default true). Add display=swap is already present, but consider preloading only; at minimum the info text already warns to disable if the theme loads Work Sans. Low-value, leave default but document. No change strictly needed.


## Snippets

### `snippets/accordion-custom-component.liquid` — Wrapper snippet that renders the <accordion-custom> web component with co-located open/disable/animation data-attributes and details-content transition styles.
- ♿ **class attribute is not HTML-escaped** `[low | moderate | high]`  
  Line 32 outputs class="{{ class }}" with no escape filter. Since callers pass this string, an escape/strip would harden it; but as this is a theme-internal contract consumed by many render calls and the web component reads these attributes, treat as low-priority hardening only. Do not change the data-* contract.

### `snippets/add-to-cart-button.liquid` — Renders the primary Add to cart <button> inside the <add-to-cart-component> web component with dynamic text, disabled state, cart icon and checkmark-burst feedback.
- ♿ **Disabled button gives no reason to assistive tech** `[medium | moderate | high]`  
  At lines 38-40 the button is `disabled` via {% unless can_add_to_cart %} but there is no aria description of why (sold out / unavailable). Add an optional aria-describedby or reuse add_to_cart_text (which typically becomes 'Sold out') so screen-reader users understand state. Because ref="addToCartButton"/on:click and name="add" are JS/web-component contracts, any change must stay additive and not alter those hooks.

### `snippets/background-media.liquid` — Renders a responsive background image or autoplay/muted/looped background video (with placeholder fallback) for hero/section/block backgrounds, computing sizes from page-width settings.
- 🐛 **Video branch ignores page-width sizing, hard-codes 100vw + wrong base width** `[low | moderate | medium]`  
  Lines 25-35 always set sizes to 100vw and image_tag width:1100 against a 3840 source; the image branch instead derives sizes from settings.page_width. For non-full-width containers the video poster over-requests. Align the video branch sizes calc with the image branch, or at least document. Moderate risk since it changes the srcset sizes attr.
- ♿ **Background <video> has no accessible label/description** `[low | additive | medium]`  
  The <video> at lines 36-42 is decorative but has no aria-hidden. Add aria-hidden="true" (and it already lacks controls) so screen readers skip it; purely decorative background media should be hidden from the a11y tree. Additive.
- ⚡ **Background video preview image not eager/high-priority above the fold** `[medium | moderate | medium]`  
  The video branch (lines 31-35) renders background_video.preview_image via image_tag with no loading/fetchpriority, unlike the image branch which sets fetchpriority:high and loading:eager for section.index==1/<=3 (lines 96-110). For a first-section hero video, the poster frame will lazy-load and delay LCP. Mirror the section.index logic to set loading/fetchpriority on the video preview image_tag. Changes output but low risk.

### `snippets/bento-grid.liquid` — Positions an array of HTML item strings into a 12-cell CSS-grid bento layout, opening a new box every 11-12 items with per-count grid-template-area overrides.
- 🐛 **Missing grid-template-areas for 3, 6, 9 item counts** `[medium | moderate | medium]`  
  The overrides block (lines 166-224) defines bento-box--items- for 1,2,4,5,7,8,10,11 but not 3, 6, or 9. When box_item_count is one of those, the class bento-box--items-3/6/9 falls through to the base 12-area template ('A A A B B B ...'), leaving empty/misplaced grid areas for the final partial box. Add explicit grid-template-areas for items-3, items-6, items-9 so partial boxes fill correctly.

### `snippets/blog-comment-form.liquid` — Renders the comment form (author/email/body inputs, error/success messaging, submit) for a blog post.
- 🐛 **Textarea renders leading/trailing whitespace as its value** `[medium | moderate | low]`  
  The <textarea> on lines 123-137 wraps `{{- form.body -}}` but the opening/closing tags sit on separate indented lines, so on a validation re-render the textarea's value gains leading whitespace/newlines. Collapse to `>{{- form.body -}}</textarea>` on tighter lines (the `-` trims help but the block indentation before `{{- form.body -}}` and after can still leak). Verify the field starts empty on first load.
- 🎛️ **No schema toggle to require/label the comment CTA text** `[low | additive | low]`  
  Submit button label and heading come only from locale strings ('blogs.comment_form.post', 'blogs.comment_form.heading'). Since this is a shared snippet, no schema is possible here, but the empty `class=""` on line 24 should be given a real BEM class (e.g. `blog-post-comments__form-errors`) so the merchant can style the error list; currently it is unstyleable.
- ♿ **General error list not linked to any field via aria-describedby** `[medium | moderate | low]`  
  The error `<ul class="">` (line 24) only builds spans with IDs `blog-post-comment-author-error`/`-body-error`, but the email branch has no matching span id, so the input on lines 96-112 references `aria-describedby="blog-post-comment-email-error"` (line 110) that never exists. Add an `id="blog-post-comment-email-error"` span in the email error branch (mirror author/body) so screen readers announce the email error.

### `snippets/border-override.liquid` — Emits inline CSS custom properties (--border-width/style/color/radius plus overflow:hidden) from global border settings for a card/element.
- 🐛 **border_opacity of 0 silently yields a valid-but-invisible border while overflow still clips** `[medium | risky | medium]`  
  Line 6 computes `--border-color: rgb(... / {{ settings.border_opacity | divided_by: 100.0 }})`. When border_width is 0 this snippet still emits `overflow: hidden` (line 7 only guards on border_radius), which can clip child badges/quick-add on cards that call it (e.g. card-gallery). Consider guarding the overflow/radius output so a merchant with radius 0 but expecting overflow visible is not surprised — but treat as investigate-only since many cards rely on the clip.
- 🎛️ **No per-block border override; always inherits global settings** `[low | additive | medium]`  
  Every value is read from `settings.*` (global theme settings) with no way for a caller to pass an override map, so a merchant cannot give one card a different radius. Add optional params (e.g. `border_radius` param defaulting to `settings.border_radius`) so callers like card-gallery can opt into a local value without changing the global default.

### `snippets/button.liquid` — Renders a link styled as a button (with size-style, style_class, optional new-tab) for button-like blocks.
- 🐛 **Stylesheet targets `.link` but the anchor never gets that class** `[low | moderate | medium]`  
  The {% stylesheet %} (lines 36-45) styles `.link` hover, yet the `<a>` class list (lines 20-24) only outputs `size-style` and `block_settings.style_class` — so this hover rule is dead unless style_class happens to equal `link`. Either add `link` to the class list or remove the unused rule to cut CSS weight.
- ♿ **Disabled link (blank href) has no accessible name when label is empty** `[low | moderate | medium]`  
  Lines 13-33: when `link == blank` the element becomes `role="link" aria-disabled="true"` but its only text is `{{ block_settings.label }}` (line 32); if label is also blank the control is an empty, focus-reachable link. Add `{% if block_settings.label == blank %}aria-label="..."{% endif %}` or skip rendering when both link and label are blank.

### `snippets/buy-buttons-styles.liquid` — Consolidated CSS for the product buy-buttons area: form layout, quantity selector/rules, volume (case-pack) pricing table, pickup availability, and accelerated checkout.
- 💸 **Volume/case-pack pricing table lacks visual emphasis for B2B bulk buyers** `[high | additive | medium]`  
  `.volume-pricing__row` (lines 245-252) only zebra-stripes via `--even/--odd` and the title (`.volume-pricing__title`, 233-239) uses body weight. For a wholesale audience the tiered price break is the key decision driver; add opt-in emphasis (e.g. a `.volume-pricing__row--best` accent using brand lime `--color-primary` or bolder price cell) so the largest case-pack saving stands out. Keep default styling unchanged and gate new rules behind a new class only.
- 🎛️ **Buy-button preferred width is hard-coded, not a themeable token** `[low | additive | medium]`  
  `--buy-button-preferred-width: 185px` is fixed inline on `.buy-buttons-block` (line 10). Expose it via an existing theme token/setting fallback (e.g. `var(--buy-button-preferred-width-setting, 185px)`) so a merchant can widen the Add-to-cart/quantity row for trade layouts without editing the snippet.

### `snippets/card-gallery.liquid` — Renders a product card's image slideshow with badge-aware padding, variant-image handling, eager/lazy loading, and placeholder states.
- 💸 **No low-stock / bulk-availability signal surfaced on the card for trade buyers** `[medium | additive | high]`  
  Badge logic (lines 17-24) only flags sold-out or compare-at (sale). Wholesale buyers care about in-stock depth; add an opt-in setting (default off) to expose a low-stock/`inventory_quantity` badge via the existing `has_badges`/`badge_position` mechanism so merchants can flag limited case-pack availability without altering current output.
- ♿ **Card gallery link aria-label duplicates the product title link elsewhere on the card** `[medium | moderate | high]`  
  The wrapping `<a class="contents" aria-label="{{ product.title }}">` (lines 160-165) is typically rendered alongside a separate title/price link in the card, producing two adjacent links with the same accessible name to the same URL. Consider `aria-hidden="true"` + `tabindex="-1"` on this image link (mirroring the placeholder title link on line 182) so keyboard/AT users get one meaningful link per card.
- ⚡ **Eager-loading condition can force eager on every card in the first 5 sections** `[high | risky | high]`  
  Line 137: `if forloop.first and section.index == null or section.index < 5` — Liquid evaluates `and`/`or` left-to-right with no grouping, so `section.index < 5` alone makes ALL first images (not just the first card) eager for sections indexed 0-4, hurting LCP/bandwidth on long grids. Wrap intent explicitly, e.g. `if forloop.first and (section.index == null or section.index < 5)`, so only above-the-fold first images load eagerly.

### `snippets/cart-bubble.liquid` — Renders the header cart count bubble with an item-count badge that clamps to 99+.
- 🐛 **99+ overflow count is never rendered** `[medium | moderate | medium]`  
  The @param docs and the data-maintain-ratio logic imply a '99+' display when cart.item_count > limit, but the text span only outputs {{ cart.item_count }} when `limit == blank or cart.item_count < limit`, so when the count exceeds the limit the bubble renders EMPTY instead of '99+'. Add an {% else %} branch outputting '99+' (or limit | append: '+') so high-volume wholesale carts still show a count.
- ♿ **Cart count has no accessible text for screen readers** `[medium | additive | medium]`  
  The count span carries aria-hidden="true" and there is no visually-hidden equivalent (e.g. 'accessibility.cart_count' | t), so assistive tech announces nothing for the number of items. Add a visually-hidden localized label inside #cart-bubble-text describing the item count.

### `snippets/cart-items-component.liquid` — Wraps cart line-item markup in the <cart-items-component> custom element and defines cart view-transition animations.
- ♿ **Blur/scale view-transition animations ignore reduced-motion for cart page** `[low | moderate | high]`  
  The page-content (cart-page-content-old blur) and drawer animations only assign view-transition-name inside prefers-reduced-motion: no-preference, but the ::view-transition-old(cart-page-content) blur keyframe itself is declared unconditionally; wrap the ::view-transition-* animation rules in a prefers-reduced-motion guard so motion-sensitive B2B buyers aren't shown blur transitions.

### `snippets/cart-products.liquid` — Renders the cart line-item table (media, title, variants, unit price, quantity selector, volume pricing, remove) or the empty-cart state.
- 💸 **No free-shipping / order-minimum progress or bulk reorder cue in cart** `[high | additive | high]`  
  For a wholesale audience the line-item table (.cart-items__table) shows no free-shipping threshold progress bar or minimum-order-value indicator, both high-leverage AOV drivers. Add an opt-in block/schema setting (e.g. block_settings.free_shipping_threshold) rendering a localized progress message above the table, default off so existing carts are unchanged.
- ♿ **Cart product thumbnail has no alt text** `[medium | additive | high]`  
  The image_tag on line 159 omits `alt`, so the linked product thumbnail is announced with a filename or nothing; add alt: item.product.title (or item.image.alt | default: item.product.title) to the image_tag filter.
- ⚡ **Cart thumbnail image lacks width/height and responsive srcset** `[medium | additive | high]`  
  Line 159 renders `item.image | image_url: width: 250 | image_tag` with only class/style, no widths/sizes/loading attributes; supplying widths and sizes (the container is clamp(2.5rem,15cqi,7.5rem)) plus width/height would cut CLS and bytes on long wholesale carts. This is additive to the image_tag call.

### `snippets/cart-summary.liquid` — Renders cart totals (subtotal, discounts, estimated total), the note and discount-code accordions, tax note, and checkout/accelerated-checkout CTAs.
- 💸 **Checkout CTA is generic 'Checkout' with no trust/security or B2B reassurance** `[medium | additive | high]`  
  The .cart__checkout-button only shows 'content.checkout' | t. Add an opt-in schema setting for a short reassurance line beneath the CTA (e.g. secure-checkout / net-terms / GST-invoice note relevant to trade buyers), default blank so nothing renders unless the merchant fills it in.
- 🎛️ **Cart note label hardcoded to 'seller_note' translation** `[low | additive | high]`  
  The note summary uses 'content.seller_note' | t (lines 58/72) with no merchant override; for wholesale a 'PO number / delivery instructions' prompt is common. Add an optional block/section setting for custom note label text falling back to the current translation, keeping default behavior identical.

### `snippets/checkbox.liquid` — Reusable checkbox input + styled label snippet with support for refs, form association, required/disabled/autofocus states.
- ♿ **Injected label/value/name attributes are not HTML-escaped** `[low | additive | medium]`  
  name, value and data-label render raw {{ label }}/{{ value }}/{{ name }} (lines 23,24,27,55) with no | escape; any caller passing text containing quotes or markup could break the attribute or inject HTML. Add | escape to the attribute interpolations to harden the shared snippet without changing normal output.

### `snippets/collection-card.liquid` — Renders a collection card (image + content) for collection-list sections, supporting bento/editorial layouts and on-image placement.
- 💸 **No product-count signal on collection cards** `[medium | additive | medium]`  
  For a B2B/wholesale catalogue, buyers benefit from knowing category depth. Add an opt-in block setting (e.g. show_product_count, default false) that renders {{ collection.products_count }} inside .collection-card__content (e.g. '{{ count }} products') so merchants can surface range size on tiles like 'Toothpaste' or 'Interdental' without changing the current default.
- ♿ **Card link exposes only collection.title, no context** `[low | moderate | medium]`  
  The .collection-card__link <a> wraps a .visually-hidden span containing just {{ collection.title }}. Screen-reader users hear the title but nothing indicating it links to a collection listing. Add descriptive text, e.g. a translated 'Shop {{ collection.title }}' or an aria-label on the anchor, so the link purpose is unambiguous (WCAG 2.4.4).

### `snippets/color-schemes.liquid` — Emits :root plus per-scheme CSS custom-property blocks (colors, opacities, button/variant/input tokens, shadows) for every theme color scheme.
- 🐛 **Opacity accumulators from dark schemes leak into later light schemes** `[low | risky | high]`  
  opacity_5_15 / opacity_10_25 / opacity_35_55 / opacity_40_60 / opacity_30_60 are set inside the if/else so they are always reassigned per iteration, which is fine — but --opacity-10, --opacity-15, --opacity-50, --opacity-60 referenced on lines 79-82 are NOT defined in this snippet, so those derived tokens depend on them existing globally. If a scheme is added without those globals present, --color-foreground-muted etc. resolve to invalid values. Confirm --opacity-10/15/50/60 are defined upstream (e.g. base variables) and document the dependency to avoid silent breakage.

### `snippets/divider.liquid` — Renders a horizontal (or full-width) divider line with configurable alignment, thickness, corner radius and width percentage.
- ♿ **Decorative divider is not marked as such for assistive tech** `[low | moderate | low]`  
  The .divider container carries visual-only meaning but no role. Add role="separator" and aria-orientation="horizontal" (or aria-hidden="true" if purely decorative) to the outer <div class="divider ..."> so screen readers announce or skip it appropriately instead of treating the empty span as content.

### `snippets/editorial-blog-grid.liquid` — Places blog list items into a 12-column editorial (mirrored) CSS grid with fixed per-item column/row spans, collapsing to a stacked flex layout under 750px.
- 🐛 **Item class cycles every 4 but grid layout cycles every 8, so items 4-7 reuse mobile styles of 0-3** `[low | moderate | medium]`  
  The wrapper uses class editorial-blog__item-{{ forloop.index0 | modulo: 4 }} (0-3) while grid_column/grid_row are computed on modulo 8. On mobile (<=749px) items 4-7 fall back to the item-0..3 width/align rules, which do not match their intended editorial variety; e.g. the 5th post (index 4) gets item-0's 66% width. Cycle the mobile class on modulo 8 (item-0..7) with matching rules, or document that mobile intentionally repeats the 4-pattern, to avoid layout mismatch on long blogs.
- ♿ **Empty spacer div adds noise to accessibility tree** `[low | additive | medium]`  
  .editorial-blog__spacer is a purely visual aspect-ratio spacer; add aria-hidden="true" so it is not exposed to assistive tech as an empty element.

### `snippets/editorial-collection-grid.liquid` — Places collection/resource-list items into a 12-column editorial CSS grid with fixed per-item spans, collapsing to a stacked flex layout under 750px.
- 🐛 **Item class modulo 4 vs layout modulo 8 mismatch on mobile for items 4-7** `[low | moderate | medium]`  
  Same defect as editorial-blog-grid: wrapper class uses editorial-collection__item-{{ forloop.index0 | modulo: 4 }} (0-3) but grid positions cycle on modulo 8. Mobile rules (<=749px) only define item-0..3 widths/aspect-ratios, so the 5th-8th collections reuse the first four's mobile styling rather than their own editorial variation. Extend mobile classes to modulo 8 with matching aspect-ratios or document the intentional repeat.
- ♿ **Decorative spacer not hidden from assistive tech** `[low | additive | medium]`  
  .editorial-collection__spacer is a layout-only element; add aria-hidden="true" to keep it out of the accessibility tree.

### `snippets/editorial-product-grid.liquid` — Renders a fixed 8-item editorial masonry grid of pre-rendered product card HTML strings using explicit grid-column/grid-row placement.
- 🎛️ **Hard-coded gap has no merchant control** `[low | additive | medium]`  
  Grid gap is fixed to `var(--gap-xl)` desktop / `var(--gap-2xl)` mobile (lines 70, 96). Since this snippet already ingests pre-rendered items, expose an optional `gap` param defaulting to the current token (e.g. `gap: gap | default: 'var(--gap-xl)'`) so the calling section can pass a merchant-configured spacing without changing the default render.
- ♿ **Decorative spacer not hidden from AT** `[low | additive | medium]`  
  The `.editorial-product__spacer` div (line 11) is a purely visual aspect-ratio placeholder but is exposed in the DOM/accessibility tree. Add `aria-hidden="true"` (and it is already display:none on mobile) so screen readers and the layout tree skip it.

### `snippets/filter-remove-buttons.liquid` — Renders the active-filter 'remove' pills (including price-range pill and clear-all) for the collection/search facet UI.
- 🐛 **Operator-precedence bug in price_range condition** `[medium | risky | high]`  
  Line 18 `if filter.type == 'price_range' and filter.min_value.value != null or filter.max_value.value != null` relies on and/or precedence. In Liquid and/or are right-associative with no precedence, so this parses as `type=='price_range' and (min!=null or max!=null)` only by luck of ordering — but any filter whose iteration reaches here can mis-evaluate. Wrap explicitly: compute `assign is_price = filter.type == 'price_range'` and guard the branch on `is_price` alone, then check min/max inside. Prevents a non-price filter with a stray max_value from rendering a malformed money-range pill.
- ♿ **role=button pills missing aria-label / keyboard activation semantics** `[low | additive | high]`  
  The `<facet-remove-component>` pills (lines 22-33, 55-67) use `role="button"` and `tabindex="0"` but their accessible name is only the filter text; the visually-hidden `actions.remove` span helps, but there is no `aria-label` combining the two, so AT announces e.g. 'Whitening remove' ambiguously. Add `aria-label="{{ 'actions.remove' | t }} {{ value.label | escape }}"` on the component for a clear 'Remove Whitening' announcement.

### `snippets/fonts.liquid` — Emits <link rel=preload as=font> tags for the four configured non-system theme fonts (body, subheading, heading, accent).
- ⚡ **fetchpriority=low on preloaded fonts undercuts the preload** `[medium | moderate | medium]`  
  All four preload links (lines 9, 19, 29, 39) set `fetchpriority="low"`, which contradicts the purpose of preloading render-blocking font files and can delay first paint / worsen CLS from font swap. The body/heading fonts in particular are above-the-fold; consider omitting fetchpriority (browser default 'auto' for preloaded fonts is higher) or gating low priority only to the accent font. Verify against theme design intent before shipping.

### `snippets/format-price.liquid` — Formats a price via money/money_with_currency (respecting currency-code setting) and optionally appends the per-item suffix.
- 💸 **Per-item suffix has no separating space, hurting B2B price scanability** `[low | moderate | medium]`  
  For `type: 'per_item'` the snippet appends `content.quantity_per_item` (locale value '/ea') directly to the money string (line 27), rendering '$4.50/ea'. For a wholesale/case-pack audience scanning per-unit pricing in quick-order-list, a thin space or leading space (e.g. append ' ' before the suffix, or update the locale string to ' /ea') improves legibility of the unit price without changing the numeric value.

### `snippets/gap-style.liquid` — Outputs a responsive CSS custom property (default --gap) that scales a pixel gap value above a threshold via --gap-scale.
- 🐛 **Doc default for scale_min mismatches actual default** `[low | additive | low]`  
  The doc comment states `@param {number} [scale_min] ... Default: 20` (line 7) but the code sets `assign min = scale_min | default: 24` (line 15). This mis-documents the threshold and could lead a merchant/dev to expect scaling to start at 20px. Align the doc to 24 (or the code to 20) so callers reason correctly about when max()/scaling kicks in.

### `snippets/gift-card-recipient-form.liquid` — Renders the gift-card recipient delivery form (send-to radio, recipient email/name/message/send-on fields with error slots) wired to the gift-card-recipient-form web component.
- 🐛 **aria-controls references non-unique id 'recipient-fields'** `[medium | moderate | medium]`  
  Both radio inputs set aria-controls="recipient-fields" (lines 36,48) but the controlled div (ref="recipientFields", line 54) has no id="recipient-fields" attribute, so the ARIA relationship is broken. Add id="recipient-fields-{{ block.id }}" to the recipientFields div and reference that unique id, since multiple gift-card blocks can render on one page (collection quick-add).
- 🐛 **maxlength/aria-label say 200 but textarea allows more via mismatched var** `[low | moderate | medium]`  
  max_chars_message is assigned 200 and used for the textarea maxlength="{{ max_chars_message }}", the aria-label max_chars, and the character-count data-max. This is consistent, but the visible message label 'content.recipient_form_message' is reused as the placeholder AND as aria-label prefix (message_label_rendered), so the accessible name duplicates the placeholder text. Give the textarea a distinct visible label rather than relabeling with the same placeholder string to avoid a redundant/confusing accessible name.
- ♿ **Send-on label uses a <div for=...> which is not a real label association** `[medium | moderate | medium]`  
  The 'Send on' label is a <div for="Recipient-send-on-{{ block.id }}"> (line 176); the for attribute is invalid on a div so the date input has no programmatic label. Change it to a <label for=...> element so screen readers announce the field name.

### `snippets/grid-density-controls.liquid` — Renders mobile/desktop grid density radio toggles (single vs default, default vs zoom-out) that drive the results-list web component layout.
- 🎛️ **No merchant control to hide density controls per template** `[low | additive | medium]`  
  The controls always render for whichever viewport is passed with no schema-level opt-out. Since this is a snippet with no schema, expose an optional param like show_grid_density (default true) checked before the wrapper so callers/sections can suppress the toggle for B2B collection layouts where a fixed dense grid is preferred, without changing current default behavior.

### `snippets/group.liquid` — Renders the shared group block wrapper (background media, overlay, optional link, layout/spacing/border/size styles) that all group-extending theme blocks use.
- ♿ **Full-block overlay link has no accessible name** `[medium | additive | high]`  
  When settings.link is set, an empty <a class="group-block__link"> covers the whole block (lines 35-41) with no text, aria-label, or title, so it is announced as an unnamed link by screen readers. Add an optional settings.link_aria_label (default blank) and emit aria-label when present, or fall back to the block's heading text, keeping current output unchanged when unset.

### `snippets/header-actions.liquid` — Renders header account button, cart icon/bubble, and cart drawer (or cart link) with the header-actions and cart-drawer web components plus all related styles.
- 💸 **No free-shipping / bulk-order threshold cue in cart drawer for wholesale buyers** `[high | additive | high]`  
  The cart-drawer summary (line 234, {% render 'cart-summary' %}) shows totals but no progress cue toward a free-shipping or minimum-order threshold, a key B2B conversion lever. Add an optional section setting (e.g. free_shipping_threshold, default 0/off) and render a progress message above cart-summary only when set, so trade buyers see how far they are from qualifying without altering default output.
- ♿ **Cart count live region duplicated between cart-drawer and header-actions** `[low | risky | high]`  
  There are two role="status" live regions with ref="liveRegion" (cart-drawer dialog line 183-187 and header-actions line 254-259); overlapping assertive/status regions can cause double or conflicting announcements of cart updates. Confirm header-actions.js/cart-drawer.js target distinct refs and consider consolidating to one cart-count live region to avoid duplicate screen-reader announcements.
- ⚡ **Account SVG icon is inline-duplicated instead of using inline_asset_content** `[low | moderate | high]`  
  The account icon is a hand-written inline <svg> captured in account_icon (lines 42-57) while the cart/close icons use {{ 'icon-x.svg' | inline_asset_content }}. Moving the account glyph to a shared asset referenced via inline_asset_content would keep icon styling consistent and reduce duplicated markup, though savings are minor.

### `snippets/header-drawer.liquid` — Renders the mobile/desktop header menu drawer web component with multi-level nav, localization, and featured collection/product content.
- 🎛️ **No merchant control over featured-content item count** `[low | additive | high]`  
  max_featured_items is hardcoded to 4 (line 16) for both featured_collections and featured_products modes. Add an opt-in block.settings number setting (default 4) so the merchant can tune how many trade collections/products surface in the drawer without a code change; wire it in via `assign max_featured_items = block_settings.drawer_featured_count | default: 4`.
- ♿ **Localization details summary aria-expanded is hardcoded** `[medium | risky | high]`  
  The drawer-localization <summary> (line 620) sets aria-expanded="false" statically and never updates it as the details toggles open, so screen-reader users are always told the region/language submenu is collapsed. Add an opt-in fix by having the header-drawer web component sync aria-expanded on the on:toggle handler (line 618 details already has on:toggle="/toggle"), or move aria-expanded onto the details-driven state.
- ⚡ **Featured resource-card images lack explicit fetch/loading priority** `[low | additive | high]`  
  The featured-content resource-card renders (e.g. lines 526-532, 733-739) pass image_sizes: 'auto, 400px' but no loading/fetchpriority hint; since this drawer content is off-screen until opened, expose an opt-in block setting (e.g. drawer_featured_lazy default true) that passes loading: 'lazy' to resource-card to cut initial payload without changing current default behavior.

### `snippets/icon-or-image.liquid` — Renders either an inline SVG icon or a responsive uploaded image based on block settings.
- ♿ **Uploaded image renders with no meaningful alt text** `[medium | additive | medium]`  
  In the image_upload branch (lines 35-39) image_tag is called without an alt attribute, so the <img> falls back to an empty/auto alt and conveys nothing to screen readers. Add an optional `alt` doc param and pass `alt: alt` (defaulting to image_upload.alt) to image_tag so merchants can describe brand/trust logos; this is backward-compatible since alt currently is absent.
- ⚡ **Uploaded image has no loading/fetchpriority control** `[low | additive | medium]`  
  The image_tag call (lines 35-39) omits loading and fetchpriority, so behavior depends on browser defaults and can cause CLS or eager loading of below-fold logos. Add optional `loading` and `fetchpriority` doc params passed through to image_tag (no default change) so icon/logo blocks placed low on the page can opt into loading:'lazy'.

### `snippets/icon.liquid` — Large case/when dictionary returning inline SVG path markup for a named icon, rendered inside an <svg> wrapper.
- 🐛 **No fallback case for unknown icon names** `[low | additive | medium]`  
  The {%- case icon -%} block (line 9) ends at {%- endcase -%} (line 405) with no {%- else -%} branch, so an unrecognized or misspelled icon name yields an empty <svg> from icon-or-image.liquid (a blank box) with no diagnostic. Add an else that renders nothing visible but is safe, or a generic placeholder path, so a bad icon setting degrades gracefully instead of shipping an empty icon.

### `snippets/image.liquid` — Renders a responsive <img> for a given image object with 1x/2x/3x height-based srcset and alt fallbacks, or a text fallback when blank.
- ⚡ **srcset uses only DPR (1x/2x/3x) with no width descriptors or sizes** `[medium | additive | medium]`  
  image_srcset (lines 21-23) emits only pixel-density candidates at a single height and image_tag (lines 31-35) sets no sizes attribute, so the browser cannot pick a smaller source on small viewports; on a B2B catalog with many thumbnails this over-serves bytes. Add optional `widths`/`sizes` doc params that, when provided, generate a width-descriptor srcset — keep the current DPR path as the default so nothing changes for existing callers.
- ⚡ **No loading/fetchpriority passthrough causes potential eager loading and CLS** `[medium | additive | medium]`  
  image_tag (lines 31-35) never sets loading, fetchpriority, width, or height, so images can load eagerly and shift layout as they arrive. Add optional `loading`, `fetchpriority`, `width`, and `height` doc params passed through to image_tag (all default off/unset) so callers can mark below-fold images loading:'lazy' and reserve space to avoid CLS without altering current output.

### `snippets/jumbo-text.liquid` — Renders large full-width display text as a <jumbo-text> web component with optional per-line blur/reveal animation.
- ♿ **Expose animation as opt-out setting for reduced-motion beyond media query** `[low | additive | high]`  
  The blur/reveal effects (data-text-effect on line 73) only respect prefers-reduced-motion via CSS. Add a new default-off block setting like `disable_animation` that, when true, omits the data-text-effect/data-animation-repeat attributes so merchants can force static text without touching motion prefs. Purely additive.
- ⚡ **Duplicate <script> injection when block used multiple times** `[low | moderate | high]`  
  Line 92-96 emits `<script src='jumbo-text.js' type='module' async>` inline every render, so a page with several jumbo-text blocks (e.g. multiple headings) injects the module tag repeatedly. Wrap it so it only prints once (e.g. guard with a page-scoped variable) to avoid redundant tags; low risk but improves head cleanliness.

### `snippets/link-featured-image.liquid` — Renders a menu/mega-menu item's featured image for collection, collections, catalog, and product-derived links.
- 🐛 **catalog_link branch renders image_tag with no nil guard** `[medium | moderate | medium]`  
  In the `catalog_link` branch (lines 41-46) `product_object` comes from `collections.all.products | where: 'featured_image' | first` but the following `image_tag` is rendered unconditionally, unlike the other branches which guard on `.featured_image`. If `collections.all` has no product with a featured image, `product_object.featured_image` is nil and image_tag emits an empty/broken img. Wrap it in `{% if product_object.featured_image %}` like the sibling branches.
- ♿ **Menu featured images render with no alt text** `[low | additive | medium]`  
  None of the image_tag calls pass an `alt` (lines 19,27,37,45). For a B2B menu with category imagery, pass `alt: link.title` (or the collection title) so screen readers announce the category the thumbnail represents rather than a filename; additive attribute only.
- ⚡ **Fixed width:800 with no srcset/widths risks oversized menu thumbnails and CLS** `[medium | moderate | medium]`  
  All four branches call `image_url: width: 800 | image_tag: loading:'lazy', class:class, sizes:image_sizes` (e.g. lines 16-19) but never pass `widths:` so no srcset is generated for these small mega-menu thumbnails; add `widths: '200,400,600,800'` and a real `width`/`height` (or aspect via the image object) so smaller devices download smaller files and layout is stable.

### `snippets/list-filter.liquid` — Renders a single collection filter facet (list, pill, swatch, or image) inside a details/accordion with show-more and clear controls, wired to facet web components.
- 💸 **Surface result counts next to each filter value for wholesale buyers** `[medium | additive | high]`  
  The loop already has `value.count` available (used only to compute `is_disabled` on lines 205-207). For a B2B catalog, showing the count in the label (e.g. an opt-in `show_facet_counts` setting that appends `({{ value.count }})` to pill/checkbox/swatch labels) helps trade buyers gauge inventory breadth before clicking. Gate behind a new default-off setting so existing themes are unchanged.
- ♿ **Image-facet uses empty alt while relying on adjacent label** `[low | moderate | high]`  
  Line 238 sets `alt: ''` making the image decorative; that is acceptable because the visible `.facets__image-label` (line 272) carries `value.label`, but the surrounding `<fieldset>` aria-label (line 233) already duplicates it — verify no double announcement. Low priority; if kept decorative, no change needed, otherwise ensure the label is programmatically associated.
- ⚡ **Image-facet thumbnails use width:300 without srcset** `[low | moderate | high]`  
  Line 238 renders `value.image | image_url: width: 300 | image_tag: alt: ''` with no `widths:`; grid columns cap at 125px so on standard-DPR mobile a 300px asset is oversized and on high-DPR desktop it may be soft. Add `widths: '125,250,375'` and explicit sizing to right-size these filter swatches.

### `snippets/localization-form.liquid` — Renders the country/region and language selector form (with searchable country list) for the <localization-form-component> web component.
- ♿ **Hardcoded English 'Country/Region' heading is not translated** `[low | moderate | high]`  
  Line 110 outputs the literal string `Country/Region` for the visually-hidden `<h2>`, unlike the language heading which uses `{{ 'content.language' | t }}` (line 246). Replace with a translation key (e.g. `'accessibility.country_region' | t`, which is already used on line 129) so non-English storefronts announce it correctly to screen readers.

### `snippets/media.liquid` — Renders a media block's image or video with color scheme, positioning, border and spacing styles inside _media/_media-without-appearance theme blocks.
- 🎛️ **No merchant control over mobile media height** `[low | additive | medium]`  
  The stylesheet hardcodes --media-height-mobile behavior (line 89 defaults to auto, and .media-block__media--video forces auto at max-width:749px). Expose an opt-in block setting (e.g. media_height_mobile range) that sets --media-height-mobile inline in the style attribute alongside --image-position, defaulting to unset so current auto behavior is preserved.
- ♿ **Linked media has no accessible label** `[medium | additive | medium]`  
  When block_settings.link is present the media_block__media-link <a> (line 55) wraps only the image with no text or aria-label; if the merchant's image alt is empty the link has no accessible name. Add an optional block_settings.link_aria_label schema string and render it as aria-label on the media-block__media-link anchor (fallback to the image alt), default blank so nothing changes for existing merchants.

### `snippets/mega-menu-list.liquid` — Builds the mega-menu column layout plus optional featured products/collections/collection-images content, including grid column math and BSS lock filtering.
- 💸 **Featured products in mega menu show no price/case-pack signal to trade buyers** `[medium | additive | high]`  
  The 'featured_products' branch (line 293) renders resource-card with image_hover but passes nothing to surface price or wholesale/case-pack info, so B2B buyers browsing the nav get no purchase signal. Add an opt-in boolean (e.g. section.settings.show_menu_price passed through as show_price) forwarded to resource-card, default false so existing menus are unchanged.
- ♿ **Parent mega-menu links give no expandable-state cue** `[medium | moderate | high]`  
  mega-menu__link--parent anchors (line 112) that own a child <ul> carry no aria-haspopup/expanded semantics; screen readers cannot tell a top-level link opens a submenu. Add aria-haspopup="true" (and, if JS toggles it, aria-expanded) on links where link.links != blank; this is additive markup that does not change layout.
- ⚡ **Featured product images lack explicit eager/lazy control** `[low | moderate | high]`  
  resource-card calls in the eager_loading path (line 293) rely on defaults for loading; since these appear only on hover/focus of the menu, ensure images are lazy-loaded to avoid competing with above-the-fold LCP. Pass an explicit image_loading:'lazy' (or expose it) to resource-card in the featured-products/collections branches, defaulting to current behavior if the card already lazy-loads.

### `snippets/meta-tags.liquid` — Outputs charset/viewport, Open Graph, Twitter Card, title, canonical and description meta tags for every page.
- 🔎 **og:image uses hardcoded http: scheme** `[medium | moderate | medium]`  
  Line 64 outputs og:image content as 'http:{{ page_image | image_url }}' while og:image:secure_url (line 67) uses https:. Serving the primary og:image over http can cause mixed-content/scraper issues on an https store; change the og:image value to a protocol-relative or https: URL (https:{{ page_image | image_url }}) to match secure_url. Low-risk output change.
- 🔎 **Missing og:image:alt and product OG availability** `[low | additive | medium]`  
  The page_image block (lines 61-78) emits width/height but no og:image:alt, and the product branch (lines 80-89) omits og:availability. Add <meta property="og:image:alt" content="{{ page_image.alt | default: og_title | escape }}"> and, on product pages, an og:availability tag derived from product.available (instock/oos). Additive tags that improve social/SEO rich previews without altering existing tags.

### `snippets/overflow-list.liquid` — Renders the <overflow-list> web component shell with declarative shadow DOM, slots for list/more/overflow, and a 'more' button.
- ♿ **'More' overflow button lacks aria-expanded/haspopup state** `[medium | moderate | high]`  
  The more button (line 45) is a plain <button> with tabindex=0 that reveals the part=overflow list but carries no aria-haspopup or aria-expanded, so assistive tech cannot tell it toggles hidden items. Expose optional more-button-attributes-driven aria (or add aria-haspopup="menu" and a default aria-expanded="false") on the button; keep it opt-in via the existing more-button-attributes param to avoid touching the web-component JS contract.

### `snippets/overlay.liquid` — Renders a full-bleed absolutely-positioned overlay (solid or gradient) driven by section/block settings.
- 🐛 **color_modify + default filter precedence produces broken --overlay-color--end** `[low | moderate | low]`  
  On line 16, `settings.overlay_color | color_modify: 'alpha', 0 | default: 'rgb(0 0 0 / 0)'` applies color_modify before default; if overlay_color is blank, color_modify returns blank and the `default` kicks in, but if overlay_color is a valid color the alpha-0 result is used correctly — however when overlay_color is unset the FIRST var (--overlay-color line 15) defaults to rgb(0 0 0 / .25) while the end color defaults to a different literal, which is fine, but the fragile chaining means a malformed color yields an empty custom property and the gradient silently fails. Precompute the two colors in a {% liquid %} block with explicit blank checks so the gradient endpoints always derive from the same base color.
- 🎛️ **Expose overlay opacity as an opt-in setting** `[low | additive | low]`  
  The overlay hardcodes alpha via the passed `overlay_color` only. Add an optional `overlay_opacity` param (default unset) that, when provided, wraps --overlay-color in `color_modify: 'alpha'` so merchants can dial overlay strength on hero/media images without editing color pickers. Default-off leaves current rendering unchanged.

### `snippets/pagination-controls.liquid` — Renders accessible previous/next + numbered pagination with mobile ellipsis collapsing and anchor-positioned hover indicator, optionally AJAX-driven via on_click_handler.
- 🎛️ **Expose pagination sizing/radius as schema-free CSS vars for merchant override** `[low | additive | medium]`  
  Lines 166-168 hardcode `--pagination-size`, `--pagination-inset`, `--pagination-radius` on `.pagination`. These are already custom properties; document them or surface an optional `radius`/`size` render param (default to current values) so section authors can align pagination pill radius with the brand's rounded card style without editing this shared snippet. Default values unchanged.
- ♿ **Numbered page links lack visible focus offset consistency on current page** `[medium | moderate | medium]`  
  The `.pagination__link--current` span (line 127) uses aria-current='page' correctly, but its color is set to var(--color-background) on a filled background via ::before. On collections with long product lists (large B2B catalog), verify contrast of the current-page number (--color-background text on --color-foreground fill) meets 4.5:1 with the navy #37456E / lime palette; if the theme's foreground token is lime #9CCB3B, white/near-white current text on lime fails WCAG AA. Add an explicit --pagination-current-color setting or force navy text.

### `snippets/predictive-search-empty-state.liquid` — Renders the predictive-search dropdown empty state, optionally showing up to 4 products from a merchant-chosen empty_state_collection (or collections.all).
- 💸 **Empty-state limit hardcoded to 4; expose count for B2B browse-to-buy** `[medium | additive | medium]`  
  Lines 26 and 34 hardcode `paginate collection.products by 4` and `limit: 4`. For a wholesale catalog where the empty search state is prime real estate to surface best-selling case-pack products, add an opt-in `empty_state_count` setting (default 4) so the merchant can show 6-8 featured/trade products. Keep 4 as default to avoid changing current layout.
- ♿ **role=listbox container with no active option semantics for empty state** `[medium | moderate | medium]`  
  The wrapper (line 12) declares `role="listbox"` and `aria-expanded="true"` even when only a heading + product grid render and no selectable option/descendant has role=option. When products render via resource-card these are not option-role children, so screen readers announce an empty listbox. Consider dropping role=listbox on the empty/browse state (or switch to role=region with an aria-label) since it is a browse grid, not a combobox result list.

### `snippets/predictive-search-products-list.liquid` — Renders the product list markup for predictive-search empty-state and recently-viewed products, using resource-card, with BSS Commerce hide-price gating per product.
- 🐛 **Duplicate int_id/product assignment in order_ids loop** `[low | moderate | high]`  
  In the order_ids branch, `int_id` and `product` are assigned twice: once inline on line 49 (before the BSS lock check) and again on lines 50-51. The second assignment (50-51) is dead/redundant work since line 49 already resolved `product` for the lock check and it is unchanged. Remove lines 50-51 to avoid re-running the `find: 'id'` filter twice per recently-viewed item (perf on repeated dropdown opens). Behavior is identical.
- 💸 **recently-viewed limit defaults to 8 but empty-state passes limit:4 with no reorder safety** `[medium | additive | high]`  
  Line 41/91 default limit to 8. For B2B quick-reorder, recently-viewed products in search are a strong reorder cue; ensure the recently-viewed branch surfaces enough items and consider adding an 'Add to cart' / quick-reorder affordance via resource-card params rather than image-only cards. Keep as additive param to resource-card so default card rendering is unchanged.
- ♿ **resultsItems/recentlyViewedItems <li> cards lack role=option under listbox parent** `[medium | risky | high]`  
  The parent predictive-search-dropdown declares role=listbox, but these product `<li>` items (lines 52, 67, 94) carry no role=option / id, so keyboard combobox navigation and SR announcements are incomplete. Add role=option and an id to each card <li> (opt-in, coordinate with the search web component's JS focus logic) so aria-activedescendant navigation works. Marked risky because it touches the JS-driven focus contract.

### `snippets/predictive-search-resource-carousel.liquid` — Renders a slideshow/carousel of predictive-search result cards for a given resource type (products, collections, pages, etc.).
- 🎛️ **Expose carousel arrow-visibility threshold as a param** `[low | additive | medium]`  
  The magic number in `{% if resources.size >= 4 %}` (used twice for both `slideshow-controls` and the `slideshow--single-media` class) is hardcoded. Accept an optional `min_slides_for_arrows` param defaulting to 4 so callers/merchant sections can tune when arrows and carousel mode kick in without editing the snippet, preserving current behavior when unset.
- ♿ **Associate the carousel list with its heading via aria-labelledby** `[low | additive | medium]`  
  The `<h4 id="predictive-search-{{ resource_type }}">` title is rendered but the `render 'slideshow'` list is not linked to it. Pass an `aria-labelledby="predictive-search-{{ resource_type }}"` (or role/region label) through to the slideshow list so screen-reader users know which resource group the carousel belongs to.

### `snippets/predictive-search-styles.liquid` — Consolidated CSS stylesheet for all predictive-search dropdown/modal/results/carousel presentation.
- ♿ **Guard heavy slide/scale animations behind prefers-reduced-motion** `[low | additive | medium]`  
  Many rules (search-element-scale-in/out, search-element-slide-up/down, content-slide, and the numerous nth-child animation-delay chains) always animate. Add a `@media (prefers-reduced-motion: reduce)` block that neutralizes `animation`/`transition` on `.predictive-search-results__wrapper`, `.predictive-search-results__card` and the keyframed elements for motion-sensitive users; additive since it only applies under the user preference.

### `snippets/price-filter.liquid` — Renders the price-range facet (min/max money inputs, current range status, highest-price hint, clear button) inside an accordion for storefront filtering.
- 🐛 **has_active_values can leak between renders when snippet is used twice** `[medium | moderate | medium]`  
  `has_active_values` is only assigned when a min/max value exists (lines 158-160) but never reset to false at the top. Because `price-filter` is documented as rendered multiple times (e.g. dialog + inline, per the `id_prefix` param), a Liquid `assign` from a prior render can persist, making the clear button `tabindex="0"`/`facets__clear--active` on a second instance that actually has no active values. Initialize `assign has_active_values = false` at the start of the capture.
- ♿ **Give the two currency-symbol labels distinct accessible text** `[low | additive | medium]`  
  Both `.price-facet__label` elements only contain `{{ cart.currency.symbol }}` (lines 113 and 140), so their `<label for=...>` announces just the currency symbol with no min/max context. The inputs carry `aria-label` (price_filter_low/high) which overrides, but the visible label association is ambiguous; add `visually-hidden` qualifier text (e.g. 'from'/'to') inside each label so the label text itself conveys min vs max.

### `snippets/price.liquid` — Renders a product's price block (regular/sale/compare-at, unit price, and quantity-break volume-pricing range) for product cards and product pages, with BSS wholesale price-locking hooks.
- 🐛 **Product-card volume range uses full product price range, not the quantity-break range** `[medium | moderate | high]`  
  In the `is_product_card` branch (lines 39-42) `has_volume_pricing` is set from `product_resource.quantity_price_breaks_configured?` but `price_min`/`price_max` are assigned `product_resource.price_min`/`price_max` — the whole product's variant price span, unrelated to the actual quantity-break tiers. This can display a misleading 'volume' range (e.g. driven by variant price differences) even when the break discount is small; source the min from the configured breaks or clearly label it as the product price range.
- 💸 **Prefix the volume-pricing range with a 'From' label for wholesale clarity** `[medium | moderate | high]`  
  On product cards the volume-pricing branch renders a bare `{{ price_min }} - {{ price_max }}` (line 87) using `product_resource.price_min`/`price_max`, which reads like a variant range rather than a bulk discount. For a B2B/wholesale buyer, wrap it with a translatable 'From {{ price_min }}' or 'Case price from …' prefix (a new locale key, defaulting to the current dash format if unset) so the case-pack savings are legible at a glance; keep the existing `volume-pricing-note` small text.
- 💸 **Volume-pricing note is a static string with no savings/threshold detail** `[medium | additive | high]`  
  The `volume-pricing-note` small element (lines 142-147) only prints `content.volume_pricing_available`. For trade buyers, optionally surface the first break's minimum quantity (e.g. 'Save on cases of {{ break.minimum_quantity }}+') via an opt-in setting, giving concrete bulk-buying incentive without changing the default copy when the setting is off.

### `snippets/product-badges-styles.liquid` — Consolidated CSS for product badge positioning (top/bottom corners) and badge chip styling.
- ♿ **Badge color relies on theme vars with no contrast safeguard** `[low | additive | low]`  
  `.product-badges__badge` sets `color: var(--color-foreground)` on `background: var(--color-background)` with no minimum-contrast fallback; for a merchant using the lime #9CCB3B accent on soft #F5F7FB backgrounds this can fall below WCAG AA at the small `--font-size--xs` badge size. Optionally add dedicated `--badge-color`/`--badge-background` custom properties (defaulting to the current foreground/background vars) so the merchant can set an accessible badge color pair without affecting existing themes.

### `snippets/product-card.liquid` — Renders the product-card web component (link, quick-add flags, view-transition wiring) used in product/featured/card blocks.
- 💸 **Expose opt-in stock/urgency badge on the card** `[medium | additive | high]`  
  The card computes `has_quick_add` from `product.available` (line 25) but never surfaces inventory. Add a default-off `block_settings.show_low_stock_badge` schema toggle that, when enabled, renders `variant_to_link.inventory_quantity` (e.g. 'Only N left') inside `.product-card__content` for the B2B reorder audience. Purely additive since it defaults off.
- ♿ **Accessible link name lacks price/availability context** `[low | additive | high]`  
  The `.product-card__link` span (lines 80-82) only outputs `product.title`; screen-reader users get no price or 'sold out' cue. Append `product.price | money` and an availability string (guarded by `unless onboarding`) into the existing `visually-hidden` span so the accessible name matches the visible card without altering layout.

### `snippets/product-grid.liquid` — Renders the collection/search product grid (layout CSS, empty state, grid-view sessionStorage restore, infinite-scroll sentinels).
- 💸 **No free-shipping / bulk-order banner slot above grid** `[medium | additive | high]`  
  For a wholesale audience the grid has a `title` slot (line 117) but no merchant-controlled trust/threshold message. Add a default-empty `section.settings.grid_promo_text` richtext setting rendered above `<ul class="product-grid">` so merchants can surface case-pack/free-freight-over-$X messaging without a code change; empty default keeps current output identical.
- ♿ **Empty-state heading uses .h2 class but h2 tag inside grid** `[low | moderate | high]`  
  The empty state (lines 104-111) renders an `<h2 class="...h2">` while the populated grid uses `<h4 class="main-collection-grid__title">` (line 118) for the results header; on a collection page the section already owns the page h1/h2, so this can create a heading-order jump. Consider making the empty-state heading level configurable or aligning it with the results `h4` to keep a logical outline. Verify against the section's existing headings before changing.

### `snippets/product-information-content.liquid` — Lays out the product page media gallery and details columns (media position, equal columns, width, skip-link); wrapped by a vendored BSS hide-price guard.
- ♿ **Skip-to-content link only rendered when media is on the left** `[low | moderate | high]`  
  The skip link (lines 85-88) is gated on `settings.desktop_media_position == 'left'`, so when media is positioned right, keyboard users get no skip-to-product-info affordance even though `#ProductInformation-<id>` target logic is the same. Render the skip link for both media positions (drop the position condition, keep the `product_has_media` and `page_type == 'product'` guards) to restore parity.

### `snippets/product-media-gallery-content.liquid` — Renders the product media gallery: sorted media, slideshow/grid presentation, controls, zoom dialog, thumbnails and 3D model XR setup.
- 🎛️ **No merchant control over thumbnail loading strategy** `[low | additive | high]`  
  Zoom-dialog thumbnails are hard-coded `loading: 'lazy'` (line 549) which is fine, but the gallery has no opt-in to eager-load the first main media for LCP on the PDP. A default-off `block_settings.eager_first_media` toggle passed to `product-media` `loading` on `forloop.first` would let merchants tune LCP; defaults off so behavior is unchanged.
- ♿ **Gallery images can ship with empty alt text** `[medium | additive | high]`  
  The main gallery relies on `product-media` with `alt: media.alt`, and the zoom thumbnails (line 553) pass `alt: media.alt` with no fallback; when a merchant leaves alt blank these are empty. Add a fallback such as `media.alt | default: selected_product.title` for the thumbnail `image_tag` alt so wholesale product images always have a meaningful accessible name. Additive (only fills empty alts).
- ⚡ **Zoom thumbnail widths list is heavier than needed** `[low | moderate | high]`  
  Thumbnail `image_tag` (lines 546-554) requests `widths: '240, 352, 832, 1200'` for elements sized ~110-160px (sizes attr) / max `--thumbnail-width`; the 832 and 1200 candidates are never used and inflate the srcset. Trim to small candidates (e.g. '110, 160, 240, 352') to cut wasted bytes on the zoom dialog without visual change.

### `snippets/product-media.liquid` — Renders a single product media item (image, 3D model, or video) with responsive image, focal point, and border-radius styling.
- 🐛 **Duplicate ref: argument on image_tag drops the transition ref** `[medium | moderate | medium]`  
  In the main image_tag call (lines 43-54) the filter is passed ref twice: ref: ref_image_to_transition and, three lines later, ref: image_ref. Liquid keeps only the last named argument, so ref: image_ref always wins and the 'imagesToTransition[]' ref set when settings.transition_to_main_product is enabled is silently discarded, breaking the image-to-main-product transition. Merge into one ref: build a single value (e.g. capture image_ref first, else fall back to ref_image_to_transition) and pass ref only once.
- ♿ **Media alt falls back to empty string for images with no alt text** `[medium | moderate | medium]`  
  Both image_tag calls use alt: media.alt with no fallback. Horizon products in this B2B catalog frequently ship without per-image alt text, producing alt="" on the main gallery image. Add a fallback such as alt: media.alt | default: selected_product.title (or product.title) so screen-reader and SEO output describes the product (e.g. 'Toothpaste 100ml case') instead of an empty string.
- ⚡ **No sizes fallback lets responsive image download the largest width** `[medium | additive | medium]`  
  image_tag is passed sizes: sizes with no default; when a caller omits sizes the browser treats the image as 100vw and can pick from the very large widths list (up to 3840). Add a sensible default (e.g. sizes = sizes | default: '(min-width: 750px) 50vw, 100vw') so the srcset picks appropriately sized files and reduces bytes/CLS on product and quick-add galleries.

### `snippets/quantity-selector.liquid` — Renders the +/- quantity stepper used on product, cart, and quick-order contexts, plus a volume price-per-item readout when quantity price breaks are configured.
- 🐛 **Volume price-break selection loop picks the wrong tier (no ordering guarantee / early break)** `[high | moderate | high]`  
  The initial display_price loop (lines 130-135) iterates variant.quantity_price_breaks and breaks on the first tier where current_qty >= price_break.minimum_quantity. This assumes the breaks are ordered highest-minimum-first; if Shopify returns them ascending (the common order), it stops at the lowest qualifying tier (e.g. picks the 10-unit price even when current_qty is 100 and a cheaper 100-unit tier exists), showing an inflated 'at price / ea'. Track the best match across the whole list (largest minimum_quantity <= current_qty wins) instead of breaking on first match.
- 💸 **Surface next-tier savings prompt for wholesale buyers** `[high | additive | high]`  
  price-per-item already exposes data-price-breaks and data-at-text. For this B2B/case-pack audience, add an opt-in setting (e.g. settings.show_next_break_hint, default off) that renders a second span like 'Buy {{ next_break.minimum_quantity }} to pay {{ next_break.price | money }}/ea' inside the price-per-item component using the already-computed price_breaks_json, nudging buyers up to the next case-pack tier without changing default output.
- ♿ **aria-live=polite on the quantity input announces every keystroke** `[medium | moderate | high]`  
  The quantity <input> (line 72) carries aria-live="polite" itself; a focused number input with aria-live re-announces its own value on each change, which is noisy/confusing with screen readers. Remove aria-live from the input and instead apply aria-live to the price-per-item__text region (the value that actually updates), so assistive tech announces the recalculated per-item price rather than the raw typed number.

### `snippets/quick-add-modal-styles.liquid` — Consolidated CSS-only stylesheet for the quick-add modal layout, gallery, scroll-mask effects, and sticky buy-buttons bar.
- 🐛 **Mismatched breakpoints (749px vs 750px) leave a 1px dead zone at exactly 750px** `[low | moderate | medium]`  
  The open-state radius rule uses @media (max-width: 750px) (line 39) while the base modal rules use max-width: 749px and min-width: 750px (e.g. lines 27, 46). At a viewport of exactly 750px both the mobile radius rule and the desktop display:flex rule apply, producing inconsistent styling at that boundary. Standardise the radius rule to max-width: 749px to match the rest of the file.

### `snippets/quick-add-modal.liquid` — Markup for the quick-add dialog web component (quick-add-dialog/dialog) with close button and an empty content container filled at runtime.
- ♿ **Dialog lacks an accessible name / labelled title association** `[medium | additive | high]`  
  The <dialog class="quick-add-modal"> (lines 7-11) has no aria-label or aria-labelledby, so screen readers announce it only as 'dialog'. Since the title is injected into #quick-add-modal-content at runtime, add a static aria-label="{{ 'accessibility.quick_view' | t }}" (or similar existing locale key) on the dialog so it has a stable accessible name when opened.

### `snippets/quick-add-styles.liquid` — Consolidated CSS-only stylesheet for the floating quick-add button on product cards and its add-to-cart text/animation states.
- ♿ **Quick-add button relies on hover/focus-within opacity, hiding it from keyboard/touch by default** `[medium | additive | medium]`  
  The button is opacity:0 by default and only reveals via product-card:is(:hover, :focus-within) (line 33) plus the [stay-visible] escape hatch. On touch devices and for keyboard users tabbing to the card, discoverability of the primary quick-add CTA is poor. Consider gating a merchant setting that adds [stay-visible] (already supported at line 71) so the button can be persistently shown on mobile/touch for this B2B reorder flow without changing the current default.

### `snippets/quick-add.liquid` — Renders the quick-add product form (Add / Choose options button) inside product cards via the quick-add-component / product-form-component web components.
- 💸 **Respect and surface case-pack min quantity for B2B** `[medium | moderate | high]`  
  The hidden quantity input (line 63-65) already falls back to variant.quantity_rule.min, but the card gives buyers no indication a pack/case minimum applies. Add an opt-in schema/setting-driven small note near the add-to-cart button showing 'Min {{ variant.quantity_rule.min }}' when variant.quantity_rule.min > 1, so wholesale buyers see the case-pack quantity before adding. Default-off keeps current output unchanged.
- ♿ **Give the Choose-options button an accessible name on mobile** `[medium | additive | high]`  
  On the Choose button (line 85-111) the label text lives in span.add-to-cart-text__content.is-visually-hidden-mobile, so on mobile the button collapses to just the icon (aria-hidden svg) with no accessible name. Add aria-label="{{ choose_options_text }} - {{ product.title }}" to the <button> so screen-reader and mobile users get a name.

### `snippets/resource-card.liquid` — Unified card for products, collections, articles and pages, rendering image (with optional hover image), title and price/excerpt.
- 💸 **Expose a volume/case-pack pricing note the CSS already styles** `[medium | moderate | medium]`  
  The stylesheet defines .volume-pricing-note (line 194-204) but nothing in this snippet outputs it. For the wholesale audience, when resource_type == 'product' and product has quantity_price_breaks / volume pricing, render a short 'Buy more, save more' note under the price (opt-in) using that existing class so bulk savings are visible on the card.
- ♿ **Add alt text to collection thumbnail images** `[medium | additive | medium]`  
  The collection multi-thumbnail loop (line 129-134) renders product.featured_image via image_tag with no alt attribute, producing empty alt for up to 4 images per collection card. Pass alt: product.featured_image.alt | default: product.title so each thumbnail is described.
- ⚡ **Allow eager loading of the first card image to protect LCP** `[medium | additive | medium]`  
  The primary product/collection image (line 85-96) is hardcoded loading: 'lazy'. When resource-card is the first row above the fold (e.g. featured collection/grid), lazy loading the LCP image delays it. Accept an optional `loading` param (default 'lazy') passed through to image_tag so callers can set 'eager' for the first card without changing default behavior.

### `snippets/resource-image.liquid` — Unified responsive image renderer for resource lists (featured-blog cards, collection cards) with aspect-ratio, placeholder and overlay handling.
- ♿ **Add alt text to the rendered resource image** `[medium | additive | medium]`  
  The image_tag call (line 150-154) passes widths/class/sizes/loading but no alt, so collection and article images render with empty alt. Add alt: image_resource.alt | default: collection.title (collections) / image_source.alt (articles) to describe the image.
- ⚡ **Cap upscaling of small source images at 3840px request** `[low | additive | medium]`  
  image_url: width: 3840 (line 152) requests a 3840px master for every image even when the source is smaller, and the widths list also tops out at 3840. This is generally fine but for small collection/article thumbnails it wastes bytes; consider clamping the requested master width to image_resource.width where available, or documenting that widths above the source are dropped by Shopify. Low urgency.

### `snippets/resource-list-carousel.liquid` — Wraps resource cards in the shared slideshow component to produce a peeking-slide carousel with container-query-driven slide sizing.
- 🐛 **for loop uses slides.size after slides is being recaptured** `[low | moderate | medium]`  
  The {% for item in slides limit: slides.size %} (line 32) iterates the incoming slides array while the same `slides` variable is being reassigned by the surrounding {% capture slides %} (line 31). limit: slides.size is redundant (the loop already ends at the array end) and relies on `slides` still referring to the array during capture; simplify to {% for item in slides %} to avoid confusion and any edge-case re-evaluation.

### `snippets/resource-list.liquid` — Layout dispatcher that renders a resource list as grid, bento, carousel or editorial, with an optional separate mobile carousel for non-carousel layouts.
- ⚡ **Mobile carousel duplicates full card markup, doubling DOM/image weight** `[medium | moderate | medium]`  
  For non-carousel layouts with carousel_on_mobile (line 143-176), the full list_items_array is rendered a second time inside a hidden--desktop carousel while the desktop grid is hidden--mobile. Both card sets (and their <img> tags) exist in the DOM at once, doubling image requests on the layout that is hidden. Consider a note/param to lazy-render or share nodes, or ensure the hidden set's images use loading:'lazy' so the offscreen duplicate does not compete for bandwidth.

### `snippets/scripts.liquid` — Renders the theme's global importmap, module preloads and all component <script> tags, plus the JS Theme object (routes/translations/template).
- 🎛️ **Expose Theme.settings for free-shipping threshold to JS** `[low | additive | high]`  
  The Theme JS object (lines 282-304) exposes routes/translations but no settings. Adding a default-off/optional settings block (e.g. free_shipping_threshold, currency) would let cart/progress-bar JS read the wholesale free-shipping goal client-side without a second request; additive since nothing consumes it until a merchant opts in.
- ⚡ **Remove duplicate fly-to-cart.js load** `[low | moderate | high]`  
  fly-to-cart.js is loaded unconditionally at line 156 and then again inside the product-template block at lines 261-264. The second copy is redundant (same URL, already cached) but adds a needless duplicate <script type=module> parse on every product page; delete the lines 260-264 block so it only loads once.
- ⚡ **Scope volume/price-per-item scripts to templates that use them** `[low | moderate | high]`  
  volume-pricing.js, price-per-item.js and volume-pricing-info.js (lines 235-249) load on EVERY page even though volume/case-pack pricing only renders on product/cart/collection. For this B2B store these are core, but wrapping them in an {% if template.name == 'product' or 'cart' or 'collection' %} guard (like the existing localization guard at 251) trims JS on blog/home/policy pages without changing behaviour where they're needed.

### `snippets/search-modal.liquid` — Renders the predictive-search modal dialog (input, results, view-all footer) as a web-component with its scoped stylesheet.
- 🐛 **Meaningless default filter on hard-coded input id** `[low | additive | high]`  
  Lines 59 and 66 use for="{{ 'cmdk-input' | default: 'Search' }}" and id="{{ 'cmdk-input' | default: 'Search' }}" — the default filter never fires because the string literal 'cmdk-input' is always truthy, so it is dead/confusing code that just resolves to 'cmdk-input'. Replace with the plain literal id="cmdk-input" to remove the misleading filter (label/input pairing is unaffected).
- 💸 **Add quick B2B search suggestions / popular categories in empty state** `[medium | additive | high]`  
  The empty-state region (predictive-search-empty-state render, lines 118-122) shows nothing actionable before typing. For trade buyers a new opt-in schema-driven list of quick links (e.g. 'Toothpaste', 'Interdental', 'Bulk cases') passed into the empty-state snippet would speed reorder discovery; additive if gated behind a new default-off setting so current empty state is unchanged.
- ♿ **View-all button lacks descriptive context** `[low | moderate | high]`  
  The viewAllButton (lines 126-131) renders only the translated 'view all' label with no aria description of the current query; adding aria-label or visually-hidden text like 'View all results for <query>' (populated by predictive-search JS) improves screen-reader clarity. Keep additive by only adding an aria-describedby target the JS may populate.

### `snippets/search.liquid` — Renders the header search trigger (<search-button>) that opens the #search-modal dialog, in icon or text display style.
- 🎛️ **Allow merchant-configurable search button label** `[low | additive | medium]`  
  The button text is hard-coded to the 'content.search' translation (line 23). Accepting an optional label param (defaulting to the current translation) would let the header block show trade-focused copy like 'Search products / SKU' without a locale override; additive because it falls back to the existing translation when unset.
- ♿ **Search icon SVG is unlabelled inside icon-only mode** `[low | additive | medium]`  
  In icon display_style the visible label span is 'hidden' (line 22) and only the inline icon-search.svg shows (lines 26-28); the button's aria-label (line 19) covers this, which is correct, but the decorative svg-wrapper span should carry aria-hidden="true" to prevent double announcement in some AT. Additive attribute-only change.

### `snippets/section.liquid` — Shared wrapper that renders a section's background media, overlay, border/spacing/layout styles and injects the section's child blocks.
- 🎛️ **No fallback when section_height is an unexpected value** `[low | additive | high]`  
  The inline style block (lines 17-29) maps section_height to --section-min-height only for custom/full-screen/non-empty named values; any typo or removed preset silently yields no min-height. A safe additive improvement is documenting/validating allowed values in the schema that feeds this, but the snippet itself is a shared contract so leave markup unchanged.
- ♿ **Background/overlay containers have no semantic role or labelling guidance** `[low | additive | high]`  
  The section-background and custom-section-background divs (lines 8, 31-39) render decorative background media with no aria-hidden; when background_media is a video/image purely decorative, marking these wrappers aria-hidden="true" avoids AT reading empty regions. Additive attribute; does not alter layout.

### `snippets/size-style.liquid` — Emits CSS custom properties (--size-style-width/height and mobile variants) from a block/section's width/height settings for use in inline styles.
- 🐛 **Mobile width variable is undefined for named/preset width_mobile values** `[medium | moderate | medium]`  
  The width_mobile branch (lines 25-33) only outputs --size-style-width-mobile for 'custom', 'fill' and 'fit-content'. Unlike the desktop width block (lines 7-14) it has no {% else %} for a plain preset value (e.g. a named size), so --size-style-width-mobile / --size-style-width-mobile-min are left unset and any CSS relying on them falls back to inherited/default, causing inconsistent mobile widths. Add an else branch mirroring lines 12-13 to emit the raw settings.width_mobile value.
- 🐛 **custom_width/custom_height emitted with % but treated inconsistently vs desktop custom** `[low | moderate | medium]`  
  Desktop custom width outputs {{ settings.custom_width }}% (lines 8-9) and mobile custom outputs the same % (lines 26-27), but the intent for these controls is often px vs %; confirm the unit against the schema — if custom_width is a 0-100 percent slider this is fine, but if it's a pixel value the trailing % breaks sizing. Verify before changing; flagged as a potential defect not a certain one.

### `snippets/skip-to-content-link.liquid` — Renders an accessibility skip-to-content link that is visually hidden until focused.
- ♿ **Focus state lacks a visible focus outline** `[low | moderate | low]`  
  The .skip-to-content-link:focus rule (lines 25-35) sets background and a same-color box-shadow (var(--color-background)) but no contrasting focus ring, so keyboard users may not clearly perceive the link's boundary against the page. Add an outline using --color-foreground or a --focus-outline-color token to strengthen the visible focus indicator without changing the hidden-by-default behavior.

### `snippets/sku.liquid` — Renders the selected/first-available variant SKU inside a ref container for a product SKU block.
- 💸 **Add optional 'SKU:' label for wholesale reorder scanning** `[medium | additive | medium]`  
  For a B2B/wholesale reordering audience the SKU is a primary purchase reference, but the snippet outputs a bare value (line 18) with no visible label. Add an optional label via a new default-off/blank param (e.g. sku_label) rendered as a visually-hidden or inline prefix like 'SKU:' / 'Product code:' so trade buyers can scan and quick-reorder; keep it empty by default so existing output is unchanged.
- ♿ **Suppress empty SKU container when variant has no SKU** `[low | moderate | medium]`  
  When product_resource.selected_or_first_available_variant.sku is blank (line 9), the snippet still renders an empty <div ref="skuContainer"><span class="sku">...</span></div>, which JS/CSS may expose as an empty labelled region. Wrap the output so the container collapses when sku is blank (e.g. {%- if sku != blank -%}) to avoid an empty SKU element on products without SKUs.

### `snippets/slideshow-arrow.liquid` — Renders a single previous/next slideshow arrow button with configurable icon style, shape and size.
- 🐛 **Dead icon_size branch keys off icon_style instead of icon_size** `[low | moderate | medium]`  
  Lines 22-24 do `if icon_style contains 'large'` then set icon_size = 'large', but icon_style only ever holds 'arrow' or 'chevron' per the doc (lines 4-5); the size value lives in the icon_size param. This condition can never be true, so the caller-provided icon_size is honoured only via the class on line 30 while this branch is dead code. Remove the dead check (or fix it to test icon_size) so the intended large-icon stroke/size styles in the .slideshow-control--large stylesheet actually apply when 'large' is requested.
- 🎛️ **Add optional arrow color override param** `[low | additive | medium]`  
  The button uses fixed brand-agnostic tokens: shape backgrounds use var(--color-primary-button-background)/text (lines 75-76). Expose an opt-in param to override arrow color (e.g. arrow_color) applied as an inline --color override, letting the merchant tint arrows to navy #37456E or lime #9CCB3B on light hero imagery without editing the snippet; default to current tokens so nothing changes.

### `snippets/slideshow-arrows.liquid` — Renders the previous/next arrow pair as an overlay <slideshow-arrows> web component positioned over slideshow media.
- ♿ **Blend-mode arrows can lose contrast over mid-tone media** `[low | additive | high]`  
  When icon_shape is 'none' the component relies on mix-blend-mode: difference (line 41) for arrow visibility over media, which produces unpredictable contrast against mid-tone hero photography (common for product/lifestyle shots) and can render arrows nearly invisible. Add an opt-in fallback (e.g. a subtle text-shadow or semi-opaque backdrop on .slideshow-control) so arrows meet a minimum contrast on any image; gate it behind a param so default blend behavior is preserved.

### `snippets/slideshow-controls.liquid` — Renders slideshow pagination controls (dots, counter, thumbnails), autoplay play/pause, and arrows for the <slideshow-controls> web component.
- 🎛️ **Add optional brand color for on-media control overlay** `[low | additive | high]`  
  The counter background is hard-coded to rgb(0 0 0 / 40%) with blur (line 418-423) and dots colors derive from --color-foreground only, so on-media controls always assume a dark overlay. Expose an opt-in setting for control accent/overlay color so the merchant can match brand navy/lime; default to the existing black overlay so current appearance is unchanged.
- ♿ **Use aria-current instead of aria-selected on dot/thumbnail buttons** `[low | moderate | high]`  
  The counter markup (lines 156-158) renders '<current>/<item_count>' but has no accessible role/label announcing it as slide position, and the dots/thumbnails use aria-selected on <button> elements which is not a valid ARIA state for a plain button. Wrap the dots list semantics or use aria-current='true' instead of aria-selected on the dot/thumbnail buttons (lines 116, 169) so assistive tech correctly conveys the active slide.
- ⚡ **Eager-load first thumbnail to reduce initial CLS** `[medium | moderate | high]`  
  Thumbnail images (lines 126-131) are all rendered with loading: 'lazy' at width 144, but the first thumbnail is above-the-fold and aria-selected on load (line 116-118); lazy-loading it can cause a visible pop-in / CLS on initial paint. Set loading: 'eager' (and fetchpriority) for forloop.first only while keeping the rest lazy to reduce first-slide layout shift.

### `snippets/slideshow-slide.liquid` — Renders a single <slideshow-slide> web-component element with sizing, media-fit, hidden state and click-to-navigate behavior.
- 🐛 **Duplicate style attribute silently drops the caller's style param** `[medium | moderate | high]`  
  The element emits style twice: line 42 sets --slideshow-timeline/--product-media-fit/grid rows, then lines 47-49 emit a second style="{{ style }}" when style != blank. HTML honors only the first style attribute, so any value passed via the style param is silently ignored. Merge the caller's {{ style }} into the single style attribute on line 42 (append it after the grid-template-rows vars) and remove the second style block.
- 🐛 **attributes is rendered twice on the element** `[medium | moderate | high]`  
  {{ attributes }} appears on both line 46 and line 59, so any pass-through attributes are duplicated on <slideshow-slide>. Duplicate attributes are invalid and the second wins/ is ignored; if attributes contains an id or event binding it can double up. Remove one of the two {{ attributes }} outputs.
- ♿ **aria-hidden hard-wired to index==0 ignores initial_slide / navigation** `[low | additive | high]`  
  Line 41 sets aria-hidden="false" only for index 0 and "true" for all others, statically at render time. When the slideshow starts on a different slide (parent passes initial_slide) or after JS navigation, the exposed slide and the aria-hidden state can disagree, hiding the visible slide from assistive tech until JS corrects it. Expose an optional active_index/initial_slide param to slideshow-slide and compare index against it instead of the literal 0.

### `snippets/slideshow-styles.liquid` — Consolidated CSS-only stylesheet for the slideshow component/slides, shared by slideshow snippets and the predictive-search resource carousel for reliable CSS subsetting.
- ♿ **Grab/grabbing cursor drag affordance has no reduced-motion or pointer fallback note** `[low | additive | medium]`  
  slideshow-slides uses scroll-behavior: smooth with a prefers-reduced-motion override to auto (lines 45-56), which is good, but the reveal keyframes slide-reveal (lines 136-150) that translate/opacity-animate slides are not gated behind @media (prefers-reduced-motion: no-preference). Wrap the animation usage so reduced-motion users don't get the horizontal translate reveal, matching the existing reduced-motion handling for scroll-behavior.

### `snippets/slideshow.liquid` — Renders the <slideshow-component> wrapper with container, optional arrows, gutters, autoplay, infinite loop and slot for slides/controls.
- 🎛️ **No merchant control over drag cursor / autoplay-on-mobile behavior surfaced here** `[low | additive | high]`  
  infinite defaults on unless explicitly false (lines 94-96) and there is no way for a calling section to disable autoplay only on mobile without editing markup. Expose an optional mobile_autoplay_disabled param that adds the existing mobile-disabled attribute (already styled in slideshow-styles.liquid line 92) to slideshow-component, giving merchants opt-in control without changing defaults.
- ♿ **Autoplay slideshow has no pause/play control exposed** `[medium | additive | high]`  
  When autoplay == true (lines 87-89) the component gets autoplay and aria-live="polite" but there is no rendered pause control, which fails WCAG 2.2.2 (moving content that auto-updates must be pausable). Add an optional additive show_pause_control param that renders a pause/play toggle button inside slideshow-container when autoplay is on, default off to preserve current output.

### `snippets/sorting.liquid` — Renders the sort-by control for collection/search results as a native <select> on mobile and an accessible listbox/accordion on desktop.
- 💸 **Sort control has no default-relevance hint for wholesale buyers** `[low | additive | high]`  
  For a B2B catalog, buyers most often want price/newest ordering, but the summary status (line 96) only echoes the current option name. Add an optional additive label param (e.g. sort_prompt) rendered next to actions.sort so the merchant can prompt trade buyers ('Sort case packs by...') without altering the default 'Sort' label.
- ♿ **data-should-use-select-on-mobile emits blank when param omitted** `[medium | moderate | high]`  
  Line 20 outputs data-should-use-select-on-mobile="{{ should_use_select_on_mobile }}" which renders an empty string (not "false") when the boolean is nil, and the JS/CSS branches (lines 31, 43, 76) treat truthiness inconsistently. Default it explicitly (e.g. should_use_select_on_mobile | default: false) so the attribute is always a defined "true"/"false" string and the mobile select vs accordion choice is deterministic.

### `snippets/spacing-padding.liquid` — Utility snippet that emits inline CSS custom properties (--padding-block/inline-start/end) from a passed settings object for section/block padding.
- 🐛 **Inconsistent whitespace trimming around inserted px values** `[low | additive | medium]`  
  The block-start/inline-start lines use {{ ... | default: 0 }}px (no trim) while block-end/inline-end use {{- ... | default: 0 -}}px with trim markers, producing inconsistent surrounding whitespace inside the style attribute. Harmless to rendering but makes the emitted style string inconsistent; normalize the trim markers across all four declarations for predictable output.
- ⚡ **Emits all four padding custom properties even when values are 0** `[low | moderate | medium]`  
  Lines 10-11 always output --padding-block-start/end and --padding-inline-start/end with a default of 0px, adding inline style bytes to every consuming element even when no padding is set. This is minor but multiplied across many sections; consider only emitting a property when the corresponding setting is present (non-nil) to trim inline CSS, keeping the 0 default only where the setting exists.

### `snippets/spacing-style.liquid` — Emits responsive CSS custom properties (--padding-block-start, etc.) from block/section spacing settings, clamping small values and scaling large ones by --spacing-scale.
- 🐛 **Margin variables promised in doc are never emitted** `[low | additive | high]`  
  The @doc header says it renders 'padding and margin' styles, but the `keys` list only contains the four padding-block/inline keys; no margin-* custom properties are ever output. Either extend `keys` to include margin-block-start/end and margin-inline-start/end, or correct the doc comment to say padding-only, so callers relying on margin variables don't silently get nothing.
- 🎛️ **Hardcoded scale_min floor of 20px is not merchant-controllable** `[low | additive | high]`  
  `assign scale_min = 20` fixes the minimum spacing floor for every block using this snippet. Accept an optional `scale_min` render param (defaulting to 20 when blank) so sections needing tighter mobile spacing can override without editing the shared snippet, keeping current output identical by default.

### `snippets/stylesheets.liquid` — Preloads overflow-list.css and outputs the base.css stylesheet tag (with preload) in the document head.
- ⚡ **overflow-list.css is preloaded but never given a stylesheet_tag** `[low | moderate | high]`  
  Line 1 emits `preload_tag: as: 'style'` for overflow-list.css but there is no corresponding `stylesheet_tag`, so the browser preloads the file and may warn 'preloaded but not used' / never applies it via this snippet. Confirm overflow-list.css is loaded elsewhere; if not, add a matching `{{ 'overflow-list.css' | asset_url | stylesheet_tag }}` (or drop the preload) to avoid a wasted high-priority fetch that competes with base.css.

### `snippets/submenu-font-styles.liquid` — Derives --menu-parent-* and --menu-child-* CSS typography variables (family, style, weight, case, size, line-height, color) for 2nd/3rd level menu items from block typography settings.
- 🐛 **menu_font_style has no default branch, dropping child/parent size vars for unknown values** `[medium | moderate | high]`  
  Both `{% case settings.menu_font_style %}` blocks only handle 'regular', 'inverse', 'inverse_large' with no `{% else %}`. If menu_font_style is blank or an unexpected value, none of --menu-parent-font-size/-line-height/-color (and the child equivalents) are emitted, leaving submenu items unstyled and falling back to inherited values. Add an `{% else %}` mirroring the 'regular' case to guarantee the size/color variables always exist.
- ♿ **Subdued-opacity menu color risks failing WCAG contrast on lime/soft backgrounds** `[medium | additive | high]`  
  Parent and child colors use `rgb(var(--color-foreground-rgb) / var(--opacity-subdued-text))`, multiplying foreground by a subdued alpha. On the brand's soft #F5F7FB/#E3E7F0 menu panels this can drop below 4.5:1 for the 3rd-level 'inverse'/'inverse_large' child text. Expose an opt-in setting (e.g. `menu_child_full_contrast`) that swaps the subdued color for solid --color-foreground when enabled, defaulting off so current rendering is unchanged.

### `snippets/swatch.liquid` — Renders a variant color/image swatch span with --swatch-background and focal-point vars, mode classes (unscaled/filter/pill), unavailable state, and a visually-hidden accessible label.
- 🐛 **Empty class/style attributes always render even when no extras apply** `[low | moderate | high]`  
  extra_classes starts as '' and the class attribute embeds leading/blank whitespace plus the multiline template, producing extra whitespace nodes; more notably when swatch_value is null the style still outputs `--swatch-background: ;` (empty value) which is an invalid declaration. Guard the --swatch-background line with `{% if swatch_value %}` so unavailable swatches emit clean markup.
- ♿ **Unavailable swatch conveys state only via CSS class, not to assistive tech** `[low | additive | high]`  
  When swatch_value is null the span gets `swatch--unavailable` styling but the visually-hidden `label` (line 51) still reads only the color name, so screen-reader users aren't told the option is unavailable/out of stock. Optionally append an availability suffix to the hidden label (opt-in via a new `unavailable_label` param defaulting to blank) so B2B buyers on assistive tech perceive sold-out swatches.
- ⚡ **Swatch image URL requested at fixed width 80 ignores DPR and actual swatch size** `[low | moderate | high]`  
  For `swatch.image` (line 22) the URL is hardcoded to `image_url: width: 80`, whereas the variant-image branch correctly doubles settings.variant_swatch_width for retina. Base swatch images on `settings.variant_swatch_width | times: 2` (with an 80 fallback) so large-swatch themes stay crisp and small-swatch themes don't over-fetch.

### `snippets/tax-info.liquid` — Renders localized tax/duties/shipping disclosure text for the cart based on cart.taxes_included, cart.duties_included, shipping policy presence, and a discounts flag.
- 💸 **Expose a merchant-editable trust/tax reassurance line** `[medium | additive | low]`  
  The <small> only prints Shopify's translated tax/duty strings. For a B2B/wholesale AU audience, add an optional block/section setting (e.g. tax_info_note) passed in and rendered after the built-in string when set, so the merchant can append 'Prices shown ex-GST' or 'GST invoice provided at checkout' without editing locale files. Default empty = no change to current output.
- ♿ **Empty <small> can render with no content** `[low | moderate | low]`  
  If none of the cart.taxes_included / cart.duties_included branches match (e.g. both are nil rather than true/false), the outer <small></small> still renders empty. Wrap the whole element in a capture and only output <small> when the captured text is non-blank to avoid an empty semantic element in the cart summary.

### `snippets/text.liquid` — Renders the Horizon text theme-block: builds spacing/typography/width/background CSS custom properties and outputs the block text in a div or rte-formatter element.
- 🐛 **background_color default uses opaque white, ignoring theme scheme** `[low | moderate | medium]`  
  On line 66 --text-background-color falls back to 'rgb(255 255 255 / 1.0)' when block_settings.background is on but background_color is unset. On a navy (#37456E) scheme section this forces a hard white box. Default instead to a scheme-aware value like rgb(var(--color-background-rgb) / 1) so an enabled-but-unconfigured background blends with the section.
- 🎛️ **Add optional max-width override for wide B2B copy blocks** `[low | additive | medium]`  
  --max-width is locked to var(--max-width--{{ type }}-{{ block_settings.max_width }}) presets (normal/narrow/tight). Add an opt-in block_settings.max_width_custom (default blank) that, when set, emits --max-width: {{ block_settings.max_width_custom }} instead, letting the merchant widen dense wholesale spec/description text without a code change. Default blank preserves current preset behavior.

### `snippets/theme-styles-variables.liquid` — Global :root design-token stylesheet: emits @font-face declarations for all font roles/weights and defines the theme's CSS custom properties (layout widths, typography scale, colors, spacing, borders, animation, cart/variant tokens).
- 🎛️ **Surface stock-status colors as schema settings** `[medium | additive | high]`  
  --color-instock (#3ED660), --color-lowstock (#EE9441), --color-outofstock (#C8C8C8) are hardcoded. For a wholesale store leaning on stock/urgency signals, back these with opt-in settings.* color settings that default to the current hex values, letting the merchant align in-stock/low-stock badges with the brand lime #9CCB3B without editing this shared file.
- ⚡ **Preload the primary body font to cut FOUT/CLS** `[medium | moderate | high]`  
  All font_face calls use font_display:'swap', so the primary body font (settings.type_body_font, Work Sans) flashes a fallback and can shift layout on first paint. Emit a <link rel=preload as=font> (via primary_font | font_url) for the primary regular weight in the head partner snippet, keeping swap. This reduces text reflow on the landing/collection pages without changing rendered styles.

### `snippets/typography-style.liquid` — Emits inline typography CSS custom properties (color, fluid font-size clamp, weight, family, case, wrap, line-height, letter-spacing) from a settings object for use on an element's style attribute.
- 🐛 **Fluid clamp can emit an undefined min when font_size_rem < 3 but branch misaligns** `[low | moderate | medium]`  
  dynamic_min_rem/vw_value are only assigned inside the font_size_rem >= fluid_size_cutoff_rem (3.0rem) branch (lines 46-54), and the same guard gates their use (line 56), so they stay paired — good. But when settings.font_size is set with a non-'rem' unit (e.g. 'px' or a bare number), `split: 'rem' | first | times: 1.0` yields the raw number (e.g. 72 for '72px'), making font_size_rem huge and forcing an unintended clamp(). Validate/normalize the unit or guard on settings.font_size containing 'rem' before the numeric compare.

### `snippets/unit-price.liquid` — Renders a product/line-item unit price with its measurement plus a visually-hidden accessibility label.
- 💸 **Surface unit price as a wholesale per-unit anchor** `[medium | additive | low]`  
  For a B2B/wholesale audience, per-unit ('$/100ml', '$/each') pricing is a key purchase-decision signal on case packs. The output is currently wrapped only in <small class="unit-price">; add an optional block param (e.g. label_prefix defaulting to blank) so the merchant can prepend a short 'Per unit:' style prefix before {{ price | unit_price_with_measurement: measurement }} without changing default rendering.

### `snippets/util-autofill-img-size-attr.liquid` — Computes a responsive img sizes attribute for product-grid cards from a minimum card pixel-width and gap.
- 🐛 **Divide-by-zero when card_size resolves to 0** `[medium | moderate | medium]`  
  Line 17 coerces card_size via '| strip | replace: px,'' | plus: 0', so a missing/empty/non-numeric card_size becomes 0, making card_size_with_gap = 0 and 'max_cols = max_breakpoint | divided_by: card_size_with_gap' (line 32) a divide-by-zero. Its caller card-gallery.liquid feeds card_size from util-product-grid-card-size, which can echo empty on an unexpected product_card_size (no else branch), so this is reachable. Guard: 'if card_size_with_gap < 1 then assign card_size_with_gap = 260' (or echo '100vw' and break) before line 32.

### `snippets/util-mega-menu-img-sizes-attr.liquid` — Calculates the responsive img sizes attribute for mega-menu images based on menu content type, page width, and grid config.
- 🐛 **No default branch on settings.page_width leaves breakpoint/page_max_width unset** `[medium | moderate | medium]`  
  The 'case settings.page_width' (lines 34-44) only handles 'narrow'/'normal'/'wide'. If page_width is unset or a future value, page_max_width and breakpoint stay undefined and the emitted '(min-width: ) calc(( - 80px - ...) ...)' sizes string is malformed. Add an 'else' assigning the 'normal' defaults (page_max_width '120rem', breakpoint '125rem').
- 🐛 **cols_per_item/grid_desktop unset for unknown menu_content_type** `[low | moderate | medium]`  
  The 'case menu_content_type' (lines 54-67) only sets cols_per_item and grid_desktop for 'collection_images' and 'featured_products'; 'featured_collections' exits early at line 29, but any other value leaves items_desktop = grid_desktop | divided_by: cols_per_item (line 70) dividing by nil. Add an else defaulting cols_per_item: 1 and grid_desktop: grid_columns | default: 6.

### `snippets/util-product-grid-card-size.liquid` — Outputs the minimum product-card pixel size for collection/search product grids based on layout type, grid width, and card-size setting.
- 🐛 **case product_card_size has no else, can echo empty string** `[medium | moderate | medium]`  
  Both 'case section.settings.product_card_size' blocks (lines 21-30 and 33-42) only handle small/medium/large/extra-large. If product_card_size is unset or a future value, product_card_size stays nil and this snippet echoes empty, which downstream (card-gallery.liquid -> util-autofill-img-size-attr) coerces to 0 and divides by zero. Add an 'else' fallback (e.g. '260px' full-width / '250px' centered) in each case.

### `snippets/util-product-media-sizes-attr.liquid` — Calculates the responsive img sizes attribute for product-gallery media across single-column, grid, and center-aligned layouts.
- 🐛 **No default branch on settings.page_width in center-aligned mode** `[medium | moderate | medium]`  
  Within the content-center-aligned branch, 'case settings.page_width' (lines 70-83) only handles 'narrow'/'normal'/'wide'. An unset/future page_width leaves breakpoint and media_base_size_* undefined, producing a malformed '(min-width: ) ... ' sizes string and breaking srcset selection. Add an 'else' assigning the 'normal' values.

### `snippets/variant-main-picker.liquid` — Renders the main product variant picker (buttons, swatches, dropdowns) with BSS Login variant-locking integration.
- 🐛 **Dropdown branch never renders due to setting-value mismatch** `[high | moderate | high]`  
  The swatch/button branch checks `block_settings.variant_style == 'buttons'` (line 72) while the dropdown branch checks `block_settings.variant_style == 'dropdowns'` (line 160), but line 45 tests the setting against the singular `'dropdown'`. If the schema value is the singular 'dropdown', the `elsif block_settings.variant_style == 'dropdowns'` branch is unreachable and options with 2+ values fall through to nothing rendered. Confirm the schema option value string and align lines 45/160 (or 72) so the intended dropdown output actually renders.
- 🎛️ **No merchant control over sold-out variant handling text/behavior** `[medium | additive | high]`  
  Add an opt-in block setting (default off) e.g. `hide_sold_out_options` that, when enabled, skips rendering unavailable `product_option_value` entries in the buttons/dropdown loops. For a wholesale catalog with many discontinued SKUs this declutters the picker without changing the current default rendering.
- ♿ **Sold-out radios use aria-disabled but remain operable** `[medium | moderate | high]`  
  In the button branch, unavailable option values set only `aria-disabled="true"` (line 100) on the radio input without the native `disabled` attribute, so keyboard/AT users can still select a sold-out variant. Add a visually-distinct state and consider gating selection, or at minimum ensure the sold-out aria-label suffix (line 98) is reliably announced.

### `snippets/variant-swatches.liquid` — Renders a collection/search card swatch picker inside swatches-variant-picker-component with overflow-list and hover media preview.
- ♿ **Swatch radio aria-label omits sold-out state** `[medium | additive | high]`  
  The swatch `<input>` aria-label is just `{{ product_option_value.name }}` (line 159) with no sold-out suffix, unlike the main picker which appends the sold_out translation. Append `{% if product_option_value.available == false %} - {{ 'products.product.sold_out' | t }}{% endif %}` so AT users on collection/search cards know an option is unavailable. Also escape the name: `| escape`.
- ♿ **Overflow 'show more' button has no discernible visible affordance for count context** `[low | additive | high]`  
  The `.hidden-swatches__count` button renders its `+N` via CSS `::before` counter (line 59-65) with an aria-label of 'show_all_options'; screen readers get the generic label but not the hidden count. Consider surfacing the numeric overflow count in the accessible name (e.g. append the `--overflow-count` value) so the announced control matches the visible `+N`.

### `snippets/video.liquid` — Renders a Shopify video object or external YouTube/Vimeo URL as a deferred-media element with poster image and play/pause controls.
- 🐛 **YouTube autoplay-off branch still appends autoplay when preview image present** `[medium | moderate | medium]`  
  In the URL/YouTube branch, `if video_autoplay == false and video_preview_image != blank` appends `&autoplay=1` (lines 47-49), and the same in the Vimeo branch (66-68). This forces autoplay for the deferred (click-to-play) iframe even though the intent is no-autoplay, causing the video to start immediately once the template is injected. Verify this is intentional; if not, remove the `&autoplay=1` from the autoplay==false case.
- ♿ **Placeholder poster image can render with empty alt** `[low | additive | medium]`  
  The no-video placeholder branch renders `video_preview_image | image_tag` (line 194) without passing an `alt`, producing an empty alt for a meaningful poster. Pass `alt: video_alt` (already computed at line 31) to the placeholder poster image_tag for parity with a described video.
- ⚡ **Poster image loading attribute defaults to eager (no lazy fallback)** `[medium | moderate | medium]`  
  The poster `image_tag` passes `loading: loading` (lines 143, 194) but `loading` is never defaulted, so when a caller omits it the attribute is empty and the browser treats it as eager. Add `assign loading = loading | default: 'lazy'` in the top liquid block so below-the-fold videos don't block LCP; keep callers able to pass `'eager'` for hero placement.

### `snippets/volume-pricing-info.liquid` — Renders a popover showing quantity rules and volume/case-pack price-break tiers per variant, highlighting the active tier.
- 💸 **Volume tiers show per-each price but not the buyer's savings vs base** `[high | additive | medium]`  
  The table rows print `price/each` for each break (lines 103, 127) but never surface the discount magnitude, which is the primary purchasing motivator for trade/case-pack buyers. Add an opt-in savings badge (e.g. compute `variant.price | minus: price_break.price` and show 'Save X% / each') next to the `.volume-pricing-info__price`, gated behind a new default-off `settings.show_volume_savings` so defaults are unchanged.
- 🎛️ **Popover max-width and label are hard-coded with no merchant setting** `[low | additive | medium]`  
  `--volume-pricing-popover-max-width: 320px` (line 175) and the presence of the 'Volume Pricing Available' label depend only on the `show_label` render param. Expose an opt-in schema setting on the calling block for the label text/threshold display so merchants can tune wholesale messaging without editing the snippet; keep current 320px/label-off behavior as defaults.
- ♿ **Price-break table uses div rows with no table/list semantics** `[medium | moderate | medium]`  
  The pricing tiers are a series of `<div class="volume-pricing-info__row">` (lines 98-135) with quantity/price spans; there is no table or list markup, so AT users get no row/column association between quantity and price. Wrap in a semantic `<table>` (or role=table with row/cell roles) so the qty-to-price mapping is conveyed.

