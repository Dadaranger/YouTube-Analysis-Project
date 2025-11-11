# Multimodal Deep Learning Solution for YouTube Shorts Winner Prediction

## Executive Summary

This document describes an end-to-end deep learning solution to predict whether a YouTube Short will be promoted by the platform (a "winner"). I reformulate the problem from failed view-count regression to a binary classification task based on a product-aligned threshold (≥ 1,000 views = winner). The solution ingests multimodal inputs (temporal metadata, text, and thumbnail embeddings), fuses them via specialized encoders, and trains a classifier with modern optimization techniques (label smoothing, AdamW, LR warmup with cosine annealing, gradient clipping, early stopping). I evaluate with standard classification metrics and provide publication-grade visualizations for architecture, training dynamics, and performance.

Exact metric values and confusion matrices are rendered directly by the notebook (see evaluation and comparison figures and printed tables). This document focuses on design rationale, architecture details, training procedure, evaluation methodology, and lessons learned.

## Problem Framing

- Objective: Predict if a Short will be a “winner”.
- Target definition: Binary label from view counts.
  - Winner (1): views ≥ 1,000
  - Non-winner (0): views < 1,000
- Motivation for classification: Direct regression on views exhibited very low explanatory power (R² ~ 0), unstable errors, and a heavy-tailed distribution resistant to standard transforms. A product-relevant threshold provides a more actionable and learnable objective, aligns with platform dynamics, and supports downstream decisioning (e.g., A/B testing and upload prioritization).

## Data and Features

- Total feature dimensionality (concatenated): 1,670
  - Temporal features (6): e.g., upload hour/day/month and simple recency signals
  - Title embeddings (384): SentenceTransformer (all-MiniLM-L6-v2), sentence-level
  - Tag embeddings (384): SentenceTransformer (all-MiniLM-L6-v2), aggregated over tags
  - Description embeddings (384): SentenceTransformer (all-MiniLM-L6-v2), sentence-level
  - Thumbnail embeddings (512): CLIP image encoder on the video thumbnail
- Train/test split: Stratified split to preserve the winner/non-winner ratio.
- Scaling: Standardization for numeric temporal features; embeddings used as-is.
- Class balance: Imbalance handled via class weights in the loss.

## Baseline vs. Enhanced Approaches

### Why regression was retired

- Strong skew and heavy tails in views impaired signal capture.
- The actual view of a video is impacted by multiple factors that our dataset do not capture, such as seasonality, user behavior, serach, etc.
- Multiple regression baselines (direct and log-transformed) produced near-random fit with poor calibration.
- Reformulating to a binary target surfaced a learnable boundary and improved stability.

### Baseline model: MultimodalWinnerClassifier

- Per-modality MLP encoders with ReLU+Dropout
- Simple fusion by concatenation
- 3-layer classifier head [512 → 256 → 128 → 2]
- Class-weighted CrossEntropy, Adam, ReduceLROnPlateau

### Enhanced model: EnhancedWinnerClassifier

- Modality encoders (2 layers each):
  - Temporal: 6 → 64 → 128
  - Title: 384 → 256 → 192
  - Tags: 384 → 256 → 192
  - Description: 384 → 256 → 192
  - Thumbnail: 512 → 256 → 192
- Fusion: Concatenate → Linear to 768 + LayerNorm
- Two residual processing blocks at 768-dim with skip connections
- Deep classifier: 768 → 512 → 384 → 256 → 128 → 2
- Regularization: Dropout ≈ 0.35 throughout
- Normalization: LayerNorm in encoders and fusion
- Total parameters: ~1.2M trainable (approximate)

The enhanced design improves representation capacity, gradient flow, and optimization smoothness while staying compact enough for the dataset size.

## Training Configuration and Rationale

- Loss: LabelSmoothingCrossEntropy (ε = 0.1) with class weights
  - Reduces overconfidence and improves generalization
- Optimizer: AdamW (lr ≈ 1e-3, weight decay 1e-4)
- Scheduler: Warmup (first ~10 epochs) followed by cosine annealing over total steps
- Gradient clipping: Max-norm 1.0 to stabilize updates
- Early stopping: Patience ≈ 25 epochs on validation loss
- Batch size: Tuned for hardware; typical minibatches in the tens range
- Epochs: Configured up to 200; early stopping often halts earlier
- Device: CUDA if available; CPU fallback supported

## Evaluation Methodology

- Splits: Stratified train/test split to preserve class ratios
- Metrics: Accuracy, Precision, Recall, F1-Score, ROC-AUC, Average Precision (AUPRC)
- Curves and matrices:
  - Confusion matrix for class-specific errors
  - ROC and Precision-Recall curves for threshold-agnostic behavior
  - Learning curves (loss and accuracy) and learning rate schedule for training dynamics
- Reporting: The notebook renders a comprehensive 7-panel evaluation figure and a side-by-side baseline vs. enhanced comparison figure with annotated metrics and AUCs.

Refer to the notebook’s evaluation cells for exact metric values and the artifacts:
- Enhanced evaluation figure (multi-panel)
- Model comparison figure: baseline vs enhanced
- Printed table of metric values and relative improvements

## Results Overview

