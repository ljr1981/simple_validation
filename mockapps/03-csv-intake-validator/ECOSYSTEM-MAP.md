# CSVIntakeValidator - Ecosystem Integration

## simple_* Dependencies

### Required Libraries

| Library | Purpose | Integration Point |
|---------|---------|-------------------|
| simple_validation | Core validation engine | Cell-level validation rules |
| simple_csv | CSV file parsing | Load and parse CSV files |
| simple_json | Schema definitions | Load JSON schema files |
| simple_cli | Command-line interface | Argument parsing, help |
| simple_file | File operations | Read CSV and schema files |

### Optional Libraries

| Library | Purpose | When Needed |
|---------|---------|-------------|
| simple_template | Report generation | Markdown/HTML reports |
| simple_logger | Debug logging | When --verbose enabled |
| simple_datetime | Date validation | Date/datetime columns |
| simple_uuid | UUID validation | UUID format columns |

## Integration Patterns

### simple_validation Integration

**Purpose:** Core validation logic for cell-level validation

**Usage:**
```eiffel
class ROW_VALIDATOR

feature -- Row Validation

    validate_row (a_row: CSV_ROW; a_schema: CSV_SCHEMA): ARRAYED_LIST [ROW_ERROR]
            -- Validate all cells in row against schema.
        local
            l_validator: SIMPLE_VALIDATOR
            l_result: VALIDATION_RESULT
            l_value: STRING
            l_col_def: CSV_COLUMN_DEF
        do
            create Result.make (0)

            across a_schema.columns as col loop
                l_col_def := col
                l_value := a_row.cell (col.name)

                -- Build validator for column
                l_validator := build_validator (l_col_def)
                l_validator := l_validator.with_field_name (col.name)

                -- Validate cell
                l_result := l_validator.validate (l_value)

                -- Collect errors
                if not l_result.is_valid then
                    across l_result.errors as err loop
                        Result.extend (create {ROW_ERROR}.make (
                            a_row.row_number,
                            col.name,
                            err.message,
                            l_value
                        ))
                    end
                end
            end
        end

    build_validator (a_col: CSV_COLUMN_DEF): SIMPLE_VALIDATOR
            -- Build validator chain for column definition.
        local
            l_validator: SIMPLE_VALIDATOR
        do
            create l_validator.make

            -- Required
            if a_col.is_required then
                l_validator := l_validator.required
            end

            -- Type-specific validation
            inspect a_col.type_name
            when "integer" then
                -- Integer validation handled in SIMPLE_VALIDATION_QUICK
            when "email" then
                l_validator := l_validator.email
            when "url" then
                l_validator := l_validator.url
            when "uuid" then
                l_validator := l_validator.uuid
            else
                -- String by default
            end

            -- Length constraints
            if attached a_col.min_length as ml then
                l_validator := l_validator.min_length (ml)
            end
            if attached a_col.max_length as ml then
                l_validator := l_validator.max_length (ml)
            end

            -- Pattern constraint
            if attached a_col.pattern as p then
                l_validator := l_validator.pattern (p)
            end

            -- Enum constraint
            if attached a_col.enum_values as ev then
                l_validator := l_validator.one_of (ev)
            end

            Result := l_validator
        end

end
```

**Data flow:** Column definition -> SIMPLE_VALIDATOR chain -> Cell validation

### simple_csv Integration

**Purpose:** Parse CSV files with configurable options

**Usage:**
```eiffel
class CSV_VALIDATOR_ENGINE

feature -- CSV Loading

    load_csv_file (a_path: STRING; a_schema: CSV_SCHEMA): ARRAYED_LIST [CSV_ROW]
            -- Load and parse CSV file.
        local
            l_csv: SIMPLE_CSV
            l_options: CSV_OPTIONS
        do
            create l_csv.make

            -- Apply schema settings
            create l_options.make
            l_options.set_delimiter (a_schema.delimiter)
            l_options.set_has_header (a_schema.has_header_row)
            l_options.set_trim_whitespace (a_schema.trim_whitespace)

            l_csv.set_options (l_options)

            -- Parse file
            Result := l_csv.parse_file (a_path)

            -- Validate header if present
            if a_schema.has_header_row then
                validate_header (Result.first, a_schema)
            end
        ensure
            rows_loaded: Result.count > 0
        end

    validate_header (a_header_row: CSV_ROW; a_schema: CSV_SCHEMA)
            -- Validate CSV header matches schema columns.
        local
            l_expected: ARRAYED_LIST [STRING]
            l_actual: ARRAYED_LIST [STRING]
        do
            l_expected := a_schema.column_names
            l_actual := a_header_row.cells

            -- Check for missing columns
            across l_expected as col loop
                if not l_actual.has (col) then
                    report_error ("Missing column: " + col)
                end
            end

            -- Check for extra columns (if not allowed)
            if not a_schema.allow_extra_columns then
                across l_actual as col loop
                    if not l_expected.has (col) then
                        report_error ("Unexpected column: " + col)
                    end
                end
            end
        end

end
```

**Data flow:** File path -> SIMPLE_CSV parser -> Parsed rows

### simple_json Integration

**Purpose:** Parse schema definition files

