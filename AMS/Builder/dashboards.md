# Teamworks AMS Builder: Dashboards Guide

Companion to `manual.md`. Covers Builder 103 ("Building a Dashboard") and Builder 104 ("Customizing a Dashboard") plus the Dashboard Builder help articles. Compiled November 1-3, 2026 from https://academy.smartabase.com and https://help.teamworks.com/ams. Lesson videos were not transcribed; the Dashboard Troubleshooting Guide was read in full only through its first half, with its remaining headings skimmed. Menu names can shift; verify against the live Builder.

Generic product knowledge only; no site configuration or athlete data.

---

## 1. Concepts

- A dashboard visualizes data from **event forms** (each form is a dataset). Dashboards cannot edit data; users can click a data point and use **Go to record**.
- Structure: **Dashboard > Canvases (pages/tabs) > Sections (blocks, collapsible) > Widgets**. A canvas needs at least one section; a dashboard needs at least one canvas. Canvases cannot be duplicated; sections and widgets can be duplicated, deleted, or moved between canvases (the four-way arrow).
- **Data sources** are event forms (with a time period and optional filters) that widgets read from. Widgets read all records of the chosen field for all users in the **currently loaded group**.
- **Filtering widgets** (Select box, Date picker) drive other widgets through **data filters**.
- Uses SVG/HTML5/CSS; exportable to PDF.
- Enterprise servers: dashboards cannot be linked between sites; export and import them.

### Builder interface (Dashboards > Builder)
Header: dashboard name, status (unsaved/draft/published), tabs **Dashboard** (preview, add components, properties), **Data** (data sources), **Rulesets** (conditional formatting), **Settings**, Undo/Redo (up to 100 steps; Cmd/Ctrl+Z, Shift+Cmd+Z or Ctrl+Y), **Save Draft**, **Publish**, Help, What's new, Exit. Auto-repair fixes broken properties (missing properties, broken filters) and shows a repair message next time.

### Draft vs published
- New dashboards are drafts, visible only in the builder. **Changes are not saved automatically; press Save draft often.**
- **Publish** makes it visible to users and creates a **data permission** for the dashboard; an administrator adds it to roles. Users need the **Dashboards system permission** and the dashboard's data permission (and the Dashboards tool in their page layout if custom layouts are used). Builders/administrators can see all dashboards without role permissions.
- Editing a published dashboard: Save draft hides changes from users but prevents testing with real data. For major edits, **duplicate** the dashboard (three-dot menu > Duplicate), edit the copy, then have an administrator swap roles; rename the old one with an `Archived_` prefix.
- A dashboard cannot be unpublished; remove it from roles. Delete is permanent (offers a template backup, not data).

---

## 2. Planning a dashboard (agent checklist)

Ask Troy:
1. Purpose and audience (who decides what from it?).
2. Source event form(s) and the exact field names and types (number, option text, date).
3. Individual view, group view, or both (affects athlete filter "empty selection" and "Ignore if empty" settings).
4. Time window and typical data volume (affects data-source period and record limit).
5. Where it is used: web, mobile, printed, embedded in a form?
6. Needed thresholds/colors (rulesets), avatars, logo, drill-through cards.
7. Whether source forms need new calculated fields first (build those in the form; see `calculations.md`). Dashboards aggregate form fields; heavy per-record math belongs in form calculations.
8. Test data: at least 2-3 test accounts, multiple dates, all fields populated (import CSV with the Import data tool, or group entry).

Prefer several focused dashboards over one all-in-one dashboard; avoid canvases that require scrolling.

---

## 3. Build spec template for a dashboard

```
DASHBOARD SPEC
Name:                 Category:           Audience / roles:
Purpose:
Data sources: (form, time period, record limit per athlete, source filters)
Filters (section "Filters"):
  Athlete select box: options from <source>.About, allow empty? y/n
  Start Date picker: range, default
  End Date picker:   range, default
CANVAS 1: <title>   (logo? subtitle? background?)
  SECTION: <title> (columns/grid, collapsible?, show on print?)
    WIDGET <type> "<title>": data source; value/series/categories/columns (field, data type, aggregation, decimals);
        data filters (field, condition, filtering widget, ignore-if-empty);
        conditional formatting (ruleset, conditional field); interactions (cards, fields); no-data behavior; width
Rulesets: (name, data type, ordered rules: condition, value, color/icon)
Settings: view on mobile? caching? print orientation/paper size/canvases; avatar color style
Testing plan: (Section 9)
```

