# Lessons Learned - Pricing Tool Development

This document captures issues encountered during development and their solutions.

## 1. File Format Support

### Issue
Initial implementation only supported Excel files (.xlsx, .xls), but users needed to work with CSV files as well.

### Solution
- Added CSV parsing logic to `handleFile()` function
- Implemented custom CSV parser to handle quoted fields properly
- Updated file input to accept `.csv` files
- Added conditional logic to detect file type and use appropriate parser

**Key Learning:** Always plan for multiple file formats early. CSV parsing requires handling edge cases like quoted commas and newlines.

**Code Location:** `index.html:612-665`

---

## 2. Auto-Column Assignment Issues

### Issue
System was auto-detecting and assigning columns based on header names (e.g., "code" → Supplier Part Number). This caused unwanted mappings when column names were similar but meant for different purposes.

### Problem Statement
> "it is selecting columns i didnt want to assign"

### Solution
- Removed all auto-detection logic from `renderMappingTable()`
- Changed default for all columns to "-- Ignore Column --"
- Users now have full control over column mapping

**Key Learning:** Auto-detection seems helpful but can be frustrating when it guesses wrong. Give users manual control, especially in data mapping scenarios.

**Code Location:** `index.html:692-703`

---

## 3. Delete vs. Exclude Functionality

### Issue
Original implementation permanently deleted rows with a "Delete" button. Users needed ability to temporarily exclude rows but restore them if needed.

### Solution
- Replaced permanent delete with exclude/restore toggle
- Changed button text from "Delete" to ❌ emoji
- Added `excludedRows` Set to track excluded row indices
- Styled excluded rows with reduced opacity and strikethrough
- Excluded rows skip processing but remain in data

**Key Learning:** Permanent operations should be avoided in data editing tools. Always provide undo/restore capabilities.

**Code Location:**
- Toggle function: `index.html:1160-1167`
- Visual styling: `pricingstyles.css:416-426`
- Processing logic: `index.html:1258-1263`

---

## 4. Column Display Order

### Issue
Columns appeared in the order they were uploaded, making the interface inconsistent across different files.

### User Request
> "please change column order for step 3"

### Solution
- Created `getMappedColumnIndices()` function with fixed column order
- Defined standard order: Actions, ✓, Supplier Part Number, Description, Cost Price/Discount %, List Price/Markup %, Barcode, etc.
- Function maps field names to display names for consistent headers

**Key Learning:** Consistency in UI is more important than preserving source file order. Define a logical column order and stick to it.

**Code Location:** `index.html:788-802`, `index.html:824-835`

---

## 5. Name Format Spacing

### Issue
"Name would like to use" field was formatted as `MANUFACTURER (CODE) DESCRIPTION` but needed spaces around parentheses for readability.

### User Request
> "MANUFACTURER ( CODE ) DESCRIPTION ideally needs spaces around the code"

### Solution
Changed from:
```javascript
const nameWouldLike = `${settings.manufacturerName} (${code}) ${description}`;
```

To:
```javascript
const nameWouldLike = `${settings.manufacturerName} ( ${code} ) ${description}`;
```

**Key Learning:** Small formatting details matter for readability. Add spaces around special characters for visual clarity.

**Code Location:** `index.html:1295`

---

## 6. Upload Workflow - Auto-Progress Issue

### Issue
After uploading main file, system immediately progressed to next step. This prevented users from uploading the optional previous pricing data file.

### User Request
> "when selecting the price list it needs to have a next button instead of automatically going into step 2, so the user has a chance to select the previous data as well"

### Solution
- Added disabled "Next: Map Columns →" button to Step 2
- Button enables after file upload
- Created `proceedFromUpload()` function for manual progression
- Shows alert confirming upload and reminding about optional previous data

**Key Learning:** Don't auto-progress between steps when users might need to perform multiple actions. Always give explicit control.

**Code Location:** `index.html:61`, `index.html:654-656`, `index.html:667-672`

---

## 7. Test Data Format Inconsistency

### Issue
Test files used different formats (CSV for current, JSON for previous). This made testing inconsistent and didn't reflect real-world usage.

### User Request
> "current and previous price list test data sheets should be both CSV files, and should have both cost and list pricing in it so that these can be tested too"

### Solution
- Converted `previous-price-list.json` to CSV format
- Added both Cost Price and List Price columns to all test files
- Removed JSON file, standardized on CSV
- Updated README with multiple test scenarios

**Key Learning:** Test data should mirror production data formats. Having both cost and list prices enables testing all code paths.

**Files Changed:**
- `test-data/current-price-list.csv`
- `test-data/previous-price-list.csv` (new)
- Deleted: `test-data/previous-price-list.json`

---

## 8. Missing Mapping Options

### Issue
Users couldn't map discount or markup columns from their uploaded files. System only auto-added these columns in Step 4 based on what was missing.

### User Request
> "in step 3, discount or markup should be an option when mapping columns"

### Solution
- Added `🏷️ Discount %` to field mapping options
- Added `📈 Markup %` to field mapping options
- Allows direct import of discount/markup values from source files
- System still auto-adds if needed but prioritizes mapped columns

**Key Learning:** If a field can exist in source data, it should be mappable. Don't force auto-generation when data might already exist.

