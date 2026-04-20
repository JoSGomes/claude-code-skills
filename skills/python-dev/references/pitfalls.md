# Common Python Pitfalls for Researchers

## Data Leakage (most common research bug)

```python
# WRONG: scaler fit on all data
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # leaks test stats into train
X_train, X_test = train_test_split(X_scaled, ...)

# RIGHT: fit only on train
X_train, X_test = train_test_split(X, ...)
scaler = StandardScaler().fit(X_train)
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)

# RIGHT (better): use Pipeline
pipe = Pipeline([("scaler", StandardScaler()), ("clf", LogisticRegression())])
pipe.fit(X_train, y_train)  # scaler fitted on train only
pipe.score(X_test, y_test)
```

## Multiple Test Set Usage

```python
# WRONG: tuning on test set
for threshold in thresholds:
    score = evaluate(model, X_test, threshold)  # test set used for selection
best_threshold = max(...)

# RIGHT: use a validation set for selection
best_threshold = select_threshold(model, X_val)
final_score = evaluate(model, X_test, best_threshold)  # test used ONCE
```

## Seed Incompleteness

```python
# WRONG: partial seeding
np.random.seed(42)  # forgot torch, random, sklearn

# RIGHT: seed everything, document it
def seed_everything(seed: int) -> None:
    """Set all seeds for reproducibility. Call before any data loading."""
    import random, os
    random.seed(seed)
    os.environ["PYTHONHASHSEED"] = str(seed)
    np.random.seed(seed)
    if torch_available:
        torch.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)
```

## Mutable Default Arguments

```python
# WRONG
def process(data, results=[]):
    results.append(transform(data))
    return results

# RIGHT
def process(data, results=None):
    if results is None:
        results = []
    results.append(transform(data))
    return results
```

## Exception Handling in Pipelines

```python
# WRONG: silent failures corrupt results
for sample in dataset:
    try:
        result = process(sample)
        results.append(result)
    except:  # swallows everything, including KeyboardInterrupt
        pass

# RIGHT: log and decide
for i, sample in enumerate(dataset):
    try:
        result = process(sample)
        results.append(result)
    except (ValueError, ProcessingError) as e:
        logger.warning(f"Sample {i} failed: {e}")
        failed_samples.append(i)
    # let KeyboardInterrupt and SystemExit propagate
```

## Float Comparison

```python
# WRONG
if score == 1.0:  # floating point equality is unreliable
    ...

# RIGHT
if abs(score - 1.0) < 1e-9:
    ...
# or
import math
if math.isclose(score, 1.0, rel_tol=1e-9):
    ...
```

## Pandas SettingWithCopyWarning

```python
# WRONG: modifying a slice
subset = df[df['label'] == 1]
subset['new_col'] = 0  # SettingWithCopyWarning, may not modify df

# RIGHT
df.loc[df['label'] == 1, 'new_col'] = 0
# or
subset = df[df['label'] == 1].copy()
subset['new_col'] = 0
```

## Temporal Leakage

```python
# WRONG: random split on time-series data
X_train, X_test = train_test_split(df, random_state=42)

# RIGHT: time-based split
cutoff = df['date'].quantile(0.8)
X_train = df[df['date'] <= cutoff]
X_test = df[df['date'] > cutoff]
```

## Generator Exhaustion

```python
# WRONG: generators consumed once
data_gen = (preprocess(x) for x in raw_data)
train_model(data_gen)
evaluate_model(data_gen)  # empty! silently produces wrong results

# RIGHT: materialize or recreate
data = [preprocess(x) for x in raw_data]
# or recreate the generator for each use
```
