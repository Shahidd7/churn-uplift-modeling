# 📊 Customer Churn Prediction & Uplift Modeling for Retention Offers

> **Predicting not just who will churn, but who can be saved with a discount.**

An end-to-end machine learning project that goes beyond traditional churn prediction by using **Uplift Modeling** to identify customers whose behavior can actually be changed by a retention offer.

---

## 🎯 The Business Problem

Traditional churn models answer:

> **"Who is likely to churn?"**

But this is not necessarily the question a business needs to answer.

Suppose a telecom company identifies 1,000 customers who are likely to churn and sends all of them a discount.

This creates two problems:

### 🔴 Lost Causes

Customers who are going to churn **regardless of whether they receive an offer**.

Giving them a discount wastes marketing budget.

### 🟢 Sure Things

Customers who were going to stay **even without an offer**.

Giving them a discount unnecessarily reduces revenue.

### 🟡 Persuadables

Customers who are likely to stay **because they received the offer**.

These are the customers a retention campaign should ideally target.

This project uses **Uplift Modeling** to identify these Persuadable customers by estimating the **incremental effect of a retention offer**.

```text
Traditional Churn Model
        ↓
Who is likely to leave?
        ↓
Risk Score

Uplift Model
        ↓
Who can actually be influenced?
        ↓
Incremental Treatment Effect
```

---

# 🧠 Key Concepts

## What is Uplift Modeling?

Uplift modeling estimates the difference between two potential outcomes for the same customer:

```text
Probability of staying WITH an offer
                -
Probability of staying WITHOUT an offer
                =
                Uplift
```

A high positive uplift means the customer is more likely to be retained **because of the intervention**.

This shifts the business objective from:

> **"Find customers who are likely to churn."**

to:

> **"Find customers whose behavior we can change."**

---

# 🛠️ Approach

## 1. Data Cleaning & Leakage Prevention

The project uses the **Telco Customer Churn** dataset.

The preprocessing pipeline included:

* Cleaning missing values
* Encoding categorical variables
* Converting features into model-ready numerical representations
* Auditing features for potential data leakage
* Removing post-outcome information such as:

```text
Churn Reason
```

This ensures the model only uses information that would realistically be available **before the retention decision is made**.

---

## 2. Baseline Churn Models

Three classification models were trained to establish a progression from simple to more complex approaches:

### Logistic Regression

Used as an interpretable baseline.

It provides a simple linear relationship between customer features and churn probability.

### Random Forest

Introduced non-linear relationships and feature interactions without requiring extensive feature engineering.

### XGBoost

A gradient-boosting model that builds trees sequentially, with each new tree attempting to correct errors made by previous trees.

XGBoost achieved the highest PR-AUC among the tested churn models.

---

# 📈 Why PR-AUC Instead of Accuracy?

The dataset is imbalanced:

* Approximately **73%** of customers do not churn
* Approximately **27%** of customers churn

A model that predicts:

```text
"No Churn"
```

for every customer would already achieve approximately **73% accuracy** without identifying any churners.

Therefore, accuracy is not a useful standalone metric for this problem.

Instead, the project uses:

### Precision-Recall AUC (PR-AUC)

PR-AUC evaluates how effectively the model identifies the minority class and ranks likely churners.

This makes it more appropriate than accuracy for evaluating an imbalanced churn classification problem.

---

# 🧪 3. Simulating a Retention A/B Test

A true uplift model requires treatment/control data.

Ideally, we would have:

```text
              Customers
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
     Control             Treatment
   No Discount           Discount
        │                   │
        ↓                   ↓
    Churn Rate           Churn Rate
```

However, the public Telco dataset does not contain historical randomized retention-offer data.

Therefore, an **A/B test was simulated** using domain-based assumptions about how different customer segments might respond to an offer.

For example:

* Some customers were modeled as **Persuadables**
* Some as **Lost Causes**
* Some as **Sure Things**
* Customer characteristics such as tenure were incorporated into the simulated treatment effects

This creates a synthetic treatment effect that can be used to evaluate the uplift modeling approach.

---

# 🎲 Simulated A/B Test Results

The simulated experiment produced:

| Group     | Churn Rate |
| --------- | ---------: |
| Control   | **26.06%** |
| Treatment | **22.55%** |

Because treatment assignment was simulated randomly, both groups were designed to have comparable baseline risk.

The simulated retention offer therefore produced a lower observed churn rate in the treatment group.

> **Important:** These are simulated experimental results, not results from a real-world telecom A/B test.

---

# 🤖 4. Uplift Modeling — T-Learner

The project uses a **Two-Model / T-Learner** approach.

Instead of training one model to predict churn, two separate models are trained:

### Model 1 — Control Model

Trained on customers who received **no offer**.

```text
Features → Probability of Churn Without Offer
```

### Model 2 — Treatment Model

Trained on customers who received the **retention offer**.

```text
Features → Probability of Churn With Offer
```

The individual treatment effect is then estimated as the difference between the two predictions.

Conceptually:

```text
Uplift
=
P(Stay | Treatment)
-
P(Stay | Control)
```

Customers can then be ranked according to their estimated uplift.

---

# 📊 Business Impact Simulation

Assume the business has a limited retention budget and can target only the **top 20% of customers**.

This corresponds to approximately **1,408 customers**.

Three targeting strategies were compared:

| Strategy                | Customers Saved |
| ----------------------- | --------------: |
| Random Targeting        |           171.8 |
| Churn Probability Model |           216.5 |
| **Uplift Model**        |       **230.3** |