- The enhanced model demonstrates improved separability on ROC/PR curves compared to the baseline.
- Confusion matrix patterns indicate better true-positive discovery without disproportionate false positives relative to baseline.
- Training dynamics show stable convergence under warmup+cosine scheduling with label smoothing.
- Exact Accuracy/Precision/Recall/F1/ROC-AUC values are logged by the notebook and saved in the figures and printed tables.

## Error Analysis and Thresholding

- The 1,000-view threshold is product-driven; depending on funnel economics, an alternative threshold can be selected to optimize Precision or Recall.
- Threshold sweeps (varying the decision threshold on predicted probability) can surface operating points for:
  - High-precision scenarios (limited promotion budget)
  - High-recall scenarios (exploration or growth)
- Examine confusion matrices across thresholds to quantify trade-offs and choose business-optimal operating points.

## Practical Challenges and Solutions

1. Regression underperformance
   - Challenge: View distributions were heavy-tailed; regression produced near-random fit.
   - Solution: Reformulated as binary classification with a clear, actionable threshold.

2. Multimodality alignment and capacity
   - Challenge: Heterogeneous inputs (text/image/temporal) with different scales and distributions.
   - Solution: Dedicated per-modality encoders with LayerNorm and dropout; residual fusion blocks to ease optimization.

3. Optimization stability and generalization
   - Challenge: Overconfidence and overfitting risks.
   - Solution: Label smoothing, AdamW with weight decay, warmup+cosine scheduling, gradient clipping, early stopping.

4. Class imbalance
   - Challenge: Winner class underrepresentation can bias decision boundary.
   - Solution: Class-weighted loss and stratified split; recommend further calibration/threshold tuning.

5. Visualization quality and communication
   - Challenge: Early diagrams had overlapping labels, unclear arrows, and insufficient spacing; evaluation plots were crowded.
   - Solution: Rebuilt architecture diagram with professional palette, tight arrow connections, consistent spacing, and readable annotations; redesigned evaluation into a clear 7-panel layout with legible fonts and labeled bars.

## Reproducibility and Artifacts

- Code: All logic consolidated in the notebook with clearly separated cells for data prep, modeling, training, and evaluation.
- Environment: `requirements.txt` lists dependencies; use a pinned virtual environment for reproducibility.
- Random seeds: Set a global seed to reduce variance across runs (RNG for NumPy/PyTorch).
- Artifacts saved by the notebook:
  - Architecture diagram (high-resolution PNG)
  - Enhanced evaluation figure (multi-panel, PNG)
  - Model comparison figure (baseline vs enhanced, PNG)
  - Printed performance table in the notebook output

## Recommendations and Next Steps

- Threshold and calibration
  - Sweep decision thresholds; optionally apply Platt scaling or temperature scaling for calibrated probabilities.
- Ablations
  - Remove modalities or encoder depth to quantify contribution; test residual blocks on/off; vary dropout.
- Feature engineering
  - Add richer temporal signals (seasonality, trend), channel-level priors, and content structure cues.
- Hyperparameter optimization
  - Structured sweeps (learning rate, weight decay, dropout, smoothing ε, hidden sizes) via grid/BO methods.
- Cross-validation
  - K-fold or repeated stratified splits to improve estimate stability.
- Ensembling
  - Average over multiple seeds/checkpoints; blend with gradient-boosted trees on top of deep features.
- Interpretability
  - Token- and pixel-level attribution (e.g., Grad-CAM for thumbnails, attention/SHAP for text) to inform content strategy.

## Appendix A: Model Shapes and Parameters

- Inputs
  - Temporal: 6
  - Title: 384 (SentenceTransformer all-MiniLM-L6-v2)
  - Tags: 384 (SentenceTransformer all-MiniLM-L6-v2)
  - Description: 384 (SentenceTransformer all-MiniLM-L6-v2)
  - Thumbnail: 512 (CLIP image)
- Encoders
  - Temporal: 6 → 64 → 128
  - Title: 384 → 256 → 192
  - Tags: 384 → 256 → 192
  - Description: 384 → 256 → 192
  - Thumbnail: 512 → 256 → 192
- Fusion and Residual Processing
  - Concat → Linear to 768 → LayerNorm
  - Residual block ×2 at 768-dim
- Classifier Head
  - 768 → 512 → 384 → 256 → 128 → 2
- Regularization
  - Dropout ≈ 0.35; LayerNorm throughout
- Parameter count
  - Baseline: ~461K
  - Enhanced: ~1.2M (approx.)

## Appendix B: Training Hyperparameters

- Loss: LabelSmoothingCrossEntropy (ε = 0.1) with class weights
- Optimizer: AdamW (lr ≈ 1e-3, weight decay 1e-4)
- Scheduler: Warmup (≈10 epochs) → cosine annealing
- Gradient clipping: 1.0
- Early stopping: patience ≈ 25
- Max epochs: 200 (with early stop)
- Batch size: tuned to hardware constraints

## How to Use This Solution

1. Prepare data to match the expected schema and embedding generation steps (SentenceTransformer all-MiniLM-L6-v2 for text; CLIP for image).
2. Execute cells for data preprocessing and dataset creation.
3. Run the baseline and enhanced model training cells.
4. Execute the evaluation and comparison cells to render metrics and figures.
5. Use the comparison outputs and threshold sweeps to select an operating point that fits business constraints (precision- or recall-oriented).