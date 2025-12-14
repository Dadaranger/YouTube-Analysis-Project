# YouTube Metadata Analysis and Modeling

## Part 1: Temporal, Textual, and Machine Learning Analysis of Trending Videos

---

## 1. Introduction

This report documents the first phase of a two-part capstone project on YouTube performance prediction. Part 1 focuses on metadata-driven analysis and modeling using a large collection of YouTube Trending videos. The objective is to understand how far one can go using only upload-time metadata—titles, descriptions, posting time, channel status, and related fields—before introducing semantic models and visual information in Part 2.

### Core Research Goals

1. **Characterize Dataset Structure**: Prepare the trending-video metadata dataset for analysis
2. **Quantify Performance Drivers**: Analyze how posting time, title/description design, punctuation, and sentiment relate to view performance
3. **Build Predictive Models**: Create statistical and machine learning models that score metadata strength relative to other trending videos
4. **Identify Limitations**: Clarify the structural boundaries of metadata-only modeling and motivate the shift to multimodal deep learning in Part 2

**Important**: Because the dataset consists only of trending videos, all results represent comparisons within a successful subset, not as universal rules for all uploads.

---

## 2. Data and Preprocessing

### 2.1 Data Source and Scope

The dataset contains **13,018 YouTube Trending videos** collected between **October 3, 2023** and **September 15, 2025**.

Each record includes:

- **Video-level metadata**: videoId, title, description, duration, view count
- **Channel-level indicators**: channel verification status
- **Format annotation**: Short vs. long-form classification
- **Timestamps**: Publication time in UTC, converted to Eastern Time
- **Additional fields**: Tags and thumbnail information

The initial ingestion combined multiple CSV exports into **54,206 rows × 24 columns** before deduplication and filtering to 13,018 unique trending videos.

### 2.2 Time Normalization and Column Cleanup

**Timestamp Processing**:
1. Parsed from Zulu time format (e.g., `2024-07-03T19:59:03.038Z`)
2. Converted to localized Eastern Time (EST/EDT)
3. Derived structured temporal features:
   - `published_date` (date only)
   - `published_time` (time only)
   - `hour_et` (0–23)
   - `dow_et` (day-of-week label)

**Quality Checks**:
- ✓ published_date and published_time: 13,018 valid values (100%)
- ✓ hour_et: range 0–23 with no missing values
- ✓ dow_et: all seven days represented

**Removed Columns** (display-oriented, redundant):
- `publishedText`, `viewsText`, `durationText`
- `creatorOnRise`
- Raw thumbnail container metadata

**Result**: ~28% reduction in columns and file size without loss of analytical content.

### 2.3 Data Quality and Deduplication

| Issue | Count | Resolution |
|-------|-------|-----------|
| Missing descriptions | 1,107 (8.5%) | Filled with empty strings |
| Missing views | 1 | Dropped anomalous row |
| Missing/unparsable duration | 765 (5.9%) | Retained; handled in modeling |
| Duplicate videoIds | Multiple | Retained latest record per videoId |

**Deduplication Strategy**: After sorting by videoId and timestamp, retained records with valid publication dates. Result: 13,018 unique videos.

### 2.4 Target Transformation: Log Views

**Raw View Distribution**:
- Mean/median ratio: **3.56×** (highly skewed by viral outliers)
- Extremely right-skewed distribution

**Solution**: Adopted **log1p(views)** as the primary performance metric

**Comparison of Approaches**:
- Log mean/median ratio: **~0.95×** (close to symmetric)
- Overlap in "optimal posting times" between raw vs. log: **~40%**
  
**Interpretation**: Modeling based on raw counts is highly sensitive to extreme outliers, whereas log-transformed target produces stable, defensible estimates.

✓ **Decision**: All subsequent analyses use **log1p(views)** as the main outcome variable.

---

## 3. Temporal Analysis of Posting Time

### 3.1 Analysis Design

**Primary Metric**: Median log1p(views) in each day-of-week × hour-of-day cell (robust to outliers)

**Secondary Metric**: 75th percentile log1p(views) (stronger performers in each window)

