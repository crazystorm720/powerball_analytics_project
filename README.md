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

---

# Statistical and Probability Framework for Lottery Analysis

## I. Core Probability Concepts

### A. Basic Probability Spaces
1. **Sample Space (Ω)**
   - White balls: {1, 2, ..., 69}
   - Powerball: {1, 2, ..., 26}
   - Total possible combinations: C(69,5) × 26 = 292,201,338

2. **Independent Events**
   - Each drawing is independent
   - P(Drawing₂|Drawing₁) = P(Drawing₂)
   - No sequential dependency

### B. Combinatorial Probability
1. **Hypergeometric Distribution**
   ```python
   def hypergeometric_prob(N, K, n, k):
       """
       N = total numbers (69)
       K = favorable numbers
       n = numbers drawn (5)
       k = successful matches
       """
       return comb(K, k) * comb(N-K, n-k) / comb(N, n)
   ```

2. **Expected Value Calculations**
   ```python
   def expected_value(probabilities, payouts, ticket_cost):
       return sum(p * v for p, v in zip(probabilities, payouts)) - ticket_cost
   ```

## II. Statistical Analysis Framework

### A. Mean Reversion Metrics
1. **Z-Score Calculation**
   ```python
   def calculate_zscore(appearances, expected_freq, time_period):
       std_dev = sqrt(expected_freq * (1 - expected_freq/69) * time_period)
       return (appearances - expected_freq * time_period) / std_dev
   ```

2. **Bollinger Bands**
   - Middle Band = 20-day SMA
   - Upper Band = Middle Band + (2 × σ)
   - Lower Band = Middle Band - (2 × σ)

### B. Statistical Pressure Indicators

1. **Short-term Pressure**
   ```
   P_short = (E_freq - A_actual) / σ_short
   Where:
   - E_freq = Expected frequency
   - A_actual = Actual appearances
   - σ_short = Short-term standard deviation
   ```

2. **Long-term Pressure**
   ```
   P_long = (E_total - A_total) / σ_long
   Where:
   - E_total = Expected total appearances
   - A_total = Actual total appearances
   - σ_long = Long-term standard deviation
   ```

## III. Key Statistical Tests & Metrics

### A. Randomness Tests
1. **Chi-Square Test for Uniformity**
   ```python
   def chi_square_test(observed_freq, expected_freq):
       chi_square = sum((o - e)**2 / e 
                       for o, e in zip(observed_freq, expected_freq))
       return chi_square
   ```

2. **Runs Test**
   ```python
   def runs_test(sequence, median):
       runs = len([i for i in range(1, len(sequence))
                  if (sequence[i] >= median) != (sequence[i-1] >= median)])
       return runs
   ```

### B. Pattern Recognition Metrics

1. **Deviation from Expected Frequency**
   ```
   D = (O - E) / √(E * (1 - p))
   Where:
   - O = Observed frequency
   - E = Expected frequency
   - p = Probability of occurrence
   ```

2. **Pattern Significance Score**
   ```
   S = (Pattern_freq - Expected_freq) / σ_pattern
   ```

## IV. Regression Analysis

### A. Time Series Components
1. **Trend Analysis**
   - Moving averages
   - Exponential smoothing
   - Linear regression slopes

2. **Cyclical Patterns**
   - Fourier analysis
   - Periodogram analysis
   - Autocorrelation functions

### B. Mean Reversion Models
1. **Ornstein-Uhlenbeck Process**
   ```
   dX_t = θ(μ - X_t)dt + σdW_t
   Where:
   - θ = Mean reversion speed
   - μ = Long-term mean
   - σ = Volatility
   ```

2. **Half-Life Calculation**
   ```python
   def calculate_half_life(theta):
       return log(2) / theta
   ```

## V. Implementation Priorities

### Phase 1: Core Statistical Measures
1. Basic frequency analysis
2. Z-score calculations
3. Simple mean reversion metrics

### Phase 2: Advanced Analysis
1. Pattern recognition
2. Time series analysis
3. Regression models

### Phase 3: Predictive Modeling
1. Machine learning integration
2. Neural network analysis
3. Bayesian updating

## VI. Key Statistical Assumptions

1. **Independence Assumption**
   - Drawings are independently distributed
   - No sequential correlation

2. **Stationarity**
   - Probability structure remains constant
   - Mean reversion is consistent

3. **Normal Distribution**
   - Frequency distributions approximate normal
   - Z-scores are meaningful

## VII. Error Measurements

1. **Standard Error Calculations**
   ```python
   def standard_error(sample_size):
       return sqrt(p * (1-p) / n)
       # where p = probability, n = sample size
   ```

2. **Confidence Intervals**
   ```python
   def confidence_interval(mean, std_error, confidence=0.95):
       z = norm.ppf((1 + confidence) / 2)
       return mean ± z * std_error
   ```

## VIII. Validation Framework

1. **Backtesting Protocol**
   - Out-of-sample testing
   - Rolling window analysis
   - Performance metrics

2. **Statistical Significance**
   - p-value calculations
   - Effect size measurements
   - Power analysis
