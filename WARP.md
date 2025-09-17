# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a database benchmark suite forked from h2oai/db-benchmark that compares scalability and performance of database-like operations across multiple data processing solutions. The benchmark tests groupby and join operations on datasets of varying sizes (1e7 to 1e9 rows) across solutions like pandas, DuckDB, Polars, data.table, Spark, ClickHouse, and others.

## Architecture

### Core Components

- **Solutions**: Each solution (pandas, duckdb, polars, etc.) has its own directory containing:
  - `setup-{solution}.sh`: Installation and environment setup
  - `groupby-{solution}.{ext}`: Groupby benchmark implementation
  - `join-{solution}.{ext}`: Join benchmark implementation
  - `ver-{solution}.sh`: Version tracking
  - `upg-{solution}.sh`: Upgrade scripts
  - `VERSION`: Current version metadata

- **Control System**: `_control/` directory manages benchmark configuration:
  - `data.csv`: Defines active datasets and parameters
  - `solutions.csv`: Maps solutions to supported tasks
  - `timeout.csv`: Defines timeout limits for different data sizes

- **Launcher**: `_launcher/` contains the R-based orchestration system:
  - `launch.R`: Main batch launcher for multiple solutions/datasets
  - `solution.R`: Single solution runner with flexible parameters
  - `launcher.R`: Core orchestration functions

- **Data Generation**: `_data/` contains R scripts to generate synthetic datasets:
  - `groupby-datagen.R`: Creates groupby test datasets with configurable parameters
  - `join-datagen.R`: Creates join test datasets

## Common Development Tasks

### Running Benchmarks

**Single solution benchmark:**
```bash
# Run specific solution on default dataset
./_launcher/solution.R --solution=pandas --task=groupby --nrow=1e7

# Run with custom parameters
./_launcher/solution.R --solution=pandas --task=groupby --nrow=1e7 --k=1e2 --na=0 --sort=0 --quiet=true

# Output to file instead of console
./_launcher/solution.R --solution=pandas --task=groupby --nrow=1e7 --out=results.csv
```

**Batch benchmark run:**
```bash
# Edit run.conf to specify solutions and tasks
# Then run full benchmark
./run.sh
```

### Data Generation

**Generate groupby datasets:**
```bash
# Generate 10M rows, 100 unique groups, 0% NAs, random order
Rscript _data/groupby-datagen.R 1e7 1e2 0 0

# Generate 100M rows, 10 unique groups, 5% NAs, sorted order  
Rscript _data/groupby-datagen.R 1e8 1e1 5 1
```

### Solution Development

**Adding a new solution:**
1. Create directory named after the solution
2. Implement required scripts:
   - `setup-{solution}.sh`: Environment setup and dependencies
   - `groupby-{solution}.{ext}`: Groupby benchmark implementation
   - `join-{solution}.{ext}`: Join benchmark implementation  
   - `ver-{solution}.sh`: Version reporting
   - `upg-{solution}.sh`: Upgrade procedure
3. Add entries to `_control/solutions.csv`
4. Update `run.sh` to include upgrade/version checks
5. Follow the timing and logging pattern from existing solutions using `_helpers/helpers.{ext}`

**Solution script requirements:**
- Use `SRC_DATANAME` environment variable for data file
- Use `MACHINE_TYPE` environment variable for machine info
- Import and use helpers for logging: `write_log()` function
- Run each query twice for timing consistency
- Include memory usage tracking and validation checksums

### Configuration

**Enable/disable datasets:**
Edit `_control/data.csv` and set `active` field to 1 (enabled) or 0 (disabled)

**Add new data sizes:**
Add entries to `_control/data.csv` and corresponding timeout values to `_control/timeout.csv`

**Configure batch runs:**
Edit `run.conf`:
- `RUN_TASKS`: Tasks to run ("groupby join")
- `RUN_SOLUTIONS`: Solutions to benchmark ("pandas duckdb polars")
- `DO_UPGRADE`: Whether to upgrade solutions before running
- `FORCE_RUN`: Ignore previous runs and force re-execution

### Prerequisites

**System requirements:**
- SWAP must be disabled: `sudo swapoff -a`
- ClickHouse server must be stopped if not being used
- Java processes should not be running during benchmarks
- Ensure adequate disk space in `./data` directory

**Python solutions setup:**
```bash
# Example for pandas
virtualenv pandas/py-pandas --python=python3
source pandas/py-pandas/bin/activate
pip install pandas pyarrow psutil
deactivate
```

### Monitoring and Debugging

**Check running benchmarks:**
```bash
tail -f run.out                    # Main benchmark output
ls out/run_*.err                   # Error logs for individual runs
ls -l out/run_*.err | awk '{if ($5 != 0) print $9}'  # Non-empty error files
```

**Graceful termination:**
```bash
touch stop    # Will stop after current script finishes
touch pause   # Will pause until file is removed
```

**Results analysis:**
- `time.csv`: Detailed timing results and validation data
- `logs.csv`: Script execution logs including failures and skips
- Reports generated in `public/` directory when `DO_REPORT=true`

## Important Notes

- Each solution must handle the standard data schema: id1-id6 (grouping columns) and v1-v3 (value columns)
- Benchmark scripts run each query twice to account for caching effects
- Memory usage reporting uses `psutil` and tracks RSS in GB
- All solutions should use consistent data loading patterns and handle missing values appropriately
- The `_helpers/helpers.{py,R,jl,sh}` files provide standardized logging and utility functions