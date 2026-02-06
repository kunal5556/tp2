# Quantum Multi-Backend Agent — Part 6: Advanced Features

_Previous: [agent5.md](agent5.md) — Comparative Analysis & Visualization_
_Master: [AGENTS.md](AGENTS.md) — Configuration & Overview_

---

## Overview

This file covers advanced capabilities: custom script execution, code modification workflows, dependency troubleshooting, cloud/remote execution, and comprehensive error resolution.

---

## 1. Custom Script Execution

### Running User-Provided Scripts

When a user says "run my script," Crush should:

1. Identify the script language (Python, Bash, C++).
2. Check dependencies are available.
3. Execute with proper environment.
4. Capture and display output.

```python
#!/usr/bin/env python3
"""Script executor with dependency checking and timeout."""
import subprocess, sys, os, json, time
from pathlib import Path

def run_script(script_path, args=None, timeout=600, env_vars=None, cwd=None):
    """Run a user script safely with output capture."""
    script = Path(script_path)
    if not script.exists():
        return {"status": "error", "error": f"Script not found: {script_path}"}

    # Determine interpreter
    ext = script.suffix.lower()
    interpreters = {
        ".py": [sys.executable],
        ".sh": ["bash"],
        ".bash": ["bash"],
        ".ps1": ["powershell", "-File"],
        ".bat": ["cmd", "/c"],
        ".cpp": None,  # Needs compilation first
        ".c": None,
    }

    if ext in (".cpp", ".c"):
        # Compile first
        binary = script.with_suffix("")
        compiler = "g++" if ext == ".cpp" else "gcc"
        compile_cmd = [compiler, "-O2", "-o", str(binary), str(script), "-lm"]
        result = subprocess.run(compile_cmd, capture_output=True, text=True)
        if result.returncode != 0:
            return {"status": "error", "error": f"Compilation failed:\n{result.stderr}"}
        command = [str(binary)]
    elif ext in interpreters and interpreters[ext]:
        command = interpreters[ext] + [str(script)]
    else:
        return {"status": "error", "error": f"Unknown script type: {ext}"}

    if args:
        command.extend(args)

    # Set up environment
    env = os.environ.copy()
    if env_vars:
        env.update(env_vars)

    try:
        start = time.time()
        result = subprocess.run(
            command, capture_output=True, text=True,
            timeout=timeout, cwd=cwd or str(script.parent), env=env
        )
        elapsed = time.time() - start

        return {
            "status": "success" if result.returncode == 0 else "error",
            "exit_code": result.returncode,
            "stdout": result.stdout,
            "stderr": result.stderr,
            "execution_time": elapsed,
            "script": str(script),
        }
    except subprocess.TimeoutExpired:
        return {"status": "timeout", "error": f"Script exceeded {timeout}s timeout"}
    except Exception as e:
        return {"status": "error", "error": str(e)}
```

### Running Multiple Scripts

When a user says "run all my benchmark scripts":

```python
def run_multiple_scripts(script_paths, parallel=False, max_workers=4):
    """Run multiple scripts, optionally in parallel."""
    results = {}

    if parallel:
        from concurrent.futures import ProcessPoolExecutor, as_completed
        with ProcessPoolExecutor(max_workers=max_workers) as executor:
            futures = {
                executor.submit(run_script, sp): sp for sp in script_paths
            }
            for future in as_completed(futures):
                sp = futures[future]
                results[str(sp)] = future.result()
    else:
        for sp in script_paths:
            print(f"Running: {sp}")
            results[str(sp)] = run_script(sp)
            status = results[str(sp)]["status"]
            print(f"  -> {status}")

    return results
```

### Natural Language Script Commands

| User Says | What Crush Does |
|-----------|----------------|
| "run my_experiment.py" | Execute with Python, show output |
| "run benchmark.sh with args --qubits 10" | Execute bash script with arguments |
| "compile and run simulation.cpp" | g++ compile, then execute binary |
| "run all .py files in scripts/" | Execute each Python file in scripts/ directory |
| "run these scripts in parallel" | Use ProcessPoolExecutor |
| "run my_script.py with BACKEND=cirq" | Set environment variable, then execute |
| "run simulation.py 5 times and average" | Execute repeatedly, compute mean of results |

---

## 2. Code Modification Workflows

### Making Changes to Backend Source Code

When a user asks to modify a backend, Crush should:

1. Understand the change requested.
2. Locate the relevant source file(s).
3. Create a Git branch for the changes.
4. Make the edits.
5. Rebuild the backend.
6. Run tests to verify.

### Workflow Template

