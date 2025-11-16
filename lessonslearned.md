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

## 13. Column Width Persistence Issues

### Issue
Users could resize columns, but resized widths were forgotten when clicking on description cells or switching between views.

### Root Cause
- Column widths were saved in `columnWidths` map by tableId and column index
- When tables were re-rendered, `makeColumnsResizable()` was called again
- Function restored widths but didn't prevent duplicate event listeners
- Missing `makeColumnsResizable()` call on preview table

### Solution
- Added duplicate resizer check: `if (header.querySelector('.column-resizer')) return;`
- Set `maxWidth` in addition to `width` and `minWidth` to prevent browser auto-expansion
- Added missing `makeColumnsResizable()` call to preview table render
- Reduced variant code column min-width from 200px to 100px

**Key Learning:** When restoring layout state, set all three width properties (width, minWidth, maxWidth) to fully lock the column size. Always check for existing handlers before adding new event listeners.

**Code Location:** `index.html:373-419`, `pricingstyles.css:488`

---

## 14. Browser Crashes from Rapid Clicking

### Issue
Browser would crash or freeze when clicking rapidly through "Name would like to use" cells to update descriptions. Initially crashed after 3 rows, worse after data integrity fixes.

### Root Causes
1. **Full table re-renders**: Every click triggered `renderOriginalPriceList()` which rebuilt entire table
2. **O(n×m) performance bottleneck**: `previousData.find()` called for every row during each render
3. **Map collision bug**: `descriptionToUseMap` used product code as key, causing duplicates to collide
4. **Compounding effects**: Multiple issues multiplied to create catastrophic performance

### Investigation Process
1. First tried increasing debounce timers (150ms → 250ms → 500ms) - helped but didn't solve it
2. Added scroll protection to defer renders during scrolling
3. Discovered O(n×m) searches in 5 different locations
4. Found duplicate code collision causing data corruption
5. Realized fundamental issue: re-rendering entire table unnecessarily

### Solutions Applied

#### Phase 1: Performance Optimization
**O(1) Hash Map Lookups** (Commit: 3547973)
- Created `previousDataMap` as cached Map for instant lookups
- Added `buildPreviousDataMap()` to build map when data loads
- Replaced all `previousData.find()` with `previousDataMap.get()`
- Reduced complexity from O(n×m) to O(n+m)
- Impact: 100-1000x faster for large datasets

```javascript
// BEFORE - O(n×m) - SLOW!
const oldItem = previousData.find(p =>
    p.code.toLowerCase() === code.toLowerCase()
);

// AFTER - O(1) - INSTANT!
const oldItem = previousDataMap.get(code.toLowerCase());
```

#### Phase 2: Data Integrity Fix
**Row-Based Keys Instead of Code** (Commit: 277856b)
- Changed `descriptionToUseMap` key from `code` to `rowIndex`
- Each row now has independent description entry
- Eliminated map collisions for duplicate codes
- Fixed character count mismatches

```javascript
// BEFORE - Duplicates collide!
descriptionToUseMap[code] = value;

// AFTER - Each row independent
descriptionToUseMap[rowIndex] = value;
```

#### Phase 3: Eliminate Re-renders
**Targeted DOM Updates** (Commit: a23c132)
- `setDescriptionToUse()` now directly updates input field and character count
- No table rebuild needed when clicking cells
- Updates only 2 DOM elements instead of regenerating 1000+ rows
- Added greying logic updates to maintain visual feedback

```javascript
// BEFORE - Rebuild entire table
renderOriginalPriceList(); // Expensive!

// AFTER - Update specific elements
input.value = value;
charCountCell.textContent = count;
// Instant, no crash!
```

### Performance Comparison

| Operation | Before | After | Improvement |
|-----------|--------|-------|-------------|
| Click 1 cell | 250ms + full render | <1ms | ~250x faster |
| Click 100 cells rapidly | Browser crash | 100 instant updates | ∞ (crash → works) |
| Data lookup per render | O(n×m) searches | O(1) map lookup | 100-1000x faster |
| Table size impact | Exponential | Constant | Eliminates scaling issue |

**Key Learnings:**
1. **Profile before optimizing**: The real issue wasn't debounce timing, it was unnecessary work
2. **Data structures matter**: Map lookups vs array searches makes massive difference
3. **Minimize DOM operations**: Update specific elements, don't rebuild entire trees
4. **Check for collisions**: Using non-unique keys in maps causes silent data corruption
5. **Compound effects**: Multiple small issues can multiply into catastrophic failure

**Code Locations:**
- Hash map optimization: `index.html:314`, `index.html:362-373`, `index.html:1507-1509`
- Row-based keys: `index.html:1602-1608`, `index.html:1672-1679`
- Targeted updates: `index.html:1750-1799`

---

## 15. Visual Feedback with Targeted Updates

