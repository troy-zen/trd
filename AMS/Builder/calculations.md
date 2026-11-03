# Teamworks AMS Builder: Calculations and Linked Data Guide

Companion to `manual.md` (read that first). Covers Builder 102 ("Using Calculations") and the help articles it links. Compiled November 1-3, 2026 from https://academy.smartabase.com (Builder 102 lessons) and https://help.teamworks.com/ams. Lesson videos were not transcribed. Re-check the linked article (slugs at the end) before relying on an edge case.

Generic product knowledge only; no site configuration or athlete data.

---

## 1. Mental model

A calculation field computes its value from other data, so values stay consistent and users do not enter them. Choose by two questions:
1. **Output type:** number, text, option, date, duration, time.
2. **Data source:** current record, a table in the current record, previous records of the same form (history), another event form (linked), a profile form, or the user's account.

| Output | Current record | Table in this record | Historical (same form) | Other event form | Profile form |
|---|---|---|---|---|---|
| Number | Calculation | Table Calculation | History Calculation | Linked Value | Profile Linked Value |
| Text | Text Calculation | Table Text Calculation | History Text Calculation | Linked Text | Profile Linked Text |
| Option | Option Calculation | Table Option Calculation | n/a | Linked Option | Profile Linked Option |
| Date | Date Calculation | Table Date Calculation | Historical Date Calculation | Linked Date | Profile Linked Date |
| Duration | Duration Calculation, Date Difference, Date Duration (table) | Table Duration Calculation | n/a | n/a | n/a |
| Time | Time Calculation, Time Difference | n/a | n/a | n/a | n/a |

Other calculation-style fields: **Personal details** (account data), **Age calculation**, **Average / Sum / Minimum / Maximum** (tick the fields), **Embedded event summary**, **Embedded report**, **Embedded event**.

General build rules (all types):
- Add the field, give it a unique clear name, enter the formula in **Calculation** (some types configure in Advanced Properties).
- Advanced Properties: **Format** (decimal places), **Auto-calculate** (calculate as data is entered; usually Yes), Visible, Default show in tables, Summary field, Show in PDF reports, and the usual reporting properties.
- **Position calculations after the fields they reference.**
- Hide interim calculations with Visible = Hidden (never needed in reports) or contradictory conditional visibility (needed in reports, hidden during entry).
- Option Calculation / Table Option Calculation / Linked Option need an **options list** matching every possible result; without options a warning appears and imports fail. List order controls report sorting.

---

## 2. Writing equations

### 2.1 Syntax
- A formula is functions, field names, constants, operators: `sum(Fatigue, Soreness, Stress)/15`.
- **Field names are case-sensitive and space-sensitive.** `Sleep hours` is not `Sleep Hours`. Quote names with special characters: `"Rating of Perceived Exertion (RPE)"`. Names containing parentheses, such as `Duration (minutes)`, are used as written in the course examples.
- Operators: `+ - * / ^` and brackets; comparisons `=`, `>`, `<` only. **`>=`, `<=`, `/=` and `isblank()` are not valid.** Use `gteq(a,b)`, `lteq(a,b)`, `isempty(x)`.
- Order of operations: brackets, exponents, multiplication/division, addition/subtraction.
- Strings go in quotes. (The help text prints typographic quotes; type plain straight quotes in the builder. This is a precaution, not a documented rule.)
- Functions use British spelling (for example `textsummarise`). Spacing around commas and operators does not matter, but unbalanced brackets do.
- Use the field-insert dropdown or type names exactly to avoid typos. The "Available functions" dropdown at the bottom of the Calculation tab lists what a given calculation type supports.

