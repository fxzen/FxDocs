# Fx Design System
**Version:** 1.0

---

## Design Direction

The Fx design language should stay intentionally in the middle.

Avoid both extremes:

* Too technical or developer-tooling driven: terminal aesthetics, cyber styling, dense engineering UI, excessive sharp edges, overuse of monospace, dark hacker tone.
* Too soft or consumer-mobile: oversized controls, overly large radii, excessive whitespace, bubbly presentation, playful illustration-heavy surfaces.

Target a modern professional SaaS workspace suitable for a broad audience.

### Desired Qualities

* Clean
* Contemporary
* Calm
* Efficient
* Trustworthy
* Premium
* Data-first
* Fast to scan
* Minimal visual noise

Reference the maturity and restraint of products like Linear, Atlassian, Stripe Dashboard, and Vercel Dashboard, without copying them.

### Interaction And Visual Rules

* Prefer subtle over flashy.
* Use whitespace deliberately.
* Use color for meaning, not decoration.
* Maintain strong visual hierarchy.
* Components should feel light and crisp.
* Rounded corners should be moderate, generally `6px` to `8px`, with larger radii only where appropriate.
* Shadows should be soft and minimal.
* Borders should do most of the structural work.
* Animations should be quick and understated.
* Typography should carry hierarchy more than color.

### FxUI Library Rule

Every `FxUI` component should be generic enough to be reused across future products while still expressing the Fx design language through tokens, spacing, typography, and composition, not through one-off styling or product-specific logic.

When choosing between two visual approaches, prefer the one that is simpler, calmer, and more timeless.

### Where things live

* `src/app/globals.css` — visual **token values** (colors, CSS variables, light/dark).
* `src/lib/FxTheme.js` — visual + **layout recipes** (`FX_*`) **and shell geometry values** (viewport/header/footer heights, sidebar widths, content padding, and the `THEMES` ids).
* `src/lib/FxConstants.js` — **app/product constants** only (`APP_NAME`, `ROUTES`, `STORAGE_KEYS`, nav ids, product enums).

---

## 🎨 Colors

Color **values** are not duplicated here. The canonical token definitions (light and dark) live in `src/app/globals.css`; a living `/ds` playground will render every token from the running theme once built.

This section owns color *intent* only:

* Use color for meaning, not decoration.
* `--fx-primary` (Hero Brand Blue) is the single interactive brand action color: primary buttons, active navigation, links, CTAs.
* `--fx-accent` (Indigo) is a secondary accent for AI actions, highlights, and premium moments — never a general-purpose second button color.
* Surfaces, borders, and text tokens should carry hierarchy through structure and contrast, with borders doing most of the structural work.
* Semantic tokens (`--fx-success`, `--fx-warning`, `--fx-danger`, `--fx-info`, `--fx-info-surface`) are reserved for state, not styling.

---

## Typography

**Font Family:** `Inter, system-ui, sans-serif`

### Type Scale
| Role | Size | Weight | Usage |
| :--- | :--- | :--- | :--- |
| **Page Title** | `28px` | `600` | Page headings |
| **Section Title** | `20px` | `600` | Sections |
| **Card Title** | `16px` | `600` | Cards, sheets, dialogs |
| **Body** | `14px` | `400` | Default text |
| **Clickable** | `14px` | `500` | Links, clickable values |
| **Meta** | `13px` | `400` | Helper text |
| **Label** | `12px` | `500` | Labels, table headers |

---

## 📐 Dimensions & Layout

### Radius
| Token | Value |
| :--- | :--- |
| `Sm` | `6px` |
| `Md` | `8px` |
| `Lg` | `12px` |
| `Pill` | `999px` |

### Spacing
* **Base Unit:** `8px`
* **Scale:** `8px`, `16px`, `24px`, `32px`, `48px`, `64px`, `96px`

### Control Heights
| Control | Height |
| :--- | :--- |
| **Input** | `40px` |
| **Select** | `40px` |
| **Button Sm** | `30px` |
| **Button Md** | `34px` |
| **Button Lg** | `40px` |
| **Button Xl** | `44px` |

### Button Min-Widths (text buttons)
Text buttons carry a **minimum width** so short labels (Edit, Ok, Save) don't render as tiny/squarish boxes. `FxButton` applies this automatically to non-`iconOnly` buttons — **icon-only buttons stay square** (sized by `iconOnlySizeClasses`). Source of truth, once `FxButton` is built: `FX_BUTTON.minWidth` in `src/lib/FxTheme.js`. Do not hardcode widths at call sites; rely on the size prop.

| Size | Min width |
| :--- | :--- |
| `xs` | `48px` |
| `sm` | `64px` |
| `md` | `80px` |
| `lg` | `96px` |
| `xl` | `112px` |

### Sheet Widths
| Token | Width | Multiplier | Usage |
| :--- | :--- | :--- | :--- |
| `Sm` | `512px` | 64 × 8px | Detail flyouts, login/auth window components, small forms |
| `Md` | `768px` | 96 × 8px | Standard data tables, configuration panels |
| `Lg` | `1024px` | 128 × 8px | Full screen viewport split-sheets, metrics dashboards |
| `Xl` / `Book-Sheet` | `1184px` | 148 × 8px | Dense workbook grids, multi-column analytics viewports |
