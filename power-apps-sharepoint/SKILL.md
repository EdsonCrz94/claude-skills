---
name: power-apps-sharepoint
description: Use when designing, building, reviewing or fixing Power Apps canvas apps, Power Fx formulas, or SharePoint / Microsoft Lists data models (lists, columns, indexes, REST provisioning, delegation), including the mandatory admin configuration module for system parameters.
---

# Power Apps + SharePoint (Microsoft Lists) conventions

This skill defines how to build canvas apps on top of SharePoint lists: naming, controls, UI structure, data modeling, the mandatory configuration module, delegation, and known platform traps. Apply every rule unless the user explicitly overrides it for the task at hand.

All examples use generic placeholders: `<SiteUrl>`, `<AppCode>`, `<Module>`, `<Entity>`, `<ListName>`, `<ColumnName>`. Never invent real site URLs, tenant names, users, emails, or business data.

---

## 1. Language rules

| Element | Language |
|---|---|
| Control names, variables, collections, named formulas, components | English |
| SharePoint list names, column names, column descriptions, choice values | English |
| Power Fx formulas, code, comments in code, flow and action names | English |
| **Text shown to the end user** (labels, buttons, placeholders, `Notify` messages, empty states, tooltips) | **Spanish** |

- Names must be descriptive: `galPendingOrders`, not `gal1`; `ShippingDate`, not `Date2`.
- Store choice values in English. When the user must see them in Spanish, translate at display time with a `Switch()`, a lookup to a catalog list, or a named-formula table, never by storing Spanish values.
- No accents, spaces or special characters in any technical name.

---

## 2. Naming conventions

### 2.1 General pattern

| Element | Pattern | Example |
|---|---|---|
| Screen | `scr_<module>_<name>` (snake_case, lowercase) | `scr_orders_queue`, `scr_orders_detail` |
| Control | `<prefix><Module?><Context>` (prefix + camelCase) | `btnSubmitOrder`, `lblOrdersTitle` |
| Component | `cmp<Name>` | `cmpTopBar`, `cmpSidebar` |
| Global variable | `gbl<Name>` | `gblCurrentUser`, `gblNavKey` |
| Context variable | `loc<Name>` | `locPanelOpen` |
| Collection | `col<Name>` | `colAuditItems` |
| Named formula (App.Formulas) | `nf<Name>` or a descriptive noun | `nfTheme`, `nfCfg`, `nfCanApprove` |
| Power Automate flow | `<AppCode>_<Module>_<Action>` | `<AppCode>_Orders_UploadEvidence` |

**Optional module segment.** Control names are global across the whole app, not per screen. When an app has several modules, insert a short module code after the prefix (`lblHrsEventsTitle`, `galPrepQueue`). This prevents Power Apps from auto-renaming duplicates with `_1` suffixes.

### 2.2 Control prefixes

| Prefix | Control | Example |
|---|---|---|
| `btn` | Button | `btnSaveOrder` |
| `lbl` | Label / Text | `lblUserName` |
| `txt` | Text input | `txtSearch` |
| `num` | Number input | `numQuantity` |
| `dd` | Dropdown | `ddDepartment` |
| `cmb` | Combo box | `cmbOwner` |
| `dt` | Date picker | `dtDueDate` |
| `tgl` | Toggle | `tglIsCritical` |
| `chk` | Checkbox | `chkConfirm` |
| `rad` | Radio | `radShift` |
| `gal` | Gallery | `galOrders` |
| `tbl` | Table (modern) | `tblResults` |
| `frm` | Edit / Display form | `frmAttachments` |
| `dc` | Data card | `dcTitle` |
| `img` | Image | `imgLogo` |
| `ico` | Icon | `icoClose` |
| `rec` | Rectangle | `recCardBg` |
| `cir` | Circle | `cirAvatar` |
| `con` | Container | `conHeader` |
| `tab` | Tab list | `tabStatus` |
| `pic` | Add picture / camera | `picEvidence` |
| `att` | Attachments | `attFiles` |
| `tmr` | Timer | `tmrRefresh` |
| `cmp` | Component instance | `cmpTopBar1` |

