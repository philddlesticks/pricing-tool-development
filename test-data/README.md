# Test Data for Pricing Tool

This directory contains test files to validate all features of the pricing tool.

## Files

- **current-price-list.csv** - Current pricing data to upload in Step 2
- **previous-price-list.csv** - Previous pricing data to upload in Step 2 (optional comparison file)

Both files are in CSV format and include both Cost Price and List Price columns to enable full testing of all pricing scenarios.

## Test Scenarios Covered

### Expected Results Summary
- **Total Items:** 15
- **New Items:** 2
- **Changed Items:** 13
- **Flagged Items:** 5

---

### Flagged Items (Should appear with warning badge)

Items flagged meet criteria: ≥15% increase OR ≤-10% decrease with cost ≥£100

1. **WIDGET-001** - Premium Widget Alpha
   - Old Cost/List: £104.00/£130.00 → New: £120.00/£150.00
   - Cost Change: +15.38% increase
   - List Change: +15.38% increase
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

2. **WIDGET-002** - Standard Widget Beta
   - Old Cost/List: £120.00/£150.00 → New: £96.00/£120.00
   - Cost Change: -20% decrease
   - List Change: -20% decrease
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

3. **GADGET-002** - Home Gadget Lite
   - Old Cost/List: £160.00/£200.00 → New: £144.00/£180.00
   - Cost Change: -10% decrease (exactly at threshold)
   - List Change: -10% decrease
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

4. **PART-003** - Replacement Part C
   - Old Cost/List: £192.00/£240.00 → New: £160.00/£200.00
   - Cost Change: -16.67% decrease
   - List Change: -16.67% decrease
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

5. **SPECIAL-001** - Special Edition Item
   - Old Cost/List: £480.00/£600.00 → New: £400.00/£500.00
   - Cost Change: -16.67% decrease
   - List Change: -16.67% decrease
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

---

### NOT Flagged (Normal changes)

6. **WIDGET-003** - Economy Widget Gamma
   - Old Cost/List: £72.00/£90.00 → New: £76.00/£95.00
   - Change: +5.56% increase
   - Cost: <£100 (doesn't meet threshold)
   - NOT flagged

7. **GADGET-001** - Professional Gadget Pro
   - Old Cost/List: £196.00/£245.00 → New: £200.00/£250.00
   - Change: +2.04% increase (small change)
   - Cost: ≥£100
   - NOT flagged (change too small)

8. **TOOL-001** - Heavy Duty Tool Set
   - Old Cost/List: £224.00/£280.00 → New: £256.00/£320.00
   - Change: +14.29% increase (just under 15%)
   - Cost: ≥£100
   - NOT flagged (just below threshold)

9. **TOOL-002** - Basic Tool Kit
   - Old Cost/List: £72.00/£90.00 → New: £60.00/£75.00
   - Change: -16.67% decrease
   - Cost: <£100 (doesn't meet threshold)
   - NOT flagged

10. **PART-001** - Replacement Part A
    - Old Cost/List: £88.00/£110.00 → New: £88.00/£110.00
    - Change: 0% (unchanged)
    - NOT flagged

11. **ACCESSORY-001** - Premium Accessory
    - Old Cost/List: £64.00/£80.00 → New: £68.00/£85.00
    - Change: +6.25% increase
    - Cost: <£100
    - NOT flagged

12. **ACCESSORY-002** - Standard Accessory
    - Old Cost/List: £40.00/£50.00 → New: £36.00/£45.00
    - Change: -10% decrease
    - Cost: <£100 (doesn't meet threshold)
    - NOT flagged

---

### New Products (Not in previous data)

13. **NEWPROD-001** - Brand New Product
    - Cost/List: £239.20/£299.00
    - Status: NEW
    - Previous Description: "New product? Check if new"
    - Description to use: "MANUFACTURER ( NEWPROD-001 ) Brand New Product"

14. **NEWPROD-002** - Another New Item
    - Cost/List: £120.00/£150.00
    - Status: NEW
    - Previous Description: "New product? Check if new"
    - Description to use: "MANUFACTURER ( NEWPROD-002 ) Another New Item"

---

### Special Test Cases

**PART-002** - Tests barcode change detection
- Price unchanged: Cost £84.00/List £105.00
- But barcode changed: 5060123400000 → 5060123456864
- Previous Description: "Replacement Part B - Old Description"
- Should show previous description in Step 6

**SPECIAL-001** - Tests description change
- Previous Description: "Special Edition Item - Previous Version"
- Current Description: "Special Edition Item"
- Should use previous description in Step 6

---

## How to Test

### Basic Test (With Both Cost and List Price)

1. **Step 1:** Enter settings:
   - Manufacturer: "Test Co"
   - Account: "12345"
   - Date: today
2. **Step 2:** Upload `current-price-list.csv` as the main file
3. **Step 2:** Upload `previous-price-list.csv` as the previous pricing data
4. **Step 3:** Map columns:
   - "Supplier Code" → Supplier Part Number
   - "Product Name" → Description
   - "Cost Price" → Cost Price
   - "List Price" → List Price GBP
   - "Barcode" → Barcode
5. **Step 4:** Review data (no discount/markup columns needed since both prices are present)
6. **Step 5:** Review the preview
7. **Step 6:** Check Original Price List - should see:
   - Previous descriptions for existing items
   - "New product? Check if new" for NEWPROD-001 and NEWPROD-002
   - Format: "Test Co ( CODE ) Description"
8. **Step 7:** Review Changes & Flagged Items
   - Should show 5 flagged items (highlighted)
   - Filter checkbox should work to show only flagged items

### Alternative Test (List Price + Discount %)

To test discount functionality:
1. Follow steps 1-3 above
2. **Step 3:** Map columns differently:
   - "Supplier Code" → Supplier Part Number
   - "Product Name" → Description
   - "List Price" → List Price GBP
   - "Barcode" → Barcode
   - Leave "Cost Price" as "Ignore"
3. **Step 4:** A "Discount %" column will be auto-added
   - Enter 20% discount on all rows using Quick Tools
   - System will calculate cost price from discount

### Alternative Test (Cost Price + Markup %)

To test markup functionality:
1. Follow steps 1-3 above
2. **Step 3:** Map columns differently:
   - "Supplier Code" → Supplier Part Number
   - "Product Name" → Description
   - "Cost Price" → Cost Price
   - "Barcode" → Barcode
   - Leave "List Price" as "Ignore"
3. **Step 4:** A "Markup %" column will be auto-added
   - Enter 25% markup on all rows using Quick Tools
   - System will calculate list price from markup

### Test Mapping Discount/Markup from File

If your price list file includes discount or markup columns:
1. **Step 3:** Map the discount or markup column:
   - Select "🏷️ Discount %" or "📈 Markup %" from dropdown
2. Values will be imported from your file
3. Can still edit them in Step 4

## Expected Stats in Step 7

- Total Items: 15
- New Items: 2
- Changed Items: 13
- Flagged Items: 5 (shown in yellow highlight)

## File Format Notes

Both files use CSV format with headers:
- **Current**: Supplier Code, Product Name, Cost Price, List Price, Barcode
- **Previous**: code, description, costPrice, listPrice, barcode

The tool handles both naming conventions for field matching.
