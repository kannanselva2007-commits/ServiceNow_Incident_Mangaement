# Implement Client Script & UI Policy (Incident)

## Project Overview

This project demonstrates how ServiceNow UI Policies and Client Scripts can be used to enforce data integrity and control user interactions on Incident records.

## Objective

The objective is to demonstrate how UI Policies and Client Scripts can:
- Make fields mandatory based on conditions
- Automatically populate field values
- Make fields read-only when required
- Validate Incident data before saving
- Prevent unauthorized changes through list editing
- Improve consistency and data quality

## Skills

- Incident Management
- UI Policy
- UI Policy Actions
- Client Scripts
- Form Validation

## UI Policy

### High Impact Control

**Table:** Incident  
**Active:** true

**Condition:**
- Field: Impact
- Operator: is
- Value: 1 - High

**Actions:**
- Assignment Group → Mandatory
- Urgency → Read-only

**Reverse if false:** true

When Impact is set to High, the UI Policy is triggered.

## Client Scripts

### 1. onChange Client Script

**Name:** Auto set urgency for high impact  
**Table:** Incident  
**Type:** onChange  
**Field:** Impact

```javascript
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || newValue == '') {
        return;
    }

    if (newValue == '1') {
        g_form.setValue('urgency', '1');
        g_form.addInfoMessage(
            'Urgency set to High for High impact incident.'
        );
    }
}
```

### 2. onSubmit Client Script

**Name:** Prevent save if Assigned To missing  
**Table:** Incident  
**Type:** onSubmit

This script prevents the Incident from being saved when Impact is High and Assigned To is empty.

```javascript
function onSubmit() {
    if (g_form.getValue('impact') == '1' &&
        g_form.getValue('assigned_to') == '') {

        g_form.showErrorBox(
            'assigned_to',
            'Assigned To is mandatory for High impact incidents.'
        );

        return false;
    }

    return true;
}
```

### 3. onCellEdit Client Script

**Name:** Prevent state change via list edit  
**Table:** Incident  
**Type:** onCellEdit  
**Field:** State

This script prevents users from changing the State field directly from the Incident list.

```javascript
function onCellEdit(sysIDs, table, oldValues, newValue, callback) {
    alert(
        'State cannot be updated using list editing. Please open the Incident.'
    );

    callback(false);
}
```

## Testing

### Mandatory Enforcement

1. Open **Incident → Create New**.
2. Set Impact to **High**.
3. Leave Assigned To empty.
4. Click **Submit**.
5. Verify that the Incident is not saved.
6. Verify that an error message appears.

### Successful Save

1. Fill the Assigned To field.
2. Click **Submit**.
3. Verify that the Incident is saved successfully.
4. Confirm that the configured UI Policy and Client Script behaviors work correctly.

### Reverse Condition Test

1. Open an Incident where Impact is High.
2. Change Impact from High to Medium.
3. Verify that Assigned To is no longer mandatory.
4. Verify that Urgency becomes editable.
5. Save the record.

### List Edit Blocking

1. Go to **Incident → All**.
2. Try to edit the State field directly from the list.
3. Verify that an alert appears.
4. Confirm that the State value remains unchanged.

### Form-Based Update

1. Open the Incident record.
2. Change the State field from the form.
3. Click **Update**.
4. Verify that the State change is saved successfully.

## Expected Behavior

| Condition / Action | Expected Result |
|---|---|
| Impact = High | UI Policy is triggered |
| Impact = High | Assignment Group becomes mandatory |
| Impact = High | Urgency becomes read-only |
| Impact changes to High | Urgency is automatically set to High |
| High Impact + Assigned To empty | Incident cannot be saved |
| State edited from list | Change is blocked |
| State edited from form | Change can be saved |
| Impact changes from High to Medium | UI Policy behavior is reversed |

## Conclusion

The Implement Client Script & UI Policy (Incident) project demonstrates how UI Policies and Client Scripts can work together to improve data quality, automate field behavior, validate Incident records, and control user interactions in ServiceNow.
