# Resume Theme & Architecture Guide

This repository contains a standalone, zero-dependency HTML/CSS/JS resume designed for clean screen viewing, interactive role detail toggling, and reliable letter-size PDF/print export.

---

## 1. Visual Theme & Design Tokens

Colors, spacing, and typography are centralized in `:root` custom properties and shared utility/semantic classes inside `<style>`:

* **Color Palette:**
  * `--primary-color` (`#1a202c`): Headings, name title, company names, role titles, timeline dots, and links.
  * `--text-color` (`#2d3748`): Primary body copy and bullet text.
  * `--subtext-color` (`#4a5568`): Contact info, secondary metadata (such as degree subtitle), and toggle button text.
  * `--muted-color` (`#718096`): Dates, tenures, and collapse chevrons.
  * `--border-color` (`#cbd5e0`): Section dividers, timeline connector lines, and button borders.
  * `--surface-bg` (`#f7fafc`): Subtle background for interactive controls.
* **Typography:** System UI font stack (`-apple-system`, `BlinkMacSystemFont`, `"Segoe UI"`, `Roboto`, `Helvetica`, `Arial`, `sans-serif`) with `1.4` line-height for compact, scannable density.
* **Container:** Centered card (`max-width: 900px`, `2rem` padding) on screen that flattens to `100%` width with `0` padding/shadow in `@media print` (`@page` margin `0.4in`, `letter`).

---

## 2. Structural Layout

```text
.resume-container
├── <header>                  # Name + inline-flex contact row (email, phone, location, LinkedIn)
└── .main-grid                # Desktop: CSS Grid (3fr 1fr) | Mobile (<=768px): 1fr
    ├── .summary-section      # Full-width top section (spans 1 / -1)
    ├── .left-column          # Work History grouped by company
    │   └── .company-group    # Single-row company header + total tenure
    │       └── .roles-timeline
    │           └── .timeline-role  # 2-column grid: [14px timeline track] [minmax(0, 1fr) content]
    └── .right-column         # Skills, Technology Summary, Open-Source, Certifications, Education
```

### Timeline Mechanics
* Each `.timeline-role` uses a `14px` left track (`.timeline-track`) and a content column (`minmax(0, 1fr)`).
* Each role renders a `7px` `.timeline-dot` aligned with the first line of the role header (`margin-top: 0.42rem`).
* All roles except `:last-child` draw a vertical connector line via `.timeline-role:not(:last-child) .timeline-track::after`. Because the connector is scoped to non-last items, the timeline automatically terminates cleanly at the final role's dot regardless of bullet count or collapsed state.

### Interactive Show/Hide Behavior
* Clicking any `.role-header` toggles `.collapsed` on its parent `.timeline-role`, hiding `.role-details` and rotating `.role-chevron`.
* The `#toggleAllBtn` button in the Work History heading calls `toggleAllDetails()` to collapse all expanded roles or expand all collapsed roles, then updates its label (`Hide Details` / `Show Details`).
* In `@media print`, `.toggle-btn` and `.role-chevron` are hidden so printed output stays clean.

---

## 3. Print & Pagination Conventions

* **Screen vs. Print Grid:** Screen uses CSS Grid (`3fr 1fr`). Print switches `.main-grid` to `display: flex; flex-wrap: wrap; align-items: flex-start;` with `.left-column { flex: 3; min-width: 0; }` and `.right-column { flex: 1; min-width: 0; }` so columns flow naturally across page boundaries without grid-fragmentation gaps.
* **Avoid Splitting Small Headers/Items:** Use `break-inside: avoid` and `break-after: avoid` on `.section-heading`, `.company-header`, and `.role-header`, and `break-inside: avoid` on individual list items (`.role-details li`), `.tech-category`, and `.education-item`.
* **Avoid `break-inside: avoid` on Large Containers:** Do not place `break-inside: avoid` on `.sidebar-section` or multi-bullet role wrappers, as tall blocks can get pushed to the next page even when space is available.
* **Explicit Page Breaks (`.page-break`):** Currently placed on the second company group (`Tata Consultancy Services Ltd.`). If you add or remove content and notice awkward whitespace, review whether `.page-break` is still needed.

---

## 4. Guidelines for Future Enhancements

1. **Adding a New Role Under an Existing Company:**
   * Duplicate a `.timeline-role` block inside `.roles-timeline` in chronological/reverse-chronological order matching the rest of the section.
   * Keep the `.timeline-track` + `.timeline-dot` structure intact; the `:not(:last-child)` rule will automatically handle connector lines.
   * Keep `.role-header` on a single flex row with `white-space: nowrap` on `.job-date` so dates do not wrap awkwardly.
2. **Adding a New Company:**
   * Create a `.company-group` with a `.company-header` containing `.company-name` (left) and `.company-tenure` (right), followed by a `.roles-timeline`.
   * Avoid repeating the company name inside individual role titles unless clarifying a client/division (e.g., `IT Analyst for Walmart`).
3. **Adding Sidebar Sections or Tech Categories:**
   * Wrap new sidebar blocks in `<div class="sidebar-section">` with `<div class="section-heading"><h3>Section Name</h3></div>`.
   * Use `.tech-category` with `<strong>Label:</strong> Value` for grouped technologies so each item stays intact across page breaks.
4. **Editing Styles:**
   * Reuse CSS variables (`--primary-color`, `--text-color`, `--muted-color`, `--border-color`) instead of hardcoding hex values.
   * Avoid inline `style="..."` attributes when a class already exists or can be added to `<style>`.
   * Test both desktop/mobile browser views and **Print / Save as PDF** (`Letter`, `0.4in` margins) after layout or spacing changes.