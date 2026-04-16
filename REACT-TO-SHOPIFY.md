# React → Shopify: Mental Model for React Developers

If you know React, you already understand Shopify themes. Map your existing knowledge:

---

## Concept Mapping

| React | Shopify Equivalent | Where |
|-------|-------------------|-------|
| Component file `.jsx` | Section file | `sections/banner.liquid` |
| Props | `section.settings` | Set in Customizer, used in Liquid |
| Children / slots | `section.blocks` | Merchant adds/removes in Customizer |
| `useState` / UI state | Vanilla JS | `assets/banner.js` |
| CSS Modules / styled-components | Inline stylesheet or global CSS | `{% stylesheet %}` or `assets/base.css` |
| `ReactDOM.render()` | Automatic — Shopify does it | No manual mounting needed |
| TypeScript prop types / interface | `{% schema %}` block | Bottom of every section file |
| `npm start` / hot reload | `shopify theme dev` | Terminal command |
| Sub-component | Snippet | `snippets/my-snippet.liquid` |
| `import Component from './Component'` | `{% render 'snippet-name' %}` | Inside any liquid file |
| Passing props to component | `{% render 'snippet', product: product %}` | Explicit — no auto scope |
| `children` prop | `{{ block.settings.xxx }}` inside block loop | Via `section.blocks` |
| Context / global state | `settings.*` | From `config/settings_data.json` |
| `useEffect` on mount | `document.addEventListener('DOMContentLoaded')` | In JS file |
| Component library | Snippets folder | `snippets/` — 103 reusable pieces |

---

## React Component vs Shopify Section — Side by Side

### React
```jsx
// Banner.jsx
export default function Banner({ heading, buttonText, bgImage }) {
  return (
    <div style={{ backgroundImage: `url(${bgImage})` }}>
      <h1>{heading}</h1>
      <button>{buttonText}</button>
    </div>
  );
}
```

### Shopify
```liquid
<!-- sections/custom-banner.liquid -->
<div style="background-image: url('{{ section.settings.bg_image | image_url: width: 1920 }}')">
  <h1>{{ section.settings.heading }}</h1>
  <button>{{ section.settings.button_text }}</button>
</div>

{% schema %}
{
  "name": "Custom Banner",
  "settings": [
    { "type": "image_picker", "id": "bg_image",    "label": "Background Image" },
    { "type": "text",         "id": "heading",     "label": "Heading",     "default": "Hello" },
    { "type": "text",         "id": "button_text", "label": "Button Text", "default": "Shop Now" }
  ],
  "presets": [{ "name": "Custom Banner" }]
}
{% endschema %}
```

---

## React State / Events vs Shopify JS

### React
```jsx
const [open, setOpen] = useState(false);
<button onClick={() => setOpen(true)}>Open Popup</button>
{open && <Popup />}
```

### Shopify
```html
<!-- In .liquid file -->
<button id="open-popup">Open Popup</button>
<div id="popup" style="display:none">...</div>
```
```javascript
// In assets/banner.js
document.getElementById('open-popup').addEventListener('click', () => {
  document.getElementById('popup').style.display = 'block';
});
```

---

## How Settings Flow (like Props)

```
Merchant types in Customizer
        ↓
Shopify saves to settings_data.json
        ↓
Liquid renders: {{ section.settings.heading }}
        ↓
Browser gets plain HTML string
```

No re-render. No virtual DOM. Full page is server-rendered HTML.

---

## Blocks = Dynamic Children (like .map() over props.children)

```liquid
{% for block in section.blocks %}
  {% case block.type %}
    {% when 'text' %}
      <p {{ block.shopify_attributes }}>{{ block.settings.text }}</p>
    {% when 'image' %}
      {{ block.settings.image | image_url: width: 800 | image_tag }}
  {% endcase %}
{% endfor %}
```

Merchant can add/remove/reorder blocks in Customizer. Like configurable children.

---

## File Creation Rules

| Want to create | Do this |
|---------------|---------|
| New page section | Create `sections/my-section.liquid` |
| Reusable component | Create `snippets/my-component.liquid` |
| Block (configurable child) | Create `blocks/my-block.liquid` |
| Add JS interactivity | Create `assets/my-section.js`, load in section file |
| Add styles | Write inside `{% stylesheet %}` in section OR add to `assets/base.css` |

Shopify auto-discovers all files — no imports, no registration needed.

---

## Dev Commands

```bash
# Install CLI
npm install -g @shopify/cli @shopify/theme

# Start live dev server (like npm start)
shopify theme dev --store prodip-sarker-48-teststore.myshopify.com

# Push local changes to store
shopify theme push --store prodip-sarker-48-teststore.myshopify.com

# Pull latest from store
shopify theme pull --store prodip-sarker-48-teststore.myshopify.com

# Check for Liquid errors
shopify theme check
```

---

## Liquid Cheat Sheet

```liquid
{{- variable -}}               Output variable (dashes trim whitespace)
{{ product.title }}            Object property
{{ price | money }}            Filter (like JS pipe/transform)
{{ 'key' | t }}                Translation string

{% if condition %}{% endif %}  Conditional
{% for x in array %}{% endfor %} Loop
{% render 'snippet-name' %}    Include snippet
{% render 'snippet', var: val %} Include with variable
{% assign x = 'value' %}       Create variable
{% capture x %}...{% endcapture %} Capture HTML into variable

{{ section.settings.my_id }}   Section setting
{{ block.settings.my_id }}     Block setting
{{ settings.color_background }} Global theme setting
```

---

## Common Liquid Filters (like JS methods)

```liquid
{{ string | upcase }}                    → "HELLO"
{{ string | downcase }}                  → "hello"
{{ string | truncatewords: 10 }}         → first 10 words
{{ string | strip_html }}                → remove HTML tags
{{ array | join: ', ' }}                 → "a, b, c"
{{ array | size }}                       → array length
{{ number | plus: 5 }}                   → number + 5
{{ image | image_url: width: 800 }}      → sized image URL
{{ image_url | image_tag }}              → <img> tag
{{ product.price | money }}              → "$10.00"
{{ 'string' | handleize }}               → "url-safe-slug"
```

---

## Key Differences from React

1. **No virtual DOM** — Liquid renders on server, browser gets static HTML
2. **No component state** — use vanilla JS for any dynamic behavior
3. **No imports** — snippets are included with `{% render %}`, not imported
4. **Scope is isolated** — variables don't pass into `{% render %}` automatically, must pass explicitly
5. **One file = one section** — HTML + CSS + JS loading + schema all in one `.liquid` file
6. **Schema drives Customizer** — `{% schema %}` at bottom defines what merchant can edit
7. **Blocks are optional children** — merchant adds/removes/reorders in visual editor

---

## This Store

- **Store:** `prodip-sarker-48-teststore.myshopify.com`
- **Theme:** Horizon 3.5.1 by Shopify
- **Format:** Shopify 2.0 (JSON templates)
- **Total files:** 417 (238 Liquid, 75 JS, 3 CSS, 33 SVG)