**Code Location:** `index.html:687-688`

---

## 9. CSS Organization

### Issue
All CSS was embedded in HTML file, making it harder to maintain and cluttering the file.

### User Request
> "we want to separate the CSS from the HTML, create a pricingstyles.css file for it all"

### Solution
- Extracted all `<style>` content to `pricingstyles.css`
- Added `<link rel="stylesheet" href="pricingstyles.css">` to HTML
- Improved separation of concerns
- Easier to maintain and update styles

**Key Learning:** Even for single-page applications, separate CSS improves maintainability. External stylesheets are easier to edit and version control.

**Files Created:** `pricingstyles.css`

**Code Location:** `index.html:7`

---

## 10. Multiple HTML Files Confusion

### Issue
Both `index.html` and `pricing-tool.html` existed with similar content. This caused confusion about which file to update.

### User Request
> "delete it yes, then only need to update index"

### Solution
- Deleted `pricing-tool.html`
- Standardized on `index.html` as single source
- Updated documentation to reference only `index.html`

**Key Learning:** Avoid duplicate files. Choose one canonical version and delete alternatives to prevent sync issues.

---

## 11. Step Numbering Changes

### Issue
Adding Step 1 (Settings) before the upload step required renumbering all subsequent steps and updating all references.

### Challenge
- Step references in HTML (div IDs)
- Step references in JavaScript (goToStep calls)
- Navigation button onclick handlers
- Render function conditionals

### Solution
- Systematically updated all step numbers
- Used find/replace carefully to update:
  - Step div IDs (step1-7)
  - goToStep() calls
  - renderSpreadsheet/renderPreviewOnly/renderOriginalPriceList calls
  - All back button references

**Key Learning:** When adding steps in the middle of a workflow, plan for comprehensive refactoring. Test each navigation path after changes.

**Code Locations:** Multiple throughout `index.html`

---

## 12. Settings Data Persistence

### Issue
Step 1 Settings needed to be available throughout the workflow (especially for "Name would like to use" field in Step 6).

### Solution
- Created `settings` object to store configuration
- Validated settings before allowing progression
- Settings object includes:
  - manufacturerName
  - supplierAccountNumber
  - effectiveDate
- Used settings data in `renderOriginalPriceList()`

**Key Learning:** When adding configuration steps, create a persistent state object that's accessible throughout the application lifecycle.

**Code Location:** `index.html:578-582`, `index.html:674-698`

---

## Best Practices Discovered

### 1. User Control Over Automation
Users prefer manual control over automatic behavior, even if it requires more clicks. Examples:
- Manual column mapping vs. auto-detection
- Exclude/restore vs. permanent delete
- Next button vs. auto-progression

### 2. Consistent Data Formats
Use consistent file formats across test data. Don't mix JSON and CSV unnecessarily.

### 3. Clear Visual Feedback
- Use emojis and icons for quick identification (📦, 💰, ❌)
- Use color coding (yellow for discount/markup, gray for excluded)
- Show state clearly (excluded rows with strikethrough)

### 4. Progressive Enhancement
Start with basic functionality, then add convenience features:
1. Basic mapping (manual only)
2. Optional auto-detection (if requested)
3. Smart defaults (if proven useful)

### 5. Documentation
Keep comprehensive test data documentation with expected results. The `test-data/README.md` proved invaluable for validating all edge cases.

### 6. State Management
For multi-step workflows, maintain clear state:
- `currentData` - uploaded data
- `previousData` - comparison data
- `processedData` - final output
- `columnMapping` - field assignments
- `excludedRows` - user exclusions
- `settings` - configuration

---

## Common Patterns

### Pattern: Conditional Column Addition
When a field might be mapped OR auto-generated:
```javascript
const hasDiscount = columnMapping.discount !== undefined;
if (hasDiscount) {
    // Use mapped column
} else if (needsDiscount) {
    // Auto-add column
}
```

### Pattern: Field Name to Display Name Mapping
For consistent UI labels:
```javascript
const fieldDisplayNames = {
    'code': 'Supplier Part Number',
    'listPrice': 'List Price GBP',
    // etc.
};
```

### Pattern: Row State Management
Using Sets for efficient state tracking:
```javascript
let selectedRows = new Set();
let excludedRows = new Set();

// Toggle
if (excludedRows.has(rowIndex)) {
    excludedRows.delete(rowIndex);
} else {
    excludedRows.add(rowIndex);
}
```

---

## Future Considerations

### 1. Undo/Redo Stack
Consider implementing full undo/redo for complex editing operations.

### 2. Bulk Edit History
Track changes made via bulk operations for auditing.

### 3. Column Presets
Allow saving/loading column mapping presets for frequently-used file formats.

### 4. Validation Rules
Add configurable validation (e.g., price ranges, required fields, format checks).

### 5. Export Templates
Provide template downloads showing expected format for upload files.

---

## Conclusion

Most issues stemmed from assumptions about user preferences:
- We assumed auto-detection was helpful → Users wanted control
- We assumed permanent delete was okay → Users wanted restore capability
- We assumed auto-progression was efficient → Users wanted review time

**Key Takeaway:** When in doubt, give users control and let them choose the level of automation they prefer.
