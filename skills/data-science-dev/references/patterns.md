# Data Science Patterns Reference

---

## 1. EDA (Exploratory Data Analysis)

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

def eda_summary(df: pd.DataFrame) -> None:
    """Canonical EDA starting point."""
    print(df.shape)
    print(df.dtypes)
    print(df.isnull().sum())
    print(df.describe(include='all'))

# Distribution — continuous
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
df['value'].hist(bins=50, ax=axes[0])
df['value'].plot(kind='box', ax=axes[1])
plt.tight_layout()

# Correlation heatmap
corr = df.select_dtypes('number').corr()
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm', center=0)

# Outlier detection (IQR)
Q1, Q3 = df['value'].quantile([0.25, 0.75])
IQR = Q3 - Q1
outliers = df[(df['value'] < Q1 - 1.5*IQR) | (df['value'] > Q3 + 1.5*IQR)]

# Missing data pattern
import missingno as msno
msno.matrix(df)           # visualize missingness patterns
```

**Key checks before modeling:**
1. Class imbalance: `df['target'].value_counts(normalize=True)`
2. Duplicates: `df.duplicated().sum()`
3. Cardinality of categoricals: `df.select_dtypes('object').nunique()`
4. Date range sanity: `df['date'].min(), df['date'].max()`

---

## 2. ML Clássico (sklearn)

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.model_selection import StratifiedKFold, cross_val_score
from sklearn.linear_model import LogisticRegression

# Canonical Pipeline — preprocessor + model
numeric_features = ['age', 'income']
categorical_features = ['category', 'region']

preprocessor = ColumnTransformer([
    ('num', StandardScaler(),        numeric_features),
    ('cat', OneHotEncoder(drop='first', handle_unknown='ignore'), categorical_features),
])

pipe = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier',   LogisticRegression(max_iter=1000, random_state=42)),
])

# Cross-validation — always stratified for classification
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(pipe, X_train, y_train, cv=cv, scoring='roc_auc', n_jobs=-1)
print(f"CV AUC: {scores.mean():.4f} ± {scores.std():.4f}")

# Fit and evaluate — test set used ONCE
pipe.fit(X_train, y_train)
print(f"Test AUC: {roc_auc_score(y_test, pipe.predict_proba(X_test)[:, 1]):.4f}")
```

**Hyperparameter tuning — use val set, not test:**
```python
from sklearn.model_selection import RandomizedSearchCV

search = RandomizedSearchCV(
    pipe, param_distributions={'classifier__C': [0.01, 0.1, 1, 10]},
    n_iter=20, cv=cv, scoring='roc_auc', n_jobs=-1, random_state=42,
)
search.fit(X_train, y_train)
# test set untouched until here
final_score = roc_auc_score(y_test, search.predict_proba(X_test)[:, 1])
```

---

## 3. Deep Learning (PyTorch)

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, Dataset
from torch.cuda.amp import GradScaler, autocast

