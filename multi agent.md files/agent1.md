# Quantum Multi-Backend Agent — Part 1: Quick Reference & Getting Started

_Master: [AGENTS.md](AGENTS.md) — Configuration & Overview_
_Next: [agent2.md](agent2.md) — Backend Registry & Dependencies_

**Multi-Backend Quantum Simulation Assistant for Crush AI**
_Run, benchmark, and compare quantum simulation backends through natural language._
_Powered by [Crush AI](https://github.com/charmbracelet/crush)_

---

## Quick Reference

| Item | Value |
|------|-------|
| **AI Assistant** | [Crush AI](https://github.com/charmbracelet/crush) (TUI-based) |
| **Primary Repos** | [kunal5556/LRET](https://github.com/kunal5556/LRET), [quantumlib/Cirq](https://github.com/quantumlib/Cirq) |
| **LRET Branches** | `cirq-scalability-comparison`, `pennylane-documentation-benchmarking`, `phase-7` |
| **Additional Backends** | Qiskit Aer, QuEST, qsim, cuQuantum, PennyLane, Stim |
| **Supported Platforms** | Windows (PowerShell/WSL), macOS, Linux |

---

## Document Structure

| Part | File | Contents |
|------|------|----------|
| **1** | `agent1.md` | Quick Reference, Crush Setup, Natural Language Commands |
| **2** | `agent2.md` | Backend Registry — Clone URLs, Dependencies, Build Instructions |
| **3** | `agent3.md` | Execution — Running Each Backend, Result Parsing, Error Handling |
| **4** | `agent4.md` | Benchmarking & Parallel Execution — Multi-Backend, Parameter Sweeps |
| **5** | `agent5.md` | Comparative Analysis & Visualization — Reports, Plots, Summaries |
| **6** | `agent6.md` | Advanced — Cloud, Custom Scripts, Code Modification, Troubleshooting |

---

## Getting Started with Crush AI

### Step 1: Install Crush

**Windows (WinGet):**
```powershell
winget install charmbracelet.crush
```

**macOS (Homebrew):**
```bash
brew install charmbracelet/tap/crush
```

**Linux (APT/YUM):**
```bash
# Debian/Ubuntu
echo "deb [trusted=yes] https://repo.charm.sh/apt/ /" | sudo tee /etc/apt/sources.list.d/charm.list
sudo apt update && sudo apt install crush

# Fedora/RHEL
sudo yum install crush
```

**Go Install:**
```bash
go install github.com/charmbracelet/crush@latest
```

**Verify:** Run `crush --version` to confirm installation.

---

### Step 2: Configure Your LLM Provider

Crush supports multiple LLM providers. Set one of these environment variables:

**Windows PowerShell:**
```powershell
# Pick one provider:
$env:ANTHROPIC_API_KEY = "sk-ant-..."      # Anthropic Claude
$env:OPENAI_API_KEY = "sk-..."             # OpenAI
$env:GEMINI_API_KEY = "..."                # Google Gemini

# To persist across sessions:
setx ANTHROPIC_API_KEY "sk-ant-..."
```

**macOS/Linux:**
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
# Add to ~/.bashrc or ~/.zshrc for persistence
```

Or use GitHub Copilot:
```bash
crush login copilot
```

---

### Step 3: Set Up Your Workspace

```bash
# Create a workspace directory
mkdir quantum-benchmarks && cd quantum-benchmarks

# Copy/place the agent .md files (AGENTS.md + agent1-6.md) into this directory

# Create required subdirectories
mkdir -p backends results benchmarks plots scripts
```

---

### Step 4: Launch Crush

```bash
cd quantum-benchmarks
crush
```

Crush reads the `AGENTS.md` file (and any other `.md` files you reference) to understand the project context. You can now issue natural language commands.

---

### Step 5: First Simulation

Type this in the Crush prompt:

```
clone the LRET cirq scalability backend, build it, and run a 10 qubit simulation with depth 20
```

Crush will:
1. Clone `https://github.com/kunal5556/LRET.git` (branch `cirq-scalability-comparison`) into `backends/lret-cirq-scalability/`
2. Install dependencies (CMake, Eigen3, Python packages)
3. Build the C++ project
4. Run the simulation
5. Display results with comprehensive analysis

---

## Natural Language Command Reference

### Cloning & Setup

```
clone the LRET phase 7 backend
clone and build Qiskit Aer
set up all LRET backends
clone Google Cirq and install its dependencies
set up QuEST with GPU support
install cuQuantum SDK
clone and configure PennyLane with all device plugins
```

### Running Simulations

```
# Single backend
run 10 qubit GHZ simulation on LRET phase 7
run 8 qubit random circuit on Cirq with 1% depolarizing noise
run 14 qubit QFT on Qiskit Aer with density matrix backend
simulate 20 qubits on QuEST with OpenMP parallelization

# With specific parameters
run 12 qubits, depth 30, hybrid mode on LRET PennyLane
run VQE for H2 molecule on Cirq with BFGS optimizer
run QAOA MaxCut on 8 qubits using qsim backend
simulate 16 qubits on cuQuantum with cuStateVec

# GPU acceleration
run 20 qubit simulation on Qiskit Aer with GPU
run large simulation on cuQuantum with 24 qubits
```

### Benchmarking

```
benchmark LRET phase 7 and Cirq on 8,10,12,14 qubits
benchmark all available backends on 10 qubits with depth 20
run scaling benchmark from 4 to 20 qubits on LRET cirq scalability
benchmark Qiskit Aer vs cuQuantum on GPU for 16,18,20,22 qubits
run 5-trial benchmark on LRET phase 7 with error bars
benchmark noise impact: 0%, 0.1%, 0.5%, 1%, 2% on Cirq for 10 qubits
```

### Comparative Analysis

```
compare LRET phase 7 vs Cirq vs Qiskit Aer for 12 qubits
compare fidelity between LRET cirq scalability and Google Cirq
compare all LRET branches on the same 10 qubit circuit
show me which backend is fastest for 16+ qubits
compare memory usage across all backends for scaling test
rank all backends by accuracy for noisy 10 qubit simulation
```

### Running Multiple Backends Simultaneously

```
run 10 qubit GHZ on LRET phase 7, Cirq, and Qiskit Aer at the same time
simultaneously benchmark all backends on 12 qubits depth 20
run parallel comparison: LRET cirq scalability vs qsim vs cuQuantum on 14 qubits
```

### Custom Scripts

```
run the script scripts/my_benchmark.py
run scripts/noise_sweep.py with --qubits 10 --noise-range 0.01,0.05,0.1
create a benchmark script that tests all backends on 8,10,12 qubits
execute scripts/compare_results.py on the latest benchmark data
run multiple scripts: scripts/setup.sh then scripts/benchmark.py then scripts/analyze.py
```

### Dependency Management

```
install all dependencies for LRET cirq scalability on Windows
check if Eigen3 and CMake are installed
install Python dependencies for Cirq and Qiskit
set up a virtual environment for quantum backends
install CUDA toolkit for GPU backends
check which backends are ready to run
```

### Code Modification

```
modify the noise model in LRET phase 7 to add custom amplitude damping
add a new circuit type to the Cirq executor
optimize the LRET build for AVX2 instructions
patch QuEST to increase max qubit count
add PennyLane device support to LRET
```

### Result Analysis

```
analyze the results from the last benchmark
explain the fidelity drop at 14 qubits on LRET
show memory scaling across backends
generate a summary report of all experiments today
export results to CSV for external analysis
why did QuEST outperform Cirq at 18 qubits?
```

---

## Keyboard Shortcuts in Crush TUI

| Shortcut | Action |
|----------|--------|
| `Ctrl+L` / `Ctrl+M` | Switch LLM model |
| `Ctrl+S` | Switch sessions |
| `Ctrl+P` | View commands |
| `Ctrl+G` | Help |
| `Ctrl+C` | Quit |

---

## Troubleshooting

### Crush not found
```powershell
# Windows
winget install charmbracelet.crush
# Then restart your terminal
```

### Backend clone fails
```
check if git is installed and available in PATH
```

### Build fails for LRET
```
check CMake version (needs 3.16+), check if Eigen3 is installed,
check C++17 compiler support
```

### Python import errors
```
create a virtual environment and install backend dependencies:
python -m venv .venv && source .venv/bin/activate && pip install cirq qiskit-aer pennylane
```

### GPU not detected
```
check CUDA installation: nvcc --version
check GPU availability: nvidia-smi
```

### Out of memory on large simulations
```
use LRET with low-rank mode for 20+ qubits
use QuEST with distributed MPI for very large simulations
reduce circuit depth or use qsim for state-vector-only
```

---

_Continue to [agent2.md](agent2.md) for the complete Backend Registry with clone URLs, dependencies, and build instructions._
