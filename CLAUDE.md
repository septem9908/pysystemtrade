# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pysystemtrade is Rob Carver's open-source systematic futures trading system implementing principles from "Systematic Trading". It provides both a backtesting engine and a fully automated live trading system for Interactive Brokers.

**Python version:** >=3.10
**Broker:** Interactive Brokers (via ib-insync)

## Build and Test Commands

```bash
# Install normally
pip install .

# Install editable with dev dependencies
pip install -e '.[dev]'

# Run all tests
pytest

# Run single test file
pytest sysdata/tests/test_config.py

# Ignore specific test
pytest --ignore=sysinit/futures/tests/test_sysinit_futures.py

# Run slow tests (marked with @pytest.mark.slow)
pytest --runslow

# Format code with Black (required version: 23.11.0)
black .

# Exclude venv from formatting
black . --exclude '/.venv\/.+/'
```

## Architecture

### Stage-Based Processing Pipeline

The core design uses a modular "stage" architecture:

- **System** (`systems/basesystem.py`): Orchestrates processing, holds config, data, and stages
- **SystemStage** (`systems/stage.py`): Base class for processing units; each stage calculates/processes data
- Stages are chained together to form complete trading systems

```
System (orchestrator)
  ├── stage_list (list of SystemStage objects)
  ├── data (simData or production data source)
  ├── config (system parameters/rules)
  └── log (logger)
```

### Data Abstraction: dataBlob

`sysdata/data_blob.py` provides unified interface to multiple data sources (CSV, MongoDB, Arctic, Parquet). It automatically maps class names to data implementations allowing source-agnostic development.

**Critical:** Follow the [data naming hierarchy](https://github.com/robcarver17/pysystemtrade/blob/master/docs/data.md#part-2-overview-of-futures-data-in-pysystemtrade) for data objects or automated abstraction won't work.

### Module Structure (12 main sys* packages)

- `sysbrokers/` - Broker adapters (Interactive Brokers in `IB/`)
- `syscontrol/` - Process control, monitoring, scheduling
- `syscore/` - Core utilities, pandas extensions
- `sysdata/` - Data layer (CSV, MongoDB, Arctic, Parquet sources)
- `sysexecution/` - Order execution engine, algos, order stacks
- `sysinit/` - System initialization and data transfers
- `syslogging/` - Logging infrastructure with contextual attributes
- `syslogdiag/` - Log diagnostics
- `sysobjects/` - Object definitions
- `sysproduction/` - Production system, reports, diagnostics
- `sysquant/` - Quantitative components, optimisation, estimators
- `systems/` - Core system framework, accounts, trading rules

### Pre-built Systems

Trading rules and example systems are in `systems/provided/`:
- `rules/` - Trading rules (EWMAC, carry, etc.)
- `basic/`, `example/` - Starter systems
- `futures_chapter15/` - Book examples
- `rob_system/` - Production system example

## Code Guidelines

### General

- Use explicit parameter passing (except single parameter)
- Use `arg_not_supplied` from `syscore.objects` as default argument, resolve in function
- Use type hints; verbose docstrings are not required (superseded by type hints)
- Doctests should be standalone functions; use unit tests for class methods

### Naming Conventions

- Classes: prefer mixedCase; single words are CamelCase
- Common methods: `get`, `calculate`, `read`, `write`
- Objects inheriting from dicts use `dict_` prefix

### Error Handling

- Production code should not throw errors unless unrecoverable
- If throwing error, also call `log.critical()` which emails user

## Git Workflow

- Main branches: `master` (stable), `develop` (development)
- Topic branches: `bug-<issue#>-<description>` or `feature-<issue#>-<description>`
- PRs go to `develop` branch
- Black formatting is enforced on PRs

## Key Documentation

- `docs/introduction.md` - Start here
- `docs/backtesting.md` - Backtesting guide
- `docs/production.md` - Live trading setup
- `docs/data.md` - Data architecture
- `docs/IB.md` - Interactive Brokers integration