# Reproducibility setup — call before anything else
def set_seed(seed: int) -> None:
    import random, os, numpy as np
    random.seed(seed); os.environ["PYTHONHASHSEED"] = str(seed)
    np.random.seed(seed)
    torch.manual_seed(seed); torch.cuda.manual_seed_all(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False

# Training loop — AMP + gradient accumulation
def train_epoch(model, loader, optimizer, criterion, scaler, accum_steps=4):
    model.train()
    optimizer.zero_grad()
    for i, (x, y) in enumerate(loader):
        x, y = x.cuda(), y.cuda()
        with autocast(device_type='cuda'):
            loss = criterion(model(x), y) / accum_steps
        scaler.scale(loss).backward()
        if (i + 1) % accum_steps == 0:
            scaler.unscale_(optimizer)
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            scaler.step(optimizer)
            scaler.update()
            optimizer.zero_grad()

# DataLoader worker seeds (required for full reproducibility)
def worker_init_fn(worker_id: int) -> None:
    import numpy as np, random
    seed = torch.initial_seed() % (2**32)
    np.random.seed(seed); random.seed(seed)

loader = DataLoader(dataset, batch_size=32, num_workers=4, worker_init_fn=worker_init_fn)
```

---

## 4. NLP / LLMs (HuggingFace)

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
from transformers import TrainingArguments, Trainer
import torch

# Tokenizer hygiene — always use fast tokenizer, set max_length
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased", use_fast=True)

def tokenize(batch):
    return tokenizer(
        batch["text"],
        truncation=True,
        max_length=512,
        padding="max_length",   # or "longest" for efficiency
        return_tensors="pt",
    )

# Fine-tuning — minimal but correct
model = AutoModelForSequenceClassification.from_pretrained(
    "bert-base-uncased", num_labels=2
)

training_args = TrainingArguments(
    output_dir="./results",
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    seed=42,
    per_device_train_batch_size=16,
    num_train_epochs=3,
    fp16=torch.cuda.is_available(),
)

# Embeddings — normalize before cosine similarity
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("all-MiniLM-L6-v2")
embeddings = model.encode(sentences, normalize_embeddings=True)
# cosine_sim = embeddings @ embeddings.T  (normalized → dot product = cosine)
```

---

## 5. Computação Científica (SciPy / SymPy)

```python
from scipy import optimize, stats, signal
import numpy as np

# Curve fitting
def model_func(x, a, b, c):
    return a * np.exp(-b * x) + c

popt, pcov = optimize.curve_fit(
    model_func, x_data, y_data,
    p0=[1.0, 0.1, 0.0],          # initial guess matters
    maxfev=5000,
)
perr = np.sqrt(np.diag(pcov))    # parameter uncertainty

# Statistical testing — check assumptions first
_, p_normal = stats.shapiro(sample)                         # normality test
stat, p_value = stats.ttest_ind(group_a, group_b)           # parametric
stat, p_value = stats.mannwhitneyu(group_a, group_b)        # non-parametric

# Bootstrap CI (non-parametric)
def bootstrap_ci(data, stat_fn, n_boot=10000, ci=0.95, seed=42):
    rng = np.random.default_rng(seed)
    boot_stats = [stat_fn(rng.choice(data, size=len(data), replace=True))
                  for _ in range(n_boot)]
    lo = (1 - ci) / 2
    return np.percentile(boot_stats, [lo*100, (1-lo)*100])

# Numerical stability — log-space for products of small probabilities
log_probs = np.array([np.log(p) for p in probabilities])
log_product = np.sum(log_probs)      # not: product = np.prod(probabilities)
```

---

## 6. Tabular Pesado (Polars / DuckDB)

```python
import polars as pl
import duckdb

# Polars — lazy evaluation, never loads full dataset unless .collect()
df = (
    pl.scan_parquet("data/*.parquet")       # lazy — no I/O yet
    .filter(pl.col("year") >= 2020)
    .with_columns([
        (pl.col("revenue") / pl.col("cost")).alias("margin"),
    ])
    .group_by("region")
    .agg(pl.col("margin").mean().alias("avg_margin"))
    .sort("avg_margin", descending=True)
    .collect()                              # execute here
)

# DuckDB — SQL over files without loading into pandas
con = duckdb.connect()
result = con.execute("""
    SELECT region, AVG(margin) AS avg_margin
    FROM read_parquet('data/*.parquet')
    WHERE year >= 2020
    GROUP BY region
    ORDER BY avg_margin DESC
""").df()                                   # convert to pandas only at the end

# Chunked pandas for files that fit on disk but not in RAM
chunk_results = []
for chunk in pd.read_csv("large.csv", chunksize=100_000):
    chunk_results.append(process(chunk))
result = pd.concat(chunk_results, ignore_index=True)
```

---

## 7. Experiment Tracking

```python
import mlflow
from dataclasses import dataclass, asdict
import json

# Config as dataclass — always saved alongside results
@dataclass
class ExperimentConfig:
    seed:       int   = 42
    lr:         float = 1e-3
    epochs:     int   = 50
    batch_size: int   = 32
    model_name: str   = "resnet50"
    data_path:  str   = "data/train.parquet"

cfg = ExperimentConfig()

# MLflow — log everything in one run
with mlflow.start_run():
    mlflow.log_params(asdict(cfg))

    # save config file too (not just mlflow params)
    with open("run_config.json", "w") as f:
        json.dump(asdict(cfg), f, indent=2)
    mlflow.log_artifact("run_config.json")

    for epoch in range(cfg.epochs):
        train_loss = train_epoch(...)
        val_auc    = evaluate(...)
        mlflow.log_metrics({"train_loss": train_loss, "val_auc": val_auc}, step=epoch)

    mlflow.sklearn.log_model(model, "model")

# W&B — equivalent
import wandb
run = wandb.init(project="my-project", config=asdict(cfg))
wandb.log({"train_loss": loss, "val_auc": auc}, step=epoch)
run.finish()
```