---

## 3. Controls

### 3.1 Modern first

- Always use **modern controls** by default.
- A **classic control is allowed only when the modern one cannot do what is needed**. Typical valid cases: arbitrary `Fill` with alpha transparency, custom corner radius, hover/pressed states on decorative elements, transparent click areas in gallery rows, controls with no modern equivalent yet.
- When you use a classic control, state why in one line (for example: "Classic/Button because the modern button has no custom radius").
- In YAML, classic controls **must** carry the `Classic/` namespace (`Classic/Button`, `Classic/DatePicker`). Without it Power Apps parses them as modern and rejects classic-only properties (error PA2108).

### 3.2 Modern control specifics

- **Text input.** Property names changed between versions. The current version uses `Default` for the initial value, `.Text` to read input, `Type = TextInputType.Multiline` for multi-line, and `TriggerOutput` to control when `.Text` updates. Older versions used `Value` and `Mode`. Check the control version in the app before writing formulas.
- **Dropdown.**
  - `Default` takes a record: `{Value: "<Option>"}`, not a plain string.
  - It has no `Placeholder`.
  - When inserted, it arrives with `Items = ComboBoxSample` and `ItemDisplayText = ThisItem.Value1`. Always correct these to the real source and `ThisItem.Value`.
- **Combo box.** Exposes only `ItemDisplayText`. To show two fields, concatenate them.
- **Buttons.** Buttons have no `Disabled*` color properties; the theme handles them. Use `Appearance = Primary | Secondary`.
- **Insertion method.** Insert galleries, dropdowns and forms from the Insert panel, never by pasting YAML.
  - A pasted gallery with a copied variant string can render correctly but silently ignore selection clicks.
  - Children of galleries and labels can be pasted via YAML safely.

---

## 4. UI design

### 4.1 Style

The UI must always look **minimalist and modern**:

- Flat surfaces.
- Generous whitespace.
- One accent color used only for primary actions and active states.
- Functional status colors used only for status, never for decoration.
- No gradients, heavy shadows or ornamental shapes.

### 4.2 Fixed layout

Every module screen has three zones:

```
┌──────────────────────────────────────────────┐
│ cmpTopBar  — title + context info            │
├───────────┬──────────────────────────────────┤
│ cmpSidebar│  Content area                    │
│ (menu)    │                                  │
└───────────┴──────────────────────────────────┘
```

- **Top bar** and **sidebar** are always **reusable components defined in the same app** (`cmpTopBar`, `cmpSidebar`).
  - Never rebuild them per screen.
  - The top bar shows the screen title (from a variable each screen sets) plus relevant context, such as the current user, role, shift or date.
- **Components need `Access app scope = On`.** Otherwise theme variables and globals do not resolve inside them.
- **Component instances created via YAML** must declare `Width`, `Height` and `Fill` explicitly.
- **Navigation state is derived from the screen.** Each screen sets its own state in `OnVisible`, not the button that navigated there. That keeps the sidebar highlight in sync whatever route the user took:

```
// scr_<module>_<name>.OnVisible
Set(gblScreenTitle, "<Título visible en español>");
Set(gblNavKey, "<screenKey>");
Reset(gal<Entity>)
```

- For mobile (portrait), the sidebar becomes a bottom navigation bar component. Phone apps are separate apps, because canvas orientation is fixed. They share the same lists but not the components.

### 4.3 Theme as one record

Declare all colors once, as a single record in `App.Formulas` (preferred) or `App.OnStart`. Every control references a token, and no control ever holds a literal color.

```
// App.Formulas
nfTheme = {
    Primary:      RGBA(<r>, <g>, <b>, 1),
    PrimaryDark:  RGBA(<r>, <g>, <b>, 1),
    TextMain:     RGBA(<r>, <g>, <b>, 1),
    TextMuted:    RGBA(<r>, <g>, <b>, 1),
    BgApp:        RGBA(<r>, <g>, <b>, 1),
    BgCard:       RGBA(255, 255, 255, 1),
    Border:       RGBA(<r>, <g>, <b>, 1),
    Success:      RGBA(<r>, <g>, <b>, 1),
    Warning:      RGBA(<r>, <g>, <b>, 1),
    Info:         RGBA(<r>, <g>, <b>, 1)
};
```

