# Horizon Theme 3.5.1 — Developer Reference

**Theme:** Horizon by Shopify  
**Version:** 3.5.1  
**Format:** Shopify 2.0 (JSON templates + block system)

---

## Folder Structure

```
prodip-sarker-48-teststore/
│
├── assets/              # 113 files — CSS, JS, SVG icons
│   ├── base.css         # Main stylesheet
│   ├── overflow-list.css
│   ├── template-giftcard.css
│   ├── *.js             # 75 JS files (web components, utilities)
│   └── icon-*.svg       # 33 SVG icons
│
├── blocks/              # 93 liquid files — modular UI components
│   ├── _*.liquid        # Private/internal components (underscore prefix)
│   └── *.liquid         # Public blocks (selectable in admin)
│
├── config/              # Theme settings
│   ├── settings_schema.json   # Defines all admin-editable settings
│   └── settings_data.json     # Saved setting values
│
├── layout/              # Base page wrappers
│   ├── theme.liquid     # Main layout (header, footer, scripts)
│   └── password.liquid  # Password-protected store layout
│
├── locales/             # 51 files — translations for 31 languages
│   ├── en.json          # English strings (default)
│   ├── *.json           # Other language strings
│   └── *.schema.json    # Admin interface translations
│
├── sections/            # 41 liquid files — page sections
│   ├── header.liquid
│   ├── footer.liquid
│   ├── main-*.liquid    # Page-specific main content areas
│   └── *.json           # Section group configs
│
├── snippets/            # 103 liquid files — reusable components
│   └── *.liquid
│
└── templates/           # 13 files — page type definitions
    ├── *.json           # Shopify 2.0 JSON templates (visual builder)
    └── gift_card.liquid # Legacy liquid template
```

---

## Key Files Quick Reference

| What | File |
|------|------|
| Page wrapper | `layout/theme.liquid` |
| CSS variables / theming | `snippets/theme-styles-variables.liquid` |
| Main stylesheet | `assets/base.css` |
| All admin settings | `config/settings_schema.json` |
| Saved setting values | `config/settings_data.json` |
| Header | `sections/header.liquid` |
| Footer | `sections/footer.liquid` |
| Product page | `templates/product.json` + `sections/product-information.liquid` |
| Collection page | `templates/collection.json` + `sections/main-collection.liquid` |
| Homepage | `templates/index.json` |
| Cart | `sections/main-cart.liquid` |
| Blog list | `sections/main-blog.liquid` |
| Blog post | `sections/main-blog-post.liquid` |

---

## Block Naming Convention

```
blocks/_component.liquid   → Private/internal — NOT shown in admin block picker
blocks/component.liquid    → Public — available in Shopify admin block picker
```

Same pattern applies in `snippets/`.

---

## Liquid Usage Guide

### Basic Output

```liquid
{{ product.title }}
{{ product.price | money }}
{{ settings.logo }}
```

### Conditionals

```liquid
{% if product.available %}
  <button>Add to cart</button>
{% elsif product.variants.size > 1 %}
  <p>Select a variant</p>
{% else %}
  <p>Sold out</p>
{% endif %}
```

### Loops

```liquid
{% for variant in product.variants %}
  <option value="{{ variant.id }}">{{ variant.title }}</option>
{% endfor %}

{% for product in collection.products limit: 8 %}
  {% render 'product-card', product: product %}
{% endfor %}
```

### Render Snippets

```liquid
{% render 'snippet-name' %}

{% render 'product-card', product: product, show_vendor: true %}

{% render 'price', product: product, use_variant: true %}
```

### Sections and Blocks

```liquid
{% for block in section.blocks %}
  {% case block.type %}
    {% when 'text' %}
      <p>{{ block.settings.text }}</p>
    {% when 'image' %}
      {{ block.settings.image | image_url: width: 800 | image_tag }}
  {% endcase %}
{% endfor %}
```

### CSS / Style Tags in Liquid

```liquid
{% stylesheet %}
  .my-element {
    color: {{ section.settings.text_color }};
  }
{% endstylesheet %}
```

### Schema Block (sections and blocks only)

```liquid
{% schema %}
{
  "name": "Section Name",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "Hello"
    }
  ],
  "blocks": [
    {
      "type": "item",
      "name": "Item",
      "settings": []
    }
  ],
  "presets": [
    {
      "name": "Section Name"
    }
  ]
}
{% endschema %}
```

### Common Filters

```liquid
{{ product.price | money }}                    → $10.00
{{ image | image_url: width: 800 | image_tag }}
{{ 'some-string' | t }}                        → translated string
{{ product.description | truncatewords: 20 }}
{{ 'now' | date: '%Y' }}
{{ string | handleize }}                       → url-safe slug
{{ array | join: ', ' }}
```

