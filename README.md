# Scan Insertion and ATPG using Fault Open Source Tool

This project demonstrates the Design for Testability (DFT), Automatic Test Pattern Generation (ATPG), Fault Coverage, and Test Compaction flow using the **Fault** open-source tool.

The workflow focuses on testability analysis of a counter, performing scan chain insertion, cutting scan loops, and generating test patterns.

## Prerequisites

- **Docker**: This guide uses the `fault` tool via a Docker container. Ensure Docker is installed and running.
- **Fault Docker Image**: `ghcr.io/aucohl/fault:latest`

## Technology Library & References

- **Technology Library Used**: [VLSI Technology osu035](https://www.vlsitechnology.org/html/libraries04.html)
- **Library Files in Repo**: `osu035_stdcells.lib` (Liberty format), `osu035_stdcells.v` (Verilog format) (See: [University of Oklahoma / Oklahoma State University Digital VLSI](https://github.com/AUCOHL/Fault/tree/main/Tech/osu035))
- **Fault Documentation**: [Fault Read the Docs](https://fault.readthedocs.io/en/latest/usage.html)

## Key Concepts

- **DFT (Design for Testability)**: Techniques added to hardware design to make it easier to develop and apply manufacturing tests.
- **ATPG (Automatic Test Pattern Generation)**: An electronic design automation method/technology used to find an input (or test) sequence that, when applied to a digital circuit, enables automatic test equipment to distinguish between the correct circuit behavior and the faulty circuit behavior caused by defects.
- **Fault Coverage**: The percentage of detected faults out of all possible faults in the design.
- **Test Compaction**: Techniques to reduce the number of test vectors (and thus test time/cost) while maintaining fault coverage.

## Workflow Steps

### 1. Scan Insertion (FAULT chain)

This step performs **Scan Flip-Flop replacement and insertion** to create a scan chain. By replacing standard Flip-Flops (FFs) with Scan FFs, we transform the sequential circuit into a combinational one during test mode, significantly improving observability and controllability.

![Scan Chain Insertion](chain%20insert.png)

**Command:**
```bash
docker run -ti --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  ghcr.io/aucohl/fault:latest \
  fault chain counter_trial.v \
    --clock clk \
    --reset rst \
    --liberty osu035_stdcells.lib \
    -o counter_scan1.v \
    --sout sout
```


**Output:** `counter_scan1.v`

### 2. Scan Cut (Break Scan Loops)

<<<<<<< HEAD
It is necessary to perform a "cut" in the Fault flow to break scan loops.
=======
### 2. Scan Cut (Break Scan Loops)

It is necessary to **perform a "cut"** in the Fault flow to break scan loops. As discussed in the IEEE papers on Fault (available in the course page), combinatorial loops created by the scan chain insertion can disrupt the ATPG process. The `fault cut` command identifies and breaks these loops to ensure proper test pattern generation.
>>>>>>> aaf8323 (Update README with detailed DFT concepts and step descriptions)

**Command:**
```bash
docker run -ti --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  ghcr.io/aucohl/fault:latest \
  fault cut counter_scan1.v \
    -o counter_scan_cut.v
```

**Output:** `counter_scan_cut.v`

### 3. ATPG – Large Random Search

This step performs Automatic Test Pattern Generation to achieve high fault coverage (aiming for e.g., >90%).

![ATPG Visualization](atpg1.png)
![ATPG Visualization 2](atpg2.png)

**Command:**
```bash
docker run -ti --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  ghcr.io/aucohl/fault:latest \
  fault counter_scan_cut.v \
    -c osu035_stdcells.v \
    --clock clk \
    --tvCount 500 \
    --increment 500 \
    --ceiling 5000 \
    --minCoverage 90 \
    --output patterns_xl.tv.json \
    --output-covered coverage_xl.yml
```

**Output:**
- `patterns_xl.tv.json`: Generated test patterns.
- `coverage_xl.yml`: Coverage report.

## Visualization

![Abstract Map](abcmap.png)
![Cells](cells.png)