- Use the user's brand palette when one is provided; otherwise ask for it.
- Tints and translucency use the **alpha channel of `RGBA`**. `ColorFade()` returns a solid color, not a transparent one.
- On dark backgrounds, verify text contrast (WCAG ≥ 4.5:1). Brand reds and similar mid-tone colors often fail as text color and need a lighter derived token.

### 4.4 Layout rules that prevent rework

- **Label paddings.** Every label gets `PaddingLeft/Right/Top/Bottom = 0`. New controls are born with padding 5, `Wrap = true` and `VerticalAlign = Middle`; set these explicitly.
- **Fixed-height labels.** Set `Wrap = false` on every fixed-height label. Otherwise the text wraps and invades the control below. Truncate long text with `Left(x, n) & "..."` and show the full value in `Tooltip`.
- **Corner radius.** `Rectangle` and `Label` do not support radius. When rounded corners are needed, use a decorative `Classic/Button`.
- **Gallery rows.**
  - Child controls capture clicks. Add a transparent `Classic/Button` covering the whole row with `OnSelect = Select(Parent)`.
  - Place it as the **last child**, because YAML order is z-order.
- **`TemplateSize`** does not accept `ThisItem`. Row height is fixed, so size it for the longest case.
- **Empty states.** Place the empty-state label on the screen, not inside the gallery.
  - Show it with `Visible = CountRows(gal<Entity>.AllItems) = 0`.
  - Its text must describe the active filter, not a generic "no data".
- **Save buttons.** Enable them only when required fields are filled, via `DisplayMode`.
- **`Notify`.** Messages describe what happened, in Spanish, and fire only when something actually changed.
- **Passing records between screens.** Pass the selected record in a global (`Set(gblCurrentRecord, ThisItem)`). Never read `galX.Selected` from another screen.

---

## 5. SharePoint lists (Microsoft Lists)

### 5.1 Naming

| Element | Pattern | Example |
|---|---|---|
| List | `<AppCode>_<Module>_<Entity>` (PascalCase segments) | `<AppCode>_Orders_Headers` |
| Security list | `<AppCode>_Sec_<Entity>` | `<AppCode>_Sec_UserRoles` |
| Catalog list | `<AppCode>_Cat_<Entity>` or `<AppCode>_<Module>_<Entity>` | `<AppCode>_Cat_Departments` |
| Configuration list | `<AppCode>_Cfg_<Entity>` | `<AppCode>_Cfg_Settings` |
| Column | PascalCase, no spaces, English | `ShippingDate`, `ParentID`, `IsActive` |
| Boolean column | `Is<X>` / `Has<X>` / `Can<X>` | `IsActive`, `CanApprove` |
| Foreign key column | `<ParentEntity>ID` | `OrderID`, `AuditID` |

- **Descriptions.** Every column **must have a description in English** that explains its meaning, its allowed values or units, and its source. For example: "Planned shipping date from the planning system. Used for queue ordering."
- **Display names.** Create columns with the final PascalCase name from the start, so the internal name equals the display name. Renaming later leaves a mismatched internal name, and some functions (`SortByColumns`, REST, flows) need the internal one.
- **The Title column.** Reuse `Title` as the main business key or rename its display name. Do not leave it empty and meaningless.

### 5.2 Data model principles