---

## 4. Data sources (Data tab)

- Add with the **+** button, choose the event form, Continue. The data source name defaults to the form name and can be changed.
- **Time period:** All history (optionally including today), All future, All history and future, Last n days/weeks/months/years, Next n..., Date range, Today.
- **Field filters** (like the Reports tool) restrict which records load. Press Enter after typing a value. This differs from widget filters.
- **Record limit:** maximum records per athlete in the period. Keep small (for example 10-20 for summaries, 50-100 for recent-data views).
- **Change form** for an existing source: choose to reset unmatched fields (publishable but affected widgets reset to the first field alphabetically) or leave unchanged (widgets show errors; you cannot publish until fixed; easier to find problems).
- **Nested CSV data** (for example heart rate over time): enable parse nested CSV data, pick the field, list headers one per line; fields appear as `Field Name[CSV Field Name]`. Can hurt performance; consult the Product Success Manager first.
- Sources must be added before widgets can use them; limit periods to what widgets need. "No data" on a source in the info panel can mean the viewer lacks permission to that form.

---

## 5. Filters

### 5.1 Filtering widgets
**Select box:** options as a String (or Number/Date) from a data source field (for athletes use the **About** field, which is automatic metadata) or custom options; options only appear if a record exists in the source; order by value or first/last occurrence; filter the options; **Allow empty selection** (empty by default) vs a default first option; allow multiple selection; Enable search; "Reset options on data change".**Date picker:** one date. **Date range** (From data source, Custom range, Last interval, No range); **Default date** (Minimum or Maximum date in range, Today, No default, Past/Future relative to today); Date format. Use two pickers for start and end; give each a unique title (or hide the title and put a label in Instructions).

### 5.2 Data filters on a widget (Edit Filters)
Each filter: **Filtered field**, **Condition**, **Filtering widget/value** (another widget or a custom value; press Enter to lock custom values).
- Conditions are grouped by **data type submenu: String, Number, Date.** Pick from the submenu matching the filtered field. Wrong submenu is the most common error and makes the filtering widget vanish from the dropdown.
- Athlete filter: `About` / `String - is equal to` / Athlete select box.
- Start date: `Date` / `Date - is after or same` / Start Date picker. End date: `Date` / `Date - is before or same` / End Date picker. Single day: `Date - is same`.
- Date filters use the **Date** field (the event date metadata).
- **Ignore if filtering widget has no value selected:** enable when the filter is optional so an empty selection shows group data. Pair with the select box's *Allow empty selection*. Without it, an empty filter shows "No data".
  - Individual summary dashboards: no empty selection (always an athlete).
  - Group summary with optional athlete: empty by default plus Ignore-if-empty on group aggregations; do not enable for "most recent", min, or max tiles (they would represent a single person).
- **Copy filters** between widgets (Copy filters tab).
- Contradictory filters (after Jan 15 and before Jan 10; equals X and not equals X) give no data.
- Source with a table and a filter field outside the table: widget may show only the first table row (rows are stored on separate lines).
- Many widgets have **two filter levels:** structure (data source, categories, rows) and data (columns, series, values). Apply filters at the correct level; a structure can show with empty cells when second-level filters exclude everything.

---

## 6. Widgets

Common widget properties: **General** (title/subtitle with font, color, icon; background and border color with opacity (0% = transparent); width in section columns), **Instructions** (rich text), **Data/Value** (source, field, data type, aggregation), filters, **Interaction** (record cards, click-through), no-data behavior, **Trellis**, data preview (simulated data in the builder; set min/max/points). **Data type must match the field:** Date fields use Date; numbers use Number; text and options use String (including option fields with scores or mixed numeric/text options). Wrong type shows **NaN**.

Aggregation depends on data type. Seen in the sources: Count (optional Unique), Sum, Mean, Median, Standard deviation, Minimum, Maximum, Most recent record ("Get from most recent record"), earliest/most recent and nth earliest/most recent (tiles), and Count Unique. Choose the aggregation that reflects the question (group mean vs latest individual value).

