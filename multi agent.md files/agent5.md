# Quantum Multi-Backend Agent — Part 5: Comparative Analysis & Visualization

_Previous: [agent4.md](agent4.md) — Benchmarking & Parallel Execution_
_Next: [agent6.md](agent6.md) — Advanced Features_

---

## Overview

This file defines how Crush should analyze benchmark results across multiple backends, generate comparison metrics, produce visualizations, and create summary reports.

---

## Comparison Metrics

For every cross-backend comparison, Crush should compute and present these metrics:

| Metric | Formula / Source | Unit | Lower/Higher Better |
|--------|-----------------|------|---------------------|
| **Execution Time** | `metrics.execution_time_seconds` | seconds | Lower |
| **Fidelity** | `metrics.fidelity` | 0.0–1.0 | Higher |
| **Gate Count** | `metrics.gate_count` | count | Lower |
| **Circuit Depth** | `metrics.circuit_depth` | count | Lower |
| **Peak Memory** | `metrics.memory_peak_mb` | MB | Lower |
| **Throughput** | `shots / execution_time` | shots/sec | Higher |
| **Scalability** | Time at N+2 qubits / Time at N qubits | ratio | Lower |
| **Noise Tolerance** | Fidelity at noise=0.05 / Fidelity at noise=0 | ratio | Higher |

---

## Comparison Report Generator

```python
#!/usr/bin/env python3
"""
Generate a comparative analysis report from benchmark results.
Reads individual result JSONs from a benchmark directory.
"""
import json, os, sys
from pathlib import Path
from datetime import datetime

def load_results(benchmark_dir):
    """Load all result JSONs from a benchmark directory."""
    results = []
    for f in Path(benchmark_dir).glob("*.json"):
        if f.name in ("summary.json", "comparison.json", "report.md"):
            continue
        with open(f) as fp:
            data = json.load(fp)
            if data.get("status") == "success":
                results.append(data)
    return results

def group_by_backend(results):
    """Group results by backend name."""
    groups = {}
    for r in results:
        backend = r.get("backend", "unknown")
        groups.setdefault(backend, []).append(r)
    return groups

def group_by_qubits(results):
    """Group results by qubit count."""
    groups = {}
    for r in results:
        nq = r.get("config", {}).get("n_qubits", 0)
        groups.setdefault(nq, []).append(r)
    return groups

def compute_comparison(results):
    """Compute comparison table across backends."""
    by_backend = group_by_backend(results)

    comparison = {}
    for backend, runs in by_backend.items():
        times = [r["metrics"]["execution_time_seconds"] for r in runs
                 if "execution_time_seconds" in r.get("metrics", {})]
        fidelities = [r["metrics"]["fidelity"] for r in runs
                      if "fidelity" in r.get("metrics", {})]
        gate_counts = [r["metrics"]["gate_count"] for r in runs
                       if "gate_count" in r.get("metrics", {})]
        memory = [r["metrics"]["memory_peak_mb"] for r in runs
                  if "memory_peak_mb" in r.get("metrics", {})]

        comparison[backend] = {
            "num_runs": len(runs),
            "avg_time": sum(times) / len(times) if times else None,
            "min_time": min(times) if times else None,
            "max_time": max(times) if times else None,
            "avg_fidelity": sum(fidelities) / len(fidelities) if fidelities else None,
            "avg_gate_count": sum(gate_counts) / len(gate_counts) if gate_counts else None,
            "avg_memory_mb": sum(memory) / len(memory) if memory else None,
        }

    return comparison

# ─── Report Generation ───────────────────────────────────────────

def generate_markdown_report(benchmark_dir, results, comparison):
    """Generate a Markdown comparison report."""
    backends = sorted(comparison.keys())
    lines = []

    lines.append(f"# Benchmark Comparison Report")
    lines.append(f"")
    lines.append(f"**Generated:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    lines.append(f"**Benchmark:** {Path(benchmark_dir).name}")
    lines.append(f"**Total runs:** {len(results)}")
    lines.append(f"**Backends:** {', '.join(backends)}")
    lines.append(f"")

    # Summary table
    lines.append("## Summary")
    lines.append("")
    lines.append("| Backend | Runs | Avg Time (s) | Min Time (s) | Avg Fidelity | Avg Gates | Avg Memory (MB) |")
    lines.append("|---------|------|-------------|-------------|-------------|-----------|----------------|")
    for b in backends:
        c = comparison[b]
        lines.append(
            f"| {b} | {c['num_runs']} | "
            f"{c['avg_time']:.4f} | {c['min_time']:.4f} | "
            f"{c['avg_fidelity']:.6f if c['avg_fidelity'] else 'N/A'} | "
            f"{c['avg_gate_count']:.0f if c['avg_gate_count'] else 'N/A'} | "
            f"{c['avg_memory_mb']:.1f if c['avg_memory_mb'] else 'N/A'} |"
        )
    lines.append("")

    # Rankings
    lines.append("## Rankings")
    lines.append("")

    # Fastest
    ranked_time = sorted(
        [(b, c["avg_time"]) for b, c in comparison.items() if c["avg_time"]],
        key=lambda x: x[1]
    )
    if ranked_time:
        lines.append("### Fastest Execution")
        for i, (b, t) in enumerate(ranked_time, 1):
            medal = ["🥇", "🥈", "🥉"][i-1] if i <= 3 else f"{i}."
            lines.append(f"{medal} **{b}**: {t:.4f}s")
        lines.append("")

    # Highest fidelity
    ranked_fidelity = sorted(
        [(b, c["avg_fidelity"]) for b, c in comparison.items() if c["avg_fidelity"]],
        key=lambda x: x[1], reverse=True
    )
    if ranked_fidelity:
        lines.append("### Highest Fidelity")
        for i, (b, f) in enumerate(ranked_fidelity, 1):
            medal = ["🥇", "🥈", "🥉"][i-1] if i <= 3 else f"{i}."
            lines.append(f"{medal} **{b}**: {f:.8f}")
        lines.append("")

    # Scalability analysis
    lines.append("## Scalability Analysis")
    lines.append("")
    by_backend = group_by_backend(results)
    for backend in backends:
        runs = by_backend.get(backend, [])
        runs_sorted = sorted(runs, key=lambda r: r["config"].get("n_qubits", 0))
        if len(runs_sorted) >= 2:
            lines.append(f"### {backend}")
            lines.append("")
            lines.append("| Qubits | Time (s) | Scaling Factor |")
            lines.append("|--------|----------|----------------|")
            prev_time = None
            for r in runs_sorted:
                nq = r["config"]["n_qubits"]
                t = r["metrics"]["execution_time_seconds"]
                factor = f"{t / prev_time:.2f}x" if prev_time and prev_time > 0 else "—"
                lines.append(f"| {nq} | {t:.4f} | {factor} |")
                prev_time = t
            lines.append("")

    return "\n".join(lines)


if __name__ == "__main__":
    benchmark_dir = sys.argv[1] if len(sys.argv) > 1 else "results/latest"
    results = load_results(benchmark_dir)
    if not results:
        print(f"No results found in {benchmark_dir}")
        sys.exit(1)

    comparison = compute_comparison(results)
    report = generate_markdown_report(benchmark_dir, results, comparison)

    # Save report
    report_path = Path(benchmark_dir) / "report.md"
    with open(report_path, 'w') as f:
        f.write(report)
    print(f"Report saved to: {report_path}")

    # Also save comparison JSON
    comp_path = Path(benchmark_dir) / "comparison.json"
    with open(comp_path, 'w') as f:
        json.dump(comparison, f, indent=2)
    print(f"Comparison data saved to: {comp_path}")

    # Print summary to terminal
    print("\n" + report)
```

