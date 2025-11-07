# Pricing Tool Development - Chat History

## Project Overview

Building a web-based tool for processing pricing lists into data that can be loaded into a database. The tool compares new pricing data to previous pricing data and flags significant changes (+15% or -10% with cost ≥ £100).

## Initial Requirements

**User Request:**
> I'm looking to build a web based tool for processing pricing lists into data we can load into our database. Data needs to be compared to previous pricing data to compare changes, and flag anything significant (+15% or -10% with a cost greater or equal to £100). The price lists usually come in excel format. If possible, the system will have a spreadsheet editor when you load the pricing list in, that way edits can be made. Once the file is loaded in the user needs to assign column headings so that it knows which data to use where: code, description, cost price, list price, barcode.

## Architecture Decisions

### Initial Approach
Claude suggested a multi-stage workflow application:

1. **File Upload & Parsing** - Use SheetJS (XLSX library) to parse Excel files in browser
2. **Interactive Spreadsheet Editor** - Editable grid for data manipulation
3. **Column Mapping Interface** - Assign columns to: Code, Description, Cost Price, List Price, Barcode
4. **Data Processing & Comparison** - Compare against previous pricing and flag significant changes
5. **Review & Export** - Display results, export as JSON for database import

### Technical Stack Decision

**Options Considered:**
- Pure Frontend (JavaScript/React) - Simple deployment, no backend, browser-only
- PHP Backend - Database storage, larger files, better security
- **Hybrid Approach (CHOSEN)** - Frontend for UX, backend for persistence

**Final Decision:** Hybrid approach with:
- Browser-based editing (in-memory, fast)
- Server save option (database or file formats)
- SheetJS from CDN (no build tools required)
- Single HTML file editable in Notepad++

### Storage Strategy
- **Database**: MySQL/PostgreSQL for querying and comparisons
- **File Storage**: Keep original Excel files and exported JSON for audit trail

## Development Environment

**User Requirement:** Using Notepad++ only (no Node.js, no build tools)

**Solution:** Self-contained HTML file with:
- SheetJS loaded from CDN
- Embedded HTML, CSS, JavaScript
- No installation or dependencies needed

## Feature Evolution

### Version 1: Basic Functionality
- Excel file upload
- Editable spreadsheet view
- Column mapping (Step 3)
- Comparison with previous pricing
- Flagged items display
- Export as JSON or CSV
- "Save to Server" placeholder for PHP integration

### Version 2: Discount Column Support
**User Need:** Price lists without cost price column, but with discount percentages

**Solution:**
- Added optional "Discount % (if no cost price)" mapping
- Flexible cost calculation: `Cost Price = List Price × (1 - Discount/100)`
- Uses Cost Price directly if available, OR calculates from discount

### Version 9: Manual Discount Entry
**User Need:** Not all price lists have discount columns, need manual entry

**Solution:**
- Auto-add "Discount % (Manual)" column when uploading
- Column highlighted in green
- Pre-mapped in Step 3
- Editable for each row

### Version 14: Bulk Discount Operations
**User Problem:** "Not efficient clicking between each box entering the value"

**Solution:**
- Quick Discount Tools panel
- Apply to All rows
- Apply to Selected rows (Ctrl+click row numbers)
- Clear All function
- Keyboard shortcuts: Ctrl+C, Ctrl+V, Ctrl+Shift+V

### Version 19: Multi-Discount Workflow
**User Problem:** "Some suppliers will have many different discounts"

**Solution:**
- Enhanced row selection (Ctrl+click multiple rows)
- Ctrl+Shift+V to paste to all selected rows
- Button-based group operations
- Individual cell paste with Ctrl+V

### Version 22-26: Column Naming & Bug Fixes
**Changes:**
- Renamed column to just "Discount %"
- Fixed bug where "Apply to All" created new columns
- Fixed discount column indexing issues

### Version 31: Column Mapping Moved to Step 2
**User Request:** "Would it be possible we assign columns at the edit pricing data page instead of step 3?"

**Solution:**
- Combined editing and mapping into Step 2
- Dropdowns in blue box above spreadsheet
- Step 3 became preview step

### Version 37: Dropdown Under Each Column
**User Request:** "How about putting the drop down under each column heading"

**Solution:**
- Dropdown directly under each column header (Row 2)
- Added emoji icons for easy identification
- Expanded field options:
  - 📦 Code
  - 📝 Description
  - 💰 Cost Price
  - 💷 List Price
  - 🔢 Barcode
  - 🏷️ Discount %
  - 📈 Markup %
  - 🏭 Manufacturer
  - 🌍 Country of Origin
  - -- Ignore Column --

### Version 42: Quick Tools Moved to Step 3
**User Request:** "Can the Quick discount tools be part of Step 3?"

**New Flow:**
- Step 1: Upload
- Step 2: Edit & Map
- Step 3: Quick Tools & Preview
- Step 4: Results

### Version 45: Markup Support
**User Request:** "If I try to choose cost price, I want to be able to do the same but instead of discounts, it will be for markups!"

**Solution:**
- Intelligent detection: Discount % mapped → show Discount Tools
- Markup % mapped → show Markup Tools
- Formula: `List Price = Cost Price × (1 + Markup/100)`
- Can use both simultaneously