```bash
# 1. Create a feature branch
cd backends/lret-phase7
git checkout -b feature/user-requested-change

# 2. Make edits (Crush does this via file editing tools)
# Example: Add a new noise model to the LRET simulator

# 3. Rebuild
cd build && cmake .. && make -j$(nproc)

# 4. Run tests
ctest --output-on-failure

# 5. If tests pass, inform the user
# 6. If tests fail, show errors and suggest fixes
```

### Common Modification Requests

| Request | Files to Modify | Rebuild Required |
|---------|----------------|------------------|
| "Add depolarizing noise to LRET" | `src/noise_model.cpp`, `include/noise.h` | Yes (cmake + make) |
| "Change default qubit count to 12" | Config files or main entry | Depends on backend |
| "Add custom gate" | Gate definition files + circuit builder | Yes |
| "Enable GPU support" | CMakeLists.txt + GPU kernel files | Yes (cmake -DGPU=ON) |
| "Add new circuit type" | Circuit generator module | Python: No, C++: Yes |
| "Change measurement output format" | Output/reporter module | Depends on backend |
| "Optimize for large qubit counts" | Memory management modules | Yes |

### Creating Patches

When a user wants to share changes:

```bash
# Create a patch from current changes
git diff > my_changes.patch

# Apply a patch to a fresh clone
git apply my_changes.patch
```

---

## 3. Dependency Management

### Complete Dependency Installation

When a user says "set up everything for me," Crush should run the appropriate installer:

#### Python Dependencies (All Backends)

```bash
# Create virtual environment
python -m venv .venv
# Activate (Linux/macOS)
source .venv/bin/activate
# Activate (Windows PowerShell)
.\.venv\Scripts\Activate.ps1

# Install all Python backends
pip install --upgrade pip
pip install cirq qiskit qiskit-aer qsimcirq pennylane stim matplotlib numpy scipy

# Optional GPU backends
pip install cuquantum-python  # Requires NVIDIA GPU + CUDA
pip install pennylane-lightning-gpu  # PennyLane GPU acceleration

# Verify all imports
python -c "
import cirq; print(f'Cirq {cirq.__version__}')
import qiskit; print(f'Qiskit {qiskit.__version__}')
import qiskit_aer; print(f'Qiskit Aer {qiskit_aer.__version__}')
import qsimcirq; print(f'qsimcirq OK')
import pennylane; print(f'PennyLane {pennylane.__version__}')
import stim; print(f'Stim {stim.__version__}')
print('All backends ready!')
"
```

#### System Dependencies (C++ Backends)

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install -y build-essential cmake git python3-dev python3-pip \
    libopenblas-dev libomp-dev gcc g++
```

**macOS:**
```bash
brew install cmake gcc libomp
```

**Windows:**
```powershell
# Install via winget or chocolatey
winget install CMake.CMake
winget install GnuWin32.Make
# Or use Visual Studio Build Tools
winget install Microsoft.VisualStudio.2022.BuildTools
```

### Dependency Conflict Resolution

| Conflict | Solution |
|----------|----------|
| qiskit and cirq version clash | Use separate venvs or `pip install --upgrade` both |
| CUDA version mismatch | Match cuQuantum to installed CUDA: `nvcc --version` |
| CMake too old for QuEST | Upgrade: `pip install cmake --upgrade` or `brew install cmake` |
| OpenMP not found on macOS | `brew install libomp && export CPPFLAGS="-I$(brew --prefix libomp)/include"` |
| numpy version conflict | `pip install 'numpy>=1.24,<2.0'` |

### Environment Health Check

```python
#!/usr/bin/env python3
"""Comprehensive environment health check."""
import sys, os, platform, shutil

def check_environment():
    checks = {}

    # Python
    checks["python"] = {
        "version": sys.version,
        "executable": sys.executable,
        "ok": sys.version_info >= (3, 9),
    }

    # System tools
    for tool in ["cmake", "make", "gcc", "g++", "git", "nvcc"]:
        path = shutil.which(tool)
        checks[tool] = {"available": path is not None, "path": path}

    # Python packages
    packages = {
        "cirq": "cirq",
        "qiskit": "qiskit",
        "qiskit_aer": "qiskit-aer",
        "qsimcirq": "qsimcirq",
        "pennylane": "pennylane",
        "stim": "stim",
        "numpy": "numpy",
        "matplotlib": "matplotlib",
        "scipy": "scipy",
    }
    for import_name, pip_name in packages.items():
        try:
            mod = __import__(import_name)
            ver = getattr(mod, "__version__", "unknown")
            checks[pip_name] = {"installed": True, "version": ver}
        except ImportError:
            checks[pip_name] = {"installed": False, "install": f"pip install {pip_name}"}

    # GPU
    try:
        import torch
        checks["gpu_cuda"] = {"available": torch.cuda.is_available(),
                              "device": torch.cuda.get_device_name(0) if torch.cuda.is_available() else None}
    except ImportError:
        checks["gpu_cuda"] = {"available": False, "note": "Install torch to check GPU"}

    # System info
    checks["system"] = {
        "os": platform.system(),
        "arch": platform.machine(),
        "cores": os.cpu_count(),
    }

    # Display
    print("Environment Health Check")
    print("=" * 50)
    for name, info in checks.items():
        if isinstance(info, dict):
            ok = info.get("ok", info.get("available", info.get("installed", False)))
            status = "OK" if ok else "MISSING"
            detail = info.get("version", info.get("path", info.get("install", "")))
            print(f"  {status:>7} | {name:<15} | {detail}")

    return checks

