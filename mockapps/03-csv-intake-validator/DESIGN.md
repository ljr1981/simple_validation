# CSVIntakeValidator - Technical Design

## Architecture

### Component Overview
```
+-------------------------------------------------------------+
|                     CSVIntakeValidator                       |
+-------------------------------------------------------------+
|  CLI Interface Layer                                         |
|    - Argument parsing (simple_cli)                          |
|    - Command routing (validate, init, analyze, report)      |
|    - Output formatting (text, json, csv, markdown)          |
+-------------------------------------------------------------+
|  Business Logic Layer                                        |
|    - CSV_VALIDATOR_ENGINE: Orchestration                    |
|    - SCHEMA_LOADER: Parse JSON schema definitions           |
|    - CSV_ANALYZER: Infer schema from data                   |
|    - ROW_VALIDATOR: Validate individual rows                |
|    - STATS_COLLECTOR: Collect validation statistics         |
+-------------------------------------------------------------+
|  Integration Layer                                           |
|    - simple_validation: Core validation engine              |
|    - simple_csv: CSV file parsing                           |
|    - simple_json: Schema definitions                        |
|    - simple_file: File operations                           |
|    - simple_template: Report generation                     |
+-------------------------------------------------------------+
```

### Class Design

| Class | Responsibility | Key Features |
|-------|----------------|--------------|
| CSV_VALIDATOR_CLI | Command-line interface | parse_args, execute, format_output |
| CSV_VALIDATOR_ENGINE | Core orchestration | load_schema, validate_file, generate_report |
| SCHEMA_LOADER | Parse schema definitions | load_json, validate_schema_syntax |
| CSV_SCHEMA | Column definitions | columns, required, custom_rules |
| CSV_COLUMN_DEF | Single column definition | name, type, format, constraints |
| CSV_ANALYZER | Infer schema from data | analyze_file, suggest_types |
| ROW_VALIDATOR | Validate single row | validate_row, collect_errors |
| STATS_COLLECTOR | Validation statistics | total_rows, valid_rows, error_summary |
| CSV_VALIDATION_RESULT | Overall result | is_valid, row_errors, column_stats |
| ROW_ERROR | Single row error | row_number, column, message, value |
| CSV_REPORTER | Output generation | to_text, to_json, to_csv, to_markdown |

### Command Structure
```bash
csv-intake-validator <command> [options] [arguments]

Commands:
  validate <file>          Validate CSV file against schema
  analyze <file>           Analyze file and suggest schema
  init                     Initialize schema file template
  report <file>            Generate detailed validation report
  version                  Show version information

Global Options:
  --schema FILE            Schema definition file (JSON)
  --format FORMAT          Output format: text, json, csv, markdown
  --strict                 Fail on first error
  --quiet                  Only output summary
  --no-color               Disable colored output
  --verbose                Verbose output with debug info
  --help                   Show help

Validate Options:
  --max-errors N           Stop after N errors (default: unlimited)
  --skip-rows N            Skip first N rows (headers handled separately)
  --encoding ENC           File encoding (default: utf-8)
  --delimiter CHAR         Field delimiter (default: comma)
  --sample N               Validate random sample of N rows

Analyze Options:
  --sample N               Sample N rows for analysis
  --output FILE            Write suggested schema to file

Examples:
  csv-intake-validator validate users.csv --schema users-schema.json
  csv-intake-validator validate data/*.csv --schema common-schema.json
  csv-intake-validator analyze raw-data.csv --output suggested-schema.json
  csv-intake-validator report users.csv --schema users-schema.json --format markdown
```

### Data Flow
```
CSV File  -->  simple_csv  -->  Parsed rows (ARRAYED_LIST of rows)
                    |
Schema File  -->  SCHEMA_LOADER  -->  CSV_SCHEMA (column definitions)
                    |
                    v
            ROW_VALIDATOR  -->  Validate each row against schema
                    |
                    v
            STATS_COLLECTOR  -->  Aggregate statistics
                    |
                    v
         CSV_VALIDATION_RESULT  -->  Errors, stats, summary
                    |
                    v
              CSV_REPORTER  -->  Formatted output
```