### Key Finding

Under the simulated treatment effects and targeting budget:

```text
Random Targeting
       ↓
    171.8 saved

Churn Probability Model
       ↓
    216.5 saved

Uplift Model
       ↓
   230.3 saved
```

The simulated uplift strategy retained approximately **14 more customers** than standard churn-probability targeting within the same targeting budget.

Relative to the churn-probability strategy, this corresponds to approximately a **6.4% increase in customers saved** in this simulation.

---

# 🏆 Model Results

### Best Churn Prediction Model

**XGBoost**

```text
PR-AUC: 0.694
```

This model was selected as the strongest baseline for churn probability prediction among the tested classifiers.

### Uplift Model

The T-Learner was then used to estimate heterogeneous treatment effects and rank customers according to their estimated incremental response to the retention offer.

---

# 🔬 Why This Project Goes Beyond Churn Prediction

Traditional churn prediction:

```text
Customer Data
      ↓
Churn Model
      ↓
"Will this customer leave?"
```

This project adds a second layer:

```text
Customer Data
      ↓
Churn Prediction
      ↓
Treatment Effect Estimation
      ↓
"Will an offer actually change their behavior?"
      ↓
Target Persuadable Customers
```

The key distinction is:

> **High churn probability does not necessarily mean high retention potential.**

A customer can be highly likely to churn but still be a poor target for a retention campaign if an offer would not change their behavior.

---

# ⚠️ Limitations & Next Steps

## 1. Simulated Treatment Effects

The biggest limitation is that the public telecom dataset does not contain real randomized retention-offer data.

Therefore:

* Treatment/control assignment was simulated.
* Treatment effects were simulated using domain assumptions.
* The resulting uplift performance should **not** be interpreted as evidence of real-world causal effectiveness.

A production implementation should be trained and evaluated using data from an actual randomized experiment.

---

## 2. T-Learner Model Divergence

The Two-Model T-Learner showed some model divergence, including learning a negative correlation between certain estimated treatment effects and the simulated ground truth.

This highlights an important challenge in uplift modeling:

> Estimating individual treatment effects is considerably harder than predicting churn probability.

Future experiments could evaluate alternative approaches such as:

* S-Learners
* X-Learners
* Other meta-learners
* Dedicated uplift modeling frameworks such as `scikit-uplift`

---

## 3. Synthetic Ground Truth

Because the treatment effects were simulated, the model is effectively being evaluated against assumptions built into the simulator.

A stronger validation framework would use:

```text
Real Randomized Experiment
          ↓
Treatment / Control Groups
          ↓
Observed Outcomes
          ↓
Estimated Individual Treatment Effects
          ↓
Uplift Model Validation
```

---

# 🚀 Next Steps

Potential improvements include:

* Validate the approach on real retention campaign data
* Run a randomized holdout experiment
* Compare T-Learner, S-Learner, and X-Learner approaches
* Experiment with `scikit-uplift`
* Optimize campaign targeting based on retention value rather than customer count
* Incorporate offer cost and customer lifetime value
* Evaluate uplift using Qini curves and AUUC
* Build a Streamlit dashboard for campaign simulation
* Add model explainability using SHAP
* Test different campaign budgets and targeting thresholds

---

# 💻 Tech Stack

| Category            | Tools                                       |
| ------------------- | ------------------------------------------- |
| Programming         | Python                                      |
| Data Processing     | Pandas, NumPy                               |
| Visualization       | Matplotlib, Seaborn                         |
| Machine Learning    | Scikit-learn, XGBoost                       |
| Classification      | Logistic Regression, Random Forest, XGBoost |
| Uplift Modeling     | XGBoost T-Learner                           |
| Evaluation          | PR-AUC                                      |
| Potential Extension | scikit-uplift                               |

---

# 📂 Project Workflow

```text
Telco Customer Dataset
          ↓
Data Cleaning
          ↓
Leakage Audit
          ↓
Feature Encoding
          ↓
Train/Test Split
          ↓
─────────────────────────────
│                           │
↓                           ↓
Churn Prediction       Treatment Simulation
│                           │
├─ Logistic Regression      ├─ Control Group
├─ Random Forest             └─ Treatment Group
└─ XGBoost                        ↓
│                         Simulated Treatment Effects
↓                                 ↓
PR-AUC Evaluation          T-Learner Uplift Model
│                                 ↓
└───────────────┬─────────────────┘
                ↓
        Customer Ranking
                ↓
      Top 20% Targeting
                ↓
        Business Simulation
                ↓
       Customers Saved
```

---

# 📌 Key Takeaways

* **Churn prediction** identifies customers who are likely to leave.
* **Uplift modeling** attempts to identify customers whose behavior can be changed by an intervention.
* **PR-AUC** is more informative than accuracy for this imbalanced churn dataset.
* **XGBoost achieved a PR-AUC of 0.694** as the strongest baseline churn model.
* Under the simulated treatment effects, **uplift-based targeting retained more customers than random or churn-probability targeting** at the same 20% targeting budget.
* The uplift results are **simulation-based** and require validation using real randomized treatment/control data before making real business decisions.

---

## 👨‍💻 Project Objective

The goal of this project is not simply to build another churn classifier.

It is to demonstrate the transition from:

> **Predictive Modeling → Causal Decision Making**

Instead of asking only:

> *"Who is going to churn?"*

the project asks the more actionable question:

> **"Who can we actually save?"**