if __name__ == "__main__":
    check_environment()
```

---

## 4. Repository Cloning Workflows

### Clone All Backends at Once

When a user says "clone all backends" or "set up workspace":

```bash
#!/bin/bash
# clone_all_backends.sh — Clone all quantum simulation backends
set -e

WORKSPACE="$(pwd)/backends"
mkdir -p "$WORKSPACE"

echo "=== Cloning LRET Branches ==="
git clone -b cirq-scalability-comparison \
    https://github.com/kunal5556/LRET.git "$WORKSPACE/lret-cirq-scalability"
git clone -b pennylane-documentation-benchmarking \
    https://github.com/kunal5556/LRET.git "$WORKSPACE/lret-pennylane"
git clone -b phase-7 \
    https://github.com/kunal5556/LRET.git "$WORKSPACE/lret-phase7"

echo "=== Cloning QuEST ==="
git clone https://github.com/QuEST-Kit/QuEST.git "$WORKSPACE/quest"

echo "=== Cloning qsim (source) ==="
git clone https://github.com/quantumlib/qsim.git "$WORKSPACE/qsim-source"

echo "=== Cloning Cirq (source, optional) ==="
git clone https://github.com/quantumlib/Cirq.git "$WORKSPACE/cirq-source"

echo ""
echo "Python backends (install via pip, no clone needed):"
echo "  pip install cirq qiskit qiskit-aer qsimcirq pennylane stim"
echo ""
echo "All backends cloned to: $WORKSPACE/"
ls -1 "$WORKSPACE/"
```

**PowerShell equivalent for Windows:**

```powershell
# clone_all_backends.ps1
$Workspace = Join-Path (Get-Location) "backends"
New-Item -ItemType Directory -Force -Path $Workspace | Out-Null

Write-Host "=== Cloning LRET Branches ==="
git clone -b cirq-scalability-comparison `
    https://github.com/kunal5556/LRET.git "$Workspace\lret-cirq-scalability"
git clone -b pennylane-documentation-benchmarking `
    https://github.com/kunal5556/LRET.git "$Workspace\lret-pennylane"
git clone -b phase-7 `
    https://github.com/kunal5556/LRET.git "$Workspace\lret-phase7"

Write-Host "=== Cloning QuEST ==="
git clone https://github.com/QuEST-Kit/QuEST.git "$Workspace\quest"

Write-Host "=== Cloning qsim (source) ==="
git clone https://github.com/quantumlib/qsim.git "$Workspace\qsim-source"

Write-Host ""
Write-Host "Python backends (install via pip):"
Write-Host "  pip install cirq qiskit qiskit-aer qsimcirq pennylane stim"
Write-Host ""
Write-Host "All backends cloned to: $Workspace"
Get-ChildItem $Workspace -Name
```

---

## 5. Comprehensive Troubleshooting

### Common Issues and Solutions

#### Build Failures

| Problem | Symptoms | Solution |
|---------|----------|----------|
| CMake not found | `cmake: command not found` | Install: `pip install cmake` or `brew install cmake` or `winget install CMake.CMake` |
| Missing C++ compiler | `gcc: command not found` | Ubuntu: `apt install build-essential` / macOS: `xcode-select --install` / Windows: Install VS Build Tools |
| OpenMP not available | Linker errors about `-lgomp` | Ubuntu: `apt install libomp-dev` / macOS: `brew install libomp` |
| CUDA not found | `nvcc: command not found` | Install CUDA Toolkit from NVIDIA. GPU backends are optional. |
| Linking errors | Undefined references | Check library paths: `export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH` |

#### Python Import Errors

| Problem | Solution |
|---------|----------|
| `ModuleNotFoundError: cirq` | `pip install cirq` |
| `ModuleNotFoundError: qiskit_aer` | `pip install qiskit-aer` |
| `ModuleNotFoundError: qsimcirq` | `pip install qsimcirq` |
| `ModuleNotFoundError: pennylane` | `pip install pennylane` |
| `ModuleNotFoundError: stim` | `pip install stim` |
| `ModuleNotFoundError: cuquantum` | `pip install cuquantum-python` (requires NVIDIA GPU) |
| Version conflict between packages | Create a clean venv: `python -m venv .venv && source .venv/bin/activate` |