**Usage:**
```eiffel
class SCHEMA_LOADER

feature -- Schema Loading

    load_schema (a_path: STRING): CSV_SCHEMA
            -- Load schema from JSON file.
        local
            l_json: SIMPLE_JSON
            l_content: STRING
            l_obj: SIMPLE_JSON_OBJECT
        do
            l_content := file_reader.read_all (a_path)
            create l_json.make

            if attached l_json.parse (l_content) as parsed then
                if attached {SIMPLE_JSON_OBJECT} parsed as obj then
                    Result := parse_schema_object (obj)
                end
            end
        ensure
            valid_schema: Result.is_valid
        end

    parse_schema_object (a_obj: SIMPLE_JSON_OBJECT): CSV_SCHEMA
            -- Parse schema from JSON object.
        local
            l_columns: ARRAYED_LIST [CSV_COLUMN_DEF]
            l_settings: SIMPLE_JSON_OBJECT
        do
            create Result.make

            -- Parse settings
            if attached a_obj.object_at ("settings") as settings then
                Result.set_has_header (settings.boolean_at ("header_row"))
                Result.set_delimiter (settings.string_at ("delimiter").item (1))
                Result.set_trim_whitespace (settings.boolean_at ("trim_whitespace"))
            end

            -- Parse columns
            if attached a_obj.array_at ("columns") as cols then
                across cols as col loop
                    if attached {SIMPLE_JSON_OBJECT} col as col_obj then
                        Result.add_column (parse_column_def (col_obj))
                    end
                end
            end

            -- Parse rules
            if attached a_obj.array_at ("rules") as rules then
                across rules as rule loop
                    if attached {SIMPLE_JSON_OBJECT} rule as rule_obj then
                        Result.add_rule (parse_rule (rule_obj))
                    end
                end
            end
        end

end
```

**Data flow:** JSON file -> SIMPLE_JSON -> CSV_SCHEMA

### simple_cli Integration

**Purpose:** Command-line argument parsing and routing

**Usage:**
```eiffel
class CSV_VALIDATOR_CLI

feature {NONE} -- Initialization

    make
            -- Initialize CLI and parse arguments.
        local
            l_cli: SIMPLE_CLI
        do
            create l_cli.make ("csv-intake-validator", "CSV data validation tool")

            -- Commands
            l_cli.add_command ("validate", "Validate CSV against schema")
            l_cli.add_command ("analyze", "Analyze file and suggest schema")
            l_cli.add_command ("init", "Initialize schema template")
            l_cli.add_command ("report", "Generate validation report")

            -- Options
            l_cli.add_option ("schema", "s", "Schema file path", True)
            l_cli.add_option ("format", "f", "Output format", True)
            l_cli.add_option ("max-errors", "m", "Maximum errors", True)
            l_cli.add_option ("encoding", "e", "File encoding", True)
            l_cli.add_flag ("strict", "", "Fail on first error")
            l_cli.add_flag ("quiet", "q", "Only output summary")

            -- Parse and route
            l_cli.parse
            execute_command (l_cli)
        end

end
```

## Dependency Graph
```
csv_intake_validator
    |
    +-- simple_validation (required)
    |       +-- simple_mml
    |       +-- gobo regexp
    |
    +-- simple_csv (required)
    |
    +-- simple_json (required)
    |
    +-- simple_cli (required)
    |
    +-- simple_file (required)
    |
    +-- simple_template (optional)
    |       +-- simple_json
    |
    +-- simple_datetime (optional)
    |
    +-- simple_uuid (optional)
    |
    +-- simple_logger (optional)
    |
    +-- ISE base (required)
```

## ECF Configuration
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system name="csv_intake_validator" uuid="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX">
    <description>CSV data validation against schema definitions</description>

    <target name="csv_intake_validator">
        <root class="CSV_VALIDATOR_CLI" feature="make"/>

        <option warning="warning" syntax="provisional">
            <assertions precondition="true" postcondition="true" check="true" invariant="true"/>
        </option>

        <setting name="console_application" value="true"/>

        <!-- Core simple_* dependencies -->
        <library name="simple_validation" location="$SIMPLE_EIFFEL/simple_validation/simple_validation.ecf"/>
        <library name="simple_csv" location="$SIMPLE_EIFFEL/simple_csv/simple_csv.ecf"/>
        <library name="simple_json" location="$SIMPLE_EIFFEL/simple_json/simple_json.ecf"/>
        <library name="simple_cli" location="$SIMPLE_EIFFEL/simple_cli/simple_cli.ecf"/>
        <library name="simple_file" location="$SIMPLE_EIFFEL/simple_file/simple_file.ecf"/>

        <!-- Optional dependencies -->
        <library name="simple_template" location="$SIMPLE_EIFFEL/simple_template/simple_template.ecf"/>
        <library name="simple_datetime" location="$SIMPLE_EIFFEL/simple_datetime/simple_datetime.ecf"/>
        <library name="simple_uuid" location="$SIMPLE_EIFFEL/simple_uuid/simple_uuid.ecf"/>
        <library name="simple_logger" location="$SIMPLE_EIFFEL/simple_logger/simple_logger.ecf"/>

        <!-- ISE dependencies -->
        <library name="base" location="$ISE_LIBRARY/library/base/base.ecf"/>

        <!-- Application source -->
        <cluster name="src" location=".\src\" recursive="true"/>
    </target>

    <target name="csv_intake_validator_tests" extends="csv_intake_validator">
        <root class="TEST_APP" feature="make"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL/simple_testing/simple_testing.ecf"/>
        <cluster name="tests" location=".\tests\" recursive="true"/>
    </target>
</system>
```

## Library Synergy Benefits

| Combination | Synergy |
|-------------|---------|
| simple_validation + simple_csv | Validate parsed CSV cells with fluent rules |
| simple_csv + simple_json | Parse CSV data, schema from JSON |
| simple_cli + simple_file | Clean arg parsing with file validation |
| simple_template + simple_json | Rich HTML/Markdown reports |
| simple_datetime + simple_validation | Enhanced date/time validation |
| simple_uuid + simple_validation | UUID format validation |
