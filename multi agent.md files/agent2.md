# Quantum Multi-Backend Agent — Part 2: Backend Registry & Dependencies

_Previous: [agent1.md](agent1.md) — Quick Reference & Getting Started_
_Next: [agent3.md](agent3.md) — Execution Systems_

---

## Backend Registry

This file contains the complete clone, dependency, and build instructions for every supported quantum simulation backend. Crush should use this as the authoritative reference when the user asks to set up, clone, build, or install dependencies for any backend.

---

## 1. LRET Cirq Scalability

**Purpose:** LRET branch focused on Cirq-vs-LRET scalability comparison benchmarks.

### Clone
```bash
git clone --branch cirq-scalability-comparison https://github.com/kunal5556/LRET.git backends/lret-cirq-scalability
cd backends/lret-cirq-scalability
```

### Dependencies

**System (C++ build):**
| Package | Install Command (Ubuntu) | Install Command (macOS) | Install Command (Windows) |
|---------|------------------------|------------------------|--------------------------|
| CMake 3.16+ | `sudo apt install cmake` | `brew install cmake` | `winget install Kitware.CMake` |
| GCC/G++ 9+ or Clang 10+ | `sudo apt install g++` | `xcode-select --install` | Install Visual Studio Build Tools |
| Eigen3 | `sudo apt install libeigen3-dev` | `brew install eigen` | `vcpkg install eigen3` |
| OpenMP | `sudo apt install libomp-dev` | `brew install libomp` | Included with MSVC |
| MPI (optional) | `sudo apt install libopenmpi-dev` | `brew install open-mpi` | Install MS-MPI |
| HDF5 (optional) | `sudo apt install libhdf5-dev` | `brew install hdf5` | `vcpkg install hdf5` |

**Python:**
```bash
pip install numpy scipy matplotlib pennylane cirq qiskit pytest
```

### Build
```bash
cd backends/lret-cirq-scalability
mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . -j$(nproc)    # Linux/macOS
cmake --build . -j%NUMBER_OF_PROCESSORS%   # Windows
```

**Windows (PowerShell) alternative:**
```powershell
cd backends\lret-cirq-scalability
mkdir build -Force; cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release --parallel
```

### Verify
```bash
./build/quantum_sim --help
./build/test_simple
cd python && pytest tests/ -v
```

### Key Files
- `src/fdm_simulator.cpp` — FDM simulator core
- `src/simd_kernels.cpp` — SIMD-optimized operations
- `python/qlret/cirq_compare.py` — Cirq comparison utilities
- `samples/` — JSON circuit configurations

---

## 2. LRET PennyLane Hybrid

**Purpose:** LRET branch with PennyLane device integration, hybrid quantum-classical workflows, documentation and benchmarking.

### Clone
```bash
git clone --branch pennylane-documentation-benchmarking https://github.com/kunal5556/LRET.git backends/lret-pennylane
cd backends/lret-pennylane
```

### Dependencies

Same C++ system dependencies as LRET Cirq Scalability (see above), plus:

**Python (additional):**
```bash
pip install pennylane pennylane-lightning jax jaxlib autograd
```

### Build
```bash
cd backends/lret-pennylane
mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . -j$(nproc)
```

### Verify
```bash
./build/quantum_sim --help
python -c "from qlret.pennylane_device import LRETDevice; print('PennyLane device OK')"
cd python && pytest tests/ -v
```

### Key Files
- `python/qlret/pennylane_device.py` — PennyLane quantum device
- `python/qlret/jax_interface.py` — JAX integration for autodiff
- `docs/` — Documentation and benchmarking guides

---

## 3. LRET Phase 7 Unified

**Purpose:** LRET unified branch (phase-7) with all features: QEC, autodiff, GPU, MPI, checkpointing.

### Clone
```bash
git clone --branch phase-7 https://github.com/kunal5556/LRET.git backends/lret-phase7
cd backends/lret-phase7
```

### Dependencies

Same C++ system dependencies as LRET Cirq Scalability, plus:

| Package | Install Command (Ubuntu) | Purpose |
|---------|------------------------|---------|
| CUDA Toolkit 11+ | `sudo apt install nvidia-cuda-toolkit` | GPU acceleration |
| cuDNN | NVIDIA Developer Downloads | Neural network decoder |

**Python:**
```bash
pip install numpy scipy matplotlib pennylane cirq jax jaxlib torch pytest
```

### Build (with GPU)
```bash
cd backends/lret-phase7
mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DUSE_CUDA=ON -DUSE_MPI=ON
cmake --build . -j$(nproc)
```

