# Quantum Multi-Backend Agent — Part 3: Execution Systems

_Previous: [agent2.md](agent2.md) — Backend Registry & Dependencies_
_Next: [agent4.md](agent4.md) — Benchmarking & Parallel Execution_

---

## Overview

This file defines how Crush should run each backend, parse results, handle errors, and produce the standardized result schema used for cross-backend comparison.

---

## Standardized Result Schema

Every backend execution MUST produce results in this canonical JSON format so that cross-backend comparisons are possible:

```json
{
  "run_id": "exp_20260206_abc123",
  "backend": "lret-phase7",
  "status": "success",
  "config": {
    "n_qubits": 10,
    "depth": 20,
    "circuit_type": "ghz",
    "noise_model": null,
    "noise_strength": 0.0,
    "mode": "hybrid",
    "shots": 1000,
    "seed": 42
  },
  "metrics": {
    "execution_time_seconds": 1.234,
    "fidelity": 0.9987,
    "gate_count": 19,
    "circuit_depth": 10,
    "memory_peak_mb": 256.5,
    "num_qubits_simulated": 10
  },
  "measurements": {
    "histogram": {"0000000000": 512, "1111111111": 488},
    "total_shots": 1000
  },
  "environment": {
    "platform": "linux",
    "python_version": "3.11.5",
    "gpu_available": false,
    "cpu_cores": 8,
    "ram_gb": 16
  },
  "timestamps": {
    "started_at": "2026-02-06T10:30:00Z",
    "completed_at": "2026-02-06T10:30:01.234Z"
  },
  "raw_output": {
    "stdout": "...",
    "stderr": ""
  }
}
```

---

## Running LRET Backends (Cirq Scalability / PennyLane / Phase 7)

All three LRET branches share the same C++ simulator core. The differences are in which Python interfaces and comparison tools are available.

### Running via C++ Binary (quantum_sim)

```bash
# Basic simulation with JSON config
./backends/lret-phase7/build/quantum_sim samples/basic_gates.json

# Custom parameters via JSON
cat > /tmp/experiment.json << 'EOF'
{
  "n_qubits": 10,
  "depth": 20,
  "mode": "hybrid",
  "noise_level": 0.01,
  "circuit_type": "ghz",
  "output_format": "json"
}
EOF
./backends/lret-phase7/build/quantum_sim /tmp/experiment.json
```

### Running via Python Interface

```python
#!/usr/bin/env python3
"""Run LRET simulation via Python interface."""
import sys, json, time, os
sys.path.insert(0, "backends/lret-phase7/python")

from qlret import LRETSimulator  # Adjust import based on branch

config = {
    "n_qubits": 10,
    "depth": 20,
    "mode": "hybrid",
    "noise_level": 0.0,
    "circuit_type": "ghz",
}

start = time.time()
sim = LRETSimulator(**config)
result = sim.run()
elapsed = time.time() - start

output = {
    "run_id": f"lret_{int(time.time())}",
    "backend": "lret-phase7",
    "status": "success",
    "config": config,
    "metrics": {
        "execution_time_seconds": elapsed,
        "fidelity": result.get("fidelity", None),
        "gate_count": result.get("gate_count", None),
        "memory_peak_mb": result.get("memory_mb", None),
        "num_qubits_simulated": config["n_qubits"],
    },
    "measurements": result.get("measurements", {}),
}

print(json.dumps(output, indent=2))
```

### Running LRET PennyLane Hybrid

```python
#!/usr/bin/env python3
"""Run LRET via PennyLane device."""
import pennylane as qml
import numpy as np
import time, json

sys.path.insert(0, "backends/lret-pennylane/python")
from qlret.pennylane_device import LRETDevice

n_qubits = 10
dev = qml.device(LRETDevice, wires=n_qubits)

@qml.qnode(dev)
def ghz_circuit():
    qml.Hadamard(wires=0)
    for i in range(1, n_qubits):
        qml.CNOT(wires=[0, i])
    return qml.state()

start = time.time()
state = ghz_circuit()
elapsed = time.time() - start

fidelity = abs(state[0])**2 + abs(state[-1])**2
print(json.dumps({
    "backend": "lret-pennylane",
    "metrics": {"execution_time_seconds": elapsed, "fidelity": float(fidelity)},
    "config": {"n_qubits": n_qubits, "circuit_type": "ghz"},
}, indent=2))
```

