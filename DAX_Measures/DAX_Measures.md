# 📐 DAX Measures Documentation

All custom DAX measures engineered for the **Retail Sales & Customer RFM Analytics** Power BI model, organized by business domain.

---

## 💰 Revenue, Cost & Profitability (`Sales - Fact`)

### 1. Return Value (`Return`)
Calculates total sales value associated with returned transactions.
```dax
Return = 
CALCULATE(
    SUM('Sales - Fact'[Net_Sales]),
    NOT(ISBLANK('Sales - Fact'[flag]))
)
```

### 2. Net Revenue (`Net_Revenue`)
Calculates actual realized revenue after deducting returns.
```dax
Net_Revenue = SUM('Sales - Fact'[Net_Sales]) - [Return]
```

### 3. Net Total Cost (`Net_Total_Cost`)
Calculates total product cost excluding returned orders.
```dax
Net_Total_Cost = 
SUMX(
    'Sales - Fact',
    IF(NOT(ISBLANK('Sales - Fact'[flag])), 0, 'Sales - Fact'[Total_cost])
)
```

### 4. Net Profit (`Net_Profit`)
Calculates net profit realized from non-returned transactions.
```dax
Net_Profit = [Net_Revenue] - [Net_Total_Cost]
```

### 5. Profit Margin % (`PM%`)
```dax
PM% = [Net_Profit] / [Net_Revenue]
```

### 6. Average Order Value (`AOV`)
```dax
AOV = [Net_Revenue] / DISTINCTCOUNT('Sales - Fact'[Transaction_ID])
```

---

## 📈 Time Intelligence (`Sales - Fact`)

### 7. Previous Month Revenue (`Revenue LM`)
```dax
Revenue LM = 
CALCULATE(
    [Net_Revenue],
    DATEADD('Calendar'[Date], -1, MONTH)
)
```

### 8. Month-over-Month Growth % (`MoM Growth %`)
```dax
MoM Growth % = DIVIDE([Net_Revenue] - [Revenue LM], [Revenue LM], 0)
```

---

## 🔄 Returns & Loss Analysis (`Sales - Fact`)

### 9. Return Rate % (`Return_Rate`)
Percentage of total unique transactions that were returned.
```dax
Return_Rate = 
DIVIDE(
    CALCULATE(
        DISTINCTCOUNT('Sales - Fact'[Transaction_ID]),
        NOT(ISBLANK('Sales - Fact'[flag]))
    ),
    DISTINCTCOUNT('Sales - Fact'[Transaction_ID]),
    0
)
```

### 10. Return Loss % (`Return_Loss %`)
Percentage of gross net sales lost due to returns.
```dax
Return_Loss % = [Return] / SUM('Sales - Fact'[Net_Sales])
```

---

## 👥 Customer Retention & Acquisition (`Customer_Dim`)

### 11. Customer Retention Rate % (`CRR %`)
Tracks the percentage of customers from the previous month who made a repeat purchase in the current month.
```dax
CRR % = 
VAR CurrentCustomers =
    CALCULATETABLE(VALUES('Sales - Fact'[Customer_ID]))

VAR PreviousMonthCustomers =
    CALCULATETABLE(
        VALUES('Sales - Fact'[Customer_ID]),
        DATEADD('Calendar'[Date], -1, MONTH)
    )

VAR RetainedCustomers =
    INTERSECT(CurrentCustomers, PreviousMonthCustomers)

RETURN
DIVIDE(
    COUNTROWS(RetainedCustomers),
    COUNTROWS(PreviousMonthCustomers),
    0
)
```

### 12. New Customers % (`New Customers %`)
Identifies the share of active customers making their first-ever purchase within the selected period.
```dax
New Customers % = 
VAR TotalCust = DISTINCTCOUNT('Sales - Fact'[Customer_ID])
VAR NewCust =
    COUNTROWS(
        FILTER(
            VALUES('Sales - Fact'[Customer_ID]),
            VAR CustID = 'Sales - Fact'[Customer_ID]
            VAR FirstOrderDate =
                CALCULATE(
                    MIN('Sales - Fact'[Order_Date]),
                    ALL('Sales - Fact'),
                    'Sales - Fact'[Customer_ID] = CustID
                )
            RETURN
                FirstOrderDate >= MIN('Sales - Fact'[Order_Date]) &&
                FirstOrderDate <= MAX('Sales - Fact'[Order_Date])
        )
    )
RETURN
    DIVIDE(NewCust, TotalCust)
```

---

## 🏬 Stores & Products (`POS_Stores` & `Products_Dim`)

### 13. Active Stores
```dax
Active Stores = DISTINCTCOUNT('POS_Stores'[STORE_ID])
```

### 14. Total Regions
```dax
Total of Region = DISTINCTCOUNT('POS_Stores'[Region])
```

### 15. Total Products
```dax
Total Products = DISTINCTCOUNT('Products_Dim'[Product_ID])
```

### 16. Total Brands
```dax
Total Brands = DISTINCTCOUNT('Products_Dim'[Brand])
```
