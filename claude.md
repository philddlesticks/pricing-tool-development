# Pricing Tool Development

## Project Overview

A web-based tool for processing pricing lists that can be loaded into a database. The tool compares new pricing data against previous pricing data and flags significant changes (≥15% increase OR ≤-10% decrease with cost ≥£100).

## Current Architecture

### Technical Stack
- **Single HTML File** (`index.html`) - Self-contained application
- **External Stylesheet** (`pricingstyles.css`) - Separated CSS for maintainability
- **SheetJS Library** - Loaded from CDN for Excel/CSV parsing
- **Client-Side Processing** - All processing happens in the browser
- **No Build Tools Required** - Editable in any text editor

### File Format Support
- Excel files (`.xlsx`, `.xls`)
- CSV files (`.csv`)
- JSON (for previous pricing data comparison)

## Current Workflow (7 Steps)

### Step 1: Settings
Configure pricing list metadata before uploading:
- **Manufacturer Name** - Brand or manufacturer name (required)
- **Supplier Account Number** - Account identifier (required)
- **Price List Effective Date** - Date selector (required)

All fields validated before proceeding.

### Step 2: Upload Pricing File
- Upload main pricing file (Excel or CSV)
- Optional: Upload previous pricing data (Excel, CSV, or JSON)
- "Next" button enables after file upload
- User can upload both files before proceeding

### Step 3: Map Columns
Assign columns from uploaded file to data fields:

**Available Mappings:**
- 📦 Supplier Part Number * (required)
- 📝 Description
- 💰 Cost Price
- 💷 List Price GBP * (or Cost Price required)
- 🔢 Barcode
- 🏭 Manufacturer
- 🌍 Country of Origin
- -- Ignore Column --

**Features:**
- Dropdowns under each column header
- All columns default to "Ignore" (no auto-assignment)
- First 5 rows shown as preview
- Validation before proceeding

### Step 4: Edit Data
Full spreadsheet editor with all mapped columns:

**Column Order (Fixed):**
1. Actions (❌ exclude/restore button)
2. ✓ (row selector)
3. Supplier Part Number
4. Description
5. Cost Price OR Discount %
6. List Price GBP OR Markup %
7. Barcode
8. Manufacturer (if mapped)
9. Country of Origin (if mapped)

**Features:**
- Only shows mapped columns (ignored columns hidden)
- Row exclusion with ❌ (click to exclude, click again to restore)
- Excluded rows shown with opacity and strikethrough
- Auto-adds Discount % or Markup % column based on mapping
- Quick Pricing Tools for bulk operations
- Row selection (Ctrl+click for multiple)
- Keyboard shortcuts (Ctrl+C, Ctrl+V, Ctrl+Shift+V)

### Step 5: Review & Process
Preview of mapped data before processing:
- Shows first 10 rows
- Displays calculated prices
- Sanity check before final processing

### Step 6: Original Price List
Review variant matching and descriptions:

**Columns:**
- # (line number with exclude/restore button)
- Manufacturer
- Variant Code (editable input field)
- Description (from file)
- 🔢 (Name would like to use character count)
- Name would like to use: `MANUFACTURER ( CODE ) DESCRIPTION` (clickable)
- Previous Description (from database or "New product? Check if new") (clickable)
- 🔢 (Description to use character count)
- Description to use (editable input field)
- Cost Price (calculated)
- List Price (calculated)
- Barcode
- Previous Barcode (from database)
- Commodity Code
- Previous Commodity Code
- Is Commodity Code valid?
- Country Code
- Previous Country Code

**Interactive Features:**
- **Clickable Descriptions**: Click on "Name would like to use" or "Previous Description" to copy to "Description to use"
- **Character Count Indicators**:
  - Shows character count for Name, Previous Description, and Description to use
  - Green when Name count ≥10% longer than Description to use
  - Grey when counts are equal
  - Red when count exceeds 250 characters