**Data Stratifications**:
- All videos: 13,017 rows
- **By Format**: Shorts (590) vs. Long-form (12,427)
- **By Verification**: Verified (9,269) vs. Unverified (3,748)
- **Cross-splits**: Shorts-Verified, Shorts-Unverified, Long-Verified, Long-Unverified

**Pivot Table Structure**: Rows = day-of-week, Columns = hour-of-day, Cells = median log1p(views), 75th percentile, sample count

**Stability Criterion**: Only cells with n ≥ 20 videos included in ranking and "best" window extraction.

### 3.2 Global Patterns

**Strongest Time Slot Across All Videos**:
- **Saturday at 3 PM Eastern Time**
  - Median views: ~961,000
  - Sample size: ~385 videos

**Key Finding**: Weekend afternoons, especially Saturdays around mid-afternoon, consistently strong across multiple stratifications. Thursday 3 PM ET emerged as a leading weekday alternative for long-form content.

### 3.3 Format-Specific Behavior: Shorts vs. Long-Form

**Long-Form Content**:
- Peak performance: Weekend afternoons (Saturday 3–4 PM ET)
- Weekday alternative: Thursday 3 PM ET
- Strategy: Afternoon/evening publication aligns with viewing patterns

**Shorts**:
- Distinct advantage: Early morning hours
- Peak: **Sunday at 6 AM ET**
- Interpretation: Short-form benefits from early-day, low-competition windows when users browse casually

**Implication**: Posting strategies must differ by format.

### 3.4 Verified vs. Unverified Channels

**Verified Channels**:
- Multiple strong time windows with high baseline performance
- Broader temporal flexibility
- Reason: Established subscriber base and algorithmic favor reduce timing dependency

**Unverified Channels**:
- Pronounced spikes and troughs
- Clearest favorable windows: Afternoon slots (Thursday, Saturday, Sunday, 3–4 PM ET)
- Implication: Developing channels gain more from careful timing choices

### 3.5 Summary of Temporal Analysis

✓ Posting time is a meaningful signal, especially for unverified channels and Shorts
✓ Weekend and afternoon windows strong for long-form; early morning optimal for Shorts
✓ Temporal strategies should adapt to both format and channel authority

**Application**: Time-related features engineered for later ML models (hour_et, day-of-week, prime-time flags).

---

## 4. Text Analysis: Titles, Descriptions, and Sentiment

### 4.1 Title and Description Length

**Features Engineered**:
- `title_len_chars`, `title_len_words`
- `desc_len_chars`, `desc_len_words`

**Title Statistics**:
| Statistic | Value |
|-----------|-------|
| Mean | ~52 characters |
| Median | ~48 characters |
| IQR | 37–66 characters |
| Maximum | ~100 characters (YouTube limit) |

**Description Statistics**:
- Much wider distribution
- Reflects diverse creator strategies

**Correlation with Log Views**:
| Feature | Correlation | Interpretation |
|---------|-------------|-----------------|
| Description length | **+0.72** | Strong positive |
| Title length | **-0.10** | Weak negative |

**Key Insight**: Among trending videos, longer and more informative descriptions associate with higher performance, while longer titles do not and may even underperform.

### 4.2 Punctuation and Structural Signals

**Features Derived from Titles**:
- `has_number`: Contains any digits
- `has_question`: Contains "?"
- `has_exclamation`: Contains "!"
- `has_emoji`: Contains emoji characters

**Findings**:

| Feature | Correlation | Interpretation |
|---------|-------------|-----------------|
| Numbers | **+0.08** | Modest advantage (e.g., "5 Facts") |
| Exclamation marks | **Negative** | Heavy use correlates with lower views |
| Question marks | **Negative** | Excessive use correlates lower |
| Emojis | **-0.26** | Negative correlation with performance |

**Interpretation**: While punctuation works in some niches, for information-focused content in this dataset, overly emotional signaling appears counterproductive.

### 4.3 Sentiment Analysis

**Method**: VADER sentiment analyzer
- Continuous sentiment scores
- Discretized buckets: negative, neutral, positive

**Results**:

| Sentiment | Avg log1p(views) | Interpretation |
|-----------|-----------------|-----------------|
| Neutral | **13.21** | Highest performance |
| Positive | **12.93** | Lower than neutral |
| Negative/Curiosity | Competitive | Effective for factual content |