### Running LRET C++ Tests
```bash
cd backends/lret-phase7/build
ctest --output-on-failure
# Or individual tests:
./test_simple
./test_fidelity
./test_autodiff
./test_checkpoint
```

---

## Running Google Cirq

```python
#!/usr/bin/env python3
"""Run Cirq simulation with standardized output."""
import cirq
import numpy as np
import time, json, uuid

def run_cirq(n_qubits=10, depth=20, circuit_type="ghz",
             noise_model=None, noise_strength=0.0, shots=1000, seed=42):
    qubits = cirq.LineQubit.range(n_qubits)

    # Build circuit
    if circuit_type == "ghz":
        circuit = cirq.Circuit()
        circuit.append(cirq.H(qubits[0]))
        for i in range(1, n_qubits):
            circuit.append(cirq.CNOT(qubits[0], qubits[i]))
        circuit.append(cirq.measure(*qubits, key='m'))
    elif circuit_type == "random":
        circuit = cirq.testing.random_circuit(qubits, n_moments=depth,
                                               op_density=0.8, random_state=seed)
        circuit.append(cirq.measure(*qubits, key='m'))
    elif circuit_type == "qft":
        circuit = cirq.Circuit()
        for i in range(n_qubits):
            circuit.append(cirq.H(qubits[i]))
            for j in range(i + 1, n_qubits):
                circuit.append(cirq.CZPowGate(exponent=1 / 2**(j - i))(qubits[j], qubits[i]))
        circuit.append(cirq.measure(*qubits, key='m'))
    else:
        circuit = cirq.testing.random_circuit(qubits, n_moments=depth,
                                               op_density=0.8, random_state=seed)
        circuit.append(cirq.measure(*qubits, key='m'))

    # Add noise if specified
    if noise_model == "depolarizing" and noise_strength > 0:
        noise = cirq.ConstantQubitNoiseModel(cirq.depolarize(noise_strength))
        noisy_circuit = cirq.Circuit(cirq.ops.noise_model_insertion.NoiseModelInserter(noise)(circuit))
    else:
        noisy_circuit = circuit

    # Choose simulator
    simulator = cirq.Simulator(seed=seed)

    # Run
    start = time.time()
    if shots > 0:
        result = simulator.run(noisy_circuit, repetitions=shots)
        elapsed = time.time() - start
        histogram = dict(result.histogram(key='m'))
        hist_str = {format(k, f'0{n_qubits}b'): v for k, v in histogram.items()}
    else:
        sv_result = simulator.simulate(noisy_circuit)
        elapsed = time.time() - start
        hist_str = {}

    return {
        "run_id": f"cirq_{uuid.uuid4().hex[:8]}",
        "backend": "cirq",
        "status": "success",
        "config": {
            "n_qubits": n_qubits, "depth": depth, "circuit_type": circuit_type,
            "noise_model": noise_model, "noise_strength": noise_strength, "shots": shots,
        },
        "metrics": {
            "execution_time_seconds": elapsed,
            "circuit_depth": len(circuit),
            "gate_count": sum(1 for _ in circuit.all_operations()),
            "num_qubits_simulated": n_qubits,
        },
        "measurements": {"histogram": hist_str, "total_shots": shots},
    }

if __name__ == "__main__":
    result = run_cirq(n_qubits=10, depth=20, circuit_type="ghz")
    print(json.dumps(result, indent=2))
```

---

## Running Qiskit Aer