---

## Terminal-Friendly ASCII Visualization

For use inside the Crush TUI where graphical plots are not available:

```python
#!/usr/bin/env python3
"""ASCII chart generator for benchmark results."""

def ascii_bar_chart(data, title="", width=50, unit="s"):
    """
    Generate an ASCII horizontal bar chart.

    data: dict of {label: value}
    """
    if not data:
        return "No data to display."

    lines = []
    if title:
        lines.append(f"\n  {title}")
        lines.append(f"  {'=' * (width + 20)}")

    max_val = max(data.values())
    max_label_len = max(len(str(k)) for k in data.keys())

    for label, value in sorted(data.items(), key=lambda x: x[1]):
        bar_len = int((value / max_val) * width) if max_val > 0 else 0
        bar = "█" * bar_len + "░" * (width - bar_len)
        label_str = str(label).rjust(max_label_len)
        lines.append(f"  {label_str} │{bar}│ {value:.4f}{unit}")

    return "\n".join(lines)


def ascii_scalability_chart(backend_data, title="Scalability: Time vs Qubits"):
    """
    Generate an ASCII line chart showing qubit scalability.

    backend_data: dict of {backend_name: [(n_qubits, time), ...]}
    """
    lines = [f"\n  {title}", f"  {'=' * 60}"]

    # Collect all qubit values
    all_qubits = sorted(set(
        nq for points in backend_data.values() for nq, _ in points
    ))

    if not all_qubits:
        return "No scalability data to display."

    # Find max time for scaling
    max_time = max(t for points in backend_data.values() for _, t in points)
    chart_height = 15
    chart_width = len(all_qubits) * 4

    # Simple table format for terminal
    header = "  Qubits │ " + " │ ".join(
        f"{b:>12}" for b in backend_data.keys()
    )
    lines.append(header)
    lines.append("  " + "─" * len(header))

    for nq in all_qubits:
        row = f"  {nq:>6} │ "
        vals = []
        for backend, points in backend_data.items():
            point_dict = dict(points)
            if nq in point_dict:
                vals.append(f"{point_dict[nq]:>12.4f}")
            else:
                vals.append(f"{'N/A':>12}")
        row += " │ ".join(vals)
        lines.append(row)

    return "\n".join(lines)


def ascii_comparison_matrix(backends, metrics, title="Comparison Matrix"):
    """
    Generate a comparison matrix showing which backend wins each metric.

    backends: list of backend names
    metrics: dict of {metric_name: {backend: value}}
    """
    lines = [f"\n  {title}", ""]

    # Header
    header = f"  {'Metric':<20} │ " + " │ ".join(f"{b:>12}" for b in backends) + " │ Winner"
    lines.append(header)
    lines.append("  " + "─" * len(header))

    for metric, values in metrics.items():
        row = f"  {metric:<20} │ "
        valid_vals = {b: v for b, v in values.items() if v is not None}

        cells = []
        for b in backends:
            v = values.get(b)
            if v is not None:
                cells.append(f"{v:>12.4f}")
            else:
                cells.append(f"{'N/A':>12}")
        row += " │ ".join(cells)

        # Determine winner (lower is better for time/memory, higher for fidelity)
        if valid_vals:
            if metric in ("fidelity", "throughput", "noise_tolerance"):
                winner = max(valid_vals, key=valid_vals.get)
            else:
                winner = min(valid_vals, key=valid_vals.get)
            row += f" │ ← {winner}"
        lines.append(row)

    return "\n".join(lines)
```

