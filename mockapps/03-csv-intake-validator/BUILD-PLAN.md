# CSVIntakeValidator - Build Plan

## Phase Overview

| Phase | Deliverable | Effort | Dependencies |
|-------|-------------|--------|--------------|
| Phase 1 | MVP CLI | 5 days | simple_validation, simple_csv, simple_json, simple_cli |
| Phase 2 | Analysis + Advanced | 4 days | Phase 1, simple_datetime |
| Phase 3 | Polish + Reports | 3 days | Phase 2, simple_template |

## Phase 1: MVP

### Objective

Demonstrate core value: validate a CSV file against a JSON schema and report errors. This proves the concept works and provides immediate value for data quality validation.

### Deliverables

1. **CSV_VALIDATOR_CLI** - Basic CLI with validate command
2. **CSV_VALIDATOR_ENGINE** - Core orchestration
3. **SCHEMA_LOADER** - Parse JSON schema definitions
4. **CSV_SCHEMA** - Schema data structure
5. **CSV_COLUMN_DEF** - Column definition data structure
6. **ROW_VALIDATOR** - Validate individual rows
7. **CSV_VALIDATION_RESULT** - Validation result with errors
8. **ROW_ERROR** - Single error data structure
9. **CSV_REPORTER** - Text output format

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T1.1 | Create ECF with dependencies | Compiles with all required libraries |
| T1.2 | Implement CSV_COLUMN_DEF class | Holds name, type, constraints |
| T1.3 | Implement CSV_SCHEMA class | Holds columns, settings, rules |
| T1.4 | Implement SCHEMA_LOADER | Parses JSON schema files |
| T1.5 | Implement ROW_VALIDATOR | Validates rows against schema |
| T1.6 | Implement ROW_ERROR class | Holds row, column, message, value |
| T1.7 | Implement CSV_VALIDATION_RESULT | Tracks validity, errors, stats |
| T1.8 | Implement CSV_REPORTER.to_text | Text error report |
| T1.9 | Implement CSV_VALIDATOR_ENGINE | Loads files, runs validation |
| T1.10 | Implement CSV_VALIDATOR_CLI | CLI with validate command |
| T1.11 | Write MVP tests | Core scenarios pass |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Valid CSV | users.csv matching schema | Exit 0, "Valid: 100 rows" |
| Missing required | users.csv missing email in row 5 | Exit 1, "Row 5: email - Required" |
| Invalid format | users.csv with bad email format | Exit 1, "Row 10: email - Invalid email format" |
| Invalid type | users.csv with non-integer id | Exit 1, "Row 15: id - Must be integer" |
| Out of range | users.csv with age=200 | Exit 1, "Row 20: age - Must be at most 150" |
| Invalid enum | users.csv with status="deleted" | Exit 1, "Row 25: status - Must be one of: ..." |
| File not found | nonexistent.csv | Exit 2, "CSV file not found" |
| Schema not found | --schema missing.json | Exit 2, "Schema file not found" |
| Invalid JSON | malformed schema | Exit 4, "Invalid schema" |

### Supported Column Types (MVP)

| Type | Validation |
|------|------------|
| string | Any text, optional length constraints |
| integer | Whole number, optional min/max |
| number | Decimal number, optional min/max |
| boolean | true/false/1/0 |
| email | Email format |
| url | URL format |
| uuid | UUID format |

### Supported Constraints (MVP)

| Constraint | Applies To | Validation |
|------------|------------|------------|
| required | All types | Must not be empty |
| min_length | string | Minimum characters |
| max_length | string | Maximum characters |
| min | integer/number | Minimum value |
| max | integer/number | Maximum value |
| pattern | string | Regex match |
| enum | string | Value in list |

## Phase 2: Analysis + Advanced Features

### Objective

Add schema inference from data, unique constraints, and advanced validation features. Enable users to generate schemas automatically.

### Deliverables