### Build (without GPU)
```bash
cmake .. -DCMAKE_BUILD_TYPE=Release -DUSE_CUDA=OFF
cmake --build . -j$(nproc)
```

### Verify
```bash
./build/quantum_sim samples/basic_gates.json
./build/test_simple
./build/test_fidelity
./build/test_qec_surface_code  # QEC tests
ctest --test-dir build
```

### Key Files
- `src/gpu_simulator.cu` — CUDA-accelerated simulation
- `src/qec_adaptive.cpp` — Adaptive QEC with ML-driven decoding
- `src/mpi_parallel.cpp` — MPI parallelization
- `src/checkpoint.cpp` — State checkpointing
- `src/autodiff.cpp` — Automatic differentiation

---

## 4. Google Cirq

**Purpose:** Google's open-source quantum computing framework. State-vector and density-matrix simulation, noise models, VQE/QAOA, hardware integration.

### Clone
```bash
git clone https://github.com/quantumlib/Cirq.git backends/cirq
cd backends/cirq
```

### Dependencies (Python only)
```bash
pip install cirq cirq-core cirq-google cirq-aqt cirq-ionq
# Optional high-performance simulator:
pip install qsimcirq
```

### Verify
```bash
python -c "import cirq; q = cirq.LineQubit(0); c = cirq.Circuit(cirq.H(q), cirq.measure(q)); print(cirq.Simulator().simulate(c))"
```

### No Build Required
Cirq is a pure Python package. No CMake or C++ compilation needed.

### Key Components
- `cirq.Simulator()` — State vector simulator
- `cirq.DensityMatrixSimulator()` — Density matrix simulator
- `cirq.noise` — Noise models (depolarizing, amplitude damping, phase damping)
- `cirq.google` — Google hardware access
- `cirq.vis` — Circuit visualization

---

## 5. Qiskit Aer

**Purpose:** IBM's high-performance quantum simulation backend supporting state vector, density matrix, stabilizer, MPS, and GPU acceleration.

### Install (pip — recommended)
```bash
pip install qiskit qiskit-aer
# With GPU support:
pip install qiskit-aer-gpu
```

### Clone (from source — for development/modification)
```bash
git clone https://github.com/Qiskit/qiskit-aer.git backends/qiskit-aer
cd backends/qiskit-aer
pip install -e .
```

### Build from Source (with GPU)
```bash
cd backends/qiskit-aer
pip install -r requirements-dev.txt
python setup.py bdist_wheel -- -DAER_THRUST_BACKEND=CUDA -DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc
pip install dist/*.whl
```

### Dependencies
| Package | Install Command | Purpose |
|---------|----------------|---------|
| Python 3.9+ | System install | Runtime |
| Qiskit | `pip install qiskit` | Core framework |
| CUDA 11+ (optional) | System install | GPU acceleration |
| OpenMP | System install | CPU parallelism |
| OpenBLAS/MKL | `pip install numpy` (bundles BLAS) | Linear algebra |

### Verify
```bash
python -c "
from qiskit import QuantumCircuit
from qiskit_aer import AerSimulator
qc = QuantumCircuit(2)
qc.h(0); qc.cx(0,1); qc.measure_all()
sim = AerSimulator()
result = sim.run(qc, shots=1000).result()
print(result.get_counts())
"
```

### Simulation Methods
| Method | Use Case | Max Qubits |
|--------|----------|------------|
| `statevector` | Pure state, exact | ~30 (RAM limited) |
| `density_matrix` | Noisy circuits | ~15 |
| `stabilizer` | Clifford circuits | 1000+ |
| `matrix_product_state` | Low-entanglement | ~50 |
| `automatic` | Auto-select best | Varies |

---

## 6. QuEST (Quantum Exact Simulation Toolkit)

**Purpose:** High-performance C simulator for state-vector and density-matrix simulation. Supports OpenMP multithreading, MPI distributed computing, and GPU acceleration.

### Clone
```bash
git clone https://github.com/QuEST-Kit/QuEST.git backends/quest
cd backends/quest
```

### Dependencies
| Package | Install Command (Ubuntu) | Purpose |
|---------|------------------------|---------|
| CMake 3.1+ | `sudo apt install cmake` | Build system |
| GCC/G++ | `sudo apt install gcc g++` | C/C++ compiler |
| OpenMP | `sudo apt install libomp-dev` | CPU parallelism |
| MPI (optional) | `sudo apt install libopenmpi-dev` | Distributed computing |
| CUDA (optional) | NVIDIA CUDA Toolkit | GPU acceleration |