```python
#!/usr/bin/env python3
"""Run Qiskit Aer simulation with standardized output."""
from qiskit import QuantumCircuit
from qiskit_aer import AerSimulator
import time, json, uuid

def run_qiskit_aer(n_qubits=10, depth=20, circuit_type="ghz",
                   noise_model=None, noise_strength=0.0, shots=1000,
                   method="automatic", device="CPU", seed=42):
    # Build circuit
    qc = QuantumCircuit(n_qubits, n_qubits)
    if circuit_type == "ghz":
        qc.h(0)
        for i in range(1, n_qubits):
            qc.cx(0, i)
    elif circuit_type == "qft":
        for i in range(n_qubits):
            qc.h(i)
            for j in range(i + 1, n_qubits):
                qc.cp(np.pi / 2**(j - i), j, i)
    elif circuit_type == "random":
        from qiskit.circuit.random import random_circuit
        qc = random_circuit(n_qubits, depth, seed=seed, measure=False)
        qc.add_register(qiskit.ClassicalRegister(n_qubits))

    qc.measure(range(n_qubits), range(n_qubits))

    # Configure simulator
    sim_opts = {"method": method}
    if device == "GPU":
        sim_opts["device"] = "GPU"
    sim = AerSimulator(**sim_opts)

    # Add noise if specified
    if noise_model == "depolarizing" and noise_strength > 0:
        from qiskit_aer.noise import NoiseModel, depolarizing_error
        nm = NoiseModel()
        error_1q = depolarizing_error(noise_strength, 1)
        error_2q = depolarizing_error(noise_strength, 2)
        nm.add_all_qubit_quantum_error(error_1q, ['h', 'x', 'y', 'z', 's', 't'])
        nm.add_all_qubit_quantum_error(error_2q, ['cx', 'cz'])
        sim = AerSimulator(noise_model=nm, **sim_opts)

    start = time.time()
    result = sim.run(qc, shots=shots, seed_simulator=seed).result()
    elapsed = time.time() - start

    counts = result.get_counts()
    return {
        "run_id": f"qiskit_{uuid.uuid4().hex[:8]}",
        "backend": "qiskit-aer",
        "status": "success",
        "config": {
            "n_qubits": n_qubits, "depth": depth, "circuit_type": circuit_type,
            "noise_model": noise_model, "noise_strength": noise_strength,
            "shots": shots, "method": method, "device": device,
        },
        "metrics": {
            "execution_time_seconds": elapsed,
            "circuit_depth": qc.depth(),
            "gate_count": qc.size(),
            "num_qubits_simulated": n_qubits,
        },
        "measurements": {"histogram": counts, "total_shots": shots},
    }

if __name__ == "__main__":
    import numpy as np
    result = run_qiskit_aer(n_qubits=10, circuit_type="ghz")
    print(json.dumps(result, indent=2))
```

---

## Running QuEST

QuEST is a C library. Crush should generate a C file, compile it, and run it.

```c
/* run_quest.c — QuEST simulation runner */
#include "QuEST.h"
#include <stdio.h>
#include <time.h>

int main(int argc, char *argv[]) {
    int n_qubits = 10;
    if (argc > 1) n_qubits = atoi(argv[1]);

    QuESTEnv env = createQuESTEnv();
    Qureg qubits = createQureg(n_qubits, env);
    initZeroState(qubits);

    clock_t start = clock();

    /* GHZ circuit */
    hadamard(qubits, 0);
    for (int i = 1; i < n_qubits; i++)
        controlledNot(qubits, 0, i);

    clock_t end = clock();
    double elapsed = (double)(end - start) / CLOCKS_PER_SEC;

    /* Get fidelity vs ideal GHZ */
    Qureg ideal = createQureg(n_qubits, env);
    initZeroState(ideal);
    hadamard(ideal, 0);
    for (int i = 1; i < n_qubits; i++)
        controlledNot(ideal, 0, i);

    qreal fidelity = calcFidelity(qubits, ideal);

    printf("{\"backend\":\"quest\",\"status\":\"success\",");
    printf("\"config\":{\"n_qubits\":%d,\"circuit_type\":\"ghz\"},", n_qubits);
    printf("\"metrics\":{\"execution_time_seconds\":%.6f,\"fidelity\":%.10f,", elapsed, fidelity);
    printf("\"num_qubits_simulated\":%d}}\n", n_qubits);

    destroyQureg(qubits, env);
    destroyQureg(ideal, env);
    destroyQuESTEnv(env);
    return 0;
}
```