### Example ASCII Output

```
  Execution Time by Backend (10 qubits, GHZ)
  ====================================================================
        stim │██████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ 0.0012s
        qsim │██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ 0.0089s
        cirq │██████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ 0.0234s
  qiskit-aer │███████████████████████████████░░░░░░░░░░░░░░░░░░░│ 0.0456s
  lret-phase7│████████████████████████████████████████████████░░│ 0.0678s
   pennylane │██████████████████████████████████████████████████│ 0.0891s
```

---

## Matplotlib Visualization (Optional)

When running in an environment where matplotlib is available, Crush should generate graphical plots:

```python
#!/usr/bin/env python3
"""Generate publication-quality comparison plots."""
import json, sys
import matplotlib.pyplot as plt
import numpy as np
from pathlib import Path

def plot_scalability(results, output_dir):
    """Plot execution time vs qubit count for all backends."""
    from collections import defaultdict
    backend_data = defaultdict(lambda: {"qubits": [], "times": []})

    for r in results:
        if r.get("status") != "success":
            continue
        backend = r["backend"]
        nq = r["config"]["n_qubits"]
        t = r["metrics"]["execution_time_seconds"]
        backend_data[backend]["qubits"].append(nq)
        backend_data[backend]["times"].append(t)

    fig, ax = plt.subplots(figsize=(10, 6))
    for backend, data in sorted(backend_data.items()):
        sorted_pairs = sorted(zip(data["qubits"], data["times"]))
        qubits, times = zip(*sorted_pairs)
        ax.semilogy(qubits, times, 'o-', label=backend, linewidth=2, markersize=6)

    ax.set_xlabel("Number of Qubits", fontsize=12)
    ax.set_ylabel("Execution Time (s)", fontsize=12)
    ax.set_title("Backend Scalability Comparison", fontsize=14)
    ax.legend(fontsize=10)
    ax.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig(Path(output_dir) / "scalability.png", dpi=150)
    plt.close()
    print(f"Saved: {output_dir}/scalability.png")


def plot_fidelity_comparison(results, output_dir):
    """Plot fidelity comparison bar chart."""
    from collections import defaultdict
    fidelities = defaultdict(list)

    for r in results:
        if r.get("status") != "success":
            continue
        f = r.get("metrics", {}).get("fidelity")
        if f is not None:
            fidelities[r["backend"]].append(f)

    if not fidelities:
        print("No fidelity data to plot.")
        return

    backends = sorted(fidelities.keys())
    avg_fids = [np.mean(fidelities[b]) for b in backends]
    std_fids = [np.std(fidelities[b]) for b in backends]

    fig, ax = plt.subplots(figsize=(10, 6))
    colors = plt.cm.Set2(np.linspace(0, 1, len(backends)))
    bars = ax.bar(backends, avg_fids, yerr=std_fids, capsize=5, color=colors, edgecolor='black')

    ax.set_ylabel("Fidelity", fontsize=12)
    ax.set_title("Fidelity Comparison Across Backends", fontsize=14)
    ax.set_ylim(min(avg_fids) - 0.01, 1.0)
    ax.grid(True, axis='y', alpha=0.3)
    plt.tight_layout()
    plt.savefig(Path(output_dir) / "fidelity_comparison.png", dpi=150)
    plt.close()
    print(f"Saved: {output_dir}/fidelity_comparison.png")


def plot_noise_tolerance(results, output_dir):
    """Plot fidelity vs noise strength for each backend."""
    from collections import defaultdict
    noise_data = defaultdict(lambda: {"noise": [], "fidelity": []})

    for r in results:
        if r.get("status") != "success":
            continue
        f = r.get("metrics", {}).get("fidelity")
        noise = r.get("config", {}).get("noise_strength", 0)
        if f is not None:
            noise_data[r["backend"]]["noise"].append(noise)
            noise_data[r["backend"]]["fidelity"].append(f)

    if not noise_data:
        print("No noise tolerance data to plot.")
        return

    fig, ax = plt.subplots(figsize=(10, 6))
    for backend, data in sorted(noise_data.items()):
        sorted_pairs = sorted(zip(data["noise"], data["fidelity"]))
        noise, fids = zip(*sorted_pairs)
        ax.plot(noise, fids, 'o-', label=backend, linewidth=2, markersize=6)

    ax.set_xlabel("Noise Strength", fontsize=12)
    ax.set_ylabel("Fidelity", fontsize=12)
    ax.set_title("Noise Tolerance Comparison", fontsize=14)
    ax.legend(fontsize=10)
    ax.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig(Path(output_dir) / "noise_tolerance.png", dpi=150)
    plt.close()
    print(f"Saved: {output_dir}/noise_tolerance.png")
```

