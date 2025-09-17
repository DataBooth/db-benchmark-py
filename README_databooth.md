# DataBooth Modernisation Plan for db-benchmark-py

## Acknowledgments

This modernisation effort builds upon the exceptional foundational work of the database benchmarking community:

- **Original Creators**: The [H2O.ai team](https://github.com/h2oai/db-benchmark) who created this comprehensive database benchmark suite, establishing rigorous methodologies for comparing data processing solutions at scale.

- **Current Maintainers**: The [DuckDB Labs team](https://github.com/duckdblabs/db-benchmark) who have excellently maintained and evolved the benchmark, ensuring up-to-date comparisons across the rapidly evolving data processing landscape.

- **Broader Community**: Contributors from pandas, Polars, DuckDB, Apache Arrow, ClickHouse, Spark, and other data processing communities who have provided implementations, optimisations, and feedback.

Our goal is to make this valuable resource more accessible while preserving the scientific rigor and comprehensive coverage that makes it such an important tool for the data community.

## Vision: Making Database Benchmarking More Accessible

### Current Challenges
The existing db-benchmark is scientifically rigorous but has some barriers to adoption:
- **Complex Setup**: Requires R, bash expertise, and manual virtual environment management
- **Platform Dependencies**: bash scripts and R-centric workflow less familiar to Python developers
- **Entry Barriers**: High complexity for running Python-only benchmarks
- **Limited Reporting**: CSV-based outputs lack modern visualisation options

### Our Modernisation Goals

1. **Maintain Scientific Excellence**: Preserve all existing benchmark methodologies, validation approaches, and result accuracy
2. **Reduce Complexity**: Provide simpler paths for common use cases while keeping full flexibility
3. **Improve Accessibility**: Make Python-only benchmarks trivial to run
4. **Modernise Tooling**: Leverage current best practices in Python tooling and cross-platform development
5. **Enhance Reporting**: Provide interactive, publication-ready output formats

## Planned Enhancements

### 🐍 **Python-First Experience**
```bash
# Current workflow (requires R, bash expertise):
Rscript _data/groupby-datagen.R 1e7 1e2 0 0
./run.sh  # Complex bash script with many dependencies

# Proposed workflow (Python developers):
just generate-data --size=1e7      # Cross-platform task runner
just bench-python                  # Python-only benchmarks
just report --format=qmd           # Modern reporting
```

### 🚀 **Modern Dependency Management**
- **uv**: Lightning-fast Python package manager replacing manual virtualenv setup
- **pyproject.toml**: Centralised configuration for all Python components
- **Version Pinning**: Exact reproducibility with lock files
- **Cross-platform**: Works identically on macOS, Linux, Windows

### 🛠 **Enhanced Developer Experience**
```toml
# pyproject.toml - Single source of truth
[tool.benchmark]
python_version = ">=3.9,<4.0"
solutions = ["pandas", "polars", "duckdb", "pyarrow"]
data_sizes = [1_000_000, 10_000_000, 100_000_000]
tasks = ["groupby", "join"]

[tool.benchmark.system]
memory_limit_gb = 8
cpu_threads = "auto"
output_formats = ["qmd", "json", "csv"]
```

### 📊 **Rich System Information**
Enhanced machine reporting including:
- Python version and implementation
- CPU architecture (arm64, x86_64) 
- Detailed hardware specs (Apple Silicon detection, memory)
- Platform-specific optimisations
- Dependency versions with lock file hashes

### 📖 **Modern Reporting Options**
- **Quarto Markdown (.qmd)**: Interactive, publication-ready reports
- **GitHub Markdown**: Embedded visualisations for repository display
- **JSON API**: Machine-readable results for automated analysis
- **HTML Dashboards**: Interactive exploration of results
- **Backward Compatibility**: Maintains existing CSV outputs

### 🏗 **Proposed Architecture**

```
db-benchmark-py/
├── justfile                     # Cross-platform task definitions
├── pyproject.toml              # Central Python configuration
├── uv.lock                     # Exact dependency versions
├── 
├── _data/
│   ├── generators.py           # Python data generation
│   └── schemas.py              # Data validation schemas
├── 
├── solutions/
│   ├── pandas/
│   │   ├── pyproject.toml      # Solution-specific deps
│   │   ├── groupby_pandas.py
│   │   └── join_pandas.py
│   ├── polars/
│   ├── duckdb/
│   └── pyarrow/
├── 
├── _launcher/
│   ├── launcher.py             # Modern Python orchestration
│   ├── system_info.py          # Enhanced system detection
│   └── validators.py           # Result validation
├── 
├── _reports/
│   ├── templates/
│   │   ├── benchmark.qmd       # Quarto report template
│   │   └── github.md.jinja2    # GitHub-friendly template
│   └── generators.py           # Report generation logic
└── 
└── _helpers/
    ├── benchmark_utils.py      # Shared utilities
    ├── timing.py              # Precision timing tools
    └── visualisation.py       # Plot generation
```

### 🔧 **Implementation Phases**

#### **Phase 1: Foundation** (Immediate)
- [x] Python data generation equivalent to R scripts
- [x] Basic justfile with core commands
- [x] uv-based dependency management for Python solutions

#### **Phase 2: Core Functionality** (Short-term)
- [ ] Python launcher replacing bash orchestration
- [ ] Enhanced system information collection
- [ ] Modern logging and progress indication

#### **Phase 3: Advanced Features** (Medium-term)
- [ ] Quarto-based reporting pipeline
- [ ] Interactive result visualisation
- [ ] Automated result validation across solutions

#### **Phase 4: Ecosystem Integration** (Long-term)
- [ ] CI/CD pipeline for automated benchmarking
- [ ] API endpoints for result querying
- [ ] Integration with cloud benchmarking platforms

### 🔄 **Backward Compatibility**

**Critical Principle**: All existing functionality remains available.

- R-based workflows continue to work unchanged
- Existing bash scripts remain functional
- Current CSV output format preserved
- All solution implementations maintain compatibility
- Migration is opt-in, not required

```bash
# Existing workflow still works:
./run.sh

# New workflow available alongside:
just bench-all  # Runs both Python and R benchmarks
```

### 🎯 **Benefits for Different Users**

#### **Python Developers**
- Familiar tooling (pyproject.toml, uv, just)
- No R installation required for Python-only benchmarks
- Modern development experience with type hints, linting, testing

#### **Data Scientists**
- Interactive Quarto reports with embedded code
- Easy customisation of benchmark parameters
- Rich visualisation of performance comparisons

#### **DevOps/CI Teams**
- Reproducible builds with lock files
- Cross-platform compatibility
- Docker-friendly dependency management

#### **Researchers**
- Enhanced system information for reproducibility
- Machine-readable output formats
- Integration with computational notebooks

### 🤝 **Contributing to the Vision**

This modernisation maintains the collaborative spirit of the original project:

1. **Incremental Changes**: Each enhancement can be developed and tested independently
2. **Community Input**: Seeking feedback from solution maintainers and users
3. **Scientific Rigor**: All changes validated against existing benchmark results
4. **Open Development**: Progress tracked in public with clear documentation

### 🎖 **Recognition of Prior Art**

We stand on the shoulders of giants. The original H2O.ai db-benchmark established:
- Rigorous timing methodologies
- Comprehensive solution coverage
- Standardised data generation
- Transparent result validation

The DuckDB Labs continuation has provided:
- Continuous maintenance and updates
- Integration of new solutions
- Infrastructure for public result hosting
- Community stewardship

Our modernisation aims to make this excellent work more accessible to today's Python-centric data community while preserving everything that makes it valuable.

---

*This is a community effort to evolve an important resource. We welcome feedback, contributions, and collaboration from anyone interested in advancing database benchmarking practices.*