### Build (CPU + OpenMP)
```bash
cd backends/quest
mkdir build && cd build
cmake .. -DMULTITHREADED=ON
make -j$(nproc)
```

### Build (GPU)
```bash
cmake .. -DGPUACCELERATED=ON -DGPU_COMPUTE_CAPABILITY=70
make -j$(nproc)
```

### Build (Distributed MPI)
```bash
cmake .. -DDISTRIBUTED=ON -DMULTITHREADED=ON
make -j$(nproc)
```

### Verify
```bash
# Run QuEST example (after building with examples enabled)
cmake .. -DMULTITHREADED=ON -DUSER_SOURCE=examples/tutorial_example.c
make -j$(nproc)
./demo
```

### Python Wrapper (PyQuEST)
```bash
pip install pyquest
python -c "import pyquest; print('PyQuEST OK')"
```

### Key API
- `createQureg(numQubits, env)` — Create quantum register
- `hadamard(qureg, qubit)` — Apply Hadamard gate
- `controlledNot(qureg, ctrl, target)` — CNOT gate
- `calcFidelity(qureg, pureState)` — Calculate fidelity
- `getNumAmps(qureg)` — Get number of amplitudes

---

## 7. qsim (Google's Fast Simulator)

**Purpose:** Google's optimized C++ state-vector simulator, designed for high-performance simulation of quantum circuits. Achieves significant speedups via SIMD, OpenMP, and CUDA.

### Install (pip — recommended)
```bash
pip install qsimcirq
```

### Clone (from source)
```bash
git clone https://github.com/quantumlib/qsim.git backends/qsim
cd backends/qsim
```

### Build from Source
```bash
cd backends/qsim
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
```

### Build with CUDA
```bash
cmake .. -DCMAKE_BUILD_TYPE=Release -DQSIM_CUDA=ON
make -j$(nproc)
```

### Dependencies
| Package | Purpose |
|---------|---------|
| Python 3.9+ | Runtime |
| Cirq | Circuit definition |
| CMake 3.18+ | Build (source only) |
| CUDA 11+ (optional) | GPU acceleration |

### Verify
```bash
python -c "
import cirq
import qsimcirq
q = cirq.LineQubit.range(2)
c = cirq.Circuit(cirq.H(q[0]), cirq.CNOT(q[0], q[1]))
sim = qsimcirq.QSimSimulator()
result = sim.simulate(c)
print(result.final_state_vector)
"
```

---

## 8. cuQuantum (NVIDIA)

**Purpose:** NVIDIA's SDK for accelerating quantum circuit simulation on GPUs. Includes cuStateVec (state-vector) and cuTensorNet (tensor-network) libraries.

### Install
```bash
pip install cuquantum cuquantum-python
# Also install the Cirq/Qiskit integrations:
pip install cuquantum-python[all]
```

### Dependencies
| Package | Requirement |
|---------|-------------|
| NVIDIA GPU | Compute Capability 7.0+ (Volta or newer) |
| CUDA 11.0+ | Required |
| cuDNN (optional) | For tensor network contraction |
| Python 3.9+ | Runtime |

### Verify
```bash
python -c "
import cuquantum
print(f'cuQuantum version: {cuquantum.__version__}')
from cuquantum import custatevec
print('cuStateVec OK')
"
```

### Usage with Cirq
```python
import cirq
import qsimcirq
# qsimcirq auto-detects cuQuantum if available
sim = qsimcirq.QSimSimulator({'use_gpu': True})
```

### Usage with Qiskit
```python
from qiskit_aer import AerSimulator
sim = AerSimulator(method='statevector', device='GPU')
```

### No Build Required
cuQuantum is distributed as pip packages and NVIDIA SDK. No source compilation needed for typical use.

---

## 9. PennyLane

**Purpose:** Xanadu's cross-platform quantum ML library. Provides a unified interface to many quantum backends (simulators and hardware), automatic differentiation for quantum circuits, and hybrid quantum-classical optimization.

### Install
```bash
pip install pennylane
# Device plugins:
pip install pennylane-lightning    # Fast C++ simulator
pip install pennylane-cirq         # Cirq backend
pip install pennylane-qiskit       # Qiskit backend
pip install pennylane-lightning-gpu  # GPU (CUDA) simulator
```

### Clone (from source)
```bash
git clone https://github.com/PennyLaneAI/pennylane.git backends/pennylane
cd backends/pennylane
pip install -e .
```

### Dependencies
| Package | Purpose |
|---------|---------|
| Python 3.9+ | Runtime |
| NumPy | Array operations |
| autograd | Classical autodiff |
| toml | Configuration |

