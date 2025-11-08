# Test Data for Pricing Tool

This directory contains test files to validate all features of the pricing tool.

## Files

- **current-price-list.csv** - Current pricing data to upload in Step 2
- **previous-price-list.json** - Previous pricing data to upload in Step 2 (optional comparison file)

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
   - Old: £130.00 → New: £150.00
   - Change: +15.38% increase
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

2. **WIDGET-002** - Standard Widget Beta
   - Old: £150.00 → New: £120.00
   - Change: -20% decrease
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

3. **GADGET-002** - Home Gadget Lite
   - Old: £200.00 → New: £180.00
   - Change: -10% decrease (exactly at threshold)
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

4. **PART-003** - Replacement Part C
   - Old: £240.00 → New: £200.00
   - Change: -16.67% decrease
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

5. **SPECIAL-001** - Special Edition Item
   - Old: £600.00 → New: £500.00
   - Change: -16.67% decrease
   - Cost: ≥£100 ✓
   - **FLAGGED** ⚠️

---

### NOT Flagged (Normal changes)

6. **WIDGET-003** - Economy Widget Gamma
   - Old: £90.00 → New: £95.00
   - Change: +5.56% increase
   - Cost: <£100 (doesn't meet threshold)
   - NOT flagged

7. **GADGET-001** - Professional Gadget Pro
   - Old: £245.00 → New: £250.00
   - Change: +2.04% increase (small change)
   - Cost: ≥£100
   - NOT flagged (change too small)

8. **TOOL-001** - Heavy Duty Tool Set
   - Old: £280.00 → New: £320.00
   - Change: +14.29% increase (just under 15%)
   - Cost: ≥£100
   - NOT flagged (just below threshold)

9. **TOOL-002** - Basic Tool Kit
   - Old: £90.00 → New: £75.00
   - Change: -16.67% decrease
   - Cost: <£100 (doesn't meet threshold)
   - NOT flagged

10. **PART-001** - Replacement Part A
    - Old: £110.00 → New: £110.00
    - Change: 0% (unchanged)
    - NOT flagged

11. **ACCESSORY-001** - Premium Accessory
    - Old: £80.00 → New: £85.00
    - Change: +6.25% increase
    - Cost: <£100
    - NOT flagged

12. **ACCESSORY-002** - Standard Accessory
    - Old: £50.00 → New: £45.00
    - Change: -10% decrease
    - Cost: <£100 (doesn't meet threshold)
    - NOT flagged

---

### New Products (Not in previous data)

13. **NEWPROD-001** - Brand New Product
    - Price: £299.00
    - Status: NEW
    - Previous Description: "New product? Check if new"
    - Description to use: "(NEWPROD-001) Brand New Product"

14. **NEWPROD-002** - Another New Item
    - Price: £150.00
    - Status: NEW
    - Previous Description: "New product? Check if new"
    - Description to use: "(NEWPROD-002) Another New Item"

---

### Special Test Cases

**PART-002** - Tests barcode change detection
- Price unchanged: £105.00 → £105.00
- But barcode changed: 5060123400000 → 5060123456864
- Previous Description: "Replacement Part B - Old Description"
- Should show previous description in Step 6

**SPECIAL-001** - Tests description change
- Previous Description: "Special Edition Item - Previous Version"
- Current Description: "Special Edition Item"
- Should use previous description in Step 6

---

## How to Test

1. **Step 1:** Enter any settings (e.g., Manufacturer: "Test Co", Account: "12345", Date: today)
2. **Step 2:** Upload `current-price-list.csv` as the main file
3. **Step 2:** Upload `previous-price-list.json` as the previous pricing data
4. **Step 3:** Map columns:
   - "Supplier Code" → Supplier Part Number
   - "Product Name" → Description
   - "List Price" → List Price GBP
   - "Barcode" → Barcode
5. **Step 4:** Add discount of 20% to all items
6. **Step 5:** Review the preview
7. **Step 6:** Check Original Price List - should see:
   - Previous descriptions for existing items
   - "New product? Check if new" for NEWPROD-001 and NEWPROD-002
8. **Step 7:** Review Changes & Flagged Items
   - Should show 5 flagged items (highlighted)
   - Filter checkbox should work to show only flagged items

## Expected Stats in Step 7

- Total Items: 15
- New Items: 2
- Changed Items: 13
- Flagged Items: 5 (shown in red)
