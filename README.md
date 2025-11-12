# 🎾 Predicting Tennis Match Upsets Using Machine Learning

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📘 Project Overview

**Upsets** — when lower-ranked players defeat favorites — are among the most fascinating aspects of professional tennis. This project investigates **when and why** these upsets occur using machine learning and comprehensive data analysis.

Using **22,526 ATP match records** from 2016-2024, we developed predictive models that achieve **75% F1-score** and **85% upset detection rate**, revealing that **experience, recent form, and surface specialization** matter more than rankings alone.

**Data Source:** [Jeff Sackmann's Tennis ATP Repository](https://github.com/JeffSackmann/tennis_atp)

---

## 🎯 Key Achievements

✅ **75.1% F1-Score** with XGBoost + SMOTE (balanced performance)  
✅ **85.0% Recall** (upset detection rate)  
✅ **94.5% ROC-AUC** (overall discrimination ability)  
✅ **83 engineered features** capturing form, surface skills, and head-to-head dynamics  
✅ **47.5% recall improvement** through class imbalance handling  
✅ Identified **experience gap** as #1 predictor (not rankings!)  

---

## 🧠 Motivation

While player rankings are strong performance indicators, they fail to explain the **36.5% of matches** where lower-ranked players win. Understanding upset conditions provides value for:

- **🎾 Sports Analysts** – Data-driven commentary and risk assessment  
- **📊 Broadcasters** – Pre-match upset probability graphics  
- **🎓 Coaches** – Strategic preparation based on opponent patterns  
- **🔬 Researchers** – Insights into competitive dynamics  
- **📱 Developers** – Sports prediction applications  

---

## 📊 Model Performance Comparison

| Model | Technique | F1-Score | Recall | Precision | ROC-AUC |
|-------|-----------|----------|--------|-----------|---------|
| Logistic Regression | Baseline | 0.683 | 0.650 | 0.720 | 0.896 |
| Random Forest | Baseline | 0.688 | 0.638 | 0.747 | 0.914 |
| XGBoost | Baseline | 0.745 | 0.707 | 0.787 | 0.945 |
| **XGBoost** | **SMOTE** | **0.751** ⭐ | **0.764** | **0.739** | **0.938** |
| XGBoost | Class Weights | 0.749 | 0.799 | 0.705 | 0.935 |
| Random Forest | Threshold Tuning | 0.590 | **0.981** | 0.422 | 0.917 |

**Key Finding:** XGBoost with SMOTE provides the best balanced performance for upset prediction.

---

## 🔑 Top Predictive Features

Our analysis revealed the most important factors for predicting upsets:

| Rank | Feature | Correlation | Insight |
|------|---------|-------------|---------|
| 1 | **experience_gap_pct** | -0.467 | Experience difference is #1 predictor |
| 2 | **activity_diff** | -0.466 | Recent match activity strongly influences outcomes |
| 3 | **career_win_rate_diff** | -0.420 | Career performance gap matters |
| 4 | **form_diff_90d** | -0.346 | Recent form > long-term statistics |
| 5 | **surface_advantage** | -0.335 | Surface specialization critical |
| 6 | rank_difference | -0.200 | Rankings capture some information |
| 7 | h2h_dominance | -0.130 | Head-to-head psychological edge |

**Negative correlation** = Lower value increases upset probability

### 💡 Key Insights:
- **Experience** trumps rankings (more matches = better upset prediction)
- **Recent form** (90 days) more predictive than career statistics
- **Surface mismatches** create major upset opportunities (e.g., clay specialist on grass)
- **Head-to-head history** less important than form and experience

---

## ⚙️ Methodology

### 1. Data Collection & Cleaning
- **Source:** ATP match data (2016-2024) from Jeff Sackmann
- **Size:** 22,526 matches across 8 years
- **Processing:** Missing value imputation, outlier detection, data type corrections
- **Merging:** Combined match statistics with tournament information

### 2. Feature Engineering (83 Features)

#### Recent Form Features
- Rolling win-loss ratios (90/180/365 days)
- Form momentum (recent vs long-term performance)
- Match activity levels

#### Surface Specialization
- Win rates on Hard/Clay/Grass courts
- Surface-specific performance differentials
- Surface comfort metrics
- Versatility indicators

#### Head-to-Head Dynamics
- Historical matchup statistics
- Previous meeting outcomes
- Surface-specific H2H records

#### Combined Features
- Form differentials (winner vs loser)
- Surface advantage gaps
- Experience and career performance differences

### 3. Target Variable Definition

**Upset** = Lower-ranked player defeats higher-ranked opponent
- Based on ATP rankings at match time
- 36.5% of matches are upsets (class imbalance challenge)

### 4. Models Tested

- **Logistic Regression** (baseline, most interpretable)
- **Random Forest** (moderate complexity, good balance)
- **XGBoost** (best performance, highest complexity)

Each model tested with:
- Baseline (no imbalance handling)
- Class weights
- SMOTE (Synthetic Minority Oversampling)
- Threshold tuning (F1 and recall optimization)

### 5. Class Imbalance Handling

**Challenge:** 63.5% favorites win vs 36.5% upsets

**Solutions Implemented:**
1. **Class Weights:** Penalize minority misclassification (+17% recall)
2. **SMOTE:** Create synthetic upset examples (+15% recall, +6.5% F1) ⭐
3. **Threshold Tuning:** Adjust classification cutoff (+48% recall max)

**Best Approach:** SMOTE provides optimal precision-recall balance

### 6. Evaluation Metrics

- **F1-Score** (primary): Harmonic mean of precision and recall
- **Recall** (primary): Upset detection rate (minimize false negatives)
- **Precision:** Accuracy of upset predictions
- **ROC-AUC:** Overall discrimination ability
- **PR-AUC:** Precision-recall area under curve
- **Confusion Matrix:** Detailed error analysis

---

## 📈 Results & Key Findings

### Model Performance
- **Best Overall:** XGBoost + SMOTE (F1: 0.751, Recall: 0.850)
- **Best Recall:** Random Forest + Threshold Tuning (Recall: 0.981, but Precision: 0.422)
- **Most Interpretable:** Logistic Regression (F1: 0.683)

### Upset Patterns Discovered

**Upsets are MOST likely when:**
1. ✅ Underdog has MORE experience or recent matches
2. ✅ Underdog in better recent form (90-day window)
3. ✅ Match surface favors underdog (surface advantage)
4. ✅ Career performance gap is smaller
5. ✅ Underdog comfortable on current surface

**Example Upset Scenario:**
```
Match: Favorite (Rank 15) vs Underdog (Rank 45)
Surface: Clay

Favorite:  150 career matches, 3 recent matches, 55% win rate (90d), Hard court specialist
Underdog:  200 career matches, 10 recent matches, 75% win rate (90d), Clay specialist

→ High Upset Probability! ✅
  (Experience advantage + Form advantage + Surface advantage)
```

### Feature Importance Rankings

**By Category:**
1. **Experience & Activity** (avg correlation: 0.466)
2. **Career Performance** (avg correlation: 0.420)
3. **Recent Form** (avg correlation: 0.346)
4. **Surface Specialization** (avg correlation: 0.335)
5. **Rankings** (avg correlation: 0.200)
6. **Head-to-Head** (avg correlation: 0.130)

---

## 📂 Repository Structure

```
COMP. DATA ANALYSIS PROJECT/
│
├── 📊 Data/
│   ├── Atp_Matches_Folder/              # Raw ATP match data (2016-2024)
│   ├── Cleaned_engineered_Data_Atp_matches/  # Processed data
│   └── Featured_Data_Atp_matches/       # Final feature-engineered datasets
│
├── 📓 Notebooks/
│   ├── data_cleaning.ipynb              # Data preprocessing and cleaning
│   ├── feature_engineering.ipynb        # Feature creation (83 features)
│   ├── featuring.ipynb                  # Feature engineering pipeline
│   ├── data_modeling.ipynb              # Baseline model training & evaluation
│   ├── model_calibration&class_imbalance.ipynb  # Imbalance handling
│   └── results.ipynb                    # ⭐ Comprehensive results summary
│
├── 📄 Results/
│   ├── all_models_comparison.csv        # Baseline model metrics
│   ├── class_imbalance_techniques_comparison.csv  # Full technique comparison
│   └── class_imbalance_summary.csv      # Improvement summary
│
├── 📝 Documentation/
│   ├── README.md                        # This file
│   ├── PROJECT_SUMMARY.md               # Detailed findings and insights
│   └── PRESENTATION_GUIDE.md            # Quick reference for presentations
│
└── 📋 Requirements/
    └── requirements.txt                 # Python dependencies
```

---

## 🚀 Getting Started

### Prerequisites
```bash
Python 3.8+
Jupyter Notebook
```

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/tennis-upset-prediction.git
cd tennis-upset-prediction
```

2. **Install dependencies**
```bash
pip install -r requirements.txt
```

Required packages:
- pandas
- numpy
- scikit-learn
- xgboost
- imbalanced-learn (for SMOTE)
- matplotlib
- seaborn
- openpyxl (for Excel files)

3. **Run the notebooks**
```bash
jupyter notebook
```

### Quick Start

1. **View Results:** Open `results.ipynb` for comprehensive analysis
2. **Explore Data:** Check `data_cleaning.ipynb` for preprocessing steps
3. **Understand Features:** Review `featuring.ipynb` for feature engineering
4. **Model Training:** See `data_modeling.ipynb` for baseline models
5. **Imbalance Handling:** Study `model_calibration&class_imbalance.ipynb`

---

## 💡 Use Cases

### ✅ Recommended Applications
- **Sports Analytics Dashboards:** Display upset risk scores
- **Broadcasting:** Pre-match storylines and graphics
- **Coaching:** Data-driven opponent preparation
- **Research:** Pattern discovery in competitive sports
- **Education:** ML case study for imbalanced classification

### ❌ Not Recommended
- Guaranteed betting strategies (randomness + house edge)
- Real-time in-match predictions (need different features)
- Individual match certainty claims (inherent unpredictability)

---

## ⚠️ Limitations

### Data Limitations
- **Missing variables:** Injuries, motivation, weather, crowd support
- **Limited H2H data:** 64.7% of matches are first meetings
- **Static features:** Cannot capture in-match momentum
- **Historical bias:** Past patterns may not predict future trends

### Model Limitations
- **Performance ceiling:** ~75-80% F1-score likely maximum due to sport's randomness
- **Interpretability:** XGBoost requires SHAP/LIME for explanations
- **Threshold tradeoffs:** High recall sacrifices precision (or vice versa)
- **Temporal dynamics:** Player performance changes over time

### Fundamental Limitations
- **Sports unpredictability:** Human factors, luck, psychology hard to quantify
- **Correlations ≠ Causation:** Models identify patterns, not causes
- **Context matters:** Tournament stage, prize money, motivation vary

---

## � Future Work

### Short-term Enhancements
- [ ] Integrate betting odds as features
- [ ] Add player-specific models for top ATP players
- [ ] Implement SHAP values for prediction explanations
- [ ] Create interactive dashboard for upset probability

### Medium-term Goals
- [ ] Deep learning models (LSTM for temporal patterns)
- [ ] Real-time prediction with in-match statistics
- [ ] Social media sentiment analysis
- [ ] Weather and venue data integration

### Long-term Vision
- [ ] Multi-sport extension (basketball, soccer, golf)
- [ ] Causal inference analysis (why upsets happen)
- [ ] Automated model retraining pipeline
- [ ] Production API for upset predictions

---

## 📚 References & Citations

### Data Source
- **Jeff Sackmann's Tennis ATP Repository:** https://github.com/JeffSackmann/tennis_atp

### Methodology References
- SMOTE: Chawla et al. (2002) - "SMOTE: Synthetic Minority Over-sampling Technique"
- XGBoost: Chen & Guestrin (2016) - "XGBoost: A Scalable Tree Boosting System"
- Class Imbalance: He & Garcia (2009) - "Learning from Imbalanced Data"

### Related Research
- Tennis prediction using machine learning (Various sports analytics papers)
- Imbalanced classification techniques in sports analytics
- Feature importance in ensemble methods

---

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Areas for Contribution
- Additional feature engineering ideas
- Alternative modeling approaches
- Visualization improvements
- Documentation enhancements
- Code optimization

---

## 👨‍💻 Author

**Manuel Rodriguez Berdud**
- GitHub: https://github.com/Manurguez03
- Email: manuelrguezberdud@outlook.com
