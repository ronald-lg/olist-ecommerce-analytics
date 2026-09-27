# DAX measure catalogue

*Extracted from `powerbi/olist_ecommerce_analytics.pbix`. All measures live in the `_Measures` table, grouped by display folder.*

*Catálogo extraído del `.pbix`. Todas las medidas viven en la tabla `_Measures`, agrupadas por carpeta.*

## 01 Sales

### Average Order Value

```dax
Average Order Value =
divide([Gross Order Value], [Total Orders])
```

### Freight Revenue

```dax
Freight Revenue =
SUM('Order Items'[freight_value])
```

### Gross Order Value

```dax
Gross Order Value =
[Product Revenue] + [Freight Revenue]
```

### Gross Order Value (M)

```dax
Gross Order Value (M) =
DIVIDE([Gross Order Value], 1000000)
```

### Monthly Growth Rate

```dax
Monthly Growth Rate =
DIVIDE(
    [Gross Order Value] - [Previous Month Gross Order Value],
    [Previous Month Gross Order Value]
)
```

### Payment Difference

```dax
Payment Difference =
_Measures[Payment Value] - _Measures[Gross Order Value]
```

### Payment Difference Ratio

```dax
Payment Difference Ratio =
DIVIDE ( [Payment Value] - [Gross Order Value], [Gross Order Value] )
```

### Payment Value

```dax
Payment Value =
sum(Payments[payment_value])
```

### Previous Month Gross Order Value

```dax
Previous Month Gross Order Value =
CALCULATE(
    [Gross Order Value],
    DATEADD('Calendar'[Date], -1, MONTH)
)
```

### Product Revenue

```dax
Product Revenue =
SUM('Order Items'[price])
```

## 02 Time

### GOV Growth MoM %

```dax
GOV Growth MoM % =
DIVIDE (
    [Gross Order Value] - [Previous Month GOV],
    [Previous Month GOV]
)
```

### Previous Month GOV

```dax
Previous Month GOV =
CALCULATE (
    [Gross Order Value],
    DATEADD ( Calendar[Date], -1, MONTH )
)
```

## 03 Customers

### Active Consumers

```dax
Active Consumers =
CALCULATE(
    DISTINCTCOUNT('Customers'[customer_unique_id]),
    CROSSFILTER('Orders'[customer_id], 'Customers'[customer_id], BOTH)
)
```

### Active Consumers (Thousands)

```dax
Active Consumers (Thousands) =
DIVIDE([Active Consumers], 1000)
```

### Orders per Customer

```dax
Orders per Customer =
DIVIDE ( [Total Orders], [Total Customers] )
```

### Repeat Customer Rate

```dax
Repeat Customer Rate =
DIVIDE ( [Repeat Customers], [Total Customers] )
```

### Repeat Customers

```dax
Repeat Customers =
COUNTROWS (
    FILTER (
        VALUES ( Customers[customer_unique_id] ),
        CALCULATE ( DISTINCTCOUNT ( Orders[order_id] ) ) > 1
    )
)
```

### Total Customers

```dax
Total Customers =
DISTINCTCOUNT ( Customers[customer_unique_id] )
```

## 04 Delivery

### Average Delivery Days

```dax
Average Delivery Days =
AVERAGEX (
    FILTER (
        Orders,
        NOT ISBLANK ( Orders[order_delivered_customer_date] )
    ),
    DATEDIFF (
        Orders[order_purchase_timestamp],
        Orders[order_delivered_customer_date],
        DAY
    )
)
```

### Delivered Orders

```dax
Delivered Orders =
CALCULATE(
    DISTINCTCOUNT('Orders'[order_id]),
    FILTER(
        'Orders',
        NOT ISBLANK('Orders'[order_delivered_customer_date])
    )
)
```

### Late Deliveries

```dax
Late Deliveries =
CALCULATE(
    [Total Orders],
    'Orders'[Delivery Status] = "Late"
)
```

### Late Delivery Rate

```dax
Late Delivery Rate =
DIVIDE (
    [Late Orders],
    CALCULATE (
        [Total Orders],
        FILTER (
            Orders,
            NOT ISBLANK ( Orders[order_delivered_customer_date] )
        )
    )
)
```

### Late Orders

```dax
Late Orders =
CALCULATE (
    [Total Orders],
    FILTER (
        Orders,
        NOT ISBLANK ( Orders[order_delivered_customer_date] )
            && Orders[order_delivered_customer_date]
                > Orders[order_estimated_delivery_date]
    )
)
```

### Not Delivered Orders

```dax
Not Delivered Orders =
CALCULATE(
    DISTINCTCOUNT('Orders'[order_id]),
    FILTER(
        'Orders',
        ISBLANK('Orders'[order_delivered_customer_date])
    )
)
```

### Not Delivered Rate

```dax
Not Delivered Rate =
DIVIDE(
    [Not Delivered Orders],
    [Total Orders]
)
```

### On-Time Delivery Rate