**Compile and run:**
```bash
cd backends/quest
gcc -o run_quest run_quest.c -I . -L build -lQuEST -lm -fopenmp
./run_quest 10
```

---

## Running qsim (via Python)

```python
#!/usr/bin/env python3
"""Run qsim simulation."""
import cirq
import qsimcirq
import time, json, uuid

def run_qsim(n_qubits=10, depth=20, circuit_type="ghz", use_gpu=False, seed=42):
    qubits = cirq.LineQubit.range(n_qubits)

    if circuit_type == "ghz":
        circuit = cirq.Circuit()
        circuit.append(cirq.H(qubits[0]))
        for i in range(1, n_qubits):
            circuit.append(cirq.CNOT(qubits[0], qubits[i]))
    else:
        circuit = cirq.testing.random_circuit(qubits, n_moments=depth,
                                               op_density=0.8, random_state=seed)

    options = {'t': 8, 'f': 2}  # threads, max fused gate size
    if use_gpu:
        options['g'] = True
    sim = qsimcirq.QSimSimulator(options)

    start = time.time()
    result = sim.simulate(circuit)
    elapsed = time.time() - start

    return {
        "run_id": f"qsim_{uuid.uuid4().hex[:8]}",
        "backend": "qsim",
        "status": "success",
        "config": {"n_qubits": n_qubits, "depth": depth, "circuit_type": circuit_type, "use_gpu": use_gpu},
        "metrics": {
            "execution_time_seconds": elapsed,
            "num_qubits_simulated": n_qubits,
            "gate_count": sum(1 for _ in circuit.all_operations()),
        },
    }

if __name__ == "__main__":
    print(json.dumps(run_qsim(10, circuit_type="ghz"), indent=2))
```

---

## Running PennyLane

```python
#!/usr/bin/env python3
"""Run PennyLane simulation."""
import pennylane as qml
import numpy as np
import time, json, uuid

def run_pennylane(n_qubits=10, depth=20, circuit_type="ghz",
                  device_name="default.qubit", shots=1000, seed=42):
    dev = qml.device(device_name, wires=n_qubits, shots=shots)

    @qml.qnode(dev)
    def ghz_circuit():
        qml.Hadamard(wires=0)
        for i in range(1, n_qubits):
            qml.CNOT(wires=[0, i])
        return qml.counts()

    @qml.qnode(dev)
    def random_circuit():
        for layer in range(depth):
            for i in range(n_qubits):
                qml.RX(np.random.uniform(0, 2*np.pi), wires=i)
                qml.RY(np.random.uniform(0, 2*np.pi), wires=i)
            for i in range(0, n_qubits - 1, 2):
                qml.CNOT(wires=[i, i+1])
        return qml.counts()

    np.random.seed(seed)
    circuit_fn = ghz_circuit if circuit_type == "ghz" else random_circuit

    start = time.time()
    counts = circuit_fn()
    elapsed = time.time() - start

    # Convert counts keys to strings
    histogram = {str(k): int(v) for k, v in counts.items()} if counts else {}

    return {
        "run_id": f"pennylane_{uuid.uuid4().hex[:8]}",
        "backend": f"pennylane-{device_name}",
        "status": "success",
        "config": {
            "n_qubits": n_qubits, "depth": depth, "circuit_type": circuit_type,
            "device": device_name, "shots": shots,
        },
        "metrics": {
            "execution_time_seconds": elapsed,
            "num_qubits_simulated": n_qubits,
        },
        "measurements": {"histogram": histogram, "total_shots": shots},
    }

if __name__ == "__main__":
    print(json.dumps(run_pennylane(10, circuit_type="ghz"), indent=2))
```

---

## Running Stim

