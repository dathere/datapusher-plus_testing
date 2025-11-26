[![DataPusher+ Testing Run](https://github.com/dathere/datapusher-plus_testing/actions/workflows/main.yml/badge.svg)](https://github.com/dathere/datapusher-plus_testing/actions/workflows/main.yml)
# DataPusher+ Testing Suite
Automated testing suite for [DataPusher+ extension](https://github.com/dathere/datapusher-plus) functionality. This repository provides a complete pipeline using GitHub Actions to validate data processing workflows across multiple file formats including CSV, TSV, Excel, and JSON files.

## Requirements:
- CKAN v2.11+
- Datapusher+ v2+

## Key Features:
- Multi-format file testing 
- Real-time DataPusher job monitoring and status validation
- Automated datastore activation and data import verification
- Detailed test reporting with markdown summaries and downloadable artifacts
- Configurable test datasets and extensible file format support

Includes error handling, performance metrics, and detailed logging for troubleshooting extension issues.

## Steps to Testing with Your Own Files

1. Fork the repository
2. Add testing files in the tests/custom files directory
3. Head over to actions and run the `DataPusher+ Testing Run` workflow. Enter the configurables like the branch of the datapusher-plus, and the directory of the test files (default directory: quick)
4. Once the work flow gets completed, refer the artifacts in the Summary of the workflow

Inside you’ll find:
```
.
├── ckan_stdout.log
├── ckan_worker.log
├── test_results.csv
└── worker_analysis.csv
```

### File descriptions

`ckan_stdout.log` – Full CKAN web application stdout/stderr stream, including HTTP requests, API calls, and runtime warnings.

`ckan_worker.log` – Detailed DataPusher+ worker trace covering each ingestion step, validation, and indexing operation. This is where you'll find datapusher-plus logs

`test_results.csv` – Consolidated pass/fail matrix and timing stats for every dataset processed during the run.

`worker_analysis.csv` – Per-resource performance breakdown with download, analysis, and indexing timings for deeper diagnostics. 
