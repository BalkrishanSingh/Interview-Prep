# Module 05: GenAI & ML — Machine Learning Core Concepts

---

## 1. Primary Machine Learning Paradigms

```
┌────────────────────────────────────────────────────────────────────────┐
│                        MACHINE LEARNING PARADIGMS                      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
    ┌───────────────────────────────┼───────────────────────────────┐
    ▼                               ▼                               ▼
┌───────────────────────┐ ┌───────────────────────┐ ┌───────────────────────┐
│  SUPERVISED LEARNING  │ │ UNSUPERVISED LEARNING │ │REINFORCEMENT LEARNING │
├───────────────────────┤ ├───────────────────────┤ ├───────────────────────┤
│ • Labeled input data  │ │ • Unlabeled data      │ │ • Agent & Environment │
│   {(x_i, y_i)}        │ │ • Discover hidden     │ │ • State, Action,      │
│ • Regression &        │ │   patterns / clusters │   Reward feedback loop  │
│   Classification      │ │ • K-Means, PCA, GMM   │ │ • Q-Learning, PPO     │
└───────────────────────┘ └───────────────────────┘ └───────────────────────┘
```

---

## 2. The Bias-Variance Tradeoff

The prediction error of any supervised machine learning model decomposes into three mathematically distinct components:

$$\text{Total Expected Error} = \text{Bias}^2 + \text{Variance} + \sigma^2 \text{ (Irreducible Error)}$$

```
Error
  ▲
  │ \                         /  Total Test Error
  │  \       Optimal         /
  │   \     Complexity      /   Variance (Overfitting)
  │    \        ▼          /   /
  │     \═══════●═════════/═══/
  │      \               /
  │       \             /
  │        \___________/________ Bias (Underfitting)
  └────────────────────────────────────────► Model Complexity
```

### 2.1 Underfitting (High Bias)
- **Characteristics**: Model makes rigid, overly simplistic assumptions about data distribution (e.g., fitting a straight linear line to complex quadratic data).
- **Diagnostics**: High training error and high validation/test error.
- **Remedies**: Increase model capacity, engineer more polynomial features, reduce regularization strength.

### 2.2 Overfitting (High Variance)
- **Characteristics**: Model memorizes noise and random fluctuations in the training dataset rather than learning the general underlying function.
- **Diagnostics**: Near-zero training error, but high validation/test error (poor generalization).
- **Remedies**: Collect more training data, apply regularization ($L_1 / L_2$), introduce dropout, reduce feature dimensionality.

---

## 3. Regularization Techniques ($L_1$ vs. $L_2$)

Regularization adds a penalty parameter to the loss function to constrain model weights $w$:

$$\mathcal{L}_{\text{reg}}(w) = \mathcal{L}_{\text{data}}(w) + \lambda \cdot \Omega(w)$$

| Feature | $L_1$ Regularization (Lasso) | $L_2$ Regularization (Ridge) |
| :--- | :--- | :--- |
| **Penalty Term** | $\Omega(w) = \sum_{i=1}^d \|w_i\|$ | $\Omega(w) = \sum_{i=1}^d w_i^2$ |
| **Weight Effect** | Drives non-essential feature weights to **exact zero** ($w_i = 0$). | Shrinks weights asymptotically toward zero, but never forces exact zeros. |
| **Feature Selection** | **Performs automatic feature selection** (generates sparse models). | Retains all features; handles multicollinearity by distributing weights. |
| **Computational Geometry** | Diamond-shaped constraint contour (corners hit axes). | Circular / spherical constraint contour. |

---

## 4. Classification Evaluation Metrics

Evaluating a classification model using simple **Accuracy** ($\frac{\text{Correct}}{\text{Total}}$) is dangerous when class distributions are imbalanced (e.g., in fraud detection where 99.9% of transactions are legitimate).

### 4.1 The Confusion Matrix
```
                       Actual Positive         Actual Negative
Predicted Positive   [ True Positive (TP)  ] [ False Positive (FP) (Type I Error) ]
Predicted Negative   [ False Negative (FN) ] [ True Negative (TN)  (Type II Error)]
                       (Type II Error)
```

### 4.2 Precision, Recall & $F_1$-Score

$$\text{Precision} = \frac{\text{TP}}{\text{TP} + \text{FP}} \qquad \text{Recall (Sensitivity)} = \frac{\text{TP}}{\text{TP} + \text{FN}}$$

$$F_1\text{-Score} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}} = \frac{2\text{TP}}{2\text{TP} + \text{FP} + \text{FN}}$$

- **When to Maximize Precision**: When the cost of a **False Positive** is catastrophic (e.g., Spam classification where marking a critical work email as spam loses business).
- **When to Maximize Recall**: When the cost of a **False Negative** is fatal (e.g., Medical disease diagnosis or fraud detection where missing a true case has severe real-world consequences).
- **$F_1$-Score**: The harmonic mean of Precision and Recall. Balances both metrics, penalizing extreme skews.

### 4.3 ROC Curve & AUC
- **Receiver Operating Characteristic (ROC)**: Plots **True Positive Rate (Recall)** against **False Positive Rate ($\frac{\text{FP}}{\text{FP} + \text{TN}}$)** across all classification probability thresholds ($0.0 \to 1.0$).
- **AUC (Area Under the Curve)**: Measures threshold-independent classification capability:
  - $\text{AUC} = 1.0$: Flawless classifier.
  - $\text{AUC} = 0.5$: Baseline performance identical to random coin flipping.
