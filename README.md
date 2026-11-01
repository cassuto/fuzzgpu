# Fuzzing Open-Source GPU Hardware

FuzzGPU is a fuzzing framework designed for testing RISC-V based GPGPU hardware designs. It generates test programs with complex SIMT (Single Instruction, Multiple Threads) control flow, memory access patterns, and barrier synchronization to uncover bugs in GPGPU implementations.

---

## Quick Start

1. Follow the "A.3 Set-up" in the Artifact Appendix.
```bash
make docker
```

You can create the container with the image built in phase 1 via:
```bash
make run
```
In the container, you can enter the working directory via:
```bash
cd /root/fuzzgpu
```

Tip: After the container has been created or terminated, you can restart and attach to it with:

```bash
docker start -ai fuzzgpu-new
```


2. Follow the "A.4.2 Experiments" in the Artifact Appendix. Note that E1 is optional if you have completed "A.3 Set-up".

### (E2): Instruction completion rate (validating C1).
Execution:
```
scripts/run_E2.sh
```

Results: `results/plots/instr_complete_rate.pdf` reproduces Figure 10.

### (E3): measurement (validating C3).
Execution:
```
scripts/run_E3.sh
```
Results: `results/plots/gen_throughput_vortex_xlen64.pdf` reproduces Figure 11.
`results/plots/effective_throughput_vortex_xlen64.pdf` reproduces Figure 12.

### (E4): measurement (validating C5).
Execution:
```
scripts/run_E4.sh
```
Results: `results/plots/coverage_comparison_ventus_x32_combined.pdf` reproduces Figure 15.

### (E5): Ablation study (validating C6).
Execution:
```
scripts/run_E5.sh
```
Results: It reproduces Section 5.4.

### (E6): Fuzzing campaign (validating C2, C4, and C7).
Execution:
```
scripts/run_E6.sh.
```
Results: It reproduces Table 2, Figure 14 (`results/plots/time_breakdown_fuzzgpu.pdf`), and Figure 16 (`results/plots/bug_discover_time.pdf`).

### What to expect
The absolute values from our AE server may differ from those reported in the paper because we are using a different hardware configuration—AMD EPYC 9T95 rather than Intel® Xeon® 6982P-C due to resource limitation, and fuzzing performance is hardware-dependent.

Nevertheless, the results should be sufficient to validate the claim.

## Generate Documentation (for reusing, optionally)

Generate the Doxygen documentation:

```bash
./build.sh --docs
```

Then view the generated guide at:

```text
docs/html/index.html
```

Use the Doxygen navigation bar to browse the generated API reference:

- `Main Page` shows this project overview.
- `Namespaces` groups documented symbols by C++ namespace.
- `Classes` lists documented classes and structs.
- `Files` lists headers and source files included in the documentation.
- The search box can find symbols, files, classes, functions, and typedefs.

---

## Supported Architectures

| Architecture | XLEN | Description |
|--------------|------|-------------|
| **Vortex** | 32-bit | Open-source RISC-V GPGPU from Georgia Tech |
| **Vortex** | 64-bit | 64-bit variant of Vortex |
| **Ventus** | 32-bit | RISC-V GPGPU from Tsinghua University |

---

## Building (for reusing, optionally)

For the full list of build options, run:

```bash
./build.sh --help
```

Basic examples:

```bash
# Build all supported configurations.
./build.sh --all

# Build for Ventus 32-bit with the default configuration.
./build.sh --arch ventus --xlen 32

# Build for Vortex 64-bit with line coverage and difftest.
./build.sh --arch vortex --xlen 64 --cov-line --dft
```

---

## Reduced-Parameter Experiment Runs (for reusing, optionally)

Note: This section is intended to facilitate future reuse. The procedure for running the experiments has already been described earlier and is therefore NOT repeated in this section.