### 2.2 Function families
- **Numeric:** `abs ceiling floor trunc pow sqrt ln log mod safe sum sumifpositive`, comparisons `gt gteq lt lteq`, unit conversions `convert...` (for example `convertcmtokm`).
- **Statistical (each has an `...ifpositive` variant):** `count max min mean median stddev var sem percentile percentrank nthmax nthmin`.
- **Reference:** `firstvalue lastvalue index firstpositive lastpositive firstnonzero lastnonzero row`, historical-only `sequentialhistoricalvaluesthatequal`, `valuewhentargetmatches`.
- **Logical:** `if(test, a, b)`, `and(...)`, `or(...)`, `contains(text, part)`, `isempty(...)` (all empty), `anyempty(...)` (any empty), `rawvalue`.
- **Text:** `concatenate customtextsummarise textsummarise whitespacetextsummarise replace replaceall split splitfirst splitlast splitgreaterthan splitlessthan substring charat option`.
- **Date/time:** `date`, `dateformat`, `duration`, `weekstart`. Date formats: `y yyyy yy M MM MMM MMMM d D DDDD u`; for Entered On values also `H k K h m S s a z Z`. Time fields cannot be passed to `date()`/`dateformat()`.
- **Historical:** `historicalsum historicalmean historicalmax historicalmin historicalstddev historicalvar historicalpercentile historicalpercentrank historicalewma`.
- `ifpositive` variants ignore negative values; this is how row filtering works (return `-1` to exclude).

### 2.3 Behaviors that surprise people
- **Scored option fields return the score**, not the text. `option(Field)` returns the text (not for Database or Multiple selection). With scores, `Field = "A"` fails; use `Field = 1` or `option(Field) = "A"`.
- Multiple selection without scores returns options joined by `|`; with scores returns the sum. `replace(Field, "|", ", ")` changes the delimiter (`"\n"` for new lines).
- **Dates in numeric math are epoch milliseconds** (86,400,000 per day; hours 3,600,000; minutes 60,000). Convert for display with a Text Calculation using `date(x, "MM/dd/yyyy")`. Text calculation output is a string, not a real date: you cannot feed it into date-difference fields, and dashboards need the String data type.
- **Empty fields:** `safe(x)` returns the number or 0 when the field is empty; guard with `anyempty()`/`isempty()` when a blank input should give a blank or "Not Assessed" result. Test blank inputs explicitly rather than assuming behavior.
- **Preview limits:** Preview cannot test history calculations, linked calculations, personal details, profile linked fields, embedded reports/events/summaries, or performance standards. Test those on the web with saved records and a test role.
- Calculations referencing a field inside a table use only the same row.

---

## 3. Recipes (from the Builder 102 practice tasks)

Replace names with the real field names; keep exact case.

**Training load:** Number fields `Duration (minutes)` and `RPE` (Format 0). Calculation `Training Load` = `Duration (minutes) * RPE`. Test: 30 and 6 gives 180.

**Weighted load using option scores:** Single selection `Session Type` (Training, Competition, Recovery) with scores 1, 1.5, 0.5. Calculation `Weighted Training Load` = `Training Load * Session Type` (60 min, RPE 7, Competition gives 630).

**Two-way option calculation:** Option Calculation `Hydration Status`, options `Adequate`, `Dehydrated`: `if(Water Intake (liters) < 2, "Dehydrated", "Adequate")`.
**Three-way (nested if):** add `Over-hydrated`: `if(Water Intake (liters) < 2, "Dehydrated", if(Water Intake (liters) > 4, "Over-hydrated", "Adequate"))`.

**Multi-metric risk flag:** Option Calculation `Training Risk` with options `Not Assessed`, `High Risk`, `Moderate Risk`, `Low Risk`:
`if(anyempty(Sleep Hours, Soreness Level, Stress Level, Energy Level), "Not Assessed", if(or(Sleep Hours < 6, Soreness Level > 7, Stress Level > 8, Energy Level < 3), "High Risk", if(or(Sleep Hours < 7, Soreness Level > 5, Stress Level > 6, Energy Level < 5), "Moderate Risk", "Low Risk")))`
Pattern: empty check first, then most severe to least severe.

**Boundary logic with gteq/lteq:** `if(anyempty(Sleep Hours, Soreness Level), "Not Assessed", if(and(gteq(Sleep Hours, 8), lteq(Soreness Level, 3)), "Well Recovered", if(or(Sleep Hours < 6, Soreness Level > 7), "Poorly Recovered", "Partially Recovered")))`