**Pattern**: Neutral, matter-of-fact tones tend to associate with higher performance than overtly positive, hype-driven language.

### 4.4 Keyword-Level Observations

**Methods**:
- Stopword removal
- Word-frequency counts
- Word clouds comparing higher vs. lower performers

**Key Pattern**: Topic-specific, concrete nouns and phrases (astrophysical objects, specific events, mission names) are more common among higher performers. Generic hype words and vague phrases are prevalent among weaker performers.

---

## 5. Feature Engineering for Modeling

The modeling pipeline combined temporal and textual insights into a structured feature set.

### Temporal Features
- `hour_et` (integer and cyclic: hour_sin, hour_cos)
- Day-of-week (numeric encoding)
- Boolean flags: `is_weekend`, `is_prime_time`, `is_optimal_time`

### Text Features
- `title_len_chars`, `title_len_words`
- `desc_len_chars`, `desc_len_words`
- `title_desc_ratio` (title chars / description chars + 1)

### Punctuation Features
- Numeric: `has_number`, `has_question`, `has_exclamation`, `has_emoji`

### Channel and Format Features
- `verified` (True/False)
- `isShort` (True/False)
- `duration_sec` and derived length-based bins

**Total**: ~20–25 engineered features, directly interpretable and linked to creator decisions.

---

## 6. Machine Learning Models

### 6.1 Train–Test Split

- **Split ratio**: 80/20
- **Stratification**: Preserved overall distribution of key variables and target
- **Consistency**: Same split used across all experiments for comparability

### 6.2 Regression Baseline: Ridge Regression

**Purpose**:
1. Quantify variance explained by linear model
2. Provide reference for tree-based models

**Results**:
| Metric | Value |
|--------|-------|
| Train R² | ~0.65 |
| Test R² | ~0.66 |

**Interpretation**: Metadata carries meaningful signal, but substantial nonlinear structure exists.

### 6.3 Tree-Based Regression: Random Forest and LightGBM

#### Random Forest Regression
- Parameters: 150 trees, depth limit, minimum samples per leaf
- Train R²: **~0.86**
- Test R²: **~0.86**
- Outcome: Balanced generalization with minimal overfit

#### LightGBM Regression (Winner)
| Metric | Value |
|--------|-------|
| Train R² | **~0.91** |
| Test R² | **0.8933** |
| Test MAE | **0.8133** (log units) |
| Test RMSE | **1.0890** |
| Generalization gap | **~0.0178** |

**Selection Rationale**: LightGBM outperformed competitors on out-of-sample R² with minimal overfit.

#### Error Analysis by Stratification
- **Shorts**: R² > 0.94
- **Long-form**: R² ≈ 0.77
- Error distributions vary between verified and unverified channels

### 6.4 Regression Feature Importance

**Top Drivers of Predictions**:
1. Video duration
2. Title-to-description length ratio
3. Title character length
4. Description word count
5. Day-of-week and hour features

**Key Finding**: Top 10 features account for ~90% of predictive power. Description-related features and length ratios are consistently prominent.

### 6.5 Classification: Top-Performer Probability

**Task**: Predict membership in "top performer" group (top 25% by log1p(views))

**Evaluation Metric**: AUC-ROC (primary), F1-score, precision–recall

**Baseline (naive majority classifier)**:
- Accuracy: ~75%
- AUC-ROC: 0.50 (demonstrating need for better metrics)

#### Candidate Models Comparison

| Model | Test AUC-ROC | Test F1 | Status |
|-------|--------------|---------|--------|
| **Random Forest** | **0.7634** | **0.523** | ✓ Selected |
| Logistic Regression | Lower | Lower | Rejected |
| XGBoost | Higher overfit | Lower | Rejected |
| LightGBM Classifier | Overfits | - | Rejected |

**Selection Rationale**: Random Forest achieved best balance of AUC-ROC, F1-score, train–test consistency, and training time (<1 second).

---

## 7. Metadata Quality and Trend-Readiness Scoring

### Two-Model Framework

#### Metadata Quality Score (0–100)
- **Source**: LightGBM regression model
- **Method**: Map predicted log1p(view) percentiles to 0–100 scale based on training distribution
- **Interpretation**: Estimated performance strength relative to trending population

