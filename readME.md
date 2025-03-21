# IPL Match Prediction Model

## Overview

This machine learning model predicts Indian Premier League (IPL) cricket match outcomes with statistical analysis of historical data. It evaluates team performance, head-to-head statistics, venue factors, and toss information to forecast match results with probability estimates.

[View demo](https://github.com/dhruvxvaishnav/IPL-Prediction-Model) | [Report issues](https://github.com/dhruvxvaishnav/IPL-Prediction-Model/issues)

## Features

- **Comprehensive Data Processing**: Parses and analyzes IPL match data across multiple seasons
- **Advanced Feature Engineering**: Transforms raw cricket data into ML-ready features
- **Multi-Model Comparison**: Trains and evaluates RandomForest, XGBoost, and CatBoost models
- **Hyperparameter Optimization**: Fine-tunes models for optimal performance
- **Probabilistic Predictions**: Provides win probabilities for both teams
- **Toss Analysis**: Evaluates impact of toss outcomes on match results
- **Venue Intelligence**: Incorporates home advantage and venue-specific statistics
- **Interactive Dashboard**: Visualizes predictions in an accessible format

## Installation

```bash
# Clone the repository
git clone https://github.com/dhruvxvaishnav/IPL-Prediction-Model.git
cd IPL-Prediction-Model

# Install dependencies
pip install -r requirements.txt
```

## Data Sources

The model uses data from multiple sources:

- Historical IPL match data (2008-2023)
- Team performance statistics by season
- Head-to-head team records
- Venue statistics and home advantage metrics

## Model Architecture

The prediction system employs a pipeline architecture:

1. **Data Preprocessing**: Cleans and standardizes input data
2. **Feature Transformation**: Applies scaling to numerical features and one-hot encoding to categorical features
3. **Model Training**: Fits multiple ML models to historical data
4. **Model Selection**: Selects best performer based on accuracy metrics
5. **Prediction**: Generates probabilistic match outcome forecasts

## Performance

- **Accuracy**: ~70-75% on 2023 IPL season (test set)
- **Key Predictive Features**:
  - Team win rates
  - Head-to-head statistics
  - Home advantage
  - Toss outcome influence

## Usage

### Basic Prediction

```python
from src.model import predict_match

# Predict a match outcome
prediction = predict_match(
    model,
    'Mumbai Indians',       # Team 1
    'Chennai Super Kings',  # Team 2
    'Wankhede Stadium',     # Venue
    'Mumbai',               # City
    'Mumbai Indians',       # Toss winner
    'bat'                   # Toss decision
)
print(prediction)
# Output: "Mumbai Indians win with 53.27% probability"
```

### Multiple Scenarios

```python
# Test all possible toss outcomes
scenarios = [
    (team1, 'bat'),   # Team 1 wins toss and bats
    (team1, 'field'), # Team 1 wins toss and fields
    (team2, 'bat'),   # Team 2 wins toss and bats
    (team2, 'field')  # Team 2 wins toss and fields
]

for toss_winner, toss_decision in scenarios:
    prediction = predict_match(model, team1, team2, venue, city, toss_winner, toss_decision)
    print(f"If {toss_winner} wins toss & chooses to {toss_decision}: {prediction}")
```

## Project Structure

```
IPL-Prediction-Model/
├── README.md
├── requirements.txt
├── src/
│   ├── cricketModel.py     # Data processing utilities
├── data/
│   ├── matches.csv           # Historical match data
│   ├── team_stats.csv        # Team performance metrics
│   ├── head_to_head.csv      # Team vs team statistics
│   └── model_features.csv    # Processed features for model
├── models/
│   ├── RandomForest.pkl      # Trained RandomForest model
│   ├── XGBoost.pkl           # Trained XGBoost model
│   ├── CatBoost.pkl          # Trained CatBoost model
│   └── tuned_BestModel.pkl   # Hyperparameter-tuned best model
└── results/
    ├── model_accuracy_comparison.png    # Performance visualization
    ├── feature_importance.png           # Feature impact analysis
    └── ipl_2024_predictions_dashboard.html  # Prediction dashboard
```

## Limitations

- Does not account for player injuries or team composition changes
- Limited consideration of player form and individual statistics
- Weather and pitch conditions not factored into predictions
- Cannot predict unusual match events (rain interruptions, super overs)

## Future Improvements

- Incorporate player-level statistics and impact scores
- Add recent team form metrics (last 5 matches performance)
- Include pitch and weather condition factors
- Implement time-series analysis for team performance trends
- Develop a web application for real-time predictions

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contact

Dhruv Vaishnav - [dhruvvaishnav687@gmail.com](mailto:dhruvvaishnav687@gmail.com)

Project Link: [https://github.com/dhruvxvaishnav/IPL-Prediction-Model](https://github.com/dhruvxvaishnav/IPL-Prediction-Model)

## Acknowledgments

- [Cricsheet](https://cricsheet.org/) for providing comprehensive ball-by-ball data
- [ESPNCricinfo](https://www.espncricinfo.com/) for player statistics
- Indian Premier League for exciting cricket that makes this analysis possible
