
## Technical Reference

### Time Measurements
- All `time` values in the worker_analysis.csv are measured in **seconds**
- Timing precision includes milliseconds (e.g., 1.04 seconds)
- All timing fields use float data type

### Job Identification and Parsing

#### Job Start Detection
The beginning of a DataPusher Plus job is identified by this log pattern:
```
2025-09-18 13:32:32,569 INFO  [07e81917-ef7a-4417-96e4-cc721129a7dc] Setting log level to INFO
```
**Regex Pattern:** `(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2},\d{3}) INFO\s+\[([a-f0-9-]{36})\] Setting log level to INFO`

#### Job Success Detection
A job is marked as `SUCCESS` when the log contains:
```
DATAPUSHER+ JOB DONE!
Download: 0.22
Analysis: 0.10
COPYing: 0.12
Indexing: 0.01
Formulae processing: 0.34
Resource metadata updates: 0.14
TOTAL ELAPSED TIME: 1.04
```

#### Job Error Detection
A job is marked as `ERROR` when the log contains:
```
ckanext.datapusher_plus.utils.JobError: [error message]
```

#### Incomplete Jobs
Jobs are marked as `INCOMPLETE` if they don't contain either success or error markers.

### Data Extraction Patterns

#### File Information
- **File URL:** `Fetching from: (.+)`
- **File Name:** Extracted as the last segment of the URL path
- **File Format:** `File format: (\w+)`
- **Encoding:** `Identified encoding of the file: (\w+)`

#### Processing Status Indicators
- **qsv Version:** `qsv version found: ([\d.]+)`
- **Normalization:** "Successful" if log contains "Normalized & transcoded"
- **Valid CSV:** "TRUE" if log contains "Well-formed, valid CSV file confirmed"
- **Sorted Status:** `Sorted: (True|False)` → converted to uppercase
- **Analysis Status:** "Successful" if log contains `ANALYSIS DONE! Analyzed and prepped in ([\d.]+) seconds`

#### Data Safety and Quality
- **Database Safe Headers:**
  - `"(\d+) unsafe" header names found` → "{count} unsafe headers found"
  - "No unsafe header names found" → "All headers safe"
  - Default: "Unknown"

#### Record Processing
- **Records Detected:** `(\d+)\s+records detected`
- **Rows Copied:** `Copied (\d+) rows to`
- **Columns Indexed:** `Indexed (\d+) column/s`

### Timing Breakdown Extraction

| Field | Regex Pattern | Description |
|-------|---------------|-------------|
| `total_time` | `TOTAL ELAPSED TIME: ([\d.]+)` | Overall job processing time |
| `download_time` | `Download: ([\d.]+)` | File download duration |
| `analysis_time` | `Analysis: ([\d.]+)` | Data analysis phase duration |
| `copying_time` | `COPYing: ([\d.]+)` | Database copying operation duration |
| `indexing_time` | `Indexing: ([\d.]+)` | Index creation duration |
| `formulae_time` | `Formulae processing: ([\d.]+)` | Formula processing duration |
| `metadata_time` | `Resource metadata updates: ([\d.]+)` | Metadata update duration |

### Error Classification

The script automatically classifies errors into categories:

| Error Type | Detection Criteria | Description |
|------------|-------------------|-------------|
| `CORRUPTED_EXCEL` | Error message contains "invalid Zip archive" or "EOCD" | Excel file corruption issues |
| `QSV_ERROR` | Error message contains "qsv command failed" | QSV processing tool failures |
| `INVALID_URL` | Error message contains "Only http, https, and ftp resources may be fetched" | Invalid resource URL |
| `DATAPUSHER_ERROR` | Generic DataPusher Plus JobError | Other DataPusher-specific errors |
| `UNKNOWN_ERROR` | No specific pattern match | Unclassified errors |

### CSV Output Schema

#### Primary Fields
| Column | Type | Description |
|--------|------|-------------|
| `timestamp` | String | Job start timestamp (YYYY-MM-DD HH:MM:SS) |
| `job_id` | String | UUID of the processing job |
| `file_name` | String | Name of the processed file |
| `status` | String | SUCCESS, ERROR, or INCOMPLETE |
| `qsv_version` | String | Version of QSV tool used |
| `file_format` | String | Detected file format (CSV, XLSX, etc.) |
| `encoding` | String | File character encoding |
| `normalized` | String | "Successful" or "Failed" |
| `valid_csv` | String | "TRUE" or "FALSE" |
| `sorted` | String | "TRUE", "FALSE", or "UNKNOWN" |
| `db_safe_headers` | String | Header safety status |
| `analysis` | String | "Successful" or "Failed" |
| `records` | Integer | Number of records detected |

#### Timing Fields (all in seconds)
| Column | Type | Description |
|--------|------|-------------|
| `total_time` | Float | Total processing time |
| `download_time` | Float | File download time |
| `analysis_time` | Float | Analysis phase time |
| `copying_time` | Float | Database copy time |
| `indexing_time` | Float | Index creation time |
| `formulae_time` | Float | Formula processing time |
| `metadata_time` | Float | Metadata update time |

#### Processing Results
| Column | Type | Description |
|--------|------|-------------|
| `rows_copied` | Integer | Number of rows copied to database |
| `columns_indexed` | Integer | Number of columns indexed |
| `error_type` | String | Classified error type (empty for successful jobs) |
| `error_message` | String | First JobError message found (quotes escaped) |

### Advanced Analytics Features

#### Data Quality Scoring Algorithm
Base score: 100, with penalties applied:
- Invalid CSV: -30 points
- Unsorted data: -10 points
- Unsafe headers: -5 points per unsafe header (max -25)
- Failed normalization: -20 points
- Failed analysis: -25 points

Bonuses applied:
- UTF-8 encoding: +5 points
- >1000 records: +5 points

#### Performance Anomaly Detection
- Uses statistical analysis (mean + 2 standard deviations)
- Identifies jobs with processing times significantly above normal
- Requires minimum 3 successful jobs for analysis

#### Failure Pattern Analysis
Analyzes failures by:
- File format distribution
- Time-of-day patterns  
- File size categories (small <100, medium <10k, large ≥10k records)
- Sequential failure detection (within 5-minute windows)
- Recurring file failures
