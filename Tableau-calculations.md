# Tableau Calculated Fields

## Time Intelligence

### Current Period vs Prior Period

#### 1. Current Period

```bash
// Current period
IF [Full Date] <= [0 - Yesterday] AND
   DATETRUNC( [Period Selector], [Full Date]) = DATETRUNC([Period Selector], [0 - Yesterday]) 
THEN [Trip Id]

// Current weekday
ELSEIF [Period Selector] = 'weekday' 
    AND [Full Date] = [0 - Yesterday]    
    THEN [Trip Id]

// Current day
ELSEIF [Period Selector] = 'day' 
    AND [Full Date] = [0 - Yesterday]    
    THEN [Trip Id]
END
```

#### 2. Prior Period

```bash
// WEEKDAY
IF [Period Selector] = 'weekday' 
    AND DATEPART('weekday', [Full Date]) = DATEPART('weekday', DATEADD('week', -1, [0 - Yesterday]))
    AND DATEPART('week', [Full Date]) = DATEPART('week', DATEADD('week', -1, [0 - Yesterday]))
    AND DATE([Full Date]) >= DATE(DATEADD('week',-1,([0 - Yesterday])))    
    THEN [Trip Id]

// DAY
ELSEIF [Period Selector] = 'day' AND [Full Date] = DATEADD('day', -1, [0 - Yesterday])    
    THEN [Trip Id]

// WEEK
ELSEIF [Period Selector] = 'week'
    AND DATEPART('week', [Full Date]) = DATEPART('week', DATEADD('week', -52, [0 - Yesterday]))
    AND DATE([Full Date]) <= DATE(DATEADD('week', -52, [0 - Yesterday])) // Same day of the same week last year
    AND DATEPART('weekday', [Full Date]) <= DATEPART('weekday', DATEADD('week', -52, [0 - Yesterday]))
    AND DATEPART('year', [Full Date]) = DATEPART('year', DATEADD('year', -1, [0 - Yesterday]))
    THEN [Trip Id]

// MONTH, YEAR, QUARTER
ELSEIF [Period Selector] <> 'weekday' AND [Period Selector] <> 'day' AND [Period Selector]  <>  'week'  
    AND DATEPART([Period Selector], [Full Date]) = DATEPART([Period Selector], DATEADD('year', -1, [0 - Yesterday]))
    AND DATEPART('year', [Full Date]) = DATEPART('year', DATEADD('year', -1, [0 - Yesterday]))
    AND [Full Date] <= DATEADD('year', -1, [0 - Yesterday])
    THEN [Trip Id]
END
```

#### 3. Prior Period for Bar Chart

```bash
// MONTH, QUARTER, YEAR
IF  [Period Selector] <> 'weekday' AND [Period Selector] <> 'day' AND [Period Selector] <> 'week'
    AND DATEPART([Period Selector], [Full Date]) = DATEPART([Period Selector], DATEADD('year', -1, [0 - Yesterday]))
    AND DATEPART('year', [Full Date]) = DATEPART('year', DATEADD('year', -1, [0 - Yesterday]))    
    THEN [Trip Id]

// WEEK
ELSEIF [Period Selector] = 'week'
    AND DATEPART([Period Selector], [Full Date]) = DATEPART([Period Selector], DATEADD('year', -1, [0 - Yesterday]))
    AND DATEPART('year', [Full Date]) = DATEPART('year', DATEADD('year', -1, [0 - Yesterday]))        
    THEN [Trip Id]     

// WEEKDAY
ELSEIF [Period Selector] = 'weekday'     
    AND DATEPART('week', [Full Date]) = DATEPART('week', DATEADD('week', -1, [0 - Yesterday]))
    AND DATE([Full Date]) >= DATE(DATEADD('month',-1,([0 - Yesterday])))     
    THEN [Trip Id]    

// DAY
ELSEIF [Period Selector] = 'day' AND [Full Date] = DATEADD('day', -1, [0 - Yesterday])    
    THEN [Trip Id]
END
```

