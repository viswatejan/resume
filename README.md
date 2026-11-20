# Resume Theme & Architecture Guide

A self-contained HTML/CSS/JS resume with local **Nunito** typography, designed for clean screen reading, interactive role collapsing, and deterministic Letter-size PDF/print output.

---

## 1. Design Tokens (`:root`)

All colors are centralized as CSS custom properties at the top of `<style>`:

| Variable | Value | Purpose |
| :--- | :--- | :--- |
| `--primary-color` | `#1a202c` | Name heading, section titles, company names, job titles, bold lead-ins, timeline dots, contact icons |
| `--text-color` | `#2d3748` | Primary body copy and bullet points |
| `--subtext-color` | `#4a5568` | Contact bar text, button labels, education metadata |
| `--muted-color` | `#718096` | Tenure dates, role dates, timeline collapse chevrons |
| `--border-color` | `#cbd5e0` | Section divider lines, vertical timeline track, button borders |
| `--surface-bg` | `#f7fafc` | Subtle background fill for interactive controls |

- **Page Canvas:** `#edf2f7` on screen; overridden to `#ffffff` in `@media print`.
- **Typography:** `"Nunito"` loaded locally via `@font-face` (`./fonts/Nunito-Regular.ttf` [400 normal], `./fonts/Nunito-Italic.ttf` [400 italic], `./fonts/Nunito-SemiBold.ttf` [600 normal], `./fonts/Nunito-Bold.ttf` [700 normal]) with `font-display: swap` and `1.4` base line-height.

---

## 2. Layout Architecture

### Header & Contact Bar
- Centered `<h1>` in uppercase with `1px` letter spacing.
- `.contact-info` uses a wrapping Flexbox row (`gap: 0.4rem 0.9rem`).
- Each `.contact-item` pairs a `14px × 14px` inline SVG icon with text or a link (`mailto:`, `tel:`, or external URL) and sets `white-space: nowrap` so icons never orphan from their labels.

### Main Grid (`.main-grid`)
- **Screen (`> 768px`):** CSS Grid (`grid-template-columns: 1fr 3fr; gap: 1.25rem;`).
  * `.summary-section` spans both columns (`grid-column: 1 / -1`).
  * `.left-column` holds sidebar sections (**Skills**, **Technology Summary**, **Open-Source**, **Certifications**, **Education**).
  * `.right-column` holds **Work History**.
- **Mobile (`@media screen and (max-width: 768px)`):**
  * Scoped strictly to `screen` so it never triggers during print/PDF export (Letter printable width is `~739px`, which is `< 768px`).
  * Switches `.main-grid` to `display: flex; flex-direction: column;` and reorders columns (`right-column` `order: 1`, `left-column` `order: 2`) so **Work History** appears first on mobile viewports, followed by the sidebar below it.

---

## 3. Work History Timeline & Interactivity

### Self-Adjusting Vertical Timeline
Each role inside `.roles-timeline` uses a two-column grid (`.timeline-role`):
- **Track (`14px`):** `.timeline-track` renders a `7px` circle (`.timeline-dot`). Every role except `:last-child` draws a `2px` vertical connector line via `::after` that stretches automatically to match the role's content height.
- **Content (`minmax(0, 1fr)`):** `.timeline-content` holds `.role-header` and `.role-details`.

### Collapsible Roles
- Clicking `.role-header` toggles `.collapsed` on the parent `.timeline-role`, hiding `.role-details` and rotating `.role-chevron` (`-90deg`).
- Clicking `#toggleAllBtn` (**Hide Details** / **Show Details**) collapses all roles if any are expanded, or expands all if all are collapsed, then syncs the button label.

---

## 4. Print & PDF Pagination Rules (`@media print`)

- **Page Setup:** `@page { size: letter; margin: 0.4in; }` with `print-color-adjust: exact` on `body` so timeline dots and tracks render reliably.
- **Flexbox Switch:** CSS Grid columns do not reliably split across pages in browser print engines. `@media print` switches `.main-grid` to `display: flex; flex-wrap: wrap; align-items: flex-start;` with `flex: 1` on `.left-column` and `flex: 3` on `.right-column` (`min-width: 0` on both).
- **Natural Sidebar Flow:** Do **not** apply `break-inside: avoid` to `.sidebar-section`. Allowing long sidebar blocks (like **Technology Summary**) to break between individual `.tech-category` items prevents the entire section from jumping to page 2.
- **Pagination Guards:**
  * `break-inside: avoid` is applied only to atomic items: `.company-header`, `.role-header`, `.role-details li`, `.tech-category`, and `.education-item`.
  * `break-after: avoid` is applied to `.section-heading`, `.company-header`, and `.role-header` so headings are never stranded at the bottom of a page.
  * `.page-break` (`break-before: page`) is placed on the second `.company-group` to start it cleanly at the top of page 2.
- **Print UI Cleanup:** `.toggle-btn` and `.role-chevron` are hidden (`display: none !important`), and `.role-title-wrap` resets `gap: 0`.

---

## 5. Guidelines for Future Edits

- **Font Weights & Styles:** Stick to defined weights (`400` normal/italic, `600` normal, `700` normal) or add matching `@font-face` rules and `.ttf` files in `./fonts/` before introducing additional weights or bold/semibold italics.
- **Adding a Role to an Existing Company:** Copy a `.timeline-role` block inside `.roles-timeline`. The vertical connector line adjusts automatically; keep the oldest role last so `:not(:last-child)` stops the line at the bottom dot.
- **Adding a New Company:** Copy a `.company-group` block. Add or remove the `.page-break` class depending on where you want page 2 to begin.
- **Adding Technology Categories:** Wrap each category in `<div class="tech-category"><strong>Category:</strong> Items</div>` so print page breaks occur cleanly between categories rather than mid-line.
- **Updating Media Queries:** Always keep mobile overrides inside `@media screen and (...)`—never bare `@media (...)`—to avoid breaking side-by-side print rendering.