- **Greying Logic**: Cells grey out (#999) when they match the "Description to use"
- **Resizable Columns**: Drag column edges to resize, widths persist across interactions
- **Editable Fields**:
  - Variant Code can be edited
  - Description to use can be typed directly with real-time character count updates

**Color Coding:**
- **Yellow background**: New products
- **Red background**: Duplicate variant codes (case-insensitive)
- **Pink background**: "Name would like to use" column
- **Orange background**: "Previous Description" column
- **Green background**: "Description to use" column
- **Grey text (#999)**: Matched descriptions (when equals "Description to use")

**Logic:**
- Matches items against previous pricing data by code (O(1) hash map lookup)
- Existing items: defaults to previous description
- New items: defaults to formatted name with manufacturer
- Each row has independent state (uses rowIndex as key, not product code)
- Updates are instant with targeted DOM manipulation (no full re-renders)

### Step 7: Review Changes & Flagged Items
Final results with comparison:

**Statistics:**
- Total Items
- New Items
- Changed Items
- Flagged Items (shown in red)

**Flagging Criteria:**
Items are flagged if:
- (Cost change ≥15% OR ≤-10%) AND cost ≥£100
- OR (List change ≥15% OR ≤-10%) AND list price ≥£100

**Features:**
- Filter to show only flagged items
- Export as JSON or CSV
- Save to Server option (placeholder for backend integration)

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

- **Ctrl+C**: Copy cell value
- **Ctrl+V**: Paste to current cell
- **Ctrl+Shift+V**: Paste to all selected rows (Discount/Markup columns only)
- **Arrow Keys**: Navigate between cells
- **Enter**: Move down to next row
- **Ctrl+Click**: Select multiple rows

## Color Coding

### Step 4 (Edit Data)
- **Blue background**: Selected rows
- **Yellow background**: Discount/Markup columns
- **Gray with strikethrough**: Excluded rows
- **Light blue**: Column mapping dropdowns

### Step 6 (Original Price List)
- **Yellow background (#ffe082)**: New products (all fields)
- **Red background (#ffcccc)**: Duplicate variant codes
- **Pink background (#ffe0f0)**: "Name would like to use" column
- **Orange background (#ffe5cc)**: "Previous Description" column
- **Green background (#ccffcc)**: "Description to use" column
- **Grey text (#999)**: Matched descriptions (when cell value equals "Description to use")
- **Green text (limegreen)**: Character count when Name ≥10% longer than Description to use
- **Red text**: Character count when exceeds 250 characters

### Step 7 (Review Changes)
- **Light blue highlight**: Flagged items
- **Yellow highlight**: New products

## Data Processing

### Row Handling
- Excluded rows (marked with ❌) are skipped during processing
- Empty rows are filtered out
- Line numbers are sequential (excluding excluded rows)

### Price Calculation Logic
1. **If only List Price mapped**: Add Discount % column, calculate cost
2. **If only Cost Price mapped**: Add Markup % column, calculate list price
3. **If both mapped**: Direct editing, no percentage columns added

### Comparison Logic
- Matches by product code
- Compares cost price and list price changes
- Flags significant changes based on criteria
- Tracks new vs. existing products

## Performance Optimizations

### Hash Map Lookups (O(1) vs O(n))
Previous pricing data is loaded into a `Map` for instant lookups:
```javascript
// Build once when data loads
previousDataMap = new Map();
previousData.forEach(item => {
    previousDataMap.set(item.code.toLowerCase(), item);
});

// Use everywhere - O(1) instead of O(n)
const oldItem = previousDataMap.get(code.toLowerCase());
```

**Impact**: 100-1000x faster for large datasets (1000+ rows)

### Targeted DOM Updates
When clicking description cells or typing, only specific elements update:
```javascript
// Updates only 2-3 elements instead of rebuilding entire table
const input = document.getElementById(`descInput_${rowIndex}`);
const charCountCell = document.getElementById(`charCount_${rowIndex}`);
input.value = value;
charCountCell.textContent = count;
```

**Impact**: Eliminates browser crashes from rapid clicking, instant updates

### Row-Based State Keys
Each row uses unique `rowIndex` instead of potentially duplicate `code`:
```javascript
// Prevents collision when duplicate codes exist
descriptionToUseMap[rowIndex] = value; // not descriptionToUseMap[code]
```

**Impact**: Prevents data corruption with duplicate variant codes

### Column Width Persistence
Column widths are saved and restored with all three CSS properties:
```javascript
header.style.width = width + 'px';
header.style.minWidth = width + 'px';
header.style.maxWidth = width + 'px'; // Prevents auto-expansion
```

**Impact**: Custom column widths persist across all interactions

### Result
- No re-renders when clicking cells
- Handles 1000+ rows without performance issues
- Instant character count updates
- Stable with duplicate variant codes
- Flawless rapid clicking through all rows

## Security & Data Integrity

### No-Escaping Architecture
All data is treated as pure data, never as code:

**Approach:**
- Pass only identifiers (rowIndex, cellId) through onclick attributes
- Retrieve actual data from DOM using `textContent`
- Never pass data values through HTML attributes or JavaScript strings

**Example:**
```html
<!-- BEFORE - Required escaping, broke with backslashes -->
<td onclick="setDesc('${description.replace(/\\/g, '\\\\')}')">

<!-- AFTER - Data as data, works with ANY characters -->
<td id="cell_${rowIndex}" onclick="setDescFromCell(${rowIndex}, 'cell_${rowIndex}')">
```

```javascript
function setDescFromCell(rowIndex, cellId) {
    const cell = document.getElementById(cellId);
    const value = cell.textContent; // Pure data retrieval
    // Works with: backslashes, quotes, newlines, unicode, etc.
}
```

**Benefits:**
- Works with ANY characters (backslashes, quotes, newlines, tabs, unicode, etc.)
- No escaping logic needed
- No character corruption
- Simpler, more robust code
- True data integrity

## Test Data

Located in `/test-data/`:
- `current-price-list.csv` - 15 test products
- `previous-price-list.json` - Historical data for comparison
- `README.md` - Detailed test scenarios documentation

**Test Coverage:**
- 5 flagged items (various scenarios)
- 2 new products
- Unchanged items
- Small changes
- Edge cases (just under/at thresholds)
- Barcode changes
- Description changes

## Future PHP Integration

### Required Endpoints
```php
GET  /api/pricing/latest    - Fetch last saved pricing
GET  /api/pricing/history   - List previous imports
POST /api/pricing/save      - Save new pricing data
```

### Database Schema
```sql
pricing_imports (
  id, manufacturer_name, supplier_account,
  effective_date, import_date, filename, status
)

pricing_items (
  id, import_id, code, description,
  cost_price, list_price, barcode,
  manufacturer, country_of_origin
)

pricing_changes (
  id, import_id, item_id, change_type,
  old_value, new_value, flagged
)
```

## Browser Compatibility

- Modern browsers with ES6 support
- JavaScript must be enabled
- No server-side requirements
- Works completely offline (except SheetJS CDN load)

## Documentation Files

### claude.md (this file)
Current implementation documentation. Describes the workflow, features, and technical details.

### lessonslearned.md
**IMPORTANT:** This file must be maintained and updated whenever issues are encountered and resolved.

Records all errors, bugs, and user-reported issues along with:
- Problem description
- User request (if applicable)
- Solution implemented
- Code locations
- Key learnings

**Update this file when:**
- A bug is fixed
- A feature is changed due to user feedback
- An assumption proves incorrect
- A design decision is reversed
- A workaround is implemented

This file serves as institutional knowledge for future development and helps avoid repeating past mistakes.

## Deployment

### GitHub Pages
The tool can be deployed as a static site:

**Files:**
- `index.html` - Main application
- `pricingstyles.css` - Stylesheet
- `test-data/` - Test files (optional)

**To Deploy:**
1. Push to `main` branch
2. Enable GitHub Pages in repository settings
3. Access at: `https://<username>.github.io/<repo-name>/`

**Requirements:**
- Modern web browser
- Internet connection (for SheetJS CDN)
- No backend needed for basic functionality