**Personalized text:** Text Calculation `Sleep Feedback`:
`if(isempty(Sleep Hours), " ", if(Sleep Hours < 7, concatenate("You slept ", Sleep Hours, " hours. Try to get at least 7 hours tonight."), "Good job getting adequate sleep!"))`

**Score vs text of a slider:** `Intensity` scored 1 to 5. `Duration (minutes) * Intensity` multiplies using the score. Text Calculation `option(Intensity)` returns the label; `Intensity` alone returns the score.

**Days between two dates:** Date fields `Today's Date` and `Goal Deadline` (Default to None). Date Difference Calculation: From Date `Today's Date`, To Date `Goal Deadline`, Display Units Days (rounded to whole number; units days, months, or years).

**Add days to a date:** Calculation `Deadline Unix_interim` = `Goal Deadline` (Format 0); Calculation `Review Date Unix (Interim)` = `Deadline Unix_interim + (7 * 86400000)`; Text Calculation `Review Date` = `date(Review Date Unix_interim, "MM/dd/yyyy")`. Streamlined: `date(Goal Deadline + (7 * 86400000), "MM/dd/yyyy")`. Hide interim fields.

**Duration math:** Duration Calculation `Sprint Time - (5 * 1000)` (subtract 5 seconds; set Format such as mm:ss). Sum of two duration fields: `Race 1 + Race 2` with matching Format.

**Time difference:** Time Difference Calculation needs Time, Linked Time or Time Calculation fields; it cannot reference event start/finish directly, so create two Time Calculation fields (Event Start, Event Finish) and difference them (units hours/minutes/seconds, web only).

**Age at event:** Age calculation, Age At = Event date or another date field, Display Units days/months/years (rounded down).

**Table row calculation and summary:** table fields `0-5m (seconds)`, `5-10m (seconds)` (Format 0.000, Table Rows 2); Calculation `Sprint Time (seconds)` = `0-5m (seconds) + 5-10m (seconds)` as a table field. Table Calculation `Fastest Sprint Time` (Entry Mode Normal, outside the table): aggregation `min`, equation `Sprint Time (seconds)`. Count rows: aggregation `count`, equation `0-5m (seconds)`.

**Conditional table aggregation:** add `Valid Test` (Yes/No) as a table field. Table Calculation `Best Valid Time`: aggregation `minifpositive`, equation `if(Valid Test = "Yes", Sprint Time (seconds), -1)`. Count valid: aggregation `countifpositive`, equation `if(Valid Test = "Yes", 1, -1)`. Other table-calc aggregations: `sumifpositive`, `row()` for targeting rows (`if(row() > 2, Load, -1)`), and for text/option/date tables `count`, `firstvalue`, `lastvalue`, `lastenteredvalue`, `textsummarise`, `whitespacetextsummarise`.

**Rolling 7-day total (history):** Number `Daily Training Minutes`; History Calculation `Weekly Training Minutes` = `historicalsum(Historical Daily Training Minutes, Daily Training Minutes)`, Date restriction Last 1 Week. Test on the web with several dated records; enable Save and enter another to enter test records quickly.
**Average and flag:** `Weekly Average` = `historicalmean(Historical Daily Training Minutes, Daily Training Minutes)`, Last 1 Week; History Text Calculation `Training Variation` = `if(Daily Training Minutes > Weekly Average, "Above Average", "Below Average")`; Resave after adding.

**Account details:** Personal details field, Personal details property = Date of birth (also names, email, phones, addresses, username, sex, language, active status, UUID, profile picture with width/height). Read-only, populates only on the web.
**Profile value:** Profile Linked Text/Option/Value/Date selects the profile form and field; read-only; the athlete needs a saved profile record.

**Linked option:** Linked Option `Favorite Food` with the same option list as the source; Calculation: Form, Field, Value Newest, Date restriction All history; Advanced: Manual update False, Linked update range End of current event, Ignore blanks No. Linked calculations cannot read future-dated records.
**Group by / Match to:** Group By (field in the source form) and Match To (field in this form) must both be option-type fields or Option Calculations. Example: `Season` in the source matched to `Current Season` here returns the newest value for the matching season; no match gives blank.