- **Layers.** Separate lists into layers: **security** (users, roles, capability flags), **catalog / master** (slow-changing reference data), **configuration** (system parameters edited by administrators, §6), **transactional** (records created by the app), and **log / history** (append-only events).
- **Capability flags.** Model permissions as capability flags (`CanCreate`, `CanApprove`) on a roles list. Formulas ask "can this user do X?", never "is the role equal to 'Admin'?".
- **Status vs. outcome.** Separate *progress* (`Status`) from *result* (`Outcome`); do not overload one column with both.
- **Snapshots.** Snapshot reference values (standards, weights, item text) into the transactional row at creation time. Later catalog changes must not rewrite history.
- **Stored results.** Store calculated results (scores, totals) at submit time, when they serve as evidence.
- **Checklists.** Checklists and repeated items are **data rows**, rendered by a gallery from a collection, never as N hard-coded controls.
- **Scope.** Keep active scope separate from future phases. Do not create columns "for later".
- **Data dictionary.** Document each list as a data dictionary (column, type, required, indexed, description) before building it.

### 5.3 Relationships (foreign keys)

**Default pattern: a Number column that holds the parent's `ID`**, named `<ParentEntity>ID` and indexed.

```
Filter(<AppCode>_<Module>_Log, OrderID = gblCurrentOrder.ID)
```

- It delegates cleanly (numeric equality).
- It avoids the per-view lookup column threshold.
- It keeps flows and REST writes simple.

**Polymorphic child tables** (for example, one attachments or notes list shared by several parents) use `ParentType` (Choice) plus `ParentID` (Number), both indexed:

```
Filter(<AppCode>_<Module>_Files, ParentType.Value = "<ParentType>", ParentID = gblCurrent.ID)
```

**Alternative: Lookup columns.** Use them when the user wants SharePoint-side referential display or cascading behavior. Filter on the lookup's `.Id`, and keep the number of lookup columns per list low.

### 5.4 Indexes

- Create an index on **every column used to filter, sort or search** in the app, in flows, or in views. This includes foreign keys, `Status`, dates used in ranges, and business keys.
- Create indexes while the list is empty or small. Adding them after a list passes 5,000 items can fail.
- Indexes do not make a formula delegable, and delegation does not replace indexes. Past 5,000 items, SharePoint rejects filtered queries on non-indexed columns regardless of the formula.
- A list supports at most 20 indexes. Choose them from the real query patterns.

### 5.5 Provisioning via REST (browser console or script)

When building lists programmatically with the SharePoint REST API:

- **Creating fields.** Create fields with `/_api/web/lists/getbytitle('<ListName>')/fields/createfieldasxml`. Success returns **HTTP 200**, not 201.
- **Default view.** New fields **do not appear in the default view**. Always run a separate `views/getbytitle('All Items')/viewfields/addviewfield('<ColumnName>')` pass.
- **Internal names.** Never create fields with spaces in the display name; that produces `_x0020_` internal names.
- **Boolean fields.** Include `<Default>0</Default>` in the schema XML.
- **Content types.** `Options: 8` (AddToAllContentTypes) adds the field to every content type without errors.
- **Updating a Choice field.** Send the full `Choices` array in the MERGE body, with the existing values preserved.
- **Renaming `Title`.** Use MERGE with `IF-MATCH: *`; it works reliably.
- **Descriptions and indexes.** Set the description via the field's `Description` property and indexes via `Indexed: true` in a MERGE.
- **Batching.** Batch the work: 2–3 lists per execution. Large scripts can drop the connection mid-run and leave lists half-built.
- **Idempotence.** Make scripts idempotent: check whether a list or field exists before creating it.
- **Verification.** Verify counts by enumerating items. `ItemCount` is unreliable after recycle-bin operations.

---

## 6. App configuration module (mandatory)

Every app ships with a configuration module. Business parameters live in a SharePoint list that administrators edit from inside the app, never as literals in formulas or flows. If the app sends a reminder every 15 days, `15` is a setting that an administrator can change to 7 or 30 without opening the app in the editor or touching a flow.

### 6.1 What counts as a setting

A value is a setting when a business user could reasonably ask to change it without a redesign:

- Frequencies and intervals (reminder every N days, escalation after N hours).
- Thresholds and limits (approval amount, maximum attachments, score cut-offs, tolerances).
- Deadlines and SLAs.
- Recipients (shared mailboxes, distribution lists). Prefer a role or group over an individual person.
- Feature switches (enable a module, require evidence before closing).
- Texts that change with policy (notification subjects and bodies, help texts).

