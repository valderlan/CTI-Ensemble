## Description

The **Threat-Intelligence-Ensemble** is an IP address risk classification system that leverages Machine Learning techniques and Cyber Threat Intelligence (CTI). The project implements multiple machine learning models, including ensemble methods, to classify IPs into different risk levels (Low, Medium, High) based on cyber threat intelligence signals.

The system uses an end-to-end pipeline that includes:
- Data preprocessing and normalization
- Automated IP labeling using weighted voting
- Training multiple ML models
- Comparative performance evaluation
- Real-time prediction

## Supported Models

The system supports the following Machine Learning models:

### Individual Models
- **Random Forest**: An ensemble of decision trees with voting
- **SVM (Support Vector Machine)**: Maximum-margin classifier
- **Neural Network (MLP)**: Multi-layer perceptron
- **Extra Trees**: Extremely randomized trees for robust and fast ensemble learning
- **Decision Tree**: Single decision tree
- **KNN (K-Nearest Neighbors)**: Neighborhood-based classifier
- **CNN (Convolutional Neural Network)**: Convolutional neural network

### Ensemble Models
- **AdaBoost Classifier**: Boosting-based ensemble that focuses on hard-to-classify samples
- **Voting Classifier**: Combines multiple models via voting (soft/hard)
- **Stacking Classifier**: Stacks models using a meta-learner


The system also supports training with and without **SMOTE** (Synthetic Minority Over-sampling Technique) for class balancing.

## How the System Classifies IPs

The CTI-Ensemble operates in multiple stages to classify IP address risk:

1. **Intelligence Collection**: Integrates data from multiple CTI (Cyber Threat Intelligence) sources
2. **Normalization**: Standardizes features such as threat counts, reputation scores, etc.
3. **Automated Labeling**: Applies a weighted voting algorithm to automatically label IPs
4. **Model Training**: Trains multiple ML algorithms on the labeled data
5. **Ensemble Learning**: Combines predictions from different models to improve accuracy
6. **Real-Time Prediction**: Classifies new IPs using trained models



## Installation

This project uses **uv** to manage dependencies and virtual environments.

### Prerequisites
- Python >= 3.13
- uv (package manager)

### Installation Steps

1. **Clone the repository:**
```bash
git clone git@github.com:LarcesUece/CTI-Ensemble.git
cd CTI-Ensemble
```

