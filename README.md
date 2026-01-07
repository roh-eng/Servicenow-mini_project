
# Family Expenses Tracker

ServiceNow Mini Demo Project

## Overview

Family Expenses Tracker is a ServiceNow application designed to consolidate daily family expense transactions into date-wise summary records. The application demonstrates core ServiceNow platform capabilities, including custom table design, business rules, server-side scripting, table relationships, and auto-number configuration.

The system automatically aggregates individual daily expenses and maintains a single summarized record per date, providing clear and accurate visibility into family spending.


## Business Objective

To provide automated, date-wise expense tracking by rolling up individual daily transactions into summarized records without manual intervention.


## System Design

### Custom Tables

#### 1. u_family_expenses

Purpose: Stores auto-generated daily expense summaries.

Fields:

* Number (Auto-number)
* Date
* Amount (Integer)
* Expense Details (String, 800 characters)

Access:

* Read-only

Auto-Number Prefix:

* MFE

---

#### 2. u_daily_expenses

Purpose: Stores individual daily expense transactions.

Fields:

* Number (Auto-number)
* Family Member Name (Reference to sys_user)
* Date
* Expense (Integer)
* Comments (String)

Access:

* Editable

Auto-Number Prefix:

* DFE

---

## Core Functionality

### Automatic Daily Aggregation

A Business Rule executes on the u_daily_expenses table before record insert and update.

Behavior:

* Searches for an existing summary record in u_family_expenses for the same date
* If found:

  * Adds the expense amount to the existing total
  * Appends transaction details to the summary description
* If not found:

  * Creates a new summary record for that date

This ensures exactly one summary record exists per date.


### Table Relationships

A parent-child relationship is configured between Family Expenses and Daily Expenses.

Relationship Condition:

```
current.date = parent.date
```

Result:

* Each summary record displays only the daily expense records associated with the same date
* Prevents cross-date data visibility



### Auto-Number Configuration

Auto-numbering is configured using Number Maintenance.

Settings:

* Prefix: MFE / DFE
* Numeric length: 7 digits
* Starting value: 1000001

Example:

* MFE1000001
* DFE1000001



## Technical Implementation

### Business Rule: Daily Expenses to Family Expenses Sync

Table: u_daily_expenses
When: Before Insert, Before Update
Type: Server-side

```javascript
var familyExpenses = new GlideRecord('u_family_expenses');
familyExpenses.addQuery('date', current.date);
familyExpenses.query();

if (familyExpenses.next()) {
    familyExpenses.amount += current.expense;
    familyExpenses.expense_details += ', ' + current.comments + ' - ' + current.expense;
    familyExpenses.update();
} else {
    var newSummary = new GlideRecord('u_family_expenses');
    newSummary.date = current.date;
    newSummary.amount = current.expense;
    newSummary.expense_details = current.comments + ' - ' + current.expense;
    newSummary.insert();
}
```


## Sample Workflow

1. Create a daily expense:

   * Mobile Recharge – 500
   * Date: 20-Jan

   Result:

   * A new Family Expense summary is created with amount 500

2. Create another daily expense on the same date:

   * Books – 250

   Result:

   * The existing summary is updated to a total of 750

3. View the Family Expenses record:

   * Related list shows only daily expenses for 20-Jan


## Deployment Instructions

1. Import the provided Update Set (55+ updates)
2. Navigate to:

   * Family Expenditure > Family Expenses
   * Family Expenditure > Daily Expenses
3. Create daily expense records
4. Verify automatic summary creation and updates


## Skills Demonstrated

* Custom table creation and field configuration
* Auto-number setup using Number Maintenance
* Business Rules and server-side scripting with GlideRecord
* Parent-child table relationships with query conditions
* Reference fields (sys_user)
* Read-only form configuration
* Form layout customization
* Update Set packaging and deployment

## Project Metadata

* Platform: ServiceNow Personal Developer Instance
* Application Type: Custom Scoped Application
* Complexity Level: Intermediate
* Development Time: Approximately 1 hour
* Production Readiness: Yes