### Version 51: Smart Validation & Flexible Column
**User Problem:** Couldn't proceed without List Price column when using Cost + Markup

**Solution:**
- Column renamed to "Discount/Markup %" (yellow/orange highlight)
- Flexible validation:
  - Requires Code + at least one price field
  - List Price but no Cost → need Discount %
  - Cost Price but no List → need Markup %
  - Both prices → no percentage field needed

### Version 60: Full Spreadsheet in Step 3
**User Problem:** "Apply to selected rows only showing mapped data first 5 rows"

**Solution:**
- Step 3 now shows full editable spreadsheet (not just preview)
- Row selection works properly
- Apply bulk operations see changes immediately

### Version 64: On-Demand Column Creation
**User Request:** "The yellow column Discount/Markup % should not exist until you add a column using a button"

**Solution:**
- No auto-added columns
- Two buttons: "➕ Add Discount % Column" and "➕ Add Markup % Column"
- Columns added only when needed
- Smart duplicate prevention

### Version 74: Final Refinements (Current)
**User Requests:**
1. "Add a tiny delete button on the discount % or markup % columns"
2. "Move the step 3 discount/markup part into step 2"

**Final Solution:**
- ✕ delete button in Discount/Markup column headers
- Quick Tools moved back to Step 2 (all-in-one editing)
- Step 3 simplified to preview only (first 10 rows)

## Final Workflow

### Step 1: Upload File
- Upload Excel file
- Optional: Upload previous pricing for comparison

### Step 2: Edit Data & Map Columns (All-in-One)
- Map columns using dropdowns under headers
- ➕ Add Discount % or Markup % columns as needed
- ✕ Delete unwanted columns
- Edit any cell data directly
- Select rows (Ctrl+click) and apply bulk discounts/markups
- Use keyboard shortcuts (Ctrl+C, Ctrl+V, Ctrl+Shift+V)

### Step 3: Preview
- Clean preview table (first 10 rows)
- Shows calculated prices
- Sanity check before processing

### Step 4: Review Changes & Flagged Items
- Comparison results
- Flagged items (≥15% increase OR ≤-10% decrease with cost ≥£100)
- Export as JSON or CSV
- Save to Server option

## Key Formulas

### Cost Price from Discount
```javascript
Cost Price = List Price × (1 - Discount/100)
// Example: List £500, Discount 20% → Cost £400
```

### List Price from Markup
```javascript
List Price = Cost Price × (1 + Markup/100)
// Example: Cost £400, Markup 25% → List £500
```

### Price Change Flagging
```javascript
const flagged = (
  (costChange >= 15 || costChange <= -10) && item.costPrice >= 100
) || (
  (listChange >= 15 || listChange <= -10) && item.listPrice >= 100
);
```

## Keyboard Shortcuts

- **Ctrl+C**: Copy cell value (flashes yellow)
- **Ctrl+V**: Paste to current cell
- **Ctrl+Shift+V**: Paste to all selected rows
- **Arrow Keys**: Navigate between cells
- **Enter**: Move down to next row
- **Ctrl+Click**: Select multiple rows

## Next Steps for PHP Integration

### Required PHP Endpoints

```php
// GET /api/pricing/latest - Fetch last saved pricing for comparison
// GET /api/pricing/history - List all previous imports
// POST /api/pricing/save - Save new pricing data
```

### Database Schema
```sql
-- For querying and comparisons
pricing_imports (id, import_date, filename, status)
pricing_items (id, import_id, code, description, cost_price, list_price, barcode)
pricing_changes (id, import_id, item_id, change_type, old_value, new_value, flagged)
```

### File Storage Structure
```
/uploads/pricing/YYYY-MM-DD_filename.xlsx  (original files)
/data/pricing/YYYY-MM-DD_import.json       (processed data)
```

## Development Notes

- Development Environment: Notepad++ (no build tools)
- Technology: Single HTML file with embedded CSS/JS
- External Dependency: SheetJS from CDN
- Browser Compatibility: Modern browsers with ES6 support
- File saved as: `pricing-tool.html`

## User Experience Highlights

1. **Emoji Icons** - Visual identification of column types
2. **Color Coding**:
   - Green: Auto-added columns
   - Yellow/Orange: Discount/Markup columns
   - Blue: Selected rows
   - Yellow: Flagged items in results
3. **Smart Validation** - Flexible field requirements
4. **Efficient Bulk Operations** - Multiple methods for applying discounts/markups
5. **In-Memory Editing** - Fast, responsive interface
6. **Server Save** - Persistent storage when needed

## Version History Summary

- **v1**: Basic upload, edit, map, compare, export
- **v2**: Discount column support
- **v9**: Manual discount entry
- **v14**: Bulk operations & keyboard shortcuts
- **v19**: Multi-discount workflow
- **v22-26**: Bug fixes & column naming
- **v31**: Column mapping moved to Step 2
- **v37**: Dropdowns under headers + emojis + new fields
- **v42**: Quick tools moved to Step 3
- **v45**: Markup support
- **v51**: Smart validation & flexible column naming
- **v60**: Full spreadsheet in Step 3
- **v64**: On-demand column creation
- **v74**: Delete buttons + all-in-one Step 2 (CURRENT)