```python
#!/usr/bin/env python3
"""Run Stim simulation (Clifford circuits only)."""
import stim
import time, json, uuid
from collections import Counter

def run_stim(n_qubits=10, circuit_type="ghz", shots=10000, seed=42):
    c = stim.Circuit()

    if circuit_type == "ghz":
        c.append("H", [0])
        for i in range(1, n_qubits):
            c.append("CNOT", [0, i])
    elif circuit_type == "random_clifford":
        import random
        random.seed(seed)
        gates_1q = ["H", "S", "X", "Y", "Z"]
        for _ in range(n_qubits * 5):
            gate = random.choice(gates_1q)
            qubit = random.randint(0, n_qubits - 1)
            c.append(gate, [qubit])
            if n_qubits > 1:
                q1, q2 = random.sample(range(n_qubits), 2)
                c.append("CNOT", [q1, q2])

    c.append("M", list(range(n_qubits)))

    start = time.time()
    sampler = c.compile_sampler(seed=seed)
    samples = sampler.sample(shots=shots)
    elapsed = time.time() - start

    # Build histogram
    bitstrings = [''.join(str(int(b)) for b in row) for row in samples]
    histogram = dict(Counter(bitstrings))

    return {
        "run_id": f"stim_{uuid.uuid4().hex[:8]}",
        "backend": "stim",
        "status": "success",
        "config": {"n_qubits": n_qubits, "circuit_type": circuit_type, "shots": shots},
        "metrics": {
            "execution_time_seconds": elapsed,
            "num_qubits_simulated": n_qubits,
        },
        "measurements": {"histogram": histogram, "total_shots": shots},
        "notes": "Stim only supports Clifford circuits (H, S, CNOT, etc.)"
    }

if __name__ == "__main__":
    print(json.dumps(run_stim(10, circuit_type="ghz"), indent=2))
```

---

## Error Handling

When running any backend, Crush should handle these errors:

| Error Type | Detection | Recovery |
|-----------|-----------|----------|
| **Import Error** | `ModuleNotFoundError` | Prompt user to install: `pip install <package>` |
| **Build Failure** | Non-zero exit code from cmake/make | Show error log, suggest checking dependencies |
| **Timeout** | Execution exceeds user-specified or default timeout (300s) | Kill process, report partial results |
| **Out of Memory** | MemoryError or OOM killer | Suggest fewer qubits or different backend |
| **GPU Error** | CUDA errors | Fallback to CPU, inform user |
| **Invalid Config** | Parameter validation failure | Show valid ranges and defaults |
| **File Not Found** | Binary or config file missing | Prompt to build or clone backend |

### Error Recovery Template
```python
import subprocess, json, sys

def run_backend_safe(command, timeout=300):
    """Run a backend command with error handling."""
    try:
        result = subprocess.run(
            command, shell=True, capture_output=True, text=True, timeout=timeout
        )
        if result.returncode != 0:
            return {"status": "error", "error": result.stderr, "exit_code": result.returncode}
        try:
            return json.loads(result.stdout)
        except json.JSONDecodeError:
            return {"status": "success", "raw_output": result.stdout}
    except subprocess.TimeoutExpired:
        return {"status": "timeout", "error": f"Exceeded {timeout}s timeout"}
    except FileNotFoundError:
        return {"status": "error", "error": "Backend binary not found. Please build first."}
    except MemoryError:
        return {"status": "error", "error": "Out of memory. Try fewer qubits."}
```

---

## Result Storage

All results should be saved to `results/` with timestamped filenames:

```bash
results/
├── lret-phase7_10q_ghz_20260206_103000.json
├── cirq_10q_ghz_20260206_103005.json
├── qiskit-aer_10q_ghz_20260206_103010.json
└── comparison_20260206_103015.json
```

**Naming convention:** `{backend}_{qubits}q_{circuit}_{date}_{time}.json`

---

_Continue to [agent4.md](agent4.md) for Benchmarking & Parallel Execution — multi-backend benchmarks, parameter sweeps, and simultaneous runs._
