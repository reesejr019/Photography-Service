# SnapShot Photography Studio — Build Plan & Conversation Log

## Project Overview

**SnapShot Photography Studio** is a multi-page static website for a fictional photography studio. It is built with plain HTML and CSS (no frameworks), with client-side JavaScript for interactive features backed by the browser's `localStorage` API. No backend or build tooling is required.

---

## Site Structure

```
exercise2/
├── index.html          # Home page
├── about.html          # About Us page
├── services.html       # Services & Pricing page
├── contact.html        # Contact form page
├── reviews.html        # Client reviews (localStorage-backed)
├── gallery.html        # Full portfolio + community uploads (localStorage-backed)
├── styles.css          # Single shared stylesheet
└── docs/
    └── BUILD-PLAN.md   # This file
```

---

## Pages & Features

### Existing Pages (pre-gallery)

| Page | Key Content |
|------|-------------|
| `index.html` | Hero section, features grid, 6-image gallery preview, testimonials, CTA |
| `about.html` | Studio story, mission & values, team grid, stats section |
| `services.html` | 6 service cards, 3 wedding pricing tiers (Essential / Premium / Luxury), FAQ |
| `contact.html` | Contact info sidebar, enquiry form, map placeholder |
| `reviews.html` | Star-rating form, user reviews persisted in `localStorage` (`snapshot_reviews`), 3 seed reviews always shown |

### Gallery Page (added)

