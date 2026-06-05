# Repeating Rows in Canvas Power Apps — Complete Guide

> A comprehensive reference for building repeating row functionality in Microsoft Power Apps Canvas applications. Covers all available methods, best practices, known limitations, and performance tips.

---

## Table of Contents

- [Overview](#overview)
- [Method 1: Gallery Control](#method-1-gallery-control)
- [Method 2: Flexible Height Gallery](#method-2-flexible-height-gallery)
- [Method 3: Data Table Control](#method-3-data-table-control)
- [Method 4: Collection + ForAll for Dynamic Rows](#method-4-collection--forall-for-dynamic-rows)
- [Method 5: Repeating Sections with Components](#method-5-repeating-sections-with-components)
- [Method 6: HTML Table via HtmlText Control](#method-6-html-table-via-htmltext-control)
- [Adding & Removing Rows Dynamically](#adding--removing-rows-dynamically)
- [Editable Repeating Rows (Inline Edit)](#editable-repeating-rows-inline-edit)
- [Best Practices](#best-practices)
- [Limitations](#limitations)
- [Performance Considerations](#performance-considerations)
- [Common Errors & Troubleshooting](#common-errors--troubleshooting)
- [Comparison Table](#comparison-table)
- [References](#references)

---

## Overview

Canvas Power Apps does not natively support a traditional "repeating rows" control like InfoPath or HTML forms. Instead, you simulate repeating rows using **Gallery controls**, **Collections**, **Components**, or **HTML rendering**. Each approach has different trade-offs in terms of flexibility, performance, and complexity.

---

## Method 1: Gallery Control

The **Gallery** is the most common and recommended method for repeating rows in Canvas Power Apps.

### How It Works

A Gallery binds to a data source (table, collection, or array) and renders one "row template" per record — automatically repeating for every record in the source.

### Setup Steps

1. Insert a **Vertical Gallery** from the **Insert** menu.
2. Set the `Items` property to your data source:

```powerapps
Gallery1.Items = Employees
```

3. Add controls inside the gallery (Labels, TextInputs, Icons) for each column.
4. Reference the current row's fields using `ThisItem`:

```powerapps
// Label for employee name
Label1.Text = ThisItem.EmployeeName

// Label for department
Label2.Text = ThisItem.Department

// Conditional formatting per row
Label1.Color = If(ThisItem.IsActive, Color.Green, Color.Red)
```

### Filtering & Sorting Rows

```powerapps
// Filter
Gallery1.Items = Filter(Employees, Department = "Sales")

// Sort
Gallery1.Items = Sort(Employees, LastName, SortOrder.Ascending)

// Search + Filter combined
Gallery1.Items = SortByColumns(
    Search(Employees, SearchInput.Text, "EmployeeName", "Department"),
    "EmployeeName",
    If(SortToggle, SortOrder.Descending, SortOrder.Ascending)
)
```

### Row Selection

```powerapps
// Track selected row
Set(varSelectedEmployee, Gallery1.Selected)

// Highlight selected row
Gallery1.TemplateFill = If(
    ThisItem.IsSelected,
    RGBA(0, 120, 212, 0.1),
    RGBA(255, 255, 255, 1)
)
```

---

## Method 2: Flexible Height Gallery

A **Flexible Height Gallery** allows each row to dynamically resize based on its content — useful when rows have variable-length text or optional sections.

### When to Use

- Rows contain multiline text (e.g., comments, descriptions)
- Some rows show/hide sub-sections based on data
- Accordion-style expandable rows

### Setup

1. Insert a Gallery → Change **Layout** to **Flexible height** in the properties panel.
2. Set the template height to auto-expand:

```powerapps
// TemplateSize property of the gallery — set dynamically
Gallery1.TemplateSize = If(
    ThisItem.IsExpanded,
    200,
    60
)
```

3. Show/hide inner controls per row:

```powerapps
// Detail section visible only for expanded rows
DetailContainer.Visible = ThisItem.IsExpanded
```

### Expand/Collapse Toggle per Row

```powerapps
// On toggle button OnSelect inside gallery
Patch(
    colExpandState,
    LookUp(colExpandState, ID = ThisItem.ID),
    { IsExpanded: !ThisItem.IsExpanded }
)
```

---

## Method 3: Data Table Control

The **Data Table** is a built-in read-only grid that auto-generates column headers and repeating rows.

### When to Use

- Read-only tabular display
- Rapid prototyping
- No need for row-level custom formatting

### Setup

```powerapps
// Set Items
DataTable1.Items = Employees

// Columns are configured in the properties panel
// Each column has a FieldName (bound to data) and HeaderText
```

### Limitations of Data Table

- **Read-only** — no inline editing support
- Cannot customize row templates (no icons, buttons, or colors per row)
- Limited styling options
- No row selection events
- **Not recommended for production apps** — limited flexibility

---

## Method 4: Collection + ForAll for Dynamic Rows

Use a **Collection** as the data source for a Gallery to create, modify, and manage rows entirely in memory — ideal for form scenarios where rows are built by the user before saving.

### Initialize a Collection

```powerapps
// On App Start or Screen OnVisible
ClearCollect(
    colOrderLines,
    {
        RowID: 1,
        ProductName: "",
        Quantity: 0,
        UnitPrice: 0,
        Total: 0
    }
)
```

### Add a New Row

```powerapps
// Add Row button OnSelect
Collect(
    colOrderLines,
    {
        RowID: Max(colOrderLines, RowID) + 1,
        ProductName: "",
        Quantity: 0,
        UnitPrice: 0,
        Total: 0
    }
)
```

### Remove a Row

```powerapps
// Delete icon inside Gallery OnSelect
Remove(colOrderLines, ThisItem)
```

### Update a Row (Inline Edit)

```powerapps
// TextInput OnChange inside Gallery for Quantity field
Patch(
    colOrderLines,
    ThisItem,
    { Quantity: Value(QuantityInput.Text) }
)
```

### Computed Columns with ForAll

```powerapps
// Recalculate totals before saving
ClearCollect(
    colOrderLinesCalc,
    ForAll(
        colOrderLines,
        {
            RowID: RowID,
            ProductName: ProductName,
            Quantity: Quantity,
            UnitPrice: UnitPrice,
            Total: Quantity * UnitPrice
        }
    )
)
```

### Save Collection to Data Source

```powerapps
// Save all rows to SharePoint or Dataverse
ForAll(
    colOrderLines,
    Patch(
        OrderLines,
        Defaults(OrderLines),
        {
            OrderID: varCurrentOrderID,
            ProductName: ProductName,
            Quantity: Quantity,
            UnitPrice: UnitPrice
        }
    )
)
```

---

## Method 5: Repeating Sections with Components

**Power Apps Components** let you create a reusable row template and stamp it out inside a Gallery — the most scalable approach for complex row UI.

### When to Use

- Rows have complex UI (multiple inputs, dropdowns, date pickers)
- Row template is reused across multiple screens/apps
- Team development — shared component library

### Setup

1. Go to **Components** tab in the Tree View → **New Component**
2. Add **Custom Properties** for data in/out:

```powerapps
// Input property: RowData (Record type)
// Output property: RowChanged (Boolean)
```

3. Design the row layout inside the component using `Component.RowData.FieldName`
4. Use the component inside a Gallery:

```powerapps
// Component inside Gallery
ComponentRow.RowData = ThisItem
```

### Passing Data Back from Component

```powerapps
// Inside component, fire custom output property on input change
UpdateContext({ locChanged: true })

// In Gallery, listen to component output
If(ComponentRow.RowChanged, Patch(colData, ThisItem, ComponentRow.UpdatedRecord))
```

---

## Method 6: HTML Table via HtmlText Control

For **read-only display**, you can generate an HTML table string dynamically and render it with the `HtmlText` control.

### When to Use

- Export-like display (invoices, reports)
- Complex cell formatting via HTML/CSS
- Read-only, no interactivity needed

### Generate HTML Table

```powerapps
// Set HtmlText.HtmlText property
"<table style='width:100%;border-collapse:collapse;font-family:Segoe UI;font-size:14px;'>" &
"<thead><tr style='background:#0078D4;color:white;'>" &
"<th style='padding:8px;text-align:left;'>Product</th>" &
"<th style='padding:8px;text-align:right;'>Qty</th>" &
"<th style='padding:8px;text-align:right;'>Price</th>" &
"<th style='padding:8px;text-align:right;'>Total</th>" &
"</tr></thead><tbody>" &
Concat(
    colOrderLines,
    "<tr style='border-bottom:1px solid #eee;'>" &
    "<td style='padding:8px;'>" & ProductName & "</td>" &
    "<td style='padding:8px;text-align:right;'>" & Text(Quantity) & "</td>" &
    "<td style='padding:8px;text-align:right;'>" & Text(UnitPrice, "$#,##0.00") & "</td>" &
    "<td style='padding:8px;text-align:right;'>" & Text(Quantity * UnitPrice, "$#,##0.00") & "</td>" &
    "</tr>"
) &
"</tbody></table>"
```

---

## Adding & Removing Rows Dynamically

### Full Add/Remove Pattern with Auto Row Numbers

```powerapps
// ── INITIALIZE ──────────────────────────────────────────
// Screen OnVisible
ClearCollect(colLines, { ID: 1, Item: "", Qty: 1, Price: 0 });
Set(varNextID, 2)

// ── ADD ROW ─────────────────────────────────────────────
// "Add Row" button OnSelect
Collect(colLines, { ID: varNextID, Item: "", Qty: 1, Price: 0 });
Set(varNextID, varNextID + 1)

// ── REMOVE ROW ──────────────────────────────────────────
// Trash icon inside Gallery OnSelect
If(
    CountRows(colLines) > 1,
    Remove(colLines, ThisItem),
    Notify("At least one row is required", NotificationType.Warning)
)

// ── DUPLICATE ROW ────────────────────────────────────────
// Duplicate icon inside Gallery OnSelect
Collect(
    colLines,
    {
        ID: varNextID,
        Item: ThisItem.Item,
        Qty: ThisItem.Qty,
        Price: ThisItem.Price
    }
);
Set(varNextID, varNextID + 1)

// ── MOVE ROW UP ──────────────────────────────────────────
// Requires a SortOrder column in the collection
Patch(colLines, ThisItem, { SortOrder: ThisItem.SortOrder - 1 });
Patch(
    colLines,
    LookUp(colLines, SortOrder = ThisItem.SortOrder - 1 && ID <> ThisItem.ID),
    { SortOrder: ThisItem.SortOrder }
)
```

---

## Editable Repeating Rows (Inline Edit)

### TextInput Inside Gallery (Live Edit Pattern)

```powerapps
// TextInput Default value (inside Gallery)
TextInput_Item.Default = ThisItem.Item

// TextInput OnChange — immediately patch to collection
Patch(colLines, ThisItem, { Item: TextInput_Item.Text })
```

### Dropdown Inside Gallery

```powerapps
// Dropdown Items
Dropdown_Category.Items = ["Electronics", "Clothing", "Food", "Other"]

// Dropdown Default
Dropdown_Category.DefaultSelectedItems = Filter(
    ["Electronics", "Clothing", "Food", "Other"],
    Value = ThisItem.Category
)

// Dropdown OnChange
Patch(colLines, ThisItem, { Category: Dropdown_Category.Selected.Value })
```

### DatePicker Inside Gallery

```powerapps
// DatePicker DefaultDate
DatePicker_Due.DefaultDate = ThisItem.DueDate

// DatePicker OnChange
Patch(colLines, ThisItem, { DueDate: DatePicker_Due.SelectedDate })
```

### Running Total / Summary Row

```powerapps
// Outside Gallery — summary labels
Label_GrandTotal.Text = Text(
    Sum(colLines, Qty * Price),
    "$#,##0.00"
)

Label_RowCount.Text = CountRows(colLines) & " item(s)"
```

---

## Best Practices

### Data Management

- **Always use a local Collection** as the gallery's data source for editable repeating rows. Avoid binding directly to a data source (SharePoint, Dataverse) for live editing — this causes excessive network calls per keystroke.
- **Save explicitly** — provide a clear "Save" button that commits the collection to the backend using `ForAll + Patch` or `Patch` with a table argument.
- **Use `ID` fields** to uniquely identify rows, especially when rows can be added/deleted. Never rely on gallery index for identity.

```powerapps
// Good: identify by ID
Patch(colLines, LookUp(colLines, ID = varTargetID), { Qty: 5 })

// Risky: identify by position (breaks after deletions)
// Patch(colLines, First(colLines), { Qty: 5 })
```

### Gallery Design

- **Fix template height** when all rows are uniform — flexible height has a higher rendering cost.
- **Avoid deeply nested Galleries** — Power Apps does not support Gallery inside Gallery. Use a flat data model with parent/child IDs instead.
- **Limit controls per row template** — every control in the template is multiplied by the number of rows. 10 controls × 100 rows = 1,000 rendered controls.
- **Use `Visible` instead of removing controls** for conditional column display — it is cheaper than dynamically adding/removing controls.

```powerapps
// Good: toggle visibility
OptionalLabel.Visible = ThisItem.ShowOptionalField

// Avoid: putting complex nested containers with many sub-controls in every row
```

### Collections

- **`ClearCollect` on navigation** — avoid accumulating stale data. Reinitialize collections on `OnVisible` for screens users navigate back to.
- **Use `Patch` (not `Remove + Collect`) for updates** — `Patch` is atomic; `Remove + Collect` causes a visual flicker.
- **Validate before saving** — check for empty required fields before `ForAll + Patch`:

```powerapps
If(
    CountRows(Filter(colLines, IsBlank(Item))) > 0,
    Notify("All rows must have an Item name.", NotificationType.Error),
    // proceed with save
    ForAll(colLines, Patch(OrderLines, Defaults(OrderLines), ThisRecord))
)
```

### UX

- **Always show a row count** and total so users know how many rows exist, especially when the list scrolls.
- **Confirm before deleting a row** — use a confirmation dialog or `Notify` for destructive actions.
- **Provide an empty state** — show a friendly message when there are no rows yet.

```powerapps
EmptyStateLabel.Visible = CountRows(colLines) = 0
```

---

## Limitations

| Area | Limitation |
|---|---|
| **Nested Galleries** | Not supported. Galleries cannot be placed inside other Galleries. |
| **Row Reordering (Drag & Drop)** | No native drag-and-drop. Must be simulated with Up/Down buttons and a SortOrder column. |
| **Maximum Rows Rendered** | Gallery renders up to **500 rows** by default (controlled by `Items` delegation). Beyond this requires manual pagination. |
| **Delegation** | `Filter`, `Search`, `CountRows` on large tables may not fully delegate — a yellow warning appears. Non-delegable queries are capped at 500 records (configurable up to 2,000 in settings). |
| **Inline Editing Performance** | `Patch` on every `OnChange` event in a large collection can feel sluggish. Consider patching on `OnFocusLost` instead. |
| **Data Table** | Read-only. Cannot add buttons, icons, or custom formatting per row. |
| **Component Limitations** | Components cannot contain Galleries. Custom output properties have limited types. |
| **HtmlText Interactivity** | HTML rendered via `HtmlText` control is fully static — no click handlers, no form inputs. |
| **Formula Complexity** | `ForAll` and `Concat` over large collections can hit formula complexity limits or cause slow evaluation. |
| **No Virtual Scrolling** | Power Apps loads all gallery items into the DOM. Very large row counts (500+) will degrade performance regardless of scrolling position. |
| **Row Height Minimum** | Flexible Height Gallery has a minimum template height — rows cannot collapse to zero height. |
| **Offline Collections** | Collections are in-memory only and lost on app close. Use `SaveData` / `LoadData` for offline persistence (Canvas apps only, not in Teams). |

---

## Performance Considerations

### Delegation Strategy

Always push filtering to the server to avoid the 500-record cap:

```powerapps
// ✅ Delegable (SharePoint / Dataverse)
Gallery1.Items = Filter(Products, Category = "Electronics")

// ❌ Non-delegable — only searches first 500 records
Gallery1.Items = Filter(Products, StartsWith(Description, "Smart"))
```

### Reduce Re-renders

- Set gallery `Items` to a stable expression. Avoid calling `Sort`, `Filter`, and `Search` in a long chain inside `Items` if the source data changes frequently.
- Cache sorted/filtered results in a collection and bind the gallery to the collection:

```powerapps
// Button or timer refreshes the cache
ClearCollect(colFilteredProducts, Sort(Filter(Products, IsActive), Name))

// Gallery binds to the cached collection
Gallery1.Items = colFilteredProducts
```

### Batch Save with Patch Table Argument

Instead of `ForAll + Patch` (one network call per row), use `Patch` with a table — a single call:

```powerapps
// ✅ Single network call — much faster
Patch(OrderLines, colLines)

// ❌ N network calls (one per row) — slow for large sets
ForAll(colLines, Patch(OrderLines, Defaults(OrderLines), ThisRecord))
```

> **Note:** `Patch(Table, Collection)` works when the collection records match the table schema exactly and have valid IDs for existing records.

### Limit Visible Controls

```powerapps
// Show columns conditionally to reduce DOM nodes
PriceLabel.Visible = varShowPricing
NoteLabel.Visible = Len(ThisItem.Note) > 0
```

---

## Common Errors & Troubleshooting

### "This formula uses 'ThisItem' outside of a Gallery"

**Cause:** You're referencing `ThisItem` in a property outside the gallery (e.g., on a screen label).  
**Fix:** Store the selected record in a variable and reference that instead:

```powerapps
// Gallery OnSelect
Set(varSelected, Gallery1.Selected)

// Outside gallery
Label.Text = varSelected.EmployeeName
```

---

### Delegation Warning (Yellow Triangle)

**Cause:** A formula in `Items` contains a non-delegable function.  
**Fix:** Move logic server-side (use indexed columns in SharePoint, views in Dataverse) or accept the 500-record limit for small datasets. Increase the limit in **Settings → General → Data row limit** (max 2,000).

---

### Gallery Doesn't Refresh After Patch

**Cause:** Gallery is bound to a data source and the local cache hasn't refreshed.  
**Fix:** Call `Refresh()` after patching:

```powerapps
Patch(Employees, ThisItem, { Department: "HR" });
Refresh(Employees)
```

Or use a local collection as intermediary and only call `Refresh` when navigating away.

---

### Duplicate Rows Appear After `Collect`

**Cause:** `Collect` appends — if called multiple times without clearing, duplicates accumulate.  
**Fix:** Use `ClearCollect` to reinitialize, or check existence before adding:

```powerapps
If(
    IsEmpty(Filter(colLines, ID = varNewID)),
    Collect(colLines, { ID: varNewID, Item: "" })
)
```

---

### Row Edits Not Persisting in Gallery

**Cause:** TextInput `Default` resets on re-render when `Patch` isn't called, or the collection isn't updated.  
**Fix:** Always call `Patch` in `OnChange` (not `OnSelect`) and bind `Default` to the collection field, not a local variable.

```powerapps
// TextInput Default
ThisItem.ProductName

// TextInput OnChange
Patch(colLines, ThisItem, { ProductName: Self.Text })
```

---

### "Cannot insert a Gallery inside a Gallery"

**Cause:** Native limitation — nested galleries are not allowed.  
**Fix:** Flatten your data model. Use parent/child ID columns in a single flat collection, and filter child rows inside a single gallery using `Filter(colLines, ParentID = varSelectedParent.ID)`.

---

## Comparison Table

| Method | Editable | Custom UI | Nested Data | Performance | Complexity |
|---|---|---|---|---|---|
| **Vertical Gallery** | ✅ Yes | ✅ Full | ❌ No | ⭐⭐⭐⭐ | Low |
| **Flexible Height Gallery** | ✅ Yes | ✅ Full | ❌ No | ⭐⭐⭐ | Medium |
| **Data Table** | ❌ No | ❌ Limited | ❌ No | ⭐⭐⭐⭐⭐ | Very Low |
| **Collection + ForAll** | ✅ Yes | ✅ Full | ⚠️ Workaround | ⭐⭐⭐⭐ | Medium |
| **Component Rows** | ✅ Yes | ✅ Full | ❌ No | ⭐⭐⭐ | High |
| **HTML Table** | ❌ No | ✅ CSS only | ✅ Yes | ⭐⭐⭐⭐⭐ | Medium |

---

## References

- [Microsoft Docs — Gallery control](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-gallery)
- [Microsoft Docs — Collect, ClearCollect functions](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/functions/function-clear-collect-clearcollect)
- [Microsoft Docs — Patch function](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/functions/function-patch)
- [Microsoft Docs — ForAll function](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/functions/function-forall)
- [Microsoft Docs — Delegation overview](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/delegation-overview)
- [Microsoft Docs — Canvas App Components](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/create-component)
- [Microsoft Docs — Data Table control](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-data-table)

---

## Contributing

Found an error or want to add another method? PRs welcome! Please include:
- A clear description of the method or fix
- Working Power Apps formula code
- The scenario where it applies

---

*Last updated: June 2026 | Power Apps version: 3.26xxx and later*