### Schema Definition Format
```json
{
  "$schema": "csv-intake-validator/1.0",
  "name": "User Import Schema",
  "description": "Schema for validating user CSV imports",
  "settings": {
    "header_row": true,
    "delimiter": ",",
    "encoding": "utf-8",
    "trim_whitespace": true,
    "allow_extra_columns": false
  },
  "columns": [
    {
      "name": "id",
      "type": "integer",
      "required": true,
      "unique": true,
      "min": 1
    },
    {
      "name": "email",
      "type": "string",
      "required": true,
      "format": "email",
      "max_length": 254
    },
    {
      "name": "name",
      "type": "string",
      "required": true,
      "min_length": 1,
      "max_length": 100
    },
    {
      "name": "age",
      "type": "integer",
      "required": false,
      "min": 0,
      "max": 150
    },
    {
      "name": "created_at",
      "type": "string",
      "required": true,
      "format": "date",
      "pattern": "^\\d{4}-\\d{2}-\\d{2}$"
    },
    {
      "name": "status",
      "type": "string",
      "required": true,
      "enum": ["active", "inactive", "pending"]
    },
    {
      "name": "phone",
      "type": "string",
      "required": false,
      "pattern": "^\\+?[0-9]{10,15}$"
    }
  ],
  "rules": [
    {
      "type": "unique",
      "columns": ["email"],
      "message": "Duplicate email address"
    },
    {
      "type": "row_count",
      "min": 1,
      "max": 100000,
      "message": "File must have 1-100K rows"
    }
  ]
}
```

### Column Types
| Type | Validation | Example |
|------|------------|---------|
| string | Any text | "John Doe" |
| integer | Whole number | 42, -5, 0 |
| number | Decimal number | 3.14, -0.5 |
| boolean | true/false | true, false, 1, 0 |
| date | ISO date | 2025-01-15 |
| datetime | ISO datetime | 2025-01-15T10:30:00Z |
| email | Email format | user@example.com |
| url | URL format | https://example.com |
| uuid | UUID format | 550e8400-e29b-41d4-a716-446655440000 |

### Error Handling

| Error Type | Handling | User Message |
|------------|----------|--------------|
| File not found | Exit 2 | "CSV file not found: {path}" |
| Parse error | Exit 3 | "Failed to parse CSV at row {n}: {details}" |
| Schema not found | Exit 2 | "Schema file not found: {path}" |
| Invalid schema | Exit 4 | "Invalid schema: {details}" |
| Validation failure | Exit 1 | Detailed error report |
| Encoding error | Exit 5 | "Invalid encoding at row {n}" |

### Exit Codes
| Code | Meaning |
|------|---------|
| 0 | Validation passed |
| 1 | Validation failed (errors found) |
| 2 | File not found |
| 3 | Parse error |
| 4 | Schema error |
| 5 | Encoding error |
| 255 | Internal error |

### Output Formats

**Text (default):**
```
CSV Validation Report: users.csv
Schema: users-schema.json

Summary:
  Total rows:  1,000
  Valid rows:  987
  Invalid rows: 13
  Error rate:  1.3%

Errors by column:
  email:       5 errors
  age:         4 errors
  status:      4 errors

Row Errors:
  Row 15: email - Invalid email format: "user@"
  Row 23: age - Must be at least 0: "-5"
  Row 45: status - Must be one of: active, inactive, pending: "deleted"
  ...
```

**JSON:**
```json
{
  "file": "users.csv",
  "schema": "users-schema.json",
  "valid": false,
  "summary": {
    "total_rows": 1000,
    "valid_rows": 987,
    "invalid_rows": 13,
    "error_rate": 0.013
  },
  "errors": [
    {
      "row": 15,
      "column": "email",
      "message": "Invalid email format",
      "value": "user@"
    }
  ]
}
```

## GUI/TUI Future Path

**CLI foundation enables:**
- All validation logic in reusable classes (CSV_VALIDATOR_ENGINE)
- Format-agnostic result objects (CSV_VALIDATION_RESULT)
- Schema analysis independent of output

**What would change for TUI:**
- Add simple_tui for interactive error navigation
- Real-time validation progress display
- Side-by-side schema vs data view

**What would change for GUI:**
- Drag-drop file validation
- Visual schema editor
- Error highlighting in data grid
- Historical validation reports
- Team schema library

**Shared components between CLI/GUI:**
- CSV_VALIDATOR_ENGINE (100% reusable)
- SCHEMA_LOADER (100% reusable)
- ROW_VALIDATOR (100% reusable)
- CSV_VALIDATION_RESULT (100% reusable)