2. **Install uv (if you don't have it yet):**
```bash
# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Linux/macOS
curl -LsSf https://astral.sh/uv/install.sh | sh
```

3. **Create the virtual environment and install dependencies:**
```bash
uv sync

```

4. **Activate the virtual environment:**
```bash
# Windows
.venv\Scripts\activate

# Linux/macOS
source .venv/bin/activate
```

## Project Structure

```
CTI-Ensemble/
├── data_prep_pipeline.py    # Dataset preparation pipeline
├── ml_pipeline.py            # Model training pipeline
├── ip_prediction.py          # IP prediction script
├── pyproject.toml            # Dependency configuration (uv)
├── scaler_params.pkl         # Feature normalization parameters
├── config/                   # Project configuration
├── src/                      # Main source code
│   ├── data_processing.py    # Data processing
│   ├── dataset_classifier.py # Dataset classifier
│   ├── models.py             # Model definitions
│   ├── train.py              # Training
│   ├── evaluate.py           # Evaluation
│   ├── normalize.py          # Normalization
│   └── visualization.py      # Visualizations
├── datasets/                 # Input datasets
├── data/                     # Trained models and results
│   ├── models/               # Models without SMOTE
│   ├── models_s/             # Models with SMOTE
│   └── experiments/          # Experiment outputs
└── logs/                     # Execution logs
```

## Dataset Preparation Pipeline

The script [data_prep_pipeline.py](data_prep_pipeline.py) is responsible for the initial data preparation:

### Features

1. **Feature Normalization**
   - Converts IPs to a numeric format
   - Normalizes scores and counts
   - Scales features to the 0-1 range

2. **Automated Labeling**
   - Applies `WeightedVotingClassifier` to label IPs
   - Classifies into 3 levels: Low Risk, Medium Risk, High Risk
   - Uses multiple CTI features for decision-making

### Run

```bash
python data_prep_pipeline.py
```

### Input/Output
- **Input**: [datasets/dataset_ip.csv](datasets/dataset_ip.csv) (raw data)
- **Output**:
   - `datasets/dataset_ip_norm.csv` (normalized data)
   - `datasets/dataset_ip_classified.csv` (labeled data)

### Process
```
Raw Dataset → Normalization → Labeling → Prepared Dataset
```

## Training Pipeline (ML Pipeline)

The script [ml_pipeline.py](ml_pipeline.py) manages the full model training process:

### Features

1. **Data Preparation**
   - Loads the labeled dataset
   - Splits into train/test (stratified)
   - Applies SMOTE (optional) for balancing

2. **Model Training**
   - Trains all supported models
   - Optimizes hyperparameters
   - Saves trained models

3. **Full Evaluation**
   - Metrics: Accuracy, Precision, Recall, F1-Score
   - Confusion matrices
   - ROC curves
   - Feature importance
   - Detailed reports

4. **Visualizations**
   - Metric comparison charts
   - Class distribution
   - Feature correlation
   - Execution times

### Run

```bash
# Runs both experiments automatically:
# - WITHOUT SMOTE
# - WITH SMOTE
python ml_pipeline.py
```

### Output

Results are saved to `data/experiments/experiment_[WITH/WITHOUT]_smote_[timestamp]/`:
- `report/`: Results in CSV and Markdown
- `images/`: Charts and visualizations
- Models saved under `data/models/` or `data/models_s/`

## IP Prediction

The script [ip_prediction.py](ip_prediction.py) allows classifying new IP addresses:

### Operation Modes

#### 1. Single IP Prediction
```bash
python ip_prediction.py
```

In the code, configure:
```python
# Select the model
MODEL_NAME = "Voting"  # or "Random Forest", "SVM", "CNN", etc.

# Select the models directory
MODELS_DIR = "data/models_s"  # with SMOTE (default in current code)
# MODELS_DIR = "data/models"  # without SMOTE
```

#### 2. Batch Prediction (CSV)
```python
# In the code, configure:
predictor = IPClassificationPredictor(
    models_dir="data/models",
    model_name="Voting"
)

# For CSV:
results = predictor.predict_from_csv("input_ips.csv", "output_results.csv")
```

### Models Available for Prediction

- "Random Forest": Best for balanced datasets
- "SVM": High precision, slower
- "Neural Network": Good for complex patterns
- "AdaBoost": Effective boosting approach for difficult samples
- "Extra Trees": Fast ensemble with strong generalization
- "Decision Tree": Fast, interpretable
- "KNN": Good for local similarity
- "CNN": Best for sequential-like features
- "Voting": Combines multiple models
- "Stacking": **Recommended** - Advanced ensemble with a meta-learner

### Input Format

The CSV file must contain the same features used during training:
- IP address
- Threat count
- Reputation scores
- Denylist status
- Geographic information
- Etc.

### Usage Example

```python
from ip_prediction import IPClassificationPredictor

# Initialize predictor
predictor = IPClassificationPredictor(
    models_dir="data/models",
    model_name="Voting"
)

# Prediction for a single IP
ip_data = {
    'ip': '192.168.1.100',
    'threat_count': 5,
    'reputation_score': 0.7,
   # ... other features
}

result = predictor.predict_single(ip_data)
print(f"IP: {result['ip']}")
print(f"Risk Level: {result['risk_level']}")
print(f"Confidence: {result['confidence']:.2%}")
```

## Logs and Monitoring

All scripts generate detailed logs under `logs/`:
- `data_preprocessing_[timestamp].log`
- `pipeline_[timestamp].log`
- `ip_prediction_[timestamp].log`

## Experiment Results

Experiments automatically save:
- Evaluation metrics (CSV)
- Markdown reports
- Visualization charts
- Serialized models (joblib/keras)

Available at: `data/experiments/experiment_*/`


