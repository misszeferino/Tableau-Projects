# Tableau Calculated Fields - Data Science - Vulog Confluence

## Time Intelligence

### Current Period vs Prior Period

### Current Period

```tableau
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

### Prior Period
```tableau
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

### Prior Period for Bar Chart
```tableau
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

### Filter Condition for Bar Chart
```tableau
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
```tableau
// Standard deviation
{ FIXED [Vehicle Service], [Model name], YEAR([Full Date]), MONTH([Full Date]): STDEV([Distance])}

// Mean
{ FIXED [Vehicle Service], [Model name], YEAR([Full Date]), MONTH([Full Date]): AVG([Distance])}

// Detect outliers - Use as a filter on a chart
{FIXED [Trip Id]:
IF SUM([Distance]) < SUM([Distance - Mean]) - (SUM([Distance - Stdev]) * [Distance - Stdvev]) THEN 'Lower Outlier'
ELSEIF SUM([Distance]) > (SUM([Distance - Stdev]) * [Distance - Stdvev]) + SUM([Distance - Mean]) THEN 'Upper Outlier'
ELSE 'Not Outlier' END
}
```

## Pagination (avoid scroll bar)
```tableau
// Pagination Lower Range
[Pagination Lower Range] = 1

// Pagination Upper Range Limit
[Pagination Lower Range] + 10 >= {FIXED : COUNTD([User Id])}

// Pagination Range Decrease
IF [Pagination Lower Range Limit] THEN 1 
ELSE [Pagination Lower Range] - 10 END

// Pagination Range Increase
IF [Pagination Upper Range Limit] THEN [Pagination Lower Range] 
ELSE [Pagination Lower Range] +10 END

// Filtering
INDEX()>=[Pagination Lower Range] AND INDEX()<[Pagination Lower Range]+10
```

## Reference Line for Bar Chart
```tableau
WINDOW_MAX([Count Canceled Trips])