### Content widget
Static text/images/tables/links (no data source). Instructions property does the same inside canvases/sections/widgets.

### Tile
One value. Value field, data type, aggregation, decimals; caption; static icon (color, size) and layout direction; **avatars** (below title; size, alignment, max number); **interaction** (record cards, click-through); **conditional formatting** (ruleset, conditional field, legible foreground color); no-value placeholder or **Hide widget if no data**; **group aggregation** (aggregate after grouping by another field, for example daily totals then average; not for Image; earliest/most recent aggregations unavailable); grid-layout scrolling; trellis. Image data type shows an image field (click to enlarge).

### Table
A report inside the dashboard; one row per record (each form table row is its own row). Columns (title, field, data type, decimals, alignment, width, conditional formatting per column); **Sorting** (default column and direction, reset sorting on data change); **Summary rows** (mean, sum, standard deviation, min, max); max height, **pagination** (enable **Print all pages** so printouts include all rows); style (font, padding, borders, header/odd/even row colors); avatar column; cards; trellis.

### Aggregation table
Rows come from **Categories** (a field's values, for example About or Session Type); each column has its own source, filters, value field, **Category field (must match the categories' field)**, data type, aggregation. Options: show null categories, multiple-selection category, order, prefix/suffix, hide category column; column summary rows (aggregate raw values or displayed values); conditional formatting with inherit option; freeze header; avatars.
Exact patterns from the course: by athlete (category About; columns Count of About, Mean of Training Load, Sum of Duration) and by session type (Count Unique of About for number of athletes; Count of Session Type for sessions; Mean of Training Load). Put athlete/date filters on the **Categories** for the athlete filter (with Ignore-if-empty) and the date filters on both categories and columns as the course does.

### Matrix
Single source; **Rows** field, **Columns** field, **Values** field with aggregation; summary row/column; ordering by value/first occurrence/custom; conditional formatting; avatars. Empty cells are normal when no record exists.

### Time series chart
Series list: name, color, source, value field, **Time field** (usually Date), aggregation, display type (line, bar, spline, area, area-spline, scatter), decimals, data labels, Y2-axis, line style, stacking group. X-axis: time input format and tick format (`%m/%d/%Y`-style directives), rotation; Y-axis tick format and padding; legend; custom gridlines; banded regions; chart height; markers; cards on series. Series order controls layering.

### Categorical chart
Categories field (data type, show null categories, sort by name or data, limit to top/bottom n, optional user-facing sort/limit controls); series each with value field, **Category field matching the chart categories**, aggregation, display type, stacking; gridlines (for example average line: value type Aggregate series data, Mean of a series), regions, legend.

### Pie/Donut
Categories field (order: Automatic by size, name, first occurrence, custom with per-category colors), Values field and aggregation (for example Count of About), Pie vs Donut, segment labels, percentage display. Multiple-selection category fields need the matching delimiter setting.

### Radar
At least three variables (fields); series of type Data (with aggregation such as most recent or mean) or Static (benchmark); each series filtered separately (omit the athlete filter for a group average).

### XY scatter
Numeric X and Y fields; color by a field or fixed; size by a field or fixed; axis labels/padding; legend; custom gridlines for quadrants.

### Periodization diagram
Timeline tracks (year/month/week/day), custom tracks and event tracks from a form; zoom, style. Lesson text gave no practice steps; read `periodization-diagram-widget` before advising.

### Trellis (small multiples)
Any widget except content can be split by a field (for example About). Trellis properties: data source, field, data type, filters (limit which multiples appear), limit, order (value, first/last occurrence, custom exactly spelled), show titles.

---

## 7. Rulesets and conditional formatting

- Rulesets tab: **+** to create; name; one **data type** (Date, Number, String); rules with condition, comparison value (Date: today or custom), response color, icon. Rules evaluate **top to bottom**; reorder with arrows.
- Apply per widget: **tile** (Conditional formatting properties), **table/aggregation table** (per column, Advanced settings; Inherit for summary cell), **matrix** (Values properties). A ruleset icon overrides the tile icon.
- **Conditional field:** format by another field (course example: color cells using `Training Load` with data type Number). **Use legible foreground color** flips text/icon to black or white for contrast.
- Example: `Training Load Status` (Number): less than 200 red; less than 400 yellow; greater than or equal to 400 green.
- Rulesets are separate from form performance standards (those color form data entry and reports).

