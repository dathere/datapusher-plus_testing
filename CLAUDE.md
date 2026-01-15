# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an automated testing suite for [DataPusher+](https://github.com/dathere/datapusher-plus), a CKAN extension for processing and importing tabular data files into CKAN's DataStore. The suite runs via GitHub Actions and validates data processing workflows across multiple file formats.

## Repository Structure

```
datapusher-plus_testing/
├── .github/workflows/
│   ├── main.yml          # Primary test workflow
│   ├── qsv_test.yml      # QSV-specific tests
│   └── url.yml           # URL-based resource tests
├── tests/
│   ├── log_analyzer.py   # Python script for parsing DataPusher+ worker logs
│   ├── quick/            # Quick test files (default test directory)
│   ├── dpp/              # DataPusher+ specific test files
│   ├── tabular/          # Various tabular format test files
│   ├── static/           # Static test files
│   ├── base_files/       # Base/reference test files
│   └── custom/           # User-provided custom test files
```

## Running Tests

Tests are run via GitHub Actions workflow dispatch. Key parameters:
- `repo_branch`: Branch of this testing repo to use (default: main)
- `datapusher_branch`: DataPusher+ branch or commit to test (default: main)
- `testing_directory`: Directory within `tests/` containing test files (default: quick)

To test with custom files:
1. Add files to `tests/custom/` directory
2. Run the `DataPusher+ Testing Run` workflow with `testing_directory: custom`

## Supported File Formats

CSV, TSV, TAB, SSV, XLS, XLSX, XLSXB, XLSM, ODS, GeoJSON, SHP, QGIS, ZIP

## Log Analyzer Tool

The `tests/log_analyzer.py` script parses DataPusher+ worker logs and generates analytics:

```bash
# Analyze worker logs and generate CSV report
python log_analyzer.py analyze <log_file> <output_csv>

# Generate performance insights from analysis CSV
python log_analyzer.py insights <worker_csv>

# Get insight for a specific file
python log_analyzer.py file-insight <worker_csv> <filename>

# Generate executive summary
python log_analyzer.py executive-summary <worker_csv>

# Detect performance anomalies
python log_analyzer.py anomalies <worker_csv>

# Calculate business metrics
python log_analyzer.py business-metrics <worker_csv>
```

## Key Log Patterns

Job start detection:
```
YYYY-MM-DD HH:MM:SS,mmm INFO [job-uuid] Setting log level to INFO
```

Job success marker: `DATAPUSHER+ JOB DONE!`

Job error marker: `ckanext.datapusher_plus.utils.JobError:`

## Test Artifacts

After workflow completion, these artifacts are available:
- `ckan_stdout.log` - CKAN web application logs
- `ckan_worker.log` - DataPusher+ worker logs (contains processing details)
- `test_results.csv` - Pass/fail matrix with timing stats
- `worker_analysis.csv` - Per-resource performance breakdown

## Requirements

- CKAN v2.11+
- DataPusher+ v2+
- qsv (data wrangling toolkit) - installed automatically by workflow