#### Runtime Errors

| Problem | Symptoms | Solution |
|---------|----------|----------|
| Out of memory | `MemoryError` or process killed | Reduce qubit count. State vector needs 2^N * 16 bytes. 30 qubits = 16 GB. |
| Too slow | Execution hangs | Use timeout; try a faster backend (Stim for Clifford, qsim for statevector) |
| Wrong results | Fidelity < 0.99 | Check noise model; verify circuit construction; compare with analytical expectation |
| GPU OOM | CUDA out of memory | Reduce qubit count or use CPU fallback |
| Permission denied | Can't write results | Check file permissions: `chmod 755 results/` or run as appropriate user |

#### Crush-Specific Issues

| Problem | Solution |
|---------|----------|
| Crush not found | Install: `winget install charmbracelet.crush` or check PATH |
| Crush shows "no API key" | Run `crush login copilot` or set `ANTHROPIC_API_KEY` env var |
| Crush can't read agent files | Place agent .md files in the project root or configure `agents_dir` in crush.json |
| Crush timeout on long simulation | Increase timeout in crush.json or break into smaller tasks |
| Crush loses context mid-simulation | Use explicit result files; save intermediate JSON results |

---

## 6. Project Configuration Templates

### crush.json for Quantum Simulation Workspace

```json
{
  "$schema": "https://crush.ai/schema.json",
  "providers": {
    "copilot": {
      "enabled": true
    }
  },
  "agents": {
    "default": {
      "model": "copilot/claude-sonnet-4",
      "maxTokens": 16384
    }
  }
}
```

### Directory Structure for a Full Workspace

```
quantum-workspace/
├── crush.json              # Crush configuration
├── AGENTS.md               # Master agent instructions
├── agent1.md               # Quick reference
├── agent2.md               # Backend registry
├── agent3.md               # Execution systems
├── agent4.md               # Benchmarking
├── agent5.md               # Analysis & visualization
├── agent6.md               # Advanced features (this file)
├── backends/               # Cloned backend repositories
│   ├── lret-phase7/
│   ├── lret-cirq-scalability/
│   ├── lret-pennylane/
│   ├── quest/
│   ├── qsim-source/
│   └── cirq-source/
├── scripts/                # User and generated scripts
│   ├── run_benchmark.py
│   ├── compare_backends.py
│   ├── clone_all.sh
│   └── health_check.py
├── results/                # Benchmark outputs
│   ├── bench_20260206_001/
│   └── latest -> bench_20260206_001
├── plots/                  # Generated visualizations
│   ├── scalability.png
│   └── fidelity_comparison.png
└── .venv/                  # Python virtual environment
```

---

## 7. Quick Setup Checklist

When starting fresh, tell Crush:

> "Set up a quantum simulation workspace with all backends"

Crush should execute these steps:

1. Create directory structure.
2. Clone all LRET branches and other source backends.
3. Create Python virtual environment.
4. Install all Python packages.
5. Build C++ backends (LRET, QuEST).
6. Run environment health check.
7. Run a quick 4-qubit GHZ test on all available backends.
8. Report which backends are ready.

Expected output:

```
Workspace setup complete!

Ready backends:
  ✓ lret-phase7        (built, tested)
  ✓ lret-cirq-scal     (built, tested)
  ✓ lret-pennylane      (built, tested)
  ✓ cirq                (pip, verified)
  ✓ qiskit-aer          (pip, verified)
  ✓ qsim                (pip, verified)
  ✓ pennylane           (pip, verified)
  ✓ stim                (pip, verified)
  ✗ cuquantum           (no NVIDIA GPU detected)
  ✓ quest               (built, tested)

Quick test (4-qubit GHZ):
  All 9 backends produced correct GHZ state.
  Fastest: stim (0.0002s)
  Slowest: pennylane (0.023s)
```

---

## File Index

| File | Purpose |
|------|---------|
| [AGENTS.md](AGENTS.md) | Master configuration, backend registry, safety rules |
| [agent1.md](agent1.md) | Quick reference, Crush setup, natural language commands |
| [agent2.md](agent2.md) | Backend registry — clone URLs, dependencies, build instructions |
| [agent3.md](agent3.md) | Execution systems — run each backend, result schema, error handling |
| [agent4.md](agent4.md) | Benchmarking — parameter sweeps, parallel execution |
| [agent5.md](agent5.md) | Comparative analysis — metrics, ASCII/matplotlib charts, reports |
| [agent6.md](agent6.md) | Advanced — scripts, code modification, deps, troubleshooting (this file) |

---

_End of agent documentation. Return to [AGENTS.md](AGENTS.md) for the master overview._