---

## 8. Customizing (Builder 104)

- **Context:** titles, subtitles, **Instructions** (rich text, images, tables, links) on canvas/section/widget; Content widget for stand-alone text. Add a short instruction block in the Filters section (for example "leave Athlete empty for group summaries").
- **Icons:** Title icon in General properties (for example filter icon on the section, account on the athlete select box, calendar icons on date pickers).
- **Colors/theme:** Settings > theme: canvas background; set section background opacity 0% to inherit; subtle borders; keep text contrast.
- **Avatars:** compatible with tiles, tables, aggregation tables, matrix. Avatar size, max avatars (1 for one person per row). No photo shows initials; Settings > Avatars color style (Full color, Grey, Black, White). Standalone avatar: tile filtered by an athlete select box, value text size 0, avatars on (size, max 1, centered), cards off, transparent background/border.
- **Logos:** upload the image as a **Resource** in a Resources-type category (admin creates category and grants roles; uploader needs Resources and Resources-upload system permissions; every viewer needs data permission to that category). Then canvas Logo properties: enable, **exact** resource name (case-sensitive), height.
- **Accessing:** dashboards appear in the Dashboards tool, favorites on the home page, and mobile. Mobile stacks widgets full width. **View on mobile** is off by default (Settings > General). **View on event pages** lets the site owner embed a dashboard in a form (request via Teamworks).
- **Layouts:** section is 12 columns; set widget widths in columns (6+6, 4+4+4) or reduce section columns; widget heights in Advanced (tables/charts); drag to reorder; optional **grid layout** for free placement/resizing.
- **Default values for filters:** decide empty vs preselected select box; date pickers can default to "past date relative to today" (for example 30 days ago) and Today so the dashboard is useful on load. Range "No range" still allows earlier dates.
- **Interactions:** record **cards** (choose card fields; disable cards to remove click-through); chart series have card fields too.
- **Hiding widgets:** Advanced > Hide widget if no data (only after confirming the widget is correct). Replace placeholder text for no-data states.
- **Printing:** Settings > Printing defaults (orientation, paper size, canvases); section **Show on print**, **Start on new page**; collapsed sections are omitted from print; users should enable browser background graphics; Print all pages for paginated tables.
- **Caching:** if load exceeds 20 seconds, caching may be offered (data is not live; refresh overnight or via Update cached data; stops auto-update after 7 days unused). Can be disabled in General settings (Allow dashboard to be cached) or from the info icon.
- Favorites: users or admins can favorite dashboards for the home page.

---

## 9. Testing and troubleshooting

### Testing checklist (course)
Loads with no errors in under about 10 seconds; all widgets present; athlete picker lists names; date pickers have sensible defaults; selecting an athlete updates athlete-filtered widgets; empty athlete shows group data for widgets with Ignore-if-empty; date range restricts data; all filters combine; tiles/tables/aggregations/charts correct against known sample data; cards and click-through work; tooltips; dashboard refreshes after record edits; titles clear; usable on first load; test 2-3 athletes, narrow and wide ranges, empty athlete, one filter at a time then combined. Before sharing: users have Dashboards system permission, add the dashboard data permission to roles, tell users where it is.

