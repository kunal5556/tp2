# Quantum Simulation Multi-Backend Agent — Crush AI Configuration

This is a multi-backend quantum simulation research assistant powered by **Crush AI** (https://github.com/charmbracelet/crush). It enables running, benchmarking, and comparing quantum simulation backends through natural language commands in the Crush TUI.

## Supported Backends & Repositories

| Backend | Repository | Branch / Source | Language | GPU Support |
|---------|-----------|----------------|----------|-------------|
| **LRET Cirq Scalability** | [kunal5556/LRET](https://github.com/kunal5556/LRET) | `cirq-scalability-comparison` | C++17 + Python | Optional (CUDA) |
| **LRET PennyLane Hybrid** | [kunal5556/LRET](https://github.com/kunal5556/LRET) | `pennylane-documentation-benchmarking` | C++17 + Python | Optional (CUDA) |
| **LRET Phase 7 Unified** | [kunal5556/LRET](https://github.com/kunal5556/LRET) | `phase-7` | C++17 + Python | Optional (CUDA) |
| **Google Cirq** | [quantumlib/Cirq](https://github.com/quantumlib/Cirq) | `main` | Python | Via qsim |
| **Qiskit Aer** | [Qiskit/qiskit-aer](https://github.com/Qiskit/qiskit-aer) | `main` | C++ + Python | CUDA, ROCm |
| **QuEST** | [QuEST-Kit/QuEST](https://github.com/QuEST-Kit/QuEST) | `main` | C | CUDA, OpenMP |
| **qsim** | [quantumlib/qsim](https://github.com/quantumlib/qsim) | `main` | C++ + Python | CUDA |
| **cuQuantum** | NVIDIA cuQuantum SDK | N/A (pip install) | C++ + Python | CUDA (required) |
| **PennyLane** | [PennyLaneAI/pennylane](https://github.com/PennyLaneAI/pennylane) | `master` | Python | Via plugins |
| **Stim** | [quantumlib/Stim](https://github.com/quantumlib/Stim) | `main` | C++ + Python | No |

## What This Agent Can Do

1. **Run backends** and provide comprehensive analysis of results (fidelity, timing, memory, gate counts).
2. **Run benchmarks** across multiple backends with identical circuit specifications.
3. **Run comparative analysis** — side-by-side performance, accuracy, and scaling comparisons.
4. **Run backends on custom user requirements** — arbitrary qubit counts, depths, noise models, circuit types.
5. **Make changes to backend code** based on user requirements (edit, patch, optimize).
6. **Clone required GitHub repos** automatically when a backend is not yet available locally.
7. **Build and compile backends** — full dependency installation, CMake builds, pip installs.
8. **Run multiple backends simultaneously** on the same specifications for head-to-head comparison.
9. **Install, configure, and handle all dependencies** — Python packages, C++ libraries, CUDA toolkits.
10. **Run scripts or multiple scripts** on user requirements — custom Python/bash scripts, parameter sweeps.

## Crush Configuration

These agent files are designed for Crush AI. Place them alongside a `crush.json` in your project:

```json
{
  "$schema": "https://charm.land/crush.json",
  "permissions": {
    "allowed_tools": [
      "view",
      "ls",
      "grep",
      "edit",
      "bash"
    ]
  }
}
```

## Directory Structure

When Crush clones and builds backends, they are organized under a workspace directory:

```
workspace/
├── crush.json                  # Crush configuration
├── AGENTS.md                   # This file (master agent rules)
├── agent1.md                   # Quick Reference & Setup
├── agent2.md                   # Backend Registry & Dependencies
├── agent3.md                   # Execution & Running Backends
├── agent4.md                   # Benchmarking & Parallel Execution
├── agent5.md                   # Comparative Analysis & Visualization
├── agent6.md                   # Advanced: Cloud, Scripts, Customization
├── backends/                   # Cloned backend repositories
│   ├── lret-cirq-scalability/  # LRET cirq-scalability-comparison branch
│   ├── lret-pennylane/         # LRET pennylane-documentation-benchmarking branch
│   ├── lret-phase7/            # LRET phase-7 branch
│   ├── cirq/                   # Google Cirq
│   ├── qiskit-aer/             # Qiskit Aer
│   ├── quest/                  # QuEST
│   ├── qsim/                   # Google qsim
│   └── stim/                   # Stim
├── results/                    # All experiment results (JSON, CSV)
├── benchmarks/                 # Benchmark output files
├── plots/                      # Generated visualizations
└── scripts/                    # User custom scripts
```

## Common Tasks

### Clone and set up a backend
```
clone and build the LRET cirq scalability backend
```

### Run a simulation
```
run a 10 qubit GHZ simulation on Cirq with depth 20
```

### Benchmark multiple backends
```
benchmark LRET phase 7, Cirq, and Qiskit Aer on 8,10,12,14 qubits with depth 20
```

### Compare backends
```
compare fidelity and execution time between LRET cirq scalability and Google Cirq for 10 qubits
```

### Run multiple backends simultaneously
```
run 12 qubit random circuit on all available backends simultaneously and compare results
```

### Install dependencies
```
install all dependencies for Qiskit Aer including GPU support
```

### Run custom scripts
```
run the script at scripts/my_benchmark.py with arguments --qubits 10 --trials 5
```

## Agent File Reference

| File | Contents |
|------|----------|
| `AGENTS.md` | Master configuration, backend registry, capabilities overview |
| `agent1.md` | Quick Reference, Crush setup, natural language command examples |
| `agent2.md` | Backend Registry — clone URLs, dependencies, build instructions for every backend |
| `agent3.md` | Execution Systems — how to run each backend, result parsing, error handling |
| `agent4.md` | Benchmarking & Parallel Execution — multi-backend benchmarks, parameter sweeps, simultaneous runs |
| `agent5.md` | Comparative Analysis & Visualization — side-by-side analysis, plots, reports |
| `agent6.md` | Advanced Features — cloud deployment, custom scripts, code modification, troubleshooting |

## Safety Rules

- ALWAYS show the execution plan before running experiments.
- NEVER delete user data or overwrite results without explicit confirmation.
- ALWAYS validate that dependencies are installed before attempting to run a backend.
- For long-running experiments (> 5 minutes estimated), inform the user and ask for confirmation.
- When modifying backend source code, always create a backup or work on a branch.
- When cloning repositories, check if the directory already exists before cloning.

## Code Standards

- C++ backends: Use C++17 features, CMake for builds, follow existing naming conventions.
- Python backends: Use Python 3.9+, virtual environments recommended, pip for dependencies.
- Results: Output in JSON format with standardized schema for cross-backend comparison.
- Logs: All experiment runs should be logged with timestamps and parameters.