**Embedded report ("recent history"):** Embedded Report field; Event = this or another form; fields to display; Date criteria (for example Last 1 Month), record count (Include all / most recent); Advanced: Show about No, Show by No, Show in tables False, Allow add row (False for the same form, True to let users add a record in another form), Show in group mode No. Not testable in Preview.
**Embedded event (link records across forms):** Embedded Event field; source form; Show Date Filters (months ago start/end); fields to display; Advanced: Show date column, Show about column, Show by column, Default show in tables No. Users click **New** to create or **Link** to attach existing records. Embedding in both forms links both ways.
**Embedded event summary:** Embedded Event Summary field below the embedded event; Calculation = exact field name in the embedded form; the embedded event's **Show date column** must be Yes; Advanced: Ignore blanks True, Concatenate (Yes joins all values with `|`). Unlike linked calculations it can read future-dated linked records and updates without resaving.

**Updating calculations:** after adding or changing a calculation, run **Resave** (Forms list, three-dot menu, Resave) over a limited date range, wait for the email, refresh, and verify in a report.

### Debugging checklist for a broken equation (from the course)
1. Unbalanced brackets (count them; the error is often a missing `)` before a comma).
2. Field name case or spacing differs from the form.
3. Invalid symbols or functions: `>=`, `<=`, `isblank`.
4. Option field has scores but the equation compares to text.
5. Referenced field is below the calculation in the form.
6. Wrong calculation type for the data source (table vs normal; history vs current).
7. Unexpected blanks: add `anyempty`/`isempty`/`safe`.
8. Stale results: Resave after changes.
9. `_eq_X is not a known variable` warning means the name X does not match a field.

---

## 4. Linked and history calculations in depth

### 4.1 Linked calculations
Types: Linked Value (number), Linked Text, Linked Option, Linked Date, Linked Time, and Profile Linked equivalents. Settings: source form and field; **which record** (Newest, Oldest, Maximum, Minimum, Current state, Current poor state, Dynamic date); **date restriction** (All history, Date range, Last n days/weeks/months/years, Last custom week/month boundaries); **record count** (all, most recent 1 to 9); Advanced: **Linked update range** (start/end of previous day, current day, or current event), **Ignore blanks**, **Manual update** (adds a button; usually False). "All (history and future)" and "All upcoming" do not work for linked calculations. Months are measured as the length of the previous month. Users need Linked access to the source form and Read access to this one; a linked field appears empty when no record meets the criteria. Cannot be tested in Preview. Typical uses: RPE into a GPS session form to compute load; injury status into availability forms.

### 4.2 History calculations
Syntax: prefix with `Historical`: `historicalmean(Historical Training Load, Training Load)`. Include the current value as the second argument; omit it to use only past records. Do not name fields with history syntax. Draft records are excluded. Controls: **date range** (All history, Date range, Last n units, Last custom), **number of records** (restricting applies only to the historical argument, so with the current value included the count is that number plus one), and **filters** (Match all/any; they filter only historical data, so wrap the current value in `if(...)`; "Match entered data" for single-select/option-calc fields). Results appear only after saving and are blank for the first record if the current value is excluded. EWMA needs a Date calculation and a date restriction several times the decay constant. Use Save and continue to refresh.

---

## 5. Source index (help.teamworks.com/ams/s/article/<slug>)

`introduction-to-calculations`, `using-functions`, `Using-Option-Fields-and-Scores-in-Calculations`, `simple-calculations`, `table-calculations`, `text-calculations`, `option-calculations`, `date-calculation`, `duration-calculation`, `date-difference-calculation-field`, `date-duration-field`, `time-calculation`, `time-difference-calculation-field`, `age-calculation`, `linked-calculations`, `history-calculations`, `personal-details`, `embedded-report-field`, `embedded-event-field`, `embedded-event-summary-field`, `Add-or-Update-a-Calculation-Field-in-an-Existing-Form`.
Course: https://academy.smartabase.com/teamworks-ams-builder-102-using-calculations
Not read: `entered-on-calculation`, `average-sum-minimum-maximum-fields` (slugs unverified), the Resave and Smartsave articles, `linked-data-in-forms`, `related-entity-forms`.