See [Gallery Page Plan](#gallery-page-plan) below for full detail.

---

## Shared Conventions

- **CSS variables** defined in `:root` on `styles.css`:
  - `--primary-color: #1a1a2e`
  - `--secondary-color: #16213e`
  - `--accent-color: #e94560`
  - `--light-color: #f5f5f5`
  - `--text-color: #333`
  - `--text-light: #666`
- **Navigation** — fixed header with `.nav-links`; active page link carries `class="active"` for underline + accent colour
- **Mobile nav** — hamburger toggle via `toggleMenu()` (inline `<script>` on every page); `.nav-links` slides in via CSS transform
- **Footer** — four-column grid: Studio blurb | Quick Links | Services | Contact Info
- **Images** — all sourced from Unsplash with `?w=600` (thumbnails) or `?w=800`/`?w=1920` (hero/about)
- **localStorage** — two independent keys:
  - `snapshot_reviews` — user-submitted reviews on `reviews.html`
  - `snapshot_gallery` — user-uploaded photos on `gallery.html`

---

## Gallery Page Plan

### Goal

Allow visitors to browse the full studio portfolio (expanded from the 6-image home-page preview) and optionally share their own photos received from SnapShot. Works entirely client-side — no backend required.

### Sections

1. **Page Header** — "Our Gallery" heading with descriptive subtext
2. **Studio Portfolio** — filterable 12-image grid
3. **Share Your Photo** — upload form (base64 via Canvas API)
4. **Community Photos** — cards rendered from `localStorage`

### Studio Portfolio — 12 Images

| # | Label | Category (`data-category`) | Unsplash ID |
|---|-------|---------------------------|-------------|
| 1 | Wedding Photography | `wedding` | `photo-1519741497674-611481863552` |
| 2 | Wedding Photography | `wedding` | `photo-1606216794074-735e91aa2c92` |
| 3 | Wedding Photography | `wedding` | `photo-1583939003579-730e3918a45a` |
| 4 | Portrait Sessions | `portrait` | `photo-1531746020798-e6953c6e8e04` |
| 5 | Portrait Sessions | `portrait` | `photo-1559136555-9303baea8ebd` |
| 6 | Portrait Sessions | `portrait` | `photo-1507003211169-0a1dd7228f2d` |
| 7 | Event Coverage | `event` | `photo-1540575467063-178a50c2df87` |
| 8 | Event Coverage | `event` | `photo-1492684223066-81342ee5ff30` |
| 9 | Product Photography | `product` | `photo-1441986300917-64674bd600d8` |
| 10 | Commercial Photography | `commercial` | `photo-1486401899868-0e435ed85128` |
| 11 | Engagement Sessions | `engagement` | `photo-1511285560929-80b456fea0bc` |
| 12 | Engagement Sessions | `engagement` | `photo-1529634806980-85c3dd6d34ac` |

### Filter Buttons

| Label | `data-filter` |
|-------|---------------|
| All | `all` |
| Weddings | `wedding` |
| Portraits | `portrait` |
| Events | `event` |
| Product | `product` |
| Commercial | `commercial` |
| Engagements | `engagement` |

"All" starts active. Clicking a button toggles `.active` on that button and shows/hides `.gallery-item` elements by matching `data-category` to `data-filter`.

### Upload Form Fields

| Field | Input Type | Notes |
|-------|-----------|-------|
| Your Name | `text` | `required` |
| Photo Caption | `text` | `required`, short description |
| Upload Photo | `file` | `accept="image/*"`, `required`; triggers canvas resize + preview on `change` |

### JavaScript Logic (`gallery.html` inline `<script>`)

- **`toggleMenu()`** — hamburger nav toggle (same as all pages)
- **Filter logic** — `.filter-btn` click: toggle `.active`, show/hide `.gallery-item` via `display: block/none`
- **`loadPhotos()` / `savePhotos()`** — read/write `snapshot_gallery` in `localStorage`
- **`resizeImage(file, callback)`** — `FileReader` → `new Image()` → `<canvas>` scales longest side to max 1200 px → `canvas.toDataURL('image/jpeg', 0.82)` → callback
- **File input `change`** — calls `resizeImage`, updates preview `src`, shows `.photo-preview-wrap`
- **`renderCommunityPhotos()`** — clears `#community-grid`; shows placeholder if empty; otherwise builds `.community-card` elements newest-first
- **Form `submit`** — `preventDefault`, resizes image, builds photo object, saves to `localStorage`, re-renders grid, resets form, hides preview, shows `#gallery-success` for 5 s

### localStorage Schema (`snapshot_gallery`)

```json
[
  {
    "id": "photo-<timestamp>",
    "name": "Client Name",
    "caption": "Short description",
    "imageData": "data:image/jpeg;base64,...",
    "date": "YYYY-MM-DD"
  }
]
```

### New CSS Classes (appended to `styles.css`)

| Class | Purpose |
|-------|---------|
| `.gallery-filter-bar` | Flex row, centered, wrapping, `gap: 0.75rem`, `margin-bottom: 2.5rem` |
| `.filter-btn` | Pill button — outline accent by default; filled accent + white on `.active` / hover |
| `.community-grid` | Same auto-fit grid as `.gallery-grid` (`minmax(300px, 1fr)`, `gap: 1.5rem`) |
| `.community-card` | White card, `border-radius: 10px`, shadow, `overflow: hidden` |
| `.community-card img` | Full width, `aspect-ratio: 4/3`, `object-fit: cover` |
| `.community-card-info` | `padding: 1rem 1.25rem` |
| `.community-card-info h4` | Primary colour, small bottom margin |
| `.community-card-info p` | Light text, `font-size: 0.9rem` |
| `.gallery-upload-card` | Mirrors `.review-form-card` (white, `max-width: 700px`, centred, shadow) |
| `.photo-preview-wrap` | Hidden by default; shown after file pick; `border-radius: 8px`, `max-height: 300px` |
| `.photo-preview-wrap img` | Full width, `object-fit: contain`, `max-height: 300px` |
| `.gallery-success` | Green success banner (mirrors `.review-success`), hidden by default |

---

## Files Changed Summary

| Action | File | Description |
|--------|------|-------------|
| Edited | `index.html` | Added Gallery to nav + footer Quick Links |
| Edited | `about.html` | Added Gallery to nav + footer Quick Links |
| Edited | `services.html` | Added Gallery to nav + footer Quick Links |
| Edited | `contact.html` | Added Gallery to nav + footer Quick Links |
| Edited | `reviews.html` | Added Gallery to nav + footer Quick Links |
| Edited | `styles.css` | Appended gallery-specific CSS rules |
| Created | `gallery.html` | Full gallery page with portfolio, upload form, community grid |
| Created | `docs/BUILD-PLAN.md` | This document |

---

## Verification Checklist

- [ ] All 6 pages show "Gallery" in nav and footer
- [ ] `gallery.html` nav link has active styling
- [ ] Filter buttons correctly show/hide portfolio images; "All" shows all 12
- [ ] Submitting the form with empty fields triggers HTML5 validation
- [ ] Picking an image file shows an immediate preview
- [ ] Submitting a valid photo: success banner appears, form clears, card appears in Community Photos
- [ ] Refreshing the page: community photo persists via `localStorage`
- [ ] Deleting `snapshot_gallery` in DevTools → reload: community grid shows placeholder message, portfolio still visible
