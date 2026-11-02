# Teamworks AMS Builder: Form-Building Manual for an Assistant Agent

Purpose: give an AI agent enough working knowledge of the Teamworks AMS (Smartabase) Builder to help Troy design, specify, review, and safely change forms.

Compiled November 1, 2026 from the Teamworks Academy course "Teamworks AMS Builder 101: Basic Form Creation" (https://academy.smartabase.com) and the public AMS Help Center (https://help.teamworks.com/ams). Product behavior changes. When a detail matters, re-check the linked help article (slugs are listed in the Source index at the end). Lesson videos were not transcribed, so anything they alone cover is absent.

This file is generic product knowledge. It contains no site configuration, athlete data, or employer-confidential details, and it must stay that way.

---

## 1. Operating rules for the agent

1. **Troy builds; you design and review.** Unless Troy explicitly says otherwise for a specific task, do not click through a live AMS site. Produce a build spec (Section 3) he can follow in the Builder.
2. **Never give instructions that can destroy data without the safe path.** Renaming a field/option, deleting a field/section/form, or removing an option on a form that already has records can permanently delete data. Use Section 8.
3. **Ask what is missing, don't invent it.** Ask for: form type, who enters data (athlete, coach, both), when and how often, what decisions the data feeds, which reports or dashboards will use it, and whether it replaces an existing form.
4. **Say what the docs do not cover.** If an article was not read or a behavior is uncertain, say so and point to the article slug rather than guessing a property name or menu path. The UI can differ slightly from the help text.
5. **Keep confidential material out.** Do not paste athlete names, real data, or internal configurations into specs, commits, or this repository.
6. **Test on a sandbox if one exists.** On production, only edit the form being built. Practice forms use the name `Builder Training - [name]`.

---

## 2. Core concepts

### 2.1 Interfaces and access
- Builder access is a permission. The Builder interface has three modules: **Forms**, **Dashboards**, **Advanced settings**. Chrome or Firefox recommended.
- Forms tools: **Event**, **Profile**, **Database**, **Related entity** (legacy scheduling, unsupported), **Performance standards**, **Data specification**.
- Advanced settings (extra permissions): Translation files, Translation, User defined property schemas, Application details, Application creation (enterprise only).
- Separate **Administration interface** (not Builder) handles roles/data permissions, restriction policies, groups, queues, object ownership. A Builder usually cannot grant a user access to a new form; an administrator adds the form to roles through data permissions.

### 2.2 Form types
| Type | Use | Notes |
|---|---|---|
| **Event** | Dated, chronological records about a person (wellness, training, injury, testing) | Most forms. Supports history, reports, dashboards. Migrate tools exist only here. |
| **Profile** | Static info about a person, no history; only current value kept | Cannot rename fields/options or merge/archive options safely; ask the Product Success Manager. |
| **Database** | A lookup list (medications, exercises, items) embedded in other forms via a Database field | First field must be Single line text or Paragraph text and is the search term. Other fields are search parameters and can pull through to the host form. |

### 2.3 Structure
`Form` contains `Sections`, which contain `Fields`. Select the form name for form properties, a section header for section properties, a field for field properties; the sidebar changes accordingly. A form must contain at least one field to save.

### 2.4 Creating a form
Forms > Event (or Profile) > **Add form** > enter name (required), category (optional), tags (optional), description (optional, only shown in builder) > **Start with blank form** > **Build**. Or import a previously exported form (Start with: Import form from another application; give it a unique name). Database: Forms > Database > Add database form.

After creating: configure form properties, save properties, sections, fields; read warnings; Save; test; then have an administrator add data permissions to roles.

---

## 3. Build spec template (what the agent should hand Troy)

```
FORM SPEC
Form type: Event | Profile | Database
Form name:               Category:        Tags:
Purpose / decision it supports:
Who enters data (athlete / coach / both), how often, web / mobile:
Group entry needed? (Y/N)        Used in reports/dashboards? (which)
Save buttons: [Save and close] [Save and continue] [Save as draft] [Save and lock] ...
  Usage criteria / restriction policy on any button?
Form properties to set: (plural name, group entry, show last record, ...)

SECTION 1: <title>
  Instructions: <text or none>
  Visibility: always | conditional (rule) | hidden-section
  Column layout: n   Collapsible: Y/N   Show in PDF: Y/N
  FIELDS
  1. Name (unique, Capitalized, no special chars):
     Type:                    Required: Y/N
     Instructions:            Default:
     Options (one per line) / scores:
     Visibility rule:
     Advanced: Summary field F | Default show in tables F/T | Abbreviation | Format 0.00 | ...
     Calculation (type, equation):
SECTION 2: ...

TABLES: first field = table owner (Table Rows, Add Row Button, Delete/Duplicate Rows, Lock rows)
CALCULATIONS: name, type, equation, position (after referenced fields), Format
OPEN QUESTIONS / ASSUMPTIONS:
TEST PLAN: (cases below, Section 9)
```

---

## 4. Section properties
- **Title** (blank is allowed; builder shows "Section"). If group tabs are on, a title is required.
- **Instructions** shown in single entry only (not group entry).
- **Visibility**: conditional, or contradictory conditions to hide permanently.
- **Column layout** (advanced, up to 10): fields flow left to right, web only; mobile always vertical. Layout is section-wide, so use separate sections for different column counts. Hidden or conditionally hidden fields still reserve a slot, so a hidden placeholder field (for example `Hidden1`, Number or Single line text, Visible = Hidden) creates an intentional gap.
- **Collapsible sections** (web) only if enabled in site Application details.
- **Group section selected** (Yes/No) for tabbed group entry; **Show in PDF reports**.
- **Policies to Restrict Editing** (restriction policy).
- Deleting a section deletes all its fields and their saved data.
- Move with the up/down arrows or the multidirectional arrow ("move below section X").
- Legacy and rarely used: form layout, form positioning, randomization, mobile page break.

---

## 5. Field reference

### 5.1 Adding and basic settings
Open form > **Add Question** tab > choose type > select field to open properties. **Question Settings**: Name, Instructions, Entry Mode (Normal | Table; Matrix is legacy and unsupported on mobile), Required, Default Value, Defaults to Last Value (Normal mode only; not testable in Preview).

Naming rules:
- Unique within the form. Avoid special characters (`+ - * / & $ # %`). Capitalize the first letter (a warning appears otherwise).
- Reserved names that must not be used: `Date`, `About`, `Event Date`, `Event Start`, `Event Finish`.
- Names are case sensitive in equations; do not name a field with syntax that a history function uses (for example "Historical Sum ...").
- Prefer names of 16 characters or fewer, or set an Abbreviation.

Arrange fields with up/down arrows, the multidirectional arrow, or columns (Section 4). There is a "Copy properties from" helper in Advanced Properties (it also copies Abbreviation, so fix it).

### 5.2 Required fields
- Block saving until filled (except Save as draft).
- Not enforced in group entry.
- If the required field is conditionally hidden, the record can still save without it.
- On imports, a required field must have a value on every row even if conditionally hidden (visibility is not enforced on import).

### 5.3 Field types
- **Text:** Single line text (min/max length, field size), Paragraph text (field size; supports line breaks). Email address and Rich text are obsolete. Prefer a Paragraph text field over the form-level Comments setting because paragraph text appears in reports/exports.
- **Number** (recommended over Positive/Whole/Positive whole): Minimum, Maximum, **Format** (decimal places, for example `0.00`; always set it, rounding applies), Number type, Unit type, Units; less used: Comparisons, Best value, Calculates personal best, Minimum significant difference.
- **Date:** Date format, **Default to** (Today or None), Minimum/Maximum (relative to another date field), Restrict future dates. Never name it `Date`. In tables, set Default to: None.
- **Time:** dropdown of 15-minute steps. For analysis prefer a **Duration** field.
- **Option fields (nine):** Single selection, Multiple selection, Dropdown, Search list, Slider, Tick, Rating (mobile unsupported), True/false, Yes/no. Prefer Single selection or Dropdown over True/false and Yes/no (those option lists cannot be edited later without changing field type).
  - Options: one per line, then press **Update** (unsaved otherwise). Order affects report sorting.
  - **Scoring tab:** if one option has a score, all must. Scored fields return numbers in calculations (Section 7.2).
  - Orientation (vertical/horizontal; not for Multiple selection).
  - **Conditional options:** set the dependent field's *Conditional option* property to the controlling field name, and list its options as `Controlling value:` followed by that group's options on the following lines.
  - More than 15 options in a Dropdown or Single selection triggers a warning; use a Database or Search list.
  - Tick: first option = unticked, second = ticked; exactly two options. Rating: scores are star counts.
  - Slider: Slider options display (All, Alternating, First and last, First middle and last), Fill color.
  - Other advanced: Best value (put best option first or last in the list), Calculate total/average (needs scores), Show scores in tables instead of text, Denotes event status (legacy appointments).
- **Calculation fields:** see Section 7.
- **Database field:** pick the database in Advanced properties; add same-named fields after it to pull values through (names and options must match the database form exactly). Users need data permissions on both forms.
- **Single athlete / Multiple athletes:** tag other athletes in a record; role needs Edit athlete fields; set the form's Athlete field property to *Create event for athlete* to clone records. Do not use Multiple athletes in group entry.
- **File upload** (one file) and **Multiple file upload** (drag and drop). Form-level Attachments is separate.
- **Body diagram fields:** Simple body area, Body area, Body area summary (set max selections; default one).
- **Embedded report / Embedded event / Embedded event summary:** show or link records from other forms inside this form (Section 6.4).
- **Date calculation, Linked ..., History ...:** Section 7.

### 5.4 Advanced field properties (all types)
| Group | Properties |
|---|---|
| Data entry | Visible (Visible/Hidden), Preferred width (table column px), Show in group mode, Plural (group entry table label), Lock delay field |
| Reporting | **Default show in tables** (set False for most), Hide in reports, **Abbreviation**, Report column width, Calculate total, Calculate average, Show in quick edit popup, **Show in PDF reports**, PDF report column width |
| Sidebar | **Summary field** (set False for most; crowds sidebar), Abbreviation |
| Calendar | Calendar summary field, iCal summary field (both usually False) |
| Other | Restriction policy (Policies to Restrict Editing), user-defined property schemas if enabled |

Reasonable defaults: Visible = Visible; Summary field = False; Default show in tables = False except a handful (keep to 8 or fewer); Format set on every number; Abbreviation for names over 16 characters.

---

## 6. Behavior and layout features

### 6.1 Instructions
Section and field instructions show in single entry only (not group entry). Good for units ("Enter weight in pounds (lbs)").

### 6.2 Visibility
- Defaults to always visible. Set under **Visibility**: *Visible when all of* (AND) or *any of* (OR); you cannot mix both in one rule.
- Conditions by type: numbers/dates (equal, less than, greater than...), options (is / is not equal to), text (contains, equal, not equal).
- Rules: filters must reference fields in **other sections**; not itself; not a field with Hide in Reports = True; not a field with Visible = Hidden.
- Not applied in group entry or inside tables.
- **Data loss risk:** if a conditionally visible field holds data but is hidden at save, the data is lost. Prefer conditionally showing a whole **section**; section data is preserved.
- Hiding methods compared:
  1. Conditional visibility: shown only when criteria met; appears in reports.
  2. **Contradictory conditional visibility** (for example `X = A` AND `X != A`): never shown in entry but available in reports; best for calculation, summary, and linked fields.
  3. Advanced property Visible = Hidden: removes from entry and reports (still in "include all fields" CSV); for interim calculations. Hidden fields must not drive visibility.
- Conditionals based on linked/history calculations cannot be tested in Preview.

### 6.3 Tables
- Convert a field with **Entry Mode = Table**; consecutive table fields become columns. Only columns (no row-based tables). The **first** table field owns the table properties; deleting or moving it resets them.
- Table properties (on the first field): **Table Rows** (default 10; set to 1 if any calculation is in the table), **Add Row Button** text (`HIDDEN` hides it), **Delete Rows**, **Duplicate Rows**, **Print Rows**, **Lock Completed Table Rows** (+ lock delay 15 minutes to 96 hours), Optionally hide rows, Autocollapse table, Expand rows, Auto fill table.
- Column widths: Preferred width, Report column width, PDF report column width.
- Saved rows cannot be deleted; unsaved rows can.
- A calculation in a table uses only values from its own row; summarize rows with **Table calculation** fields placed outside the table (cannot be inside a table).
- In reports, each table row appears as its own report row; set table fields' Default show in tables to False and show a table calculation instead.
- Date fields in tables: Default to None.
- Form property **Default next cell** controls fill direction.
- All field types work in tables except table calculations. Visibility is not applied inside tables.

### 6.4 Linking and embedding forms
- **Database field** (Section 2.2) for long lists.
- **Embedded report:** mini-report of the same or another event form inside this form (filters, columns, About/By columns, show Add new, dynamic date range, sort, show in PDF). Viewer needs data permissions for the embedded form.
- **Embedded event:** create or link records from other forms (filters, columns). **Embedded event summary** pulls a field value from the embedded record (requires *Show date column = Yes* and the field selected in the embedded event). Not supported in group entry.
- **Linked calculations** (Section 7.3) pull values from other event or profile forms.
- Form property **Linked record pullthrough** copies identically named/typed fields into new embedded records (set on the originating form).

### 6.5 Group entry
One form, many athletes (requires Coach access). **Enable group entry** (True by default), **Group entry tabs** (sections as tabs; named sections only), section **Group section selected**, field **Show in group mode**.
Not honored in group entry: conditional visibility, required fields, instructions, usage criteria, locking, embedded events, Multiple athletes field.

### 6.6 Save properties (form properties > Save properties)
Enable each button by ticking it; optional Label, Tooltip, **Restriction policies**, **Usage criteria** (filters like visibility, "usable when all/any of...").
- **Save and close:** to Performance history (web) or home (mobile).
- **Save and continue:** saves and reopens (useful for history calculations that only run on save).
- **Save and enter another**, **Save and print** (web, single entry), **Save as draft** (web; drafts appear only in the left sidebar, not in reports, history, dashboards, calendar, or history calculations), **Save and redirect** (home or dashboard; not for enterprise-linked forms), **Cancel**.
- **Save and lock:** irreversible; not available with quick edits or group entry; optional lock delay. Reserve for medical-style records and consider restricting it to a policy.
- Auto-save: *temporary record* (default) or *real record* (created 30 seconds after open, updated every 30 seconds).
- Save audits: Yes for most forms (audit trail on self-hosted sites); only No for fully automated summary forms.
- Conflict warnings: legacy, leave No.
- Preview does not behave like real save buttons; test on web or mobile.

### 6.7 Form properties worth knowing
Basic: Form name (rename through the Edit name button in the form list so references update), Main category, Tags, Default date (default report range).
Advanced (selection): Plural name (favorites label), Enable group entry, Group entry tabs, Category icon, **Date override** (use a Date field as the event date), Athlete field (None | Create event for athlete | Show in calendar), **Auto color** (apply performance standards during entry), Entry mode (All | Linked entry only), Linked record pullthrough, Allow imports, **Key fields** (unique composite key for imports), Enable comments (prefer a paragraph field), Show last record, Show history, Compare to, Auto shorten, Attachments, Default next cell, Focus field on open, Allow copy to clipboard, Show template, Compressed summary field, report/print settings (Rows in history/reports, History columns, Show entered by, Print page format), sidebar settings (tooltip, auto history, Expand sidebar, Enable duplication), calendar settings (show in calendar, colors, iCal fields). Legacy/obsolete: Derive form by, positioning, chart properties, event types for scheduling/appointments, related events, Print all.

### 6.8 Other features (outline; read the article before advising)
Performance standards (color rules by field value; one standard applied per form at a time; rules cannot depend on other fields, so use an Option calculation to create the value), Athlete status/status indicators (one status field per site, scored best to worst, Best value set, Status indicator = Yes, configured with Application details), Critical information (highlighted fields in athlete sidebar), importance indicators (Tick or scored option plus Importance selector), Restriction policies (admin creates policy and attaches to roles; builder applies to a section, field, or save button; users need both role Write access and the policy), Translations, Quick edits, Templates and clipboard, Data import (Key fields, required fields), Data conflict management, Resources tool, user-defined property schemas.

---

## 7. Calculations

### 7.1 Choosing the type
Decide (1) output type, then (2) data source.

| Output | Current record | Table in current record | Historical | Other event form | Profile form |
|---|---|---|---|---|---|
| Number | Calculation | Table Calculation | History Calculation | Linked Value | Profile Linked Value |
| Text | Text Calculation | Table Text Calculation | History Text Calculation | Linked Text | Profile Linked Text |
| Option | Option Calculation | Table Option Calculation | n/a | Linked Option | Profile Linked Option |
| Date | Date Calculation | Table Date Calculation | Historical Date Calculation | Linked Date | Profile Linked Date |
| Duration | Duration Calculation, Date Difference, Date Duration | Table Duration Calculation | n/a | n/a | n/a |
| Time | Time Calculation, Time Difference | n/a | n/a | n/a | n/a |

Also: Average, Sum, Minimum, Maximum (tick the fields), Age calculation.

### 7.2 Writing equations
- Build: add the calculation field, name it, use the **Calculation** section (some types configure in Advanced Properties), set Format and Auto-calculate, and **position it after the fields it references**.
- Reference fields by exact name including case; quote names containing special characters (`"Rating of Perceived Exertion (RPE)"`).
- Operators: `+ - * / ^`, comparison `=`, `>`, `<` only (no `>=` `<=` `/=`; use `gteq()`/`lteq()`). Brackets control order.
- Functions are not internationalized: British spellings (for example `textsummarise`).
- **Scored option fields return the numeric score**, not text. Use `option(Field)` for the text (not available for Database and Multiple selection). In conditions, either `if(Field = 1, ...)` or `if(option(Field) = "A", ...)`. Multiple selection: without scores returns options joined by `|` (use a text calculation; `replace(Field, "|", ", ")` to change delimiter); with scores returns the sum.
- Dates in numeric calculations are epoch milliseconds (divide by 86,400,000 for days).
- Empty handling: `safe(x)` returns 0 if empty; `isempty(...)` (all empty), `anyempty(...)` (any empty).
- Key functions: `sum mean max min median count stddev var percentile percentrank nthmax nthmin` (each has an `...ifpositive` variant), `abs ceiling floor trunc pow sqrt ln log mod`, `if(cond, a, b)`, `and or contains`, `concatenate textsummarise customtextsummarise replace split substring`, `date dateformat duration weekstart`, `firstvalue lastvalue index row`.
- Table calculations aggregate a row-level expression with an aggregate such as `sumifpositive`; for example `if(Weight > (0.7 * Body Mass), Load, -1)` with `sumifpositive` ignores the -1 rows. `row()` returns the row number.
- History calculations: prefix the field with `Historical` (`historicalsum(Historical Load, Load)` includes the current record; omit the second argument to exclude it). Functions: `historicalmax/min/mean/sum/stddev/var/percentile/percentrank/ewma`. They use only saved (non-draft) records, run on save (use Save and continue), and can be limited by date range, number of records, and filters. They cannot be tested in Preview.
- Worked example: Training Load = `RPE * Duration` (Duration 90, RPE 7 gives 630).

### 7.3 Linked calculations
Pull a value from another event/profile form: choose source form and field, then which record (Newest, Oldest, Maximum, Minimum, Current state, Current poor state, Dynamic date), then date restriction (All history, Date range, Last n days/weeks/months/years, Last custom), with optional Group By and Match To filters. Linked option fields need the source option list entered. Cannot be tested in Preview.

### 7.4 Changing calculations on existing data
New calculations do not populate old records. Run **Resave** (three-dot menu on the form list; pick a time period; smallest range that works; wait for the email; refresh). Opening and saving a record also recalculates it and later records for that athlete.

---

## 8. Changing existing forms safely

Always first: **Download** the form (.txt template) or **Duplicate** it, and export the form's data (report > Excel > Full export). Warn Troy before any step in the "Unsafe" column.

| Safe (data preserved) | Unsafe without a migration/archive (data deleted) |
|---|---|
| Rename the form (use Edit name in the form list; also update the Plural name) | Rename a field in the builder |
| Add a field | Rename an option in the builder |
| Add an option (then update performance standards, alerts, dashboard static options that depend on the list) | Remove an option (history for that option returns blank) |
| Rearrange options | Delete a field, section, or form |
| Add or change a calculation (then Resave) | Hide a data-bearing field with per-field conditional visibility or Visible = Hidden |
| | Change Multiple selection to single-select type with data present |

- **Rename a field:** event forms only. Forms > Event > three-dot menu > **Migrate fields** > choose field > enter a new unique name > run. Wait for the confirmation email (check the Queues admin tool if none arrives), **refresh the builder before touching the form**, verify with a report. Then manually update visibility rules, calculation equations, linked calculations, embedded events, and dashboards that used the old name. Profile and database forms: consult the Product Success Manager.
- **Rename an option:** event forms only. Three-dot menu > **Migrate options** > pick field > rename options > Migrate > wait for email > refresh > verify. Conditional-option lists in other fields must be updated by hand (renaming the filter label before the colon is fine).
- **Archive a field:** create a section at the bottom (for example "Hidden fields") with contradictory visibility and move the field there. Do not put visibility or Hidden on the field itself.
- **Archive an option:** either archive the original field and add a new one, then use `if(isempty(archived), new, archived)` in a text calculation for reports; or use a date-driven Option calculation with conditional options (`Archived:` / `Active:` lists).
- **Delete** a field/form only when data loss is intended and backed up. Deleted data cannot be recovered through AMS; Teamworks Support may help within 5 business days.
- After a migration or resave do not open or edit the form until the email arrives; failing to refresh can cause data loss.
- Deleting a form needs ownership (Object ownership tool); first remove it from roles and anywhere it is referenced (the **Data specification** tool shows where).

---

## 9. Warnings, QA, and testing

### 9.1 Warning messages at the bottom of the builder
**Critical (field will not work):** no questions; reserved name; option field with no options; Database field without a database; related entity not selected; empty equation; `_eq_X is not a known variable` (wrong field name in equation); linked item/form/field not selected; date/time calculation missing source field; Average/Sum/Min/Max with no items; date-difference missing From/To/display units; Age calculation missing units. Also a pop-up: "Each form item has to be a unique name."
**Non-critical (should fix):** Tick with more than two options; categorical calculation with no options; 16 or more options in Dropdown/Single selection; nine or more fields shown in tables; long names without Abbreviation; number without Format; name not capitalized.

### 9.2 Testing
- **Preview** in the builder tests required fields, visibility, conditional options, orientation, tables, and non-history calculations. It cannot test: history calculations, linked calculations, performance standards, real saving, save-button behavior, Defaults to Last Value, or anything driven by saved data.
- Test the real form on web (and mobile) as a user with a test role/group. An administrator must grant data permissions for the form and any referenced forms (databases, embedded forms, linked forms).
- Suggested test pass: required behavior; every visibility rule both ways; contradictory-hidden fields are hidden; conditional options for each parent value; table rows add/delete; each calculation with edge values (blank, zero, max); save each enabled button; open a saved record; run a report and PDF; check warnings; group entry if enabled.

---

## 10. Reporting and data-entry optimization checklist
- Number fields: Format set; units in instructions.
- Default show in tables True for a few key fields only; Abbreviation on long names; Report column width set.
- Summary field False on all but one or two fields.
- Table fields: Default show in tables False plus a table calculation summary.
- PDF: Show in PDF reports at field and section level; Print page format; PDF report column width.
- Totals/averages: Calculate total / average where meaningful.
- Option fields: scores plus Best value if used for sorting, status, or color; best option at top or bottom.
- Favorites: Plural name and Category icon.
- Show last record / Show history if staff benefit from context while entering.
- Use sections for related fields and conditional section visibility instead of per-field hiding.
- Mobile: columns do not apply; Rating field and Matrix are unsupported.

---

## 11. Common design patterns
- **Daily wellness check-in:** sliders or scored single selections, a scored total (Sum or Calculation), status option calculation, optional follow-up section visible only when a score is poor.
- **Training session / load:** Duration, RPE slider, `RPE * Duration`, history calculation for weekly or rolling totals; optional table of exercises with a table calculation total.
- **Availability / status:** one scored option field set as the status indicator, with best value and performance standard colors.
- **Attendance:** one record plus Multiple athletes field with *Create event for athlete*.
- **Reference list:** Database form plus Database field with pulled-through fields.
- **Report-only field:** hide with contradictory visibility, not Visible = Hidden.

---

## 12. Glossary
Event form, Profile form, Database form; Section; Field; Question Settings; Advanced Properties; Entry Mode; Required; Preview; Summary field; Performance history; Reports tool; Favorite event; Group entry; Resave; Migrate fields / Migrate options; Restriction policy; Data permission; Role; Performance standard; Status indicator; Smartsave (scheduled automatic record creation); Data specification module; Queues (admin tool showing migration/resave jobs); Product Success Manager (Teamworks contact for sandbox/access and unsupported changes).

---

## 13. Source index (help.teamworks.com/ams/s/article/<slug>)

Course: https://academy.smartabase.com/teamworks-ams-builder-101-basic-form-creation (Course Introduction, Form Components, Optimized Data Entry, Optimized Reporting, Managing Forms, Final Quiz).

Structure and creation: `introduction-to-the-builder-interface`, `log-in-as-a-builder`, `event-forms`, `profile-forms`, `database-forms`, `create-a-form`, `form-properties`, `form-save-properties`, `section-properties`, `field-properties`.
Field types: `number-fields`, `text-fields`, `option-fields`, `date-field`, `time-field`, `single-athlete-field`, `multiple-athletes-field`, `file-upload-field`, `body-diagram-fields`, `embedded-report-field`, `embedded-event-field`, `option-fields-as-importance-indicators`.
Behavior: `Control-Field-and-Section-Visibility-During-Data-Entry`, `Add-a-Table-to-a-Form`, `Arrange-Fields-Side-by-Side-Within-a-Form`, `Optimize-Group-Entry-Mode-for-Event-Forms`, `restriction-policies`, `warning-messages-in-forms`, `reference-a-database-in-a-form`, `update-database-records`.
Calculations: `introduction-to-calculations`, `using-functions`, `Using-Option-Fields-and-Scores-in-Calculations`, `linked-calculations`, `history-calculations`.
Reporting/sidebar: `Customize-Reports-and-PDF-Exports-for-a-Form`, `Customize-the-Web-Sidebar-Settings`, `Configure-How-Event-Forms-Appear-as-Favorites`, `performance-standards`, `configure-athlete-status`, `ams-reports`, `export-data-from-ams`, `data-specification`.
Changing/managing: `Update-an-Option-Field-in-a-Form`, `Change-the-Name-of-a-Field-with-the-Migrate-Fields-Tool`, `Change-the-Name-of-a-Form`, `Add-or-Update-a-Calculation-Field-in-an-Existing-Form`, `Remove-a-Field-From-a-Form`, `Duplicate-a-Form`, `Download-a-Form`, `Delete-a-Form`, `object-ownership`.
Not read in full (consult before advising): `import-data`, `translations`, `user-defined-property-schemas`, `data-conflict-management`, `linked-data-in-forms`, `Enable-Quick-Edits-for-an-Event-Form`, `roles`, `groups`, `introduction-to-the-dashboard-builder` (dashboards belong to separate courses), `form-metadata`.