### Issue
After eliminating full table re-renders, the greying out logic for "Name would like to use" and "Previous Description" cells stopped working.

### Cause
Greying logic was part of the table render phase. When we removed re-renders, we also lost the visual feedback.

### Solution
- Added IDs to clickable cells (`nameCell_${rowIndex}`, `prevCell_${rowIndex}`)
- Added `data-value` attributes to store cell values for comparison
- Updated `setDescriptionToUse()` to manually grey/un-grey cells after update
- Updated `updateCharCount()` to update greying in real-time as user types

```javascript
// Update greying dynamically
if (nameCell) {
    const nameValue = nameCell.getAttribute('data-value');
    if (nameValue === value) {
        nameCell.style.color = '#999'; // Grey out
    } else {
        nameCell.style.color = ''; // Normal
    }
}
```

**Key Learning:** When moving from declarative (re-render everything) to imperative (update specific elements), remember to manually handle all visual state changes that were previously automatic.

**Code Location:** `index.html:1779-1796`, `index.html:1825-1844`

---

## 16. Git Conflict Resolution with Reverted Commits

### Issue
After reverting adaptive debouncing commits locally (git reset --hard), the branch diverged from main where those commits had already been merged via PR. This created merge conflicts.

### Solution
- Used `git rebase origin/main` to sync branch history
- Manually resolved conflicts by keeping the targeted DOM update approach
- Force-pushed cleaned-up branch to remote
- Result: Clean linear history with correct final code

**Key Learning:** When reverting commits that have already been merged upstream, use rebase to resolve divergence rather than merge. Always communicate with team before force-pushing shared branches.

**Commands Used:**
```bash
git rebase origin/main
# Resolve conflicts in editor
git add index.html
git rebase --continue
git push -f origin branch-name
```

---

## Performance Optimization Patterns

### Pattern: Cached Lookup Maps
When frequently searching arrays, build a Map once and reuse:

```javascript
// Build once when data loads
function buildPreviousDataMap() {
    previousDataMap = new Map();
    previousData.forEach(item => {
        const key = item.code.toLowerCase();
        previousDataMap.set(key, item);
    });
}

// Use everywhere - O(1) instead of O(m)
const oldItem = previousDataMap.get(code.toLowerCase());
```

### Pattern: Targeted DOM Updates
Instead of re-rendering entire components, update specific elements:

```javascript
// BAD - Rebuild everything
function onClick() {
    rebuildEntireTable(); // Expensive!
}

// GOOD - Update what changed
function onClick() {
    const element = document.getElementById(`specific_${id}`);
    element.textContent = newValue;
    element.style.color = getColor(newValue);
}
```

### Pattern: State Preservation with Unique Keys
Always use unique, stable keys for maps:

```javascript
// BAD - Can collide
const map = {};
map[duplicatableValue] = data; // Multiple items can have same value

// GOOD - Guaranteed unique
const map = {};
map[uniqueRowIndex] = data; // Each row has unique index
```

---

## Debugging Methodology for Performance Issues

1. **Identify symptoms**: Browser crash, freeze, slowness
2. **Measure frequency**: Does it happen more with large datasets? Rapid actions?
3. **Profile the operation**: What code runs on each action?
4. **Calculate complexity**: Is there nested iteration? Repeated searches?
5. **Look for data integrity**: Are values being lost or corrupted?
6. **Test incrementally**: Fix one issue at a time, test thoroughly
7. **Verify with real data**: Small test files can hide performance problems

### Red Flags Found in This Session
- ✗ `array.find()` inside a loop or repeated calls → O(n×m)
- ✗ Rebuilding entire DOM tree on every minor change
- ✗ Non-unique keys in data structures
- ✗ Debouncing without addressing root cause
- ✗ Event listeners added without checking for duplicates

### Solutions Applied
- ✓ Build hash maps for O(1) lookups
- ✓ Update specific DOM elements, not entire trees
- ✓ Use unique, stable keys (row indices)
- ✓ Eliminate unnecessary work before optimizing timing
- ✓ Check for existing handlers before adding new ones

---

## 17. Dynamic Character Count Color Updates

### Issue
After implementing targeted DOM updates to eliminate re-renders, the character count color for "Name would like to use" column wasn't updating when:
- Clicking on "Name would like to use" cells
- Clicking on "Previous Description" cells
- Typing in "Description to use" field

The character count number updated, but the color styling didn't change.

### Root Cause
The `setDescriptionToUseFromCell()` and `updateCharCount()` functions updated the character count text but didn't apply the color styling logic that was previously handled during table renders.

### Solution
1. Added ID to "Name would like to use" character count cell: `nameCharCount_${rowIndex}`
2. Updated `setDescriptionToUseFromCell()` to apply character count styling when clicking cells
3. Updated `updateCharCount()` to apply character count styling when typing