---

## Crush Commands for Analysis

| User Says | What Crush Does |
|-----------|----------------|
| "analyze results from the last benchmark" | Load `results/latest/`, compute comparison, show summary table |
| "compare Cirq vs Qiskit on 10 qubits" | Filter results, show side-by-side metrics |
| "show scalability chart" | Generate ASCII bar chart of time vs qubits |
| "plot scalability" | Generate matplotlib scalability.png |
| "which backend is fastest?" | Rank backends by avg execution time |
| "show fidelity comparison" | Bar chart (ASCII or matplotlib) of fidelity |
| "noise tolerance analysis" | Show fidelity degradation vs noise for each backend |
| "generate full report" | Create report.md with all tables, rankings, and charts |
| "export results as CSV" | Convert JSON results to CSV for external tools |
| "summarize in one paragraph" | Natural language summary of key findings |

---

## CSV Export

```python
#!/usr/bin/env python3
"""Export benchmark results to CSV for external analysis tools."""
import csv, json, sys
from pathlib import Path

def export_csv(benchmark_dir, output_file=None):
    """Export all results to a flat CSV."""
    results = []
    for f in Path(benchmark_dir).glob("*.json"):
        if f.name in ("summary.json", "comparison.json"):
            continue
        with open(f) as fp:
            results.append(json.load(fp))

    if not results:
        print("No results found.")
        return

    if output_file is None:
        output_file = Path(benchmark_dir) / "results.csv"

    fieldnames = [
        "backend", "status", "n_qubits", "depth", "circuit_type",
        "noise_model", "noise_strength", "shots",
        "execution_time_seconds", "fidelity", "gate_count",
        "circuit_depth", "memory_peak_mb", "num_qubits_simulated",
    ]

    with open(output_file, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames, extrasaction='ignore')
        writer.writeheader()
        for r in results:
            row = {
                "backend": r.get("backend"),
                "status": r.get("status"),
                **r.get("config", {}),
                **r.get("metrics", {}),
            }
            writer.writerow(row)

    print(f"Exported {len(results)} results to {output_file}")

if __name__ == "__main__":
    export_csv(sys.argv[1] if len(sys.argv) > 1 else "results/latest")
```

---

## Natural Language Report Summary

When a user says "summarize results," Crush should produce a paragraph like:

> **Summary:** Across 7 backends tested on GHZ circuits from 4 to 20 qubits,
> **Stim** was the fastest for Clifford circuits (0.001s at 20 qubits),
> followed by **qsim** (0.03s) and **Cirq** (0.12s). **Qiskit Aer** had the
> best fidelity at 0.999998 using the statevector method. For scalability,
> all backends showed exponential scaling beyond 24 qubits, with **Stim**
> maintaining sub-second performance up to 1000 qubits due to its stabilizer
> formalism. **LRET Phase 7** showed the best noise tolerance, maintaining
> fidelity > 0.99 with noise strength 0.05. Backends requiring GPU
> (cuQuantum, Qiskit Aer GPU) could not be tested on this system.

---

_Continue to [agent6.md](agent6.md) for Advanced Features — cloud deployment, custom scripts, code modification, and troubleshooting._