### Translation (i18n)

```liquid
{{ 'products.product.add_to_cart' | t }}
{{ 'cart.general.title' | t }}

{% # With variables %}
{{ 'products.product.inventory_count' | t: count: product.variants.first.inventory_quantity }}
```

Translation strings live in `locales/en.json` (and other language files).

### Access Theme Settings

```liquid
{{ settings.logo }}
{{ settings.color_background }}
{{ section.settings.heading }}
{{ block.settings.text }}
```

### Image Handling

```liquid
{{ product.featured_image | image_url: width: 800 | image_tag: loading: 'lazy' }}

{% # Responsive srcset %}
{{ image | image_url: width: 800 | image_tag:
  loading: 'lazy',
  widths: '200, 400, 600, 800',
  sizes: '(min-width: 768px) 50vw, 100vw'
}}
```

### URL Helpers

```liquid
{{ product.url }}
{{ collection.url }}
{{ routes.cart_url }}
{{ routes.search_url }}
{{ routes.root_url }}
```

---

## How to Add or Edit Content

### Add New Section

1. Create `sections/your-section.liquid`
2. Add HTML + Liquid
3. Add `{% schema %}` block at bottom with settings
4. Add to a template JSON file or use admin customizer

Example minimal section:
```liquid
<div class="your-section">
  <h2>{{ section.settings.heading }}</h2>
</div>

{% schema %}
{
  "name": "Your Section",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "My Section"
    }
  ],
  "presets": [{ "name": "Your Section" }]
}
{% endschema %}
```

### Add New Block

1. Create `blocks/your-block.liquid`
2. Write block HTML + schema
3. Reference it in a section schema under `"blocks"`

### Add New Snippet

1. Create `snippets/your-snippet.liquid`
2. Call it anywhere with `{% render 'your-snippet' %}`
3. Pass variables: `{% render 'your-snippet', product: product %}`

### Add Translation String

1. Open `locales/en.json`
2. Add key inside relevant object:
```json
{
  "products": {
    "product": {
      "your_new_key": "Your text here"
    }
  }
}
```
3. Use in Liquid: `{{ 'products.product.your_new_key' | t }}`
4. Repeat for other locale files if needed

### Edit Theme Colors / Typography

- Via admin: Customize → Theme settings
- Via file: `config/settings_data.json` (raw values)
- CSS variables defined in: `snippets/theme-styles-variables.liquid`

### Edit Global Layout

- Header changes → `sections/header.liquid` + `blocks/_header-*.liquid`
- Footer changes → `sections/footer.liquid` + related blocks
- Global wrapper → `layout/theme.liquid`

### Edit Styles

- Global CSS → `assets/base.css`
- Section-specific CSS → inside section file using `{% stylesheet %}`
- CSS variables → `snippets/theme-styles-variables.liquid`

---

## Shopify CLI Commands

```bash
# Start dev server (live preview)
shopify theme dev

# Push theme to store
shopify theme push

# Pull latest from store
shopify theme pull

# Check theme for errors
shopify theme check

# Package theme as zip
shopify theme package
```

---

## Template Types (JSON 2.0 vs Legacy)

| Type | Format | When |
|------|--------|------|
| Modern | `templates/*.json` | All page types — supports visual editor |
| Legacy | `templates/*.liquid` | Only `gift_card.liquid` remaining |

JSON templates reference sections by name. To add a section to a page, edit the JSON file or use the Shopify admin customizer.

Example `templates/page.json`:
```json
{
  "sections": {
    "main": {
      "type": "main-page",
      "settings": {}
    }
  },
  "order": ["main"]
}
```

---

## Update Checklist — Before Making Changes

- [ ] Identify which file owns the element (section / block / snippet)
- [ ] Check if element is set via `settings_schema.json` (admin-configurable)
- [ ] Check for related JS file in `assets/` (same name pattern)
- [ ] Check translations in `locales/en.json` for any text strings
- [ ] Test on mobile breakpoints
- [ ] Run `shopify theme check` for Liquid errors

---

## Common Gotchas

- `_blocks.liquid` in `sections/` and `snippets/` is the block rendering orchestrator — don't confuse with individual block files in `blocks/`
- Blocks prefixed with `_` (e.g. `blocks/_header-logo.liquid`) are internal components, not meant to be added directly in admin
- `settings_data.json` is auto-generated by Shopify — avoid manual edits unless necessary
- `gift_card.liquid` is the only legacy `.liquid` template — all others are `.json`
- Snippets cannot have their own schema — only sections and blocks can
- Liquid `render` tag has isolated scope — variables don't inherit automatically, pass them explicitly