#### Trend-Readiness Probability (%)
- **Source**: Random Forest classifier
- **Method**: Probability of falling into high-performing group (top quartile)
- **Interpretation**: Likelihood that video's metadata profile matches successful patterns

### Joint Interpretation Scheme

| Quality | Readiness | Interpretation |
|---------|-----------|-----------------|
| High | Low | Structurally sound but saturated/competitive topic |
| Moderate | High | Niche/topic advantage offsetting metadata gaps |
| High | High | Strong metadata in favorable niche |
| Low | Low | Multiple improvement opportunities |

### Demonstration Example

**Scenario**: Candidate video with baseline posting time, title, and description

**Baseline Scores**:
- Metadata Quality: ~40/100
- Trend-Readiness: Moderate

**Proposed Improvements**:
1. Strengthen description (longer, more informative)
2. Add numeric structure to title
3. Align posting time with empirically strong windows

**Updated Scores**:
- Metadata Quality: ~60/100
- Trend-Readiness: Improved

**Conclusion**: Concrete demonstration of how modeling outputs translate into pre-publication decision support.

---

## 8. Synthesis and Interpretation

### Key Findings

✓ **Metadata carries substantial signal**: Engineered features explain large fraction of log-view variance among successful videos

✓ **Description design is paramount**: Richer, complete descriptions consistently associate with better performance and receive highest model importance

✓ **Title structure matters more than length**: Concise, informative titles with occasional numbers outperform longer, vague, or overly emotional alternatives

✓ **Timing strategy especially important for underdog creators**: Unverified channels and Shorts benefit most from careful timing; timing can partially compensate for weaker channel authority

✓ **Sentiment and punctuation secondary but meaningful**: Neutral/factual tone and restrained punctuation align better with trending behavior than exaggerated language and emojis

✓ **Channel authority remains influential**: Verified channels show more flexibility and higher baseline performance across time windows—metadata alone cannot overcome this

### Practical Implications

The **LightGBM regression model** and **Random Forest classifier** provide workable tools for:
- Relative metadata benchmarking
- Answering "How competitive is this metadata compared to other trending videos?"
- Assessing similarity to top-performer patterns

---

## 9. Limitations and Motivation for Part 2

### Key Constraints of Part 1

1. **Selection Bias**
   - Dataset includes only trending videos
   - Models learn distinctions among successful videos, not between successful and unsuccessful
   - Results valid within trending distribution only

2. **Metadata-Only Perspective**
   - No access to video content, thumbnail semantics, audio
   - Many user preference and ranking aspects unobservable
   - Misses critical visual and semantic signals

3. **Temporal Drift**
   - Analysis spans 2023–2025
   - YouTube's recommendation system and user behavior change over time
   - Models require periodic retraining for accuracy

4. **No User-Level Personalization**
   - Individual viewing histories and preferences not modeled
   - Regional differences not captured
   - Results reflect aggregate video-level patterns only

### Motivation for Part 2

These limitations motivate the next phase, which shifts from metadata-only modeling on trending data to **multimodal deep learning** on a broader YouTube Shorts dataset:

- **SentenceTransformer embeddings**: Semantic representations of titles and descriptions
- **CLIP embeddings**: Visual semantics from thumbnail images
- **Integrated temporal features**: Time signals embedded in neural architecture
- **Enhanced interpretability**: Analyze semantic and visual influences on performance

---

## 10. Conclusion

Part 1 of this capstone project demonstrates that **careful analysis of metadata alone can yield meaningful insights** into YouTube performance within the trending ecosystem.

### Accomplishments

✓ **Robust methodology**: Systematic data cleaning, temporal analysis, text feature engineering, sentiment analysis

✓ **Clear empirical patterns**: Identified optimal posting times by format and channel authority

✓ **Strong evidence base**: Description quality and length are central performance levers

✓ **Practical framework**: Two-model system for scoring metadata quality and trend-readiness

### Path Forward

Part 1 clarifies the boundaries of metadata-only modeling and establishes the foundation for Part 2, which leverages deep learning and multimodal representations to move beyond metadata into the actual **semantics and visuals** of YouTube content.

---

**Project Status**: Part 1 Complete | Part 2: Multimodal Deep Learning (Ongoing)