### Verify
```bash
python -c "
import pennylane as qml
dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev)
def circuit():
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])
    return qml.state()

print(circuit())
"
```

### Available Devices
| Device | Backend | GPU |
|--------|---------|-----|
| `default.qubit` | PennyLane built-in | No |
| `lightning.qubit` | C++ optimized | No |
| `lightning.gpu` | CUDA-accelerated | Yes |
| `cirq.simulator` | Google Cirq | No |
| `qiskit.aer` | IBM Qiskit Aer | Optional |

---

## 10. Stim

**Purpose:** Google's ultra-fast Clifford circuit simulator. Specialized for quantum error correction research — can simulate stabilizer circuits with millions of qubits.

### Install
```bash
pip install stim
```

### Clone (from source)
```bash
git clone https://github.com/quantumlib/Stim.git backends/stim
cd backends/stim
pip install -e .
```

### Build from Source (C++)
```bash
cd backends/stim
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

### Verify
```bash
python -c "
import stim
c = stim.Circuit()
c.append('H', [0])
c.append('CNOT', [0, 1])
c.append('M', [0, 1])
sampler = c.compile_sampler()
print(sampler.sample(shots=10))
"
```

### Limitations
- Only supports Clifford gates (H, S, CNOT, etc.) — not universal.
- Best for error correction studies, not general quantum algorithms.
- Extremely fast: can simulate millions of qubits for Clifford circuits.

---

## Dependency Quick-Install Scripts

### Install All Python Backends at Once
```bash
pip install cirq qiskit qiskit-aer pennylane pennylane-lightning stim qsimcirq numpy scipy matplotlib pytest
```

### Install All C++ Build Tools (Ubuntu)
```bash
sudo apt update && sudo apt install -y \
  cmake g++ libeigen3-dev libhdf5-dev libopenmpi-dev libomp-dev \
  python3-pip python3-venv git
```

### Install All C++ Build Tools (macOS)
```bash
brew install cmake eigen hdf5 open-mpi libomp
```

### Install All C++ Build Tools (Windows)
```powershell
# Install Visual Studio Build Tools (includes MSVC, CMake)
winget install Microsoft.VisualStudio.2022.BuildTools
# Install CMake separately if needed
winget install Kitware.CMake
# Install vcpkg for Eigen3
git clone https://github.com/microsoft/vcpkg.git
.\vcpkg\bootstrap-vcpkg.bat
.\vcpkg\vcpkg install eigen3
```

---

## Backend Readiness Check

Crush should run this validation before attempting to use any backend:

```python
#!/usr/bin/env python3
"""Check which quantum backends are available."""
import importlib, shutil, subprocess, sys

backends = {
    "cirq": {"module": "cirq", "type": "python"},
    "qiskit_aer": {"module": "qiskit_aer", "type": "python"},
    "pennylane": {"module": "pennylane", "type": "python"},
    "stim": {"module": "stim", "type": "python"},
    "qsimcirq": {"module": "qsimcirq", "type": "python"},
    "cuquantum": {"module": "cuquantum", "type": "python"},
}

tools = {"cmake": "cmake --version", "g++": "g++ --version", "nvcc": "nvcc --version", "mpirun": "mpirun --version"}

print("=== Python Backend Availability ===")
for name, info in backends.items():
    try:
        mod = importlib.import_module(info["module"])
        ver = getattr(mod, "__version__", "unknown")
        print(f"  ✓ {name}: {ver}")
    except ImportError:
        print(f"  ✗ {name}: NOT INSTALLED")

print("\n=== Build Tool Availability ===")
for name, cmd in tools.items():
    if shutil.which(name):
        print(f"  ✓ {name}: available")
    else:
        print(f"  ✗ {name}: NOT FOUND")

print("\n=== LRET Backend Directories ===")
import os
backend_dirs = [
    "backends/lret-cirq-scalability",
    "backends/lret-pennylane",
    "backends/lret-phase7",
    "backends/cirq",
    "backends/qiskit-aer",
    "backends/quest",
    "backends/qsim",
    "backends/stim",
]
for d in backend_dirs:
    status = "✓ CLONED" if os.path.isdir(d) else "✗ NOT CLONED"
    build_status = ""
    build_dir = os.path.join(d, "build")
    if os.path.isdir(build_dir):
        build_status = " (built)"
    print(f"  {status}: {d}{build_status}")
```

Save this as `scripts/check_backends.py` and run with:
```bash
python scripts/check_backends.py
```

---

_Continue to [agent3.md](agent3.md) for Execution Systems — how to run each backend, result parsing, and error handling._