Not settings:

- **Structural changes** (new columns, screens, or Choice options that need new logic). Those are releases.
- **Secrets** (passwords, API keys, connection strings). Every app user can read the settings list, so secrets never go there.
- **Deployment values** that differ between dev, test and prod (site URL, list IDs). Those belong in Power Platform environment variables of the solution, not in the business settings list.

### 6.2 The settings list

`<AppCode>_Cfg_Settings`, one row per setting:

| Column | Type | Indexed | Description |
|---|---|---|---|
| `Title` | Single line of text, unique | Yes | Setting key, `<Module>_<Name>` in PascalCase (`Notifications_ReminderIntervalDays`). Never renamed after release, because formulas and flows look it up by this text. |
| `Module` | Choice | Yes | Module that owns the setting. Groups rows on the admin screen. |
| `DataType` | Choice: `Number`, `Text`, `Boolean`, `Date` | No | Which value column holds the value and which input the admin screen shows. |
| `ValueNumber` | Number | No | Value when `DataType = Number`. |
| `ValueText` | Multiple lines of text (plain) | No | Value when `DataType = Text`. |
| `ValueBoolean` | Yes/No | No | Value when `DataType = Boolean`. |
| `ValueDate` | Date only | No | Value when `DataType = Date`. |
| `DisplayLabel` | Single line of text | No | Spanish label shown to administrators. |
| `HelpText` | Multiple lines of text (plain) | No | Spanish explanation: what the setting controls and what happens when it changes. |
| `Unit` | Single line of text | No | Spanish unit shown next to the input (`días`, `horas`, `%`). |
| `MinValue` / `MaxValue` | Number | No | Allowed range for Number settings. Blank means no limit. |
| `IsEditable` | Yes/No | No | `No` makes the row read-only on the admin screen (technical values shown for reference). |

- **Typed value columns.** Use one value column per data type, not a single text column. Formulas and flows read the value without parsing, and SharePoint validates the type.
- **Seeding.** Create every key with its default value when provisioning the list (part of the §5.5 script). Record key, default and allowed range in the data dictionary.
- **Audit.** Turn on versioning for the list. Version history records who changed which value and when. Add an append-only `<AppCode>_Cfg_SettingsLog` list only if the app itself must show that history.
- **Scalars only.** The settings list holds single values. Repeating reference data (departments, reasons, product types) stays in catalog lists (§5.2), administered from the same module.

### 6.3 Access control

- **Capability flag.** Add `CanManageSettings` to the roles list (§5.2) and expose it as a named formula (`nfCanManageSettings`). Formulas check the flag, never a role name.
- **In the app.** Show the sidebar entry only when `nfCanManageSettings` is true. Also guard each admin screen, so a deep link or a stale navigation cannot open it:

```
// scr_admin_settings.OnVisible
If(!nfCanManageSettings, Navigate(<HomeScreen>));
Set(gblScreenTitle, "Configuración");
Set(gblNavKey, "adminSettings")
```

- **In SharePoint.** Hiding a screen is not security: any user with Edit on the list can change parameters directly in SharePoint. Break permission inheritance on the settings, security and catalog lists. App users get Read; only administrators get Edit.

### 6.4 Admin screens

The configuration module is a normal module (top bar, sidebar, theme, naming rules) with at least:

| Screen | Purpose |
|---|---|
| `scr_admin_settings` | System parameters from `<AppCode>_Cfg_Settings`, grouped by `Module`. |
| `scr_admin_users` | Users, roles and capability flags from the security lists. |
| `scr_admin_catalogs` | Catalog lists: add, edit, and deactivate with `IsActive`. Never delete catalog rows that transactions reference. |

When the app has few settings, users and catalogs, these can be one screen with a tab list.

Settings screen pattern:

- **Gallery.** `galAdmSettings` lists the settings of the active module: `DisplayLabel`, current value with `Unit`, and the last change (`Modified`, `Modified By`).
- **Edit panel.** Selecting a row sets `gblCurrentSetting` and opens a panel with only the input that matches `DataType` (`numAdmSettingValue`, `txtAdmSettingValue`, `tglAdmSettingValue`, `dtAdmSettingValue`, each with `Visible` driven by `DataType`). Show `HelpText` below the input.
- **Validation.** Save is enabled only when the row is editable and a Number value is inside `MinValue` / `MaxValue`. Also disable it while the input still equals the stored value, so `Notify` fires only on a real change (§4.4).

```
// btnAdmSettingsSave.DisplayMode
With(
    {
        t: gblCurrentSetting.DataType.Value,
        n: numAdmSettingValue.Value
    },
    If(
        gblCurrentSetting.IsEditable &&
        // Number values must be present and inside the allowed range
        (
            t <> "Number" ||
            (
                !IsBlank(n) &&
                (IsBlank(gblCurrentSetting.MinValue) || n >= gblCurrentSetting.MinValue) &&
                (IsBlank(gblCurrentSetting.MaxValue) || n <= gblCurrentSetting.MaxValue)
            )
        ) &&
        // The input must differ from the stored value
        !Switch(
            t,
            "Number",  n = gblCurrentSetting.ValueNumber,
            "Text",    txtAdmSettingValue.Text = gblCurrentSetting.ValueText,
            "Boolean", tglAdmSettingValue.Checked = gblCurrentSetting.ValueBoolean,
            "Date",    dtAdmSettingValue.SelectedDate = gblCurrentSetting.ValueDate
        ),
        DisplayMode.Edit,
        DisplayMode.Disabled
    )
)

// btnAdmSettingsSave.OnSelect
With(
    {t: gblCurrentSetting.DataType.Value},
    Set(
        gblCurrentSetting,
        Patch(
            <AppCode>_Cfg_Settings,
            gblCurrentSetting,
            {
                ValueNumber:  If(t = "Number",  numAdmSettingValue.Value,       gblCurrentSetting.ValueNumber),
                ValueText:    If(t = "Text",    txtAdmSettingValue.Text,        gblCurrentSetting.ValueText),
                ValueBoolean: If(t = "Boolean", tglAdmSettingValue.Checked,     gblCurrentSetting.ValueBoolean),
                ValueDate:    If(t = "Date",    dtAdmSettingValue.SelectedDate, gblCurrentSetting.ValueDate)
            }
        )
    )
);
Refresh(<AppCode>_Cfg_Settings);
Notify("Se actualizó «" & gblCurrentSetting.DisplayLabel & "».", NotificationType.Success)
```

The patch record has the same shape in every case (each column keeps its stored value unless it is the active type). This avoids a `Switch` whose branches return records with different columns.

### 6.5 Reading settings in the app

Expose settings as one named-formula record, the same way as the theme (§4.3):

```
// App.Formulas
nfCfg = {
    ReminderIntervalDays: LookUp(<AppCode>_Cfg_Settings, Title = "Notifications_ReminderIntervalDays").ValueNumber,
    MaxAttachments:       LookUp(<AppCode>_Cfg_Settings, Title = "Evidence_MaxAttachments").ValueNumber,
    IsEvidenceRequired:   LookUp(<AppCode>_Cfg_Settings, Title = "Evidence_IsRequired").ValueBoolean
};
```

- **No literals.** Formulas use `nfCfg.<Name>`. No formula contains a business value as a literal.
- **Delegation.** `Title = "<key>"` delegates, and the list is small.
- **No hidden fallbacks.** Do not write `Coalesce(nfCfg.ReminderIntervalDays, 15)`. A fallback reintroduces the hard-coded value and hides a missing key. A missing key is fixed by adding the row.
- **Freshness.** After a save, `Refresh(<AppCode>_Cfg_Settings)` makes the current session read the new value. Other open sessions pick it up when the app restarts.

### 6.6 Reading settings in flows

- **Run-time read.** Flows read each setting when they run, with *Get items* on `<AppCode>_Cfg_Settings`, Filter Query `Title eq '<key>'`, Top Count 1. Never type the value into the flow.
- **Schedules.** The *Recurrence* trigger cannot take its interval from a list. For "every N days" behavior, schedule the flow daily and decide inside it which records are due, using a `LastNotifiedOn` date column (indexed) on the record:

```
Trigger      Recurrence: every 1 day
Get items    Get_ReminderIntervalDays
             List: <AppCode>_Cfg_Settings
             Filter Query: Title eq 'Notifications_ReminderIntervalDays'
             Top Count: 1
Compose      CutoffDate
             addDays(utcNow(), mul(-1, int(first(body('Get_ReminderIntervalDays')?['value'])?['ValueNumber'])), 'yyyy-MM-dd')
Get items    Get_DueRecords
             List: <AppCode>_<Module>_<Entity>
             Filter Query: LastNotifiedOn le '@{outputs('CutoffDate')}' or LastNotifiedOn eq null
Apply to each due record
             Send the notification
             Update item: LastNotifiedOn = utcNow()
```

When an administrator changes the interval from 15 to 7 days, the next daily run already uses 7, and nobody edits the flow. If the setting row is missing, `int()` fails and the flow run fails visibly instead of silently using a wrong value.

---

## 7. Delegation

### 7.1 Principles

- Always write **delegable formulas first**. Non-delegable queries silently process only the first N rows. N is 500 by default and at most 2,000 via *Settings → General → Data row limit*.
- The authoritative check is **App Checker → Formulas**. Canvas warning triangles mix accessibility and performance issues and can mislead.
- Filter at the source, then shape locally. `Filter` / `LookUp` first; `AddColumns`, `ForAll` and heavy transforms only on the reduced set.
- `ClearCollect(col, <List>)` materializes at most the row limit. Never collect a whole large list and then filter the collection.

### 7.2 SharePoint delegation quick reference

| Pattern | Delegable? |
|---|---|
| `Filter` / `LookUp` with `=`, `<>`, `<`, `>`, `<=`, `>=` on Text, Number, Date | Yes |
| `=` on Choice (`Status.Value = "<Value>"`), Boolean, Lookup `.Id` | Yes |
| `ID` column | Only `=` |
| `&&`, `||` combining delegable predicates | Yes |
| `StartsWith(<TextColumn>, txtSearch.Text)` | Yes |
| `Sort` / `SortByColumns` on one supported column | Yes |
| `CountRows`, `Sum`, `Average`, `Max`, `Min` over a SharePoint list | **No** |
| `Search()`, `in`, `Len`, `Lower`, `Upper`, `Trim` on a column | **No** |
| `<>` on Choice, Person sub-fields (`.Email`, `.DisplayName`) | **Risky.** Often flagged; verify in App Checker |
| `Filter` whose source is `Switch(...)` / `If(...)` returning tables | **No.** The source is no longer a direct table reference |
| `First` / `FirstN` applied *before* `Filter` | **No** |
| `Filter` on a collection | Not applicable (local, but limited to what was collected) |

### 7.3 Preferred patterns

- **Search.** Use `StartsWith` on an indexed text column instead of `Search()` or `in`.
- **Tab or filter switching.** Instead of `Filter(Switch(var, "a", Filter(...), "b", Filter(...)))`, write one `Filter` over the list whose predicates encode each branch with delegable conditions. Alternatively, use separate galleries toggled with `Visible`.
- **Counts and KPIs.** Get them from `CountRows(gal<Entity>.AllItems)` over an already delegated gallery (bounded by what the gallery loaded), from a flow, from Power BI, or from a counter stored in a summary list.
- **Sorting.** Prefer `Sort(<source>, <ColumnReference>)`. `SortByColumns` requires the **internal** column name as text, which can differ from the display name.
- **Keys and variables.** Compare key columns against a variable, not a nested lookup. Pre-compute values with `With()` or `Set()`.

### 7.4 Mandatory reporting when a warning is unavoidable

When a formula cannot be made delegable, say so explicitly next to the formula, in this format:

```
⚠ Delegation warning — <control>.<property>
Reason: <which function/operator breaks delegation and why>
Limit: only the first <500 | configured N ≤ 2000> rows of <ListName> are evaluated
Current / expected rows: <n> / <growth estimate>
Risk: <Low | Medium | High>
Mitigation: <index, restructure, flow, summary list, raise row limit, monitor threshold>
```

To test delegation for real, look up a record known to sit beyond the row limit. If `LookUp` finds it but a collected copy does not, the query delegates.

---

## 8. Known platform traps

- **Choice columns:**
  - Read with `.Value` and write with `{Value: "<Option>"}`.
  - Writing a value not in the allowed choices fails silently.
- **Blank vs. zero.** Aggregations over empty sets can return `Blank()`, not `0`, especially from Power BI or DAX sources. Test with `IsBlank()`, not `= 0`.
- **Gallery scroll.** Galleries keep their scroll position between visits and refreshes:
  - Put `Reset(gal<Entity>)` in `OnVisible`.
  - After inserting a record, call `Refresh(<List>); Reset(gal<Entity>)`.
- **Controls outside a form.** A control outside an `EditForm` evaluates `Default` only when created. Only `Reset()` forces a re-read.
- **Resetting a dropdown.** `Reset()` on a dropdown returns it to its `Default`; it does not clear it.
- **Multi-step captures.**
  - Keep values in globals, because controls lose state across screens.
  - Guard `OnVisible` resets with a fresh-entry flag, so returning from the next step does not wipe the data.
- **Dependent dropdowns.**
  - Feed the child from a context variable, not directly from the parent control, so prefilled values survive recalculation.
  - Reset the child in the parent's `OnChange`.
- **Attachments.**
  - They exist only inside an `EditForm`. Use a hybrid: `Patch` for fields, a minimal `EditForm` only for attachments.
  - In `FormMode.New`, `Item` is ignored, so set defaults on the **DataCard**.
  - `Form.Updates.Attachments` returns blank. Check `CountRows(<AttachmentsControl>.Attachments) > 0` instead.
- **Column names.** Never assume a column name in a formula. The same column may resolve by display name or internal name depending on when the control was created, so read the existing formula before rewriting it.
- **Stable device identity.** For shared devices, use `SaveData` / `LoadData`; this works only in the mobile player.
- **Mobile phone apps:**
  - Font sizes are about 1.6× tablet sizes.
  - Touch targets are at least 72 logical px tall.
  - Screens do not scroll; everything must fit, or use a scrollable container.
- **Long formulas.** Formulas longer than about 200 characters pasted through automation can be truncated. Paste them manually, or write them directly into the formula bar field.
- **Flows called from apps.** When users without premium licenses run a flow that uses premium or shared connections, configure run-only connections to use the owner's embedded credentials rather than "provided by run-only user". Only do this when licensing policy allows it.

---

## 9. Definition of done (checklist)

Before delivering any app change, list schema, or formula, confirm:

- [ ] Every control, variable, collection and component follows the naming tables. Screens use `scr_<module>_<name>`.
- [ ] Every technical name is English; every user-facing text is Spanish.
- [ ] Modern controls are used everywhere. Each classic control has a one-line justification.
- [ ] Top bar and sidebar are reusable components with `Access app scope = On`. The screen sets title and nav state in `OnVisible`.
- [ ] No literal colors: all colors come from the theme record.
- [ ] Lists are named `<AppCode>_<Module>_<Entity>`, columns are PascalCase, and every column has an English description.
- [ ] Every filter, sort or search column is indexed. Relationships use `<Parent>ID` Number columns (or Lookup, with justification).
- [ ] The app has a configuration module (§6). No business parameter (interval, threshold, limit, recipient, switch) is a literal in a formula or flow; formulas read `nfCfg`, flows read `<AppCode>_Cfg_Settings` at run time.
- [ ] Admin screens are gated by `CanManageSettings`, and only administrators have Edit on the settings, security and catalog lists.
- [ ] App Checker → Formulas shows no delegation warnings, or each one is reported in the §7.4 format.
- [ ] No real URLs, tenant names, user names or business data appear in shared artifacts; placeholders are used instead.