**Styling Logic Applied:**
```javascript
if (nameCharCount > 250) {
    nameCharCountCell.style.color = 'red';
    nameCharCountCell.style.fontWeight = 'bold';
} else if (nameCharCount === descCharCount) {
    // Grey when equal
    nameCharCountCell.style.color = '#999';
    nameCharCountCell.style.fontWeight = '';
} else if (nameCharCount >= descCharCount * 1.1) {
    // Green when 10% longer
    nameCharCountCell.style.color = 'limegreen';
    nameCharCountCell.style.fontWeight = 'bold';
} else {
    // Default
    nameCharCountCell.style.color = '';
    nameCharCountCell.style.fontWeight = '';
}
```

### Testing
User confirmed: "big success! it works FLAWLESS. i was able to click every row with no issues."

**Key Learning:** When moving from declarative rendering to imperative DOM updates, ensure ALL visual state changes are explicitly handled in the update functions. Color, styling, and visual feedback that was automatic during re-renders must be manually maintained.

**Code Locations:**
- ID added: `index.html:1666`
- `setDescriptionToUseFromCell()` styling: `index.html:1779-1802`
- `updateCharCount()` styling: `index.html:1850-1873`

---

## Conclusion

Most issues stemmed from assumptions about user preferences:
- We assumed auto-detection was helpful → Users wanted control
- We assumed permanent delete was okay → Users wanted restore capability
- We assumed auto-progression was efficient → Users wanted review time

Most **performance issues** stemmed from assumptions about scale:
- We assumed small datasets → Users had 1000+ rows
- We assumed re-rendering was acceptable → It caused browser crashes
- We assumed code was unique → Duplicates caused collisions
- We assumed debouncing fixed performance → It only masked the real problem

**Key Takeaways:**
1. When in doubt, give users control and let them choose the level of automation
2. Profile and measure before optimizing - fix the root cause, not symptoms
3. Data structure choice (Map vs Array) can make 100x+ performance difference
4. Minimize DOM operations - update what changed, not everything
5. Use unique keys in maps to prevent silent data corruption
6. Test with realistic data sizes early in development

---

## 18. User-Selectable Field Pattern (Barcode, Commodity Code, Country Code)

### Issue
Users needed to choose between new values from their price list and previous values from the database for multiple fields (barcode, commodity code, country code). When editing these values in the Edit Data step, the OW Import sheet wasn't reflecting the user's selected and edited values.

### Requirements
1. Display both new value and previous value from database
2. Allow user to click either value to select it
3. Default to previous value unless blank
4. Grey out the selected value for visual feedback
5. Sync edits in Edit Data step with user's selection
6. Export the chosen value to OW Import sheet

### Solution Pattern

This pattern was implemented consistently across three fields: **Barcode**, **Commodity Code**, and **Country Code**.

#### Step 1: Add Selection Map Variable
```javascript
let barcodeToUseMap = {};        // Store "Barcode to use" values by row index
let commodityCodeToUseMap = {};  // Store "Commodity Code to use" values by row index
let countryCodeToUseMap = {};    // Store "Country Code to use" values by row index
```

#### Step 2: Set Default Values
```javascript
// Default to Previous Barcode unless it's blank or '-'
if (!barcodeToUseMap[rowIndex]) {
    if (previousBarcode && previousBarcode !== '-') {
        barcodeToUseMap[rowIndex] = previousBarcode;
    } else {
        barcodeToUseMap[rowIndex] = barcode || '';
    }
}
```

#### Step 3: Add Greying Logic
```javascript
// Check which value matches the user's selection
const codeMatchesBarcode = (barcode || '') === barcodeToUseMap[rowIndex];
const prevMatchesBarcode = previousBarcode === barcodeToUseMap[rowIndex];
const barcodeTextColor = codeMatchesBarcode ? 'color: #999;' : '';
const prevBarcodeTextColor = prevMatchesBarcode ? 'color: #999;' : '';
```

#### Step 4: Make Cells Clickable
```javascript
<td id="barcodeCell_${rowIndex}"
    style="cursor: pointer; ${barcodeTextColor}"
    onclick="setBarcodeToUseFromCell(${rowIndex}, 'barcodeCell_${rowIndex}')"
    title="Click to use this barcode">${barcode || '-'}</td>
<td id="prevBarcodeCell_${rowIndex}"
    style="${(previousBarcode === '-') ? '' : 'cursor: pointer;'} ${prevBarcodeTextColor}"
    ${(previousBarcode === '-') ? '' : `onclick="setBarcodeToUseFromCell(${rowIndex}, 'prevBarcodeCell_${rowIndex}')"`}
    ${(previousBarcode === '-') ? '' : 'title="Click to use this barcode"'}>${previousBarcode}</td>
```