```dax
On-Time Delivery Rate =
DIVIDE (
    CALCULATE (
        [Total Orders],
        FILTER (
            Orders,
            NOT ISBLANK ( Orders[order_delivered_customer_date] )
                && Orders[order_delivered_customer_date]
                    <= Orders[order_estimated_delivery_date]
        )
    ),
    CALCULATE (
        [Total Orders],
        FILTER (
            Orders,
            NOT ISBLANK ( Orders[order_delivered_customer_date] )
        )
    )
)
```

## 05 Reviews

### Average Customer Rating

```dax
Average Customer Rating =
AVERAGE ( Reviews[Review Score] )
```

### Average Review Score Delivered

```dax
Average Review Score Delivered =
CALCULATE (
    [Average Customer Rating],
    FILTER (
        Orders,
        NOT ISBLANK ( Orders[order_delivered_customer_date] )
    )
)
```

### Average Review Score Late Orders

```dax
Average Review Score Late Orders =
CALCULATE (
    [Average Customer Rating],
    FILTER (
        Orders,
        NOT ISBLANK ( Orders[order_delivered_customer_date] )
            && Orders[order_delivered_customer_date]
                > Orders[order_estimated_delivery_date]
    )
)
```

### Average Review Score On-Time

```dax
Average Review Score On-Time =
CALCULATE (
    [Average Customer Rating],
    FILTER (
        Orders,
        NOT ISBLANK ( Orders[order_delivered_customer_date] )
            && Orders[order_delivered_customer_date]
                <= Orders[order_estimated_delivery_date]
    )
)
```

### Five-Star Review Rate

```dax
Five-Star Review Rate =
DIVIDE (
    CALCULATE (
        COUNTROWS ( Reviews ),
        Reviews[Review Score] = 5
    ),
    COUNTROWS ( Reviews )
)
```

### Five-Star Reviews

```dax
Five-Star Reviews =
CALCULATE([Review Count], Reviews[Review Score] = 5)
```

### Low Review Rate

```dax
Low Review Rate =
DIVIDE (
    CALCULATE (
        COUNTROWS ( Reviews ),
        Reviews[Review Score] <= 2
    ),
    COUNTROWS ( Reviews )
)
```

### Review Count

```dax
Review Count =
COUNTROWS ( Reviews )
```

### Review Count (Thousands)

```dax
Review Count (Thousands) =
DIVIDE([Review Count], 1000)
```

### Review Share %

```dax
Review Share % =
DIVIDE(
    [Review Count],
    CALCULATE([Review Count], ALL('Reviews'[Review Score]))
)
```

## 06 Validation

### Reviewed Delivered Orders

```dax
Reviewed Delivered Orders =
CALCULATE (
    DISTINCTCOUNT ( Reviews[order_id] ),
    FILTER (
        Orders,
        NOT ISBLANK ( Orders[order_delivered_customer_date] )
    )
)
```

### Reviewed Orders

```dax
Reviewed Orders =
CALCULATE (
    DISTINCTCOUNT ( Reviews[order_id] )
)
```

### Weighted Review Check Delivered

```dax
Weighted Review Check Delivered =
[Average Review Score On-Time] * [On-Time Delivery Rate]
    + [Average Review Score Late Orders] * [Late Delivery Rate]
```

## 07 Core Counts

### Order Count (Thousands)

```dax
Order Count (Thousands) =
DIVIDE([Total Orders], 1000)
```

### Order Share %

```dax
Order Share % =
DIVIDE(
    [Total Orders],
    CALCULATE(
        [Total Orders],
        ALL(Orders[Delivery Status])
    )
)
```

### Test Orders

```dax
Test Orders =
COUNTROWS('Orders')
```

### Total Orders

```dax
Total Orders =
DISTINCTCOUNT(Orders[order_id])
```

### Total Orders (Thousands)

```dax
Total Orders (Thousands) =
divide([Total Orders],1000)
```

## Calculated columns · Columnas calculadas

```dax
-- Orders[Order Purchase Date]: date key for the Calendar relationship
Order Purchase Date =
DATE ( YEAR ( Orders[order_purchase_timestamp] ), MONTH ( Orders[order_purchase_timestamp] ), DAY ( Orders[order_purchase_timestamp] ) )

-- Orders[Delivery Status]
Delivery Status =
IF (
    ISBLANK ( 'Orders'[order_delivered_customer_date] ),
    "Not delivered",
    IF ( 'Orders'[order_delivered_customer_date] > 'Orders'[order_estimated_delivery_date], "Late", "On time" )
)
```

## Calendar table · Tabla calendario

```dax
Calendar =
ADDCOLUMNS (
    CALENDAR ( MIN ( Orders[order_purchase_timestamp] ), MAX ( Orders[order_purchase_timestamp] ) ),
    "Year", YEAR ( [Date] ),
    "Month Number", MONTH ( [Date] ),
    "Month", FORMAT ( [Date], "MMMM" ),
    "Year Month", FORMAT ( [Date], "YYYY-MM" ),
    "Year Month Sort", YEAR ( [Date] ) * 100 + MONTH ( [Date] )
)
```
