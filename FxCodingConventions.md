# Fx Coding Conventions

## Naming

* Prefix reusable shared components with `Fx`.

  * `FxButton`, `FxSheet`, `FxTable`, `FxAppShell`
* Prefix exported theme/recipe groups with `FX_`.

  * `FX_TYPOGRAPHY`, `FX_SURFACE`, `FX_LAYOUT`, `FX_STATE` (class-recipe groups in `src/lib/FxTheme.js`)
* Use plain exported constants for global app constants.

  * `APP_NAME`, `ROUTES`, `SIDEBAR_WIDTHS`, `NAV_ITEMS`
* Use `@/` imports instead of deep relative paths.

**The `Fx` boundary — the rule behind the prefix, not just the prefix itself.** `Fx` is the reusable design-system layer: generic, product-agnostic, carries no idea what the product's actual domain objects are. The product/domain layer gets its own separate prefix and composes `Fx` primitives into product behavior — never the other way around; an `Fx` component or helper never imports or references product-specific code. This is enforced by convention and file placement, not a lint rule, so it only holds as long as people (and agents) actually keep it — worth restating explicitly in any project that reuses this system, since "generic vs. product" erodes quietly if nobody's watching for it. (Each project picks its own short product prefix — e.g. `Ev`, `Xk` — and documents where that split lives in its own architecture doc.)

---

## Project Structure

* `src/components/ui` → low-level Radix/shadcn primitives — wrapped by `Fx` components, never used raw by product code.
* `src/components/FxUI` → Fx design-system components, organized into domain subfolders (e.g. Forms, Navigation, DataDisplay, Overlays — the actual set is project-specific and will grow; check the project's own architecture doc for the current list rather than assuming this one's example below is exhaustive). Each domain folder exposes an `index.js` barrel.
* `src/lib` → constants, theme, utilities, helpers — same generic-vs-product split as everywhere else (see the `Fx` boundary note above).
* `src/FxDocs` → design system and architecture documentation.

Use domain `index.js` barrels for imports where possible.

Example (folder names below are this project's current domains, illustrative of the pattern, not a fixed list):

```js
import { FxAppShell, FxAppHeader } from "@/components/FxUI/AppShell";
import { FxButton, FxInput } from "@/components/FxUI/Forms";
```

---

## File Header

Every source file starts with:

```js
/* src/path/File.js | Short purpose | Sree | YYYY-MM-DD */
```

---

## Section Divider

Use the standard divider between major logical sections.

```js
/* - - - - - - - - - - - - - - - - */
```

Placement rules:

* Place it after the full import block, not between imports.
* Place it immediately after a function block ends when separating the next major section.
* Do not leave an extra blank line before the divider.
* End source files with the divider.
* Do not overuse it inside short files.

---

## JavaScript

* JavaScript / JSX only.
* Prefer named `function` declarations.
* Keep helpers above the exported component.
* Prefer early returns.
* Keep components focused and reusable.
* **Prefer compact conditional expressions over verbose blocks.** A ternary for picking between two values, `&&` for conditional render/execution, `??`/`?.` for defaults — reach for these before a multi-line `if`/`else` when the logic is simple enough to read cleanly in one line. This isn't "cram everything onto one line" — a ternary that needs its own comment to explain what it's doing should probably be a named variable or an early return instead. The bar is readability at a glance, not line count for its own sake; terse-and-clear beats both verbose-and-clear and terse-and-cryptic.

---

## React

* Server Components by default.
* Add `"use client"` only where interactivity is required.
* Keep client-only logic in small leaf components.

---

## React keys (`key=`)

A key only needs to be unique **among its immediate siblings**, but the traps below have each cost real debugging time.

* **Never seed a remount counter at `0` and use it raw as a key.** `const [k, setK] = useState(0)` → `<Dialog key={k} />` looks fine alone, but two or more sibling elements each doing this all start at `key={0}` on mount → *"Encountered two children with the same key, `0`"*. **Namespace it:** `` key={`reject-${k}`} ``. This is the classic bare-numeric-`0` collision — the key in the error is a real number, not a string.
* **Reading the error's stack:** `updateContextProvider → reconcileChildrenArray` + a **numeric** key means the colliding siblings carry an explicit `key={<number>}` prop (a page's sibling dialogs/panels), **not** a `.map()` fallback. `<tbody>`/host-element row keys show up under a host fiber, not `updateContextProvider` — don't chase the table when the frame says provider.
* **Don't key list rows off data that can drift.** `key={row.id ?? index}` collides when mutated/localStorage data yields a duplicate or nullish id. Ensure the id is genuinely unique at the source; only fall back to index for truly static lists.
* **A `.map()` needs stable, unique, content-derived keys** — never the array index for reorderable/editable lists. Fallback chains are fine but keep them string-prefixed: `` key={item.id ?? `q-${index}`} `` (a bare `?? index` next to a numeric field can still resolve to `0`).

---

## JSX

* Add section comments only for large layout regions.

Example:

```jsx
{/* PAGE HEADER */}

{/* FILTER BAR */}

{/* TABLE */}
```

Avoid commenting every `<div>`.

---

## Styling