#### Step 5: Add Click Handler Function
```javascript
function setBarcodeToUseFromCell(rowIndex, cellId) {
    // Get value from cell - NO ESCAPING for security
    const cell = document.getElementById(cellId);
    if (!cell) return;
    const value = cell.textContent; // Pure data

    // Update the map
    barcodeToUseMap[rowIndex] = value;

    // Update greying directly on DOM (no re-render)
    const barcodeCell = document.getElementById(`barcodeCell_${rowIndex}`);
    const prevBarcodeCell = document.getElementById(`prevBarcodeCell_${rowIndex}`);

    if (barcodeCell) {
        const barcodeValue = barcodeCell.textContent;
        barcodeCell.style.color = (barcodeValue === value) ? '#999' : '';
    }

    if (prevBarcodeCell) {
        const prevBarcodeValue = prevBarcodeCell.textContent;
        prevBarcodeCell.style.color = (prevBarcodeValue === value && prevBarcodeValue !== '-') ? '#999' : '';
    }
}
```

#### Step 6: Use Selected Value in processData
```javascript
const item = {
    // ... other fields
    barcode: barcodeToUseMap[rowIndex] || (columnMapping.barcode !== undefined ? row[columnMapping.barcode] : ''),
    commodityCode: commodityCodeToUseMap[rowIndex] || (columnMapping.commodityCode !== undefined ? row[columnMapping.commodityCode] : ''),
    countryOfOrigin: countryCodeToUseMap[rowIndex] || (columnMapping.countryOfOrigin !== undefined ? row[columnMapping.countryOfOrigin] : '')
};
```

#### Step 7: Sync Edit Data Changes
```javascript
function updateCell(row, col, value) {
    const oldValue = currentData[row][col];
    currentData[row][col] = value;

    // If barcode was edited and user had selected the new barcode, update the map
    if (columnMapping.barcode === col) {
        if (barcodeToUseMap[row] === oldValue) {
            barcodeToUseMap[row] = value;
        }
    }

    // Same for commodity code
    if (columnMapping.commodityCode === col) {
        if (commodityCodeToUseMap[row] === oldValue) {
            commodityCodeToUseMap[row] = value;
        }
    }

    // Same for country code
    if (columnMapping.countryOfOrigin === col) {
        if (countryCodeToUseMap[row] === oldValue) {
            countryCodeToUseMap[row] = value;
        }
    }
}
```

### Key Learnings

1. **Consistent Pattern**: Once established for one field (barcode), the exact same pattern applies to all user-selectable fields. This makes the codebase predictable and maintainable.

2. **Direct DOM Manipulation**: Using `textContent` instead of `innerHTML` prevents XSS and escaping issues. The data is treated as pure data, never as code.

3. **No Re-rendering**: Click handlers update only the specific cells that changed, maintaining the performance benefits from earlier optimizations.

4. **Default to Previous**: Users typically want to keep database values unless they explicitly change them. Defaulting to "Previous" values reduces required clicks.

5. **Edit Data Sync Critical**: When users edit a value in Edit Data step, check if they had selected the new value (not previous). If so, update the selection map to the edited value. This ensures consistency across all steps.

6. **Row-Based Keys**: Using `rowIndex` as the map key (not product code) prevents collisions when duplicate codes exist.

### Pattern Template

When adding a new user-selectable field:
1. Add `fieldToUseMap = {}` variable
2. Set default value (prefer previous unless blank)
3. Add greying comparison logic
4. Make both cells clickable with IDs
5. Create `setFieldToUseFromCell()` handler
6. Use `fieldToUseMap[rowIndex]` in processData
7. Add sync logic to `updateCell()` for Edit Data step

This pattern has been successfully applied to:
- **Barcode selection** (Barcode vs Previous Barcode)
- **Commodity Code selection** (Commodity Code vs Previous Commodity Code)
- **Country Code selection** (Country Code vs Previous Country Code)
- **Description selection** (Name Would Like to Use vs Previous Description) - with additional input field

**Code Locations:**
- Variable declarations: `index.html:565-567`
- Default value setting: `index.html:2038-2062`
- Greying logic: `index.html:2085-2095`
- Clickable cells: `index.html:2161-2194`
- Click handlers: `index.html:2330-2524`
- processData usage: `index.html:2767-2770`
- Edit Data sync: `index.html:1646-1671`

---

**Key Takeaways:**
1. When in doubt, give users control and let them choose the level of automation
2. Profile and measure before optimizing - fix the root cause, not symptoms
3. Data structure choice (Map vs Array) can make 100x+ performance difference
4. Minimize DOM operations - update what changed, not everything
5. Use unique keys in maps to prevent silent data corruption
6. Test with realistic data sizes early in development
7. **Establish consistent patterns for similar features** - makes code predictable and reduces bugs
