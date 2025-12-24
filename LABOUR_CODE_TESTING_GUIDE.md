# Labour Code Integration - Quick Testing Guide

## Pre-Testing Setup

### Step 1: Run Database Migration
```bash
cd /path/to/project
python manage.py migrate
```

### Step 2: Create Test Labour Codes
Go to Django Admin: `/admin/tracker/labourcode/`

**Labour Code 1 (Sales Item):**
- Code: `TIRE001`
- Description: `Michelin Tire 185/65 R15`
- Category: `sales`
- Item Name: `Michelin Tire 185/65 R15`
- Brand: `Michelin`
- Quantity: `2`
- Tire Type: `New`
- Is Active: ✓

**Labour Code 2 (Service):**
- Code: `SVC001`
- Description: `Oil Change Service`
- Category: `service`
- Item Name: `Oil Filter Pro`
- Brand: `Shell`
- Quantity: `1`
- Tire Type: (leave blank)
- Is Active: ✓

**Labour Code 3 (Labour):**
- Code: `LAB001`
- Description: `General Labour Work`
- Category: `labour`
- Item Name: `Labor Work - 8 Hours`
- Brand: (leave blank)
- Quantity: (leave blank)
- Tire Type: (leave blank)
- Is Active: ✓

---

## Test Cases

### Test 1: API Lookup by Item Name
**Purpose**: Verify labour code API lookup works

**Steps**:
1. Open browser and go to:
   ```
   http://localhost:8000/tracker/api/orders/lookup-labour-code/?item_name=Michelin
   ```
2. Verify response contains `TIRE001` labour code with all item details

**Expected Result**:
```json
{
  "success": true,
  "labour_codes": [
    {
      "id": <id>,
      "code": "TIRE001",
      "description": "Michelin Tire 185/65 R15",
      "item_name": "Michelin Tire 185/65 R15",
      "brand": "Michelin",
      "quantity": 2,
      "tire_type": "New",
      "category": "sales"
    }
  ]
}
```

### Test 2: API Lookup by Code
**Purpose**: Verify exact code lookup works

**Steps**:
1. Go to:
   ```
   http://localhost:8000/tracker/api/orders/lookup-labour-code/?code=TIRE001
   ```
2. Verify response contains the correct labour code

**Expected Result**: Single labour code with code "TIRE001"

### Test 3: Create Sales Order with Labour Code
**Purpose**: Test labour code integration in started order detail

**Steps**:
1. Go to Started Orders Dashboard: `/tracker/orders/started/`
2. Create a new order (or select existing)
3. Click "Edit Order Details"
4. Change Order Type to "Sales"
5. Click "Labour Code" tab
6. Type "Michelin" in search field
7. Select the Michelin Tire labour code from results
8. Verify:
   - Code displays: "TIRE001"
   - Name displays: "Michelin Tire 185/65 R15"
   - Brand displays: "Michelin"
9. Adjust quantity if desired
10. Click "Save Changes"
11. Verify order is updated correctly

**Expected Result**:
- Order item_name = "Michelin Tire 185/65 R15"
- Order brand = "Michelin"
- Order quantity = 2 (or as adjusted)
- Order tire_type = "New"

### Test 4: Manual Entry Fallback
**Purpose**: Test manual entry when labour code not available

**Steps**:
1. Go to Started Orders Dashboard
2. Create/select a sales order
3. Click "Edit Order Details"
4. Change Order Type to "Sales"
5. Click "Manual Entry" tab
6. Fill in:
   - Item Name: "Custom Tire XYZ"
   - Brand: "Bridgestone"
   - Quantity: "4"
7. Click "Save Changes"
8. Verify order is updated with manual data

**Expected Result**:
- Order item_name = "Custom Tire XYZ"
- Order brand = "Bridgestone"
- Order quantity = 4

### Test 5: Inventory Item Selection (Backward Compatibility)
**Purpose**: Verify inventory selection still works

**Steps**:
1. Go to Started Orders Dashboard
2. Select a sales order
3. Click "Edit Order Details"
4. Change Order Type to "Sales"
5. Click "Inventory" tab
6. Select an existing inventory item
7. Verify brand auto-fills
8. Click "Save Changes"

**Expected Result**: Order updated with inventory item data

### Test 6: Service Order with Labour Code
**Purpose**: Test labour code for service orders