### Troubleshooting guide (summary)
Quick checks: published (not draft); refresh the dashboard; data exists for the loaded group; use test data.
- **"No data":** check data source and field; widget filters (field, correct data-type submenu, correct filtering widget; both levels); Data tab time period; run the same filters in Reports to see whether records exist; loosen filters; check widget-specific required fields (Category field, Time field, etc.).
- **NaN:** wrong Data type; text/option fields (including scored options and mixed numeric/text options) use String; numbers use Number; dates use Date.
- **Tile blank with "most recent":** the newest record has a blank value; add a filter excluding blanks or targeting the relevant category.
- **Empty columns/series/cells:** conflicting second-level filters; Category field mismatch between categories and columns/series.
- **Logo missing:** logo toggle, exact resource name, supported image, viewer permission to the category.
- **Filter does nothing:** no filter applied to that widget, wrong filtered field, wrong submenu, filtering widget not set. **Filtering widget missing in the dropdown:** condition from the wrong data-type submenu.
- **Empty filter shows "No data" but should show group:** enable Ignore-if-empty and Allow empty selection (only where a group value makes sense).
- **Filter widget shows no options:** no records in the source period, wrong options data type, restrictive date range.
- **Date filters off:** use `after or same` for start, `before or same` for end, `same` for a day; avoid contradictory conditions.
- **Categories missing:** Category field mismatch or spelling; empty categories can appear if null categories are shown.
- **Dashboard differs from report:** time period, record limit, filters, aggregation (dashboard mean vs report sum, most recent vs all records), cache timing.
- **Shows data for some users:** group loaded; data permissions to every source form.
- **Slow:** shorter time periods (Last 30 days for frequently updated data, Last 90 for less frequent), record limits, fewer widgets/series/columns/filters, multiple focused dashboards or canvases, caching.
- **Changes not visible after publishing:** refresh dashboard and clear browser cache.
- **Cannot see dashboard:** Dashboards system permission, the specific dashboard data permission, page layout includes the Dashboards tool.

---

## 10. Agent working rules for dashboards
- A dashboard is only as good as the form fields: if a needed metric is not a field, propose a form calculation first (and note a resave may be needed).
- Always state each widget's data type and aggregation explicitly in specs; most dashboard bugs are wrong data type, wrong filter submenu, or missing Ignore-if-empty.
- Specify the filter trio (athlete, start, end) once, then reuse via Copy filters.
- Never assume aggregation menus: list the aggregation by name and confirm it exists for the chosen data type in the live builder.
- Warn about static option names in rulesets, select-box defaults, filters, and pie colors when form option names change; they do not update automatically.
- Do not click Publish on a production dashboard on Troy's behalf without explicit instruction.

---

## 11. Source index

Course pages: https://academy.smartabase.com/teamworks-ams-builder-103-building-a-dashboard and https://academy.smartabase.com/teamworks-ams-builder-104-customizing-a-dashboard

Help articles (help.teamworks.com/ams/s/article/<slug>): `introduction-to-the-dashboard-builder`, `ams-dashboards`, `create-a-dashboard`, `edit-the-dashboard-settings`, `dashboard-canvases`, `dashboard-sections`, `manage-data-sources-in-a-dashboard`, `introduction-to-widgets`, `filter-a-dashboard-widget`, `select-box-widget`, `date-picker-widget`, `content-widget`, `tile-widget`, `table-widget`, `aggregation-table-widget`, `matrix-widget`, `time-series-chart-widget`, `categorical-chart-widget`, `pie-and-donut-chart-widget`, `radar-chart-widget`, `xy-scatter-chart-widget`, `periodization-diagram-widget`, `widget-trellising`, `rulesets-and-conditional-formatting-in-dashboards`, `dashboard-instructions`, `Display-Avatars-in-a-Dashboard`, `Add-a-Logo-to-a-Dashboard-Canvas`, `Print-or-Export-a-Dashboard`, `Print-or-Export-a-Dashboard-on-AMS`, `dashboard-caching`, `set-favorite-events-and-dashboards`, `Dashboard-Troubleshooting-Guide`, `Create-an-Optional-Athlete-Filter-in-a-Dashboard`, `Create-a-Date-Range-Filter-in-a-Dashboard`, `Trellis-a-tile-widget-to-display-the-availability-of-all-athletes-in-a-group`, `Display-Heart-Rate-Data-from-Nested-CSV-Data-in-a-Dashboard`.

Not read in full: the tutorial articles above (optional filter, date range filter, trellis tile, nested CSV), the second half of the troubleshooting guide, `matrix-widget`/`pie-and-donut-chart-widget`/`xy-scatter-chart-widget`/`radar-chart-widget`/`periodization-diagram-widget`/`time-series-chart-widget`/`categorical-chart-widget` beyond their setup and appearance sections, `Dashboard aggregation options` (article not found at the linked address), `roles` (permissions), and Admin courses (Admin 101 and 102 cover roles and permissions needed to share and test dashboards).
