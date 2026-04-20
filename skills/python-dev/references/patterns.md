# Python Patterns Reference

## Scientific Computing

### NumPy
```python
# Prefer: vectorized operations
result = np.where(mask, a, b)                    # not: [a if m else b for a,m in zip(...)]
result = arr[arr > threshold]                     # boolean indexing
matrix = np.einsum('ij,jk->ik', A, B)            # explicit dimension control

# Memory layout matters for performance
c_contig = np.ascontiguousarray(arr)              # for row-wise operations
f_contig = np.asfortranarray(arr)                 # for column-wise operations

# Modern RNG (prefer over legacy)
rng = np.random.default_rng(seed=42)
samples = rng.normal(0, 1, size=(1000,))
```

### Pandas
```python
# Avoid fragmentation
df = pd.DataFrame(data)                          # build all at once
# not: df['new_col'] = ...  (repeated column assignment)

# Use categorical for low-cardinality string columns
df['category'] = df['category'].astype('category')

# Efficient filtering
result = df.query("value > 0 and label == 'positive'")   # eval engine

# Memory usage
df.info(memory_usage='deep')

# Correct dtypes from load
df = pd.read_csv(f, dtype={'id': 'int32', 'score': 'float32'})
```

### PyTorch
```python
# Full reproducibility setup
def set_seed(seed: int) -> None:
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False

# Gradient accumulation (correct scaling)
optimizer.zero_grad()
for i, batch in enumerate(loader):
    loss = model(batch) / accumulation_steps
    loss.backward()
    if (i + 1) % accumulation_steps == 0:
        optimizer.step()
        optimizer.zero_grad()

# AMP (mixed precision)
scaler = torch.cuda.amp.GradScaler()
with torch.autocast(device_type='cuda'):
    output = model(input)
    loss = criterion(output, target)
scaler.scale(loss).backward()
scaler.step(optimizer)
scaler.update()
```

## Async I/O (Python 3.11+)

```python
# TaskGroup for structured concurrency
async def fetch_all(urls: list[str]) -> list[bytes]:
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(url)) for url in urls]
    return [t.result() for t in tasks]

# Semaphore for bounded concurrency
sem = asyncio.Semaphore(10)
async def bounded_fetch(url: str) -> bytes:
    async with sem:
        return await fetch(url)

# Proper cancellation
async def worker(queue: asyncio.Queue, stop: asyncio.Event) -> None:
    while not stop.is_set():
        try:
            item = await asyncio.wait_for(queue.get(), timeout=1.0)
            await process(item)
        except asyncio.TimeoutError:
            continue
```

## Packaging (pyproject.toml)

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "numpy>=1.26",
    "pandas>=2.0",
]

[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]
research = ["torch>=2.0", "scikit-learn>=1.3"]

[project.scripts]
myproject = "myproject.cli:main"

[tool.ruff]
line-length = 88
select = ["E", "F", "I", "N", "UP"]

[tool.mypy]
strict = true
```

## Type Hints (advanced)

```python
from typing import Protocol, TypeVar, ParamSpec, overload

T = TypeVar("T")
P = ParamSpec("P")

# Protocol for structural subtyping
class Fittable(Protocol):
    def fit(self, X: np.ndarray, y: np.ndarray) -> "Fittable": ...
    def predict(self, X: np.ndarray) -> np.ndarray: ...

# Overload for different return types
@overload
def load(path: str, as_dict: Literal[True]) -> dict: ...
@overload
def load(path: str, as_dict: Literal[False] = ...) -> pd.DataFrame: ...

# TypeGuard
def is_valid(x: object) -> TypeGuard[ValidType]:
    return isinstance(x, ValidType) and x.value > 0
```

## Experiment Configuration (Hydra/dataclasses)

```python
from dataclasses import dataclass, field

@dataclass
class TrainConfig:
    seed: int = 42
    lr: float = 1e-3
    epochs: int = 100
    batch_size: int = 32
    model: str = "resnet50"
    data_dir: str = "data/"
    output_dir: str = "outputs/${now:%Y-%m-%d_%H-%M-%S}"

# Save config alongside results
import json, dataclasses
with open(f"{run_dir}/config.json", "w") as f:
    json.dump(dataclasses.asdict(cfg), f, indent=2)
```