1. **analyze command** - Infer schema from CSV data
2. **CSV_ANALYZER** - Schema inference engine
3. **init command** - Generate schema template
4. **Unique constraints** - Column uniqueness validation
5. **Cross-column rules** - Related column validation
6. **JSON output format** - Structured JSON reports
7. **CSV output format** - Error list as CSV

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T2.1 | Implement CSV_ANALYZER | Infers types from data |
| T2.2 | Implement analyze command | Outputs suggested schema |
| T2.3 | Implement init command | Generates schema template |
| T2.4 | Add unique constraint | Detects duplicate values |
| T2.5 | Add row_count rule | Min/max row count |
| T2.6 | Add date/datetime types | Date validation |
| T2.7 | Implement JSON reporter | Valid JSON output |
| T2.8 | Implement CSV reporter | Errors as CSV |
| T2.9 | Add --max-errors option | Stop after N errors |
| T2.10 | Add --sample option | Random sample validation |
| T2.11 | Add glob pattern support | Validate multiple files |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Analyze integers | Column with 1,2,3 | Suggested type: integer |
| Analyze emails | Column with emails | Suggested type: email format |
| Analyze dates | Column with ISO dates | Suggested type: date |
| Unique violation | Duplicate emails | Exit 1, "email: Duplicate value" |
| Row count min | 0 rows, min=1 | Exit 1, "File must have at least 1 row" |
| Row count max | 1M rows, max=100K | Exit 1, "File exceeds 100K row limit" |
| JSON output | --format json | Valid JSON report |
| CSV output | --format csv | Valid CSV error list |
| Max errors | --max-errors 10 | Stops at 10 errors |

## Phase 3: Production Polish

### Objective

Production-ready with comprehensive reports, performance optimization, and documentation.

### Deliverables

1. **report command** - Generate detailed validation report
2. **Markdown reporter** - Rich markdown output
3. **Statistics collection** - Detailed validation stats
4. **Performance optimization** - Large file handling
5. **Progress indicator** - For large files
6. **Documentation** - README, schema guide

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T3.1 | Implement report command | Detailed validation report |
| T3.2 | Implement markdown reporter | Clean markdown output |
| T3.3 | Add column statistics | Min/max/avg/distinct per column |
| T3.4 | Add error summary | Errors by column/type |
| T3.5 | Optimize large files | Streaming validation |
| T3.6 | Add progress indicator | Shows % complete |
| T3.7 | Add --encoding option | Handle different encodings |
| T3.8 | Add --delimiter option | Custom delimiters |
| T3.9 | Write README.md | Installation, usage |
| T3.10 | Write schema guide | Full schema reference |
| T3.11 | Final testing | All edge cases |

## ECF Target Structure
```xml
<!-- Library target (reusable) -->
<target name="csv_intake_validator">
    <root class="CSV_VALIDATOR_CLI" feature="make"/>
    <library name="simple_validation" location="..."/>
    <library name="simple_csv" location="..."/>
    <library name="simple_json" location="..."/>
    <library name="simple_cli" location="..."/>
    <library name="simple_file" location="..."/>
    <cluster name="src" location=".\src\"/>
</target>

<!-- Test target -->
<target name="csv_intake_validator_tests" extends="csv_intake_validator">
    <root class="TEST_APP" feature="make"/>
    <library name="simple_testing" location="..."/>
    <cluster name="tests" location=".\tests\"/>
</target>
```

## Build Commands
```bash
# Compile CLI (workbench for development)
/d/prod/ec.sh -batch -config csv_intake_validator.ecf -target csv_intake_validator -c_compile

# Run tests
/d/prod/ec.sh -batch -config csv_intake_validator.ecf -target csv_intake_validator_tests -c_compile
./EIFGENs/csv_intake_validator_tests/W_code/csv_intake_validator.exe

# Compile finalized (production)
/d/prod/ec.sh -batch -config csv_intake_validator.ecf -target csv_intake_validator -finalize -c_compile
```

## Success Criteria

| Criterion | Measure | Target |
|-----------|---------|--------|
| Compiles | Zero errors | 100% |
| Tests pass | All tests | 100% |
| MVP commands | validate works | Functional |
| All commands | All 4 commands work | Functional |
| Performance | 100K rows | <30 seconds |
| Memory | 100K rows | <500MB |
| Documentation | README complete | Yes |

## File Structure
```
csv_intake_validator/
    csv_intake_validator.ecf
    README.md
    CHANGELOG.md
    src/
        csv_validator_cli.e
        csv_validator_engine.e
        schema_loader.e
        csv_schema.e
        csv_column_def.e
        csv_analyzer.e
        row_validator.e
        stats_collector.e
        csv_validation_result.e
        row_error.e
        csv_reporter.e
    tests/
        test_app.e
        test_schema_loader.e
        test_row_validator.e
        test_analyzer.e
    schemas/
        user-import.json
        product-catalog.json
        transaction-log.json
    examples/
        users.csv
        users-invalid.csv
        users-schema.json
```