The `scripts/run_E2.sh` to `scripts/run_E6.sh` wrappers reproduce the paper
experiments. Some experiments may be run with reduced parameters to complete in
a reasonable amount of time, while conserving the tendency shown in the paper.
Reduced runs are intended for quick validation of trends, not for regenerating
the exact paper numbers.

The easiest way to shorten a run is to open the corresponding `scripts/run_*.sh`
file and reduce the variables near the top of the script. The wrappers
`run_E2.sh` to `run_E6.sh` usually call one or two lower-level scripts; edit
those lower-level scripts directly.

### E2: Instruction Completion Rate

Edit `scripts/run_instr_complete_rate.sh`.

Recommended quick-run changes:

- Reduce `NUM_SEEDS` from `5000` to `100` or `500`.
- Keep only one entry in `ARCH_XLEN_CONFIGS`.
- Keep only one entry in `METHODS`.
- Increase `JOBS` only if the machine has enough cores and memory.

### E3: Fuzzed Instruction Throughput

Edit `scripts/run_fuzz_instr_throughput.sh`.

Recommended quick-run changes:

- Reduce `TARGET_TIME` from `60` seconds to `10` or `20`.
- Keep `ARCH` and `XLEN` to a single target.
- Use the reduced result to check relative throughput trends; short timing runs
  are noisier than the paper-scale run.

### E4: Baseline Coverage

Edit `scripts/run_baseline.sh`.

Recommended quick-run changes:

- Reduce `TARGET_TIME` from `1800` seconds to `300` or less.
- Keep only one entry in `ARCH_XLEN_CONFIGS`.
- Keep only one coverage type in `COV_TYPES`, for example `expr`.
- Keep only one method in `ALL_METHODS`, or run the script with `--method
  fuzzgpu`.

### E5: Ablation Study

Edit both `scripts/run_ablation_program_metric.sh` and
`scripts/run_ablation_coverage.sh`.

Recommended quick-run changes for `scripts/run_ablation_program_metric.sh`:

- Reduce `--num-seeds` from `5000` to `100` or `500`.
- Keep `--arch ventus` for the OpenGPGPU result used in the paper.
- Keep `--force-max-num-instrs 10000` unless testing a much smaller smoke run.

Recommended quick-run changes for `scripts/run_ablation_coverage.sh`:

- Reduce `NUM_SEEDS` from `100` to a smaller value such as `20` or `50`.
- Set `COV_TYPES` to only `expr` for the OpenGPGPU expression coverage result.
- Set `CONFIGS` to a subset such as `baseline no_emu_guide` for a very short
  check, or leave it empty to run all ablations.
- Set `MAX_TIME` to cap each ablation configuration if needed.

`scripts/tab_ablation.py` uses the latest `expr` ablation coverage result for
the OpenGPGPU expression coverage table.

### E6: Bug Discovery and Runtime Breakdown

Edit `scripts/run_fuzzing.sh`.

Recommended quick-run changes:

- Reduce `TARGET_TIME` from `300` seconds to `60`.
- Reduce `START_SEED_INDICES` from ten entries to one or two entries.
- Keep only one entry in `ARCH_XLEN_CONFIGS`.
- Keep only one method in `ALL_METHODS`, or run the script with `--method
  fuzzgpu`.


## Citation

If you find our work useful, please consider citing:

```bibtex
@inproceedings {320689,
author = {Zibo Gao and Jie Wang and Qihang Zhou and Lixiao Shan and Junjie Hu and Xiaoqi Jia and Zhiqiang Lv},
title = {Fuzzing {Open-Source} {GPU} Hardware with {SIMT} Program Generation},
booktitle = {35th USENIX Security Symposium (USENIX Security 26)},
year = {2026},
isbn = {978-1-939133-58-8},
address = {Baltimore, MD},
pages = {2465--2484},
url = {https://www.usenix.org/conference/usenixsecurity26/presentation/gao-zibo},
publisher = {USENIX Association},
month = aug
}
```
