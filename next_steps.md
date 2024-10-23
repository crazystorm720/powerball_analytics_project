## getting started

```python
import pandas as pd
import numpy as np
from datetime import datetime, date
from typing import Dict, List, Tuple

class PressureAnalyzer:
    """Analyzes lottery number pressure based on historical appearances"""
    
    def __init__(self):
        self.NUMBERS_DRAWN = 5
        self.TOTAL_NUMBERS = 69
        self.EXPECTED_FREQ = self.NUMBERS_DRAWN / self.TOTAL_NUMBERS
        
    def load_and_prepare(self, csv_path: str) -> pd.DataFrame:
        """
        Expects CSV with columns: draw_date, n1, n2, n3, n4, n5
        Returns clean DataFrame
        """
        df = pd.read_csv(csv_path, parse_dates=['draw_date'])
        df['numbers'] = df.apply(lambda x: [x['n1'], x['n2'], x['n3'], x['n4'], x['n5']], axis=1)
        return df.sort_values('draw_date')
    
    def calculate_appearances(self, data: pd.DataFrame) -> Dict[int, int]:
        """Calculate how many times each number has appeared"""
        appearances = {num: 0 for num in range(1, self.TOTAL_NUMBERS + 1)}
        for numbers in data['numbers']:
            for num in numbers:
                appearances[num] += 1
        return appearances
    
    def calculate_days_since_last(self, data: pd.DataFrame) -> Dict[int, int]:
        """Calculate days since each number last appeared"""
        last_seen = {num: None for num in range(1, self.TOTAL_NUMBERS + 1)}
        latest_draw = data['draw_date'].max()
        
        # Get last appearance for each number
        for idx, row in data.iterrows():
            for num in row['numbers']:
                last_seen[num] = row['draw_date']
        
        # Convert to days since last appearance
        days_since = {
            num: (latest_draw - date).days if date else 999
            for num, date in last_seen.items()
        }
        return days_since
    
    def calculate_pressure(self, data: pd.DataFrame) -> pd.DataFrame:
        """
        Calculate pressure scores for all numbers
        Returns DataFrame with pressure analysis
        """
        total_drawings = len(data)
        expected_appearances = total_drawings * self.EXPECTED_FREQ
        
        appearances = self.calculate_appearances(data)
        days_since = self.calculate_days_since_last(data)
        
        results = []
        for number in range(1, self.TOTAL_NUMBERS + 1):
            actual = appearances[number]
            deviation = (expected_appearances - actual) / expected_appearances
            days_gap = days_since[number]
            
            # Pressure increases with both deviation and days since last appearance
            pressure = deviation * (1 + (days_gap / 30))
            
            results.append({
                'number': number,
                'appearances': actual,
                'expected': expected_appearances,
                'deviation': deviation,
                'days_since_last': days_gap,
                'pressure_score': pressure
            })
            
        return pd.DataFrame(results)

    def get_high_pressure_numbers(self, pressure_df: pd.DataFrame, top_n: int = 10) -> pd.DataFrame:
        """Return top N numbers under highest pressure to appear"""
        return pressure_df.nlargest(top_n, 'pressure_score')

# Example usage:
analyzer = PressureAnalyzer()
data = analyzer.load_and_prepare('drawing_history.csv')
pressure_df = analyzer.calculate_pressure(data)
high_pressure = analyzer.get_high_pressure_numbers(pressure_df)

```

And here are the corresponding PowerBI measures that work with this analysis:



# PowerBI Implementation

## Core Tables Required
1. Drawings (from your CSV)
   - draw_date
   - n1, n2, n3, n4, n5

2. Numbers (reference table)
   - number (1-69)

## Key Measures

```
// Appearances
Number Appearances = 
CALCULATE(
    COUNTROWS(Drawings),
    FILTER(
        Drawings,
        OR(
            Drawings[n1] = SELECTEDVALUE(Numbers[number]),
            Drawings[n2] = SELECTEDVALUE(Numbers[number]),
            Drawings[n3] = SELECTEDVALUE(Numbers[number]),
            Drawings[n4] = SELECTEDVALUE(Numbers[number]),
            Drawings[n5] = SELECTEDVALUE(Numbers[number])
        )
    )
)

// Expected Appearances
Expected Appearances = 
COUNTROWS(Drawings) * (5/69)

// Deviation
Deviation = 
VAR Expected = [Expected Appearances]
VAR Actual = [Number Appearances]
RETURN
(Expected - Actual) / Expected

// Days Since Last
Days Since Last = 
VAR LastDate = 
    CALCULATE(
        MAX(Drawings[draw_date]),
        FILTER(
            Drawings,
            OR(
                Drawings[n1] = SELECTEDVALUE(Numbers[number]),
                Drawings[n2] = SELECTEDVALUE(Numbers[number]),
                Drawings[n3] = SELECTEDVALUE(Numbers[number]),
                Drawings[n4] = SELECTEDVALUE(Numbers[number]),
                Drawings[n5] = SELECTEDVALUE(Numbers[number])
            )
        )
    )
RETURN
    DATEDIFF(LastDate, TODAY(), DAY)

// Pressure Score
Pressure Score = 
[Deviation] * (1 + ([Days Since Last] / 30))
```

## Key Visualizations

1. Pressure Heat Map
```
Visual: Matrix
Rows: Numbers[number]
Values: [Pressure Score]
Conditional formatting: Data bars
Color scale: Red-Yellow-Green (reversed)
```

2. High Pressure Numbers
```
Visual: Table
Values: 
    Numbers[number]
    [Days Since Last]
    [Deviation]
    [Pressure Score]
Sort: [Pressure Score] Descending
Top N filter: 10
```

3. Historical Trend
```
Visual: Line chart
X-axis: Drawings[draw_date]
Y-axis: [Number Appearances]
Small multiples: Top 5 pressure numbers
```

## Page Layout
1. Heat Map (50% width)
2. High Pressure Table (25% width)
3. Historical Trend (25% width)
4. Key metrics cards above:
   - Total drawings analyzed
   - Average deviation
   - Highest pressure score
   - Days since last drawing


This gives you:
1. A complete pressure analysis implementation
2. Clear PowerBI measures and visualizations
3. Flexibility to adjust parameters
4. Easy integration with your git-synced CSV

The Python code can be containerized and run whenever your CSV updates, while PowerBI will refresh based on the latest analysis results.

Would you like me to:
1. Detail any specific part further?
2. Add specific validation checks?
3. Expand any visualization components?
