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
- # (line number)
- Variant Code (supplier part number)
- Description (from file)
- Name would like to use: `MANUFACTURER ( CODE ) DESCRIPTION`
- Previous Description (from database or "New product? Check if new")
- Description to use (defaults to previous if found, otherwise uses formatted name)
- Cost Price (calculated)
- List Price (calculated)
- Barcode
- Previous Barcode (from database)

**Logic:**
- Matches items against previous pricing data by code
- Existing items: uses previous description
- New items: uses formatted name with manufacturer

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

- **Blue background**: Selected rows
- **Yellow background**: Discount/Markup columns
- **Gray with strikethrough**: Excluded rows
- **Yellow highlight**: Flagged items in Step 7
- **Light blue**: Column mapping dropdowns

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