#### 4. Filter Condition for Bar Chart
Filter for the bar chart of the Executive Summary, for “quarter” and “day”. To not show empty days/quarters.

```bash
// FOR QUARTER AND DAY
// Prior PERIOD
IF [Period Selector] = 'quarter' 
    AND DATEPART([Period Selector], [Full Date]) = DATEPART([Period Selector], DATEADD('year', -1, [0 - Yesterday]))
    AND DATEPART('year', [Full Date]) = DATEPART('year', DATEADD('year', -1, [0 - Yesterday]))    
THEN TRUE

ELSEIF [Period Selector] = 'day' 
    AND [Full Date] = DATEADD('day', -1, [0 - Yesterday])   
THEN TRUE

// Current period
ELSEIF [Period Selector] = 'quarter'
    AND [Full Date] <= [0 - Yesterday] 
    AND DATETRUNC( [Period Selector], [Full Date]) = DATETRUNC([Period Selector], [0 - Yesterday]) 
THEN TRUE

ELSEIF [Period Selector] = 'day' 
    AND [Full Date] = [0 - Yesterday]    
THEN TRUE

// FOR MONTH, WEEK, YEAR, WEEKDAY
ELSEIF [Period Selector] = 'weekday' 
        OR [Period Selector] = 'month' 
        OR [Period Selector] = 'year'
        OR [Period Selector] = 'week'
THEN TRUE
ELSE FALSE
END
```

## Outliers Detection - Filter (mean + 3 * stdev)

1. Create a calculated field for the Standard deviation
```bash
// Standard deviation
{ FIXED [Vehicle Service], [Model name], YEAR([Full Date]), MONTH([Full Date]): STDEV([Distance])}
```
2. Create a calculated field for the Mean
```bash
// Mean
{ FIXED [Vehicle Service], [Model name], YEAR([Full Date]), MONTH([Full Date]): AVG([Distance])}
```
2. Create a calculated field for the outliers detection
```bash
// Detect outliers - Use as a filter on a chart
{FIXED [Trip Id]:
IF SUM([Distance]) < SUM([Distance - Mean]) - (SUM([Distance - Stdev]) * [Distance - Stdvev]) THEN 'Lower Outlier'
ELSEIF SUM([Distance]) > (SUM([Distance - Stdev]) * [Distance - Stdvev]) + SUM([Distance - Mean]) THEN 'Upper Outlier'
ELSE 'Not Outlier' END
}
```

## Pagination (avoid scroll bar)
Example based on the Canceled Trips dashboard, chart “Top Users by nb. canceled trips”

1. Create a parameter: Pagination Lower Range ( integer, allowed value: all, current value: 1)
2. 
3. Create a T/F calculated field: Pagination Lower Range Limit
     
```bash
[Pagination Lower Range] = 1
```

3. Create a T/F calculated field: Paginagtion Upper Range Limit
   
```bash
// Pagination Upper Range Limit
[Pagination Lower Range] + 10 >= {FIXED : COUNTD([User Id])}
```
4. Create a calculated field: Pagination Range Decrease
   
```bash 
// Pagination Range Decrease
IF [Pagination Lower Range Limit] THEN 1 
ELSE [Pagination Lower Range] - 10 END
```
5. Create a calculated field: Pagination Range Increase
   
```bash
// Pagination Range Increase
IF [Pagination Upper Range Limit] THEN [Pagination Lower Range] 
ELSE [Pagination Lower Range] +10 END
```
6. Create a filter: Pagination Filter (select True)

```bash 
// Filtering
INDEX()>=[Pagination Lower Range] AND INDEX()<[Pagination Lower Range]+10
```

7. For the bar chart, add a ref. line: Edit > Entire Table > Value > “Pagination Range Ref Line”
Create a calculated field: Pagination Range Ref Line
```bash
WINDOW_MAX([Count Canceled Trips])
```