* Use Fx theme tokens.
* Never hardcode colors.
* Prefer `cn()` for conditional classes.
* Reuse existing spacing, typography, radius and layout recipes.
* Consume reusable class recipes from `src/lib/FxTheme.js` (`FX_TYPOGRAPHY`, `FX_SURFACE`, `FX_RADIUS`, etc.) instead of re-inlining arbitrary values.
* Responsibility split — `globals.css` owns **visual token values** (colors, CSS variables, light/dark); `src/lib/FxTheme.js` owns **visual + layout recipes AND shell geometry values** (`FX_*` class recipes plus dimension constants like `APP_HEADER_HEIGHT`, `SIDEBAR_WIDTHS`, `APP_CONTENT_PADDING`, and the `THEMES` ids); `src/lib/FxConstants.js` owns **app/product constants only** (`APP_NAME`, `ROUTES`, `STORAGE_KEYS`, nav ids, product enums).
* **Define styling constants at the TOP of the file.** Hoist className strings and style maps (e.g. `LABEL_CLASS`, `TONE_TEXT`, `TONE_BAR`) to named constants near the top — do not scatter the same class inline or bury style maps mid-file.

---

## Component Design

* `ui` = generic primitives.
* `FxUI` = branded reusable components.
* `app` = product composition.

Do not place product-specific logic inside `FxUI`.

**File-count discipline.** Before adding a new component file, check whether an existing component can absorb the need via a prop or variant instead. Prefer one well-designed shared component with props over several near-duplicate or one-off files — this matters more than usual in a from-scratch rebuild, where it's easy to let file count balloon without noticing.

**Enter advances.** Single-line `FxInput`s move focus to the next field on Enter by default (scoped to the nearest dialog/sheet/form). Opt out with `advanceOnEnter={false}` where Enter has its own job (search, inline save-on-Enter editors already win via `preventDefault`). Multi-line entry (`FxTextarea`) is exempt.

**Auto-focus whatever field just became relevant.** Whenever a UI action makes a new field "the one you're about to type into," it should already have the cursor in it — no extra click needed. Four recurring cases: (1) a checkbox/toggle reveals a field — focus it the instant it appears; (2) switching between mutually exclusive input modes (e.g. a Card/UPI selector) — focus that mode's first field on every switch, not just the first time; (3) entering edit mode on a read/edit-toggle card — focus the first editable field, not just swap in the form; (4) opening a sheet/dialog for data entry — the first relevant field gets focus on open, not the panel or its close button. A plain `autoFocus` prop only fires on true first mount, so it won't refire on a later reveal or mode switch — use a ref + effect keyed on the triggering state, calling `.focus()` imperatively. For sheets/dialogs with their own default open-focus behavior (Radix, etc.), override it explicitly (see `SheetContent`'s `onOpenAutoFocus` in `src/components/ui/sheet.js`) rather than racing it with a plain effect, or the library wins and steals focus back. When auditing a screen for this, check every conditionally-rendered field, every mode toggle, and every read→edit switch — each tends to "work fine" in isolation, so it's easy to miss the set as a whole.

**Tooltips.** Hover-hints on controls always use `FxTooltip` (`FxUI/Overlays` — the dark DS tooltip), never the native `title` attribute: `<FxTooltip label="Fit width"><button …/></FxTooltip>`. Keep `aria-label` on the control. When the control is also a Radix trigger, `FxTooltip` wraps the trigger (`<FxTooltip><DropdownMenuTrigger asChild>…`). The `title` attribute is allowed only as a truncated-text overflow affordance on content, never on interactive controls.

**Button label casing.** Default to Title Case — capitalize each word (`Get Started`, `Log In`, `See Pricing`), not sentence case (`Get started`) or lowercase. Exceptions (e.g. ALL CAPS for a specific marketing treatment) are allowed per button when the design deliberately calls for it — override at the call site, don't change the default.

---

## Editing

* Follow the existing file structure before introducing abstractions.
* Keep naming, formatting and comment style consistent.
* Import app-wide constants from `@/lib/FxConstants`.

## DRY & Reuse

* **No duplicated logic.** A small utility used (or likely to be used) in more than one place lives ONCE in a shared module — generic helpers in `src/lib/FxUtils.js`; domain/format helpers in the relevant lib (`EvFormat`, `EvSelectors`, …). Import them; never re-declare per file.
* **Check before writing** a helper — search for an existing one and reuse or extend it.
* **Parameterize small differences** instead of forking near-identical copies — e.g. one `scoreTone(value, fallback)` rather than a copy per consumer.
* Best logic, cleanest shared abstraction over copy-paste; build the reusable piece first, then compose product behavior from it.

## Storage Keys

* Never bake a client/project name (e.g. "acme") into persisted keys — clients change, the code keeps the `Fx` signature.
* There is exactly ONE localStorage key — the app root **`FxID8r`** (`FX_STORAGE_ROOT`) — holding a single JSON object; every setting is a FIELD inside it, so the dev inspector shows just one expandable key.
* Field names live in `STORAGE_KEYS` (`src/lib/FxConstants.js`); read/write them ONLY via `getStored` / `setStored` from `src/lib/FxStorage.js` — never touch `window.localStorage` directly at a call site.
* Keep field names and values terse and self-evident (e.g. `Theme: "light" | "dark"`, `Sidebar-Open: "Y" | "N"`).
