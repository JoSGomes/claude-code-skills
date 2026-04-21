# Data Science Research Pitfalls

> Bug priority: data leakage > seed incompleteness > test set reuse > all others.
> A model that doesn't generalize is suspicious; check these before blaming the architecture.

---

## 1. Data Leakage (most critical)

```python
# WRONG: preprocessor sees test data during fit
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)          # leaks test statistics into training
X_train, X_test = train_test_split(X_scaled, test_size=0.2, random_state=42)

# RIGHT: fit only on train, transform test
X_train, X_test = train_test_split(X, test_size=0.2, random_state=42)
scaler = StandardScaler().fit(X_train)
X_train_s = scaler.transform(X_train)
X_test_s  = scaler.transform(X_test)

# BEST: Pipeline enforces this automatically
pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("clf",    LogisticRegression()),
])
pipe.fit(X_train, y_train)      # scaler fitted on X_train only
pipe.score(X_test, y_test)
```

**Variants to check:**
- `LabelEncoder`, `OrdinalEncoder`, `TfidfVectorizer`, `CountVectorizer` — any `.fit()` before split
- Target encoding without fold-aware implementation (MeanEncoder leaks target statistics)
- Feature selection (SelectKBest, RFE) computed on full dataset before split
- PCA/dimensionality reduction fitted on full X

---

## 2. Temporal Leakage (time-series specific)

```python
# WRONG: random split destroys temporal order
X_train, X_test = train_test_split(df, random_state=42)  # shuffles time

# WRONG: rolling feature looks into the future
df['rolling_mean'] = df['value'].rolling(7).mean()       # includes future rows if df not sorted

# RIGHT: time-based split
df = df.sort_values('date')
cutoff = df['date'].quantile(0.8)
X_train = df[df['date'] <= cutoff]
X_test  = df[df['date'] >  cutoff]

# RIGHT: sklearn TimeSeriesSplit for CV
from sklearn.model_selection import TimeSeriesSplit
tscv = TimeSeriesSplit(n_splits=5)
for train_idx, val_idx in tscv.split(X):
    ...

# RIGHT: rolling feature — sort first, then compute
df = df.sort_values('date').reset_index(drop=True)
df['rolling_mean'] = df['value'].rolling(7, min_periods=1).mean()
```

---

## 3. Seed Incompleteness

```python
# WRONG: partial seed — results differ between runs
np.random.seed(42)              # forgot torch, random, os.environ

# RIGHT: seed everything in one place
import random, os
import numpy as np

def seed_everything(seed: int) -> None:
    """Call at the very start, before any data loading or model init."""
    random.seed(seed)
    os.environ["PYTHONHASHSEED"] = str(seed)
    np.random.seed(seed)
    try:
        import torch
        torch.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)
        torch.backends.cudnn.deterministic = True
        torch.backends.cudnn.benchmark = False   # may reduce speed
    except ImportError:
        pass
    try:
        import tensorflow as tf
        tf.random.set_seed(seed)
    except ImportError:
        pass
```

**Checklist:**
- `random.seed` — Python stdlib random
- `np.random.seed` (or `np.random.default_rng(seed)` for modern API)
- `torch.manual_seed` + `torch.cuda.manual_seed_all`
- `torch.backends.cudnn.deterministic = True`
- `sklearn` estimators: pass `random_state=seed` explicitly
- `train_test_split(random_state=seed)`
- DataLoader worker seeds via `worker_init_fn`

---

## 4. Test Set Reuse (p-hacking)

```python
# WRONG: model selection based on test set
best_model = None
for model in candidates:
    score = model.fit(X_train, y_train).score(X_test, y_test)   # test used in loop
    if score > best_score:
        best_model = model

# WRONG: threshold tuning on test set
for threshold in np.arange(0.1, 0.9, 0.05):
    score = f1_score(y_test, probs > threshold)                  # test used for tuning

# RIGHT: separate validation and test sets
X_train, X_rest, y_train, y_rest = train_test_split(X, y, test_size=0.3, random_state=42)
X_val,   X_test, y_val,   y_test = train_test_split(X_rest, y_rest, test_size=0.5, random_state=42)

# model selection on val, final evaluation on test ONCE
best_model = select_model(X_train, y_train, X_val, y_val)
final_score = best_model.score(X_test, y_test)                   # test used exactly once
```

---

## 5. Generator Exhaustion

```python
# WRONG: generator consumed silently on second pass
data_gen = (preprocess(x) for x in raw_data)
model.fit(data_gen)
metrics = evaluate(model, data_gen)     # data_gen is exhausted — evaluate sees 0 samples

# WRONG: PyTorch DataLoader iterator reuse
loader_iter = iter(train_loader)
batch1 = next(loader_iter)
# ... somewhere later in the loop ...
for batch in loader_iter:               # already partially consumed
    ...

# RIGHT: materialize when you need multiple passes
data = [preprocess(x) for x in raw_data]

# RIGHT: create a fresh DataLoader iterator per epoch
for epoch in range(epochs):
    for batch in train_loader:          # DataLoader creates a new iterator each time
        ...
```

---

## 6. Pandas SettingWithCopyWarning

```python
# WRONG: chained indexing — may or may not modify the original
subset = df[df['label'] == 1]
subset['new_col'] = 0                  # SettingWithCopyWarning; behavior undefined

# RIGHT: use .loc on the original
df.loc[df['label'] == 1, 'new_col'] = 0

# RIGHT: explicit copy when you want a separate frame
subset = df[df['label'] == 1].copy()
subset['new_col'] = 0                  # modifies subset only, intentionally
```

---

## 7. Float Comparison

```python
# WRONG: exact float equality
if accuracy == 1.0:    ...
if loss == 0.0:        ...
if prob_sum == 1.0:    ...

# RIGHT
import math
if math.isclose(accuracy, 1.0, rel_tol=1e-9):  ...
if loss < 1e-9:                                  ...
if abs(prob_sum - 1.0) < 1e-6:                  ...
```

---

## 8. Notebook Hidden State

```python
# Symptom: notebook produces different results depending on cell execution order
# Causes:
# - Cell defines `df = df.dropna()` — run twice = double-filter
# - `model` variable reassigned in a deleted cell, still in kernel memory
# - `scaler.fit(X_train)` run in multiple cells — last one wins

# Mitigations:
# 1. Restart kernel + Run All before sharing / publishing results
# 2. Keep stateful operations (fit, split) in a single cell
# 3. Use `importlib.reload` if modifying local modules during development
# 4. Pin the execution count at the top: "This notebook was last run clean on YYYY-MM-DD"
```

**Additional notebook pitfalls:**
- `%matplotlib inline` vs `plt.show()` — missing `plt.show()` in non-interactive runs causes empty plots
- Large intermediate DataFrames kept in memory across all cells — `del df_tmp; gc.collect()` when done
- `tqdm` in notebooks: use `tqdm.notebook.tqdm`, not `tqdm.tqdm` (breaks progress bar rendering)