**Steps**:
1. Go to Started Orders Dashboard
2. Select a service order (or change type to Service)
3. Click "Edit Order Details"
4. Order Type should be "Service"
5. Verify:
   - Labour Code tab is available
   - Inventory tab is hidden
   - Manual Entry tab is available
6. Try searching for labour codes
7. Save changes

**Expected Result**: Service order can use labour codes if needed

### Test 7: Priority Order Test
**Purpose**: Verify data priority (Labour Code > Manual Entry > Inventory)

**Scenario A**: Labour Code provided
- Fill labour code search and select an item
- Leave manual entry fields blank
- Expected: Labour code data is used

**Scenario B**: Manual Entry provided
- Don't select labour code
- Fill manual entry fields
- Expected: Manual data is used

**Scenario C**: Inventory item provided
- Don't use labour code or manual entry
- Select inventory item
- Expected: Inventory data is used

### Test 8: Form Validation
**Purpose**: Verify form validation works

**Steps**:
1. Go to order edit page: `/tracker/orders/<id>/edit/`
2. Try to create/save a sales order without item selection
3. Verify error message appears
4. Try using labour code dropdown instead
5. Verify it works

**Expected Result**: Proper validation messages and error handling

### Test 9: Empty Labour Code Fields
**Purpose**: Verify items work with empty optional fields

**Steps**:
1. Create a labour code with only required fields:
   - Code: "EMPTY001"
   - Description: "Minimal Labour Code"
   - Category: "labour"
   - Leave item fields empty
2. Try to use it in order update
3. Verify graceful handling

**Expected Result**: No errors; system handles empty item fields

### Test 10: Search Performance
**Purpose**: Verify search debouncing and performance

**Steps**:
1. Open Edit Order Details modal
2. Click Labour Code tab
3. Type "M" and hold for several seconds
4. Monitor browser console (F12 > Network tab)
5. Verify API calls are throttled/debounced

**Expected Result**: Fewer API calls than keypresses (debouncing working)

---

## Browser Console Debugging

### Enable Debug Logs
Open browser DevTools (F12) and check:

**Network Tab**:
- Look for requests to `/tracker/api/orders/lookup-labour-code/`
- Verify response times
- Check for errors

**Console Tab**:
- JavaScript errors should be minimal
- Look for 404 or 500 errors

**Elements Tab**:
- Verify DOM structure of tabs
- Check form fields are properly bound

---

## Common Issues & Troubleshooting

### Issue 1: Migration Failed
**Solution**:
```bash
python manage.py makemigrations
python manage.py migrate
```

### Issue 2: Labour Code Tab Not Showing
**Solution**:
- Verify order type is "Sales", "Service", or "Labour"
- Check browser console for JavaScript errors
- Verify template changes were saved

### Issue 3: Search Returns No Results
**Solution**:
- Verify labour codes are marked as `is_active = True`
- Check spelling of search term
- Try searching by code instead of name

### Issue 4: Item Data Not Saving
**Solution**:
- Check Django logs for errors
- Verify form submission completed (no validation errors)
- Check Network tab in browser for response errors

### Issue 5: Performance Issues
**Solution**:
- Clear browser cache
- Check database query count (Django Debug Toolbar)
- Verify indexes are created by migration

---

## Quick Verification Checklist

After implementation, verify:

- [ ] Database migration ran successfully
- [ ] No database errors in Django logs
- [ ] Labour code admin page shows new fields
- [ ] Labour code form has new fields with proper widgets
- [ ] API endpoint returns correct response
- [ ] Started order detail page loads without errors
- [ ] Edit modal opens and shows three tabs
- [ ] Labour code search works
- [ ] Manual entry fields appear and accept input
- [ ] Inventory tab still works
- [ ] Form validation works for sales orders
- [ ] Data is saved correctly to Order model
- [ ] Backward compatibility maintained (old orders still work)

---

## Performance Baseline

After completing tests, document:
- API response time for labour code lookup: ____ms
- Page load time with new template: ____ms
- Search debounce delay working: YES / NO
- No console errors: YES / NO

---

## Sign-Off

- **Tested By**: _______________
- **Date**: _______________
- **All Tests Passed**: YES / NO
- **Issues Found**: _______________
- **Notes**: _______________
