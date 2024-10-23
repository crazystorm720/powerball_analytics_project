# Powerball Analytics Project

## Project Overview
A data-driven approach to analyzing Powerball lottery drawings using statistical analysis and visualization. This project focuses on tracking number frequencies, calculating statistical pressure, and identifying patterns through PowerBI analytics.

## Core Components
1. Data Repository (GitHub)
2. Analysis Engine (Python)
3. Visualization Layer (PowerBI)

## Project Structure
```
powerball-analytics/
├── data/
│   ├── raw/                      # Raw drawing results CSV files
│   │   └── powerball_results.csv
│   └── processed/                # Processed datasets
├── scripts/
│   ├── data_collection/          # Scripts to fetch and update drawing data
│   └── analysis/                 # Statistical analysis scripts
├── powerbi/
│   └── templates/                # PowerBI template files
├── docs/
│   └── data_dictionary.md        # Data field definitions
└── README.md
```

## Data Structure

### powerball_results.csv
```csv
draw_date,white_1,white_2,white_3,white_4,white_5,powerball,jackpot_amount
2024-01-01,12,24,36,48,60,18,340000000
```

### Statistical Metrics (processed data)
```csv
number,appearances,days_since_last,pressure_score,expected_frequency
12,45,12,-1.23,0.0724
```

## Phase 1 Implementation

### 1. Data Collection & Storage
- Store drawing results in GitHub as CSV
- Simple update process for new drawings
- Basic data validation
- Version control for all changes

### 2. Core Analysis (Python)
```python
def calculate_basic_pressure(number_history, lookback_days=90):
    """
    Calculate basic statistical pressure for each number
    """
    expected_freq = lookback_days * (5/69)  # For white balls
    appearances = sum(number_history[-lookback_days:])
    std_dev = np.sqrt(expected_freq * (1 - 5/69))
    
    return (expected_freq - appearances) / std_dev

def process_drawing_history(csv_path):
    """
    Process raw drawing history into analysis metrics
    """
    df = pd.read_csv(csv_path)
    metrics = defaultdict(dict)
    
    for num in range(1, 70):
        history = [num in row for row in zip(df['white_1'], 
                                           df['white_2'], 
                                           df['white_3'], 
                                           df['white_4'], 
                                           df['white_5'])]
        metrics[num] = {
            'pressure_score': calculate_basic_pressure(history),
            'days_since_last': len(history) - max(loc for loc, val 
                                                in enumerate(history) if val),
            'appearances': sum(history)
        }
    
    return pd.DataFrame.from_dict(metrics, orient='index')
```

### 3. Initial PowerBI Dashboard
- Focus on core visualizations:
  1. Number frequency heatmap
  2. Days since last appearance
  3. Basic pressure scores
  4. Historical trends

#### Key DAX Measures
```
// Basic Pressure Score
PressureScore = 
VAR ExpectedFreq = AVERAGE(DrawingHistory[Appearances])
RETURN
(ExpectedFreq - LASTDATE(DrawingHistory[Appearances])) /
SQRT(ExpectedFreq * (1 - ExpectedFreq))

// Days Since Last Appearance
DaysSinceLastAppearance = 
DATEDIFF(
    MAX(DrawingHistory[LastAppearance]),
    TODAY(),
    DAY
)
```

## GitHub Usage

### Repository Structure
- Main branch for stable code
- Development branch for new features
- Data updates through pull requests
- Automated validation checks

### Workflow
1. Update drawing results through PR
2. Trigger automated analysis
3. Update processed datasets
4. Sync with PowerBI

## Assumptions & Limitations
1. Data Freshness:
   - Drawing results updated within 24 hours
   - Analysis runs daily

2. Statistical Approach:
   - Focus on basic pressure scores initially
   - Expand to complex patterns later

3. PowerBI Refresh:
   - Daily refresh schedule
   - Direct connection to GitHub CSV

## Next Steps
1. Set up GitHub repository
2. Create initial CSV structure
3. Implement basic Python analysis
4. Build core PowerBI dashboard

## Future Enhancements
1. Advanced statistical models
2. Pattern recognition
3. Automated data collection
4. Real-time updates

## Getting Started

1. Clone the repository:
```bash
git clone https://github.com/your-username/powerball-analytics.git
```

2. Set up Python environment:
```bash
pip install pandas numpy scipy
```

3. Initial data processing:
```bash
python scripts/analysis/process_drawings.py
```

4. Connect PowerBI to processed data:
- Use "Web" data source
- Point to raw GitHub CSV URL
- Apply transformations as needed

## Contributing
1. Fork the repository
2. Create feature branch
3. Submit pull request
4. Include test cases

## License
MIT License - feel free to use and modify
