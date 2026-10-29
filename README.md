# Predictive Fault Analysis & Resilience Engineering

A native Windows desktop application (PySide6) and headless engine that learns how target software behaves under controlled stress, predicts failure within 30 seconds using per-target Machine Learning (RandomForest), analyzes failure cascades across component dependency graphs, and prescribes actionable resilience fixes.

---

## Key Capabilities & Safety Guarantees

### 1. Target Adapters (v1)
- **`process`**: Standalone executables, python scripts, CLI batch workers, and memory-leaking targets.
- **`http_service`**: Microservices and REST APIs (FastAPI) on dynamic ports.
- **`python_function`**: In-process or threaded Python callables.
- **Strict Boundary**: All engine components communicate using only the standardized **Common Metrics Schema** (`MetricSample`).

### 2. Guarded Safety Model
- **Ownership & Approval**: Faults can only be injected against processes launched by the application or explicitly confirmed by the user.
- **Allowlisted Hosts**: Strict localhost (`127.0.0.1`, `localhost`, `::1`) allowlisting.
- **Safety Caps**:
  - Max CPU stress: 90% hard cap
  - Max Memory stress: 2048 MB hard cap
  - Max fault duration: 120 seconds hard cap
- **Guaranteed Cleanup & Auto-Rollback**: Every fault has a background timer and context manager ensuring full cleanup.
- **Prominent Emergency STOP Button**: Instantly terminates and rolls back all active faults across all targets.
- **Audit Logging**: Every action and fault event is logged to SQLite with cryptographic tokens.

### 3. ML Prediction Pipeline (30-Second Warning)
- **Real Experiment Data Only**: Never relies on synthetic mock labels.
- **Causal Feature Extraction**: Strictly past-looking rolling averages, rates of change, and volatility. Zero future leakage.
- **Future Window Labeling**: Computes whether failure occurs within $(t, t + 30\text{s}]$.
- **GroupKFold Cross-Validation**: Splits data strictly by experiment ID, preventing temporal leakage.
- **Baseline Comparison**: Compares against a static rule-threshold baseline.
- **Scorecard**: Reports Precision, Recall, F1, PR-AUC, and **Mean Lead Time** (advance warning in seconds).

### 4. Cascade & Blast Radius Analysis (NetworkX)
- Evaluates system impact when more than 1 component is registered.
- Identifies Single Points of Failure (SPOF) where failure causes $\ge 50\%$ system outage.
- Simulates failure cascade propagation chains.

### 5. Resilience Fix Recommendations
- Automatically detects memory leak slopes, concurrency bottlenecks, and unhandled error rates.
- Suggests Circuit Breakers, Exponential Backoff with Jitter, Bulkheads, Token-Bucket Rate Limiters, and SPOF redundancy.
- Generates comprehensive HTML, Markdown, and JSON audit reports.

---

## Directory Structure

```
predictive-resilience/
├── engine/
│   ├── config/          # Settings, port manager, safety controller
│   ├── metrics/         # Common metrics schema & collector
│   ├── storage/         # SQLite WAL persistence & audit trail
│   ├── targets/         # Process, HTTP, PythonFunction adapters
│   ├── load_testing/    # Subprocess load runner & worker
│   ├── fault_injection/ # Safe chaos injector with auto-rollback
│   ├── experiments/     # Scenario runner & profiling pipeline
│   ├── prediction/      # Causal feature engineering & real-time predictor
│   ├── graph_analysis/  # NetworkX cascade & blast radius
│   ├── recommendations/ # Resilience recommendation engine
│   ├── reports/         # HTML/Markdown/JSON generator
│   └── cli.py           # Headless CLI entry point
├── models/
│   ├── training/        # RandomForest GroupKFold trainer
│   ├── evaluation/      # Precision, Recall, F1, PR-AUC, Mean Lead Time
│   └── saved/           # Serialized models & feature lists
├── desktop/
│   ├── main.py          # Native Windows app entry point
│   ├── views/           # PySide6 main window & tabs
│   ├── widgets/         # Safety confirmation dialog, pyqtgraph charts, cascade tree
│   └── threads/         # Responsive QThread workers
├── target_system/       # Demo FastAPI targets, CLI worker, memory leak target
├── storage/             # SQLite DB and reports storage
└── tests/               # Comprehensive unit and integration test suite
```

---

## Running the Desktop Application

```powershell
.\run-local.bat
```

The launcher creates a local virtual environment, installs Python requirements, and opens the native Windows UI. To host that GUI remotely, use a Windows VM with an interactive desktop session; see [deploy/windows-vm/README.md](deploy/windows-vm/README.md).

## Running the Browser Console

The React/FastAPI browser console runs locally in Docker or on an AWS Linux EC2 instance:

```sh
cp .env.example .env
# Set APP_API_TOKEN to a private random secret in .env
docker compose -f compose.yaml -f compose.local.yaml up --build
```

Open `http://localhost:8080` and enter the token. For AWS HTTPS deployment instructions, DNS, and firewall requirements, see [deploy/aws/README.md](deploy/aws/README.md).

The browser console runs the engine and demo targets on the server. Keep its API token private; it authorizes process management and explicitly confirmed fault injection.

## Running via CLI

```powershell
# View recent safety audit log
py engine\cli.py audit

# Train ML model for a target
py engine\cli.py train --target memory_leak_target

# Export resilience audit report
py engine\cli.py report --target memory_leak_target
```

## Running Tests

```powershell
py -m pytest tests -v
```
