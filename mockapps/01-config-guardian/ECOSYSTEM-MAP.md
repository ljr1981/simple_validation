# ConfigGuardian - Ecosystem Integration

## simple_* Dependencies

### Required Libraries

| Library | Purpose | Integration Point |
|---------|---------|-------------------|
| simple_validation | Core validation engine | Field validation rules |
| simple_yaml | YAML config parsing | Load YAML files and rules |
| simple_json | JSON config parsing | Load JSON files, JSON output |
| simple_cli | Command-line interface | Argument parsing, help |
| simple_file | File operations | Read config and rule files |

### Optional Libraries

| Library | Purpose | When Needed |
|---------|---------|-------------|
| simple_toml | TOML config parsing | When validating TOML files |
| simple_template | Report generation | For markdown/HTML reports |
| simple_logger | Debug logging | When --verbose enabled |
| simple_env | Environment variables | For config path expansion |

## Integration Patterns

### simple_validation Integration

**Purpose:** Core validation logic for all field-level rules

**Usage:**
```eiffel
class CONFIG_GUARDIAN_ENGINE

feature -- Validation

    validate_field (a_path: STRING; a_value: ANY; a_rule: GUARDIAN_RULE): VALIDATION_RESULT
            -- Validate single field against rule.
        local
            l_validator: SIMPLE_VALIDATOR
        do
            create l_validator.make

            -- Apply rule constraints
            if a_rule.is_required then
                l_validator := l_validator.required
            end
            if attached a_rule.min_length as ml then
                l_validator := l_validator.min_length (ml)
            end
            if attached a_rule.max_length as ml then
                l_validator := l_validator.max_length (ml)
            end
            if attached a_rule.pattern as p then
                l_validator := l_validator.pattern (p)
            end
            if attached a_rule.one_of_values as vals then
                l_validator := l_validator.one_of (vals)
            end

            -- Set field context
            l_validator := l_validator.with_field_name (a_path)

            -- Execute validation
            Result := l_validator.validate (a_value)
        end

end
```

**Data flow:** Rule definition -> SIMPLE_VALIDATOR chain -> VALIDATION_RESULT

### simple_yaml Integration

**Purpose:** Parse YAML configuration files and rule definitions

**Usage:**
```eiffel
class CONFIG_PARSER

feature -- YAML Parsing

    parse_yaml (a_path: STRING): YAML_DOCUMENT
            -- Parse YAML file into navigable structure.
        local
            l_parser: SIMPLE_YAML
            l_content: STRING
        do
            l_content := file_reader.read_all (a_path)
            create l_parser.make
            Result := l_parser.parse (l_content)
        ensure
            document_valid: Result.is_valid
        end

    yaml_value_at (a_doc: YAML_DOCUMENT; a_path: STRING): detachable ANY
            -- Get value at dotted path like "server.port".
        local
            l_parts: LIST [STRING]
            l_current: YAML_NODE
        do
            l_parts := a_path.split ('.')
            l_current := a_doc.root
            across l_parts as p loop
                if attached {YAML_MAP} l_current as m then
                    l_current := m.item (p)
                end
            end
            Result := l_current.value
        end

end
```

**Data flow:** File path -> YAML string -> YAML_DOCUMENT -> Nested values

### simple_json Integration

**Purpose:** Parse JSON files and generate JSON output

**Usage:**
```eiffel
class GUARDIAN_REPORTER

feature -- JSON Output

    to_json (a_result: GUARDIAN_RESULT): STRING
            -- Generate JSON report.
        local
            l_json: SIMPLE_JSON_OBJECT
            l_errors: SIMPLE_JSON_ARRAY
        do
            create l_json.make
            l_json.put_boolean ("valid", a_result.is_valid)
            l_json.put_integer ("error_count", a_result.error_count)
            l_json.put_integer ("warning_count", a_result.warning_count)

            create l_errors.make
            across a_result.errors as e loop
                l_errors.extend (error_to_json (e))
            end
            l_json.put_array ("errors", l_errors)

            Result := l_json.to_string
        end

end
```

**Data flow:** GUARDIAN_RESULT -> SIMPLE_JSON objects -> JSON string

### simple_cli Integration

**Purpose:** Command-line argument parsing and help generation

**Usage:**
```eiffel
class CONFIG_GUARDIAN_CLI

feature {NONE} -- Initialization

    make
            -- Initialize CLI and parse arguments.
        local
            l_cli: SIMPLE_CLI
        do
            create l_cli.make ("config-guardian", "Configuration file validator")

            -- Commands
            l_cli.add_command ("validate", "Validate config file against rules")
            l_cli.add_command ("init", "Initialize rule file")
            l_cli.add_command ("list-rules", "List built-in rule types")

            -- Global options
            l_cli.add_option ("rules", "r", "Rule file path", True)
            l_cli.add_option ("format", "f", "Output format", True)
            l_cli.add_flag ("strict", "s", "Treat warnings as errors")
            l_cli.add_flag ("quiet", "q", "Only output errors")
            l_cli.add_flag ("verbose", "v", "Verbose output")

            -- Parse and route
            l_cli.parse
            if l_cli.has_command then
                execute_command (l_cli.command, l_cli)
            else
                l_cli.show_help
            end
        end

end
```

**Data flow:** Command-line args -> SIMPLE_CLI -> Parsed options -> Command execution

### simple_file Integration

**Purpose:** File existence checks, reading config and rule files

**Usage:**
```eiffel
class CONFIG_GUARDIAN_ENGINE

feature -- File Operations

    load_config_file (a_path: STRING): detachable CONFIG_DOCUMENT
            -- Load and parse config file.
        local
            l_file: SIMPLE_FILE
        do
            create l_file.make (a_path)
            if l_file.exists then
                Result := parse_by_extension (a_path, l_file.read_all)
            end
        end

    detect_format (a_path: STRING): STRING
            -- Detect config format from file extension.
        local
            l_file: SIMPLE_FILE
        do
            create l_file.make (a_path)
            inspect l_file.extension.as_lower
            when "yaml", "yml" then Result := "yaml"
            when "json" then Result := "json"
            when "toml" then Result := "toml"
            when "ini", "cfg" then Result := "ini"
            else Result := "unknown"
            end
        end

end
```

**Data flow:** File path -> SIMPLE_FILE -> Content string -> Parser

## Dependency Graph
```
config_guardian
    |
    +-- simple_validation (required)
    |       +-- simple_mml
    |       +-- gobo regexp
    |
    +-- simple_yaml (required)
    |       +-- simple_json
    |
    +-- simple_json (required)
    |
    +-- simple_cli (required)
    |
    +-- simple_file (required)
    |
    +-- simple_toml (optional)
    |
    +-- simple_template (optional)
    |       +-- simple_json
    |
    +-- simple_logger (optional)
    |
    +-- simple_env (optional)
    |
    +-- ISE base (required)
```

## ECF Configuration
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system name="config_guardian" uuid="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" xmlns="http://www.eiffel.com/developers/xml/configuration-1-22-0">
    <description>Configuration file validator with custom business rules</description>

    <target name="config_guardian">
        <root class="CONFIG_GUARDIAN_CLI" feature="make"/>

        <option warning="warning" syntax="provisional" manifest_array_type="mismatch_warning">
            <assertions precondition="true" postcondition="true" check="true" invariant="true"/>
        </option>

        <setting name="console_application" value="true"/>
        <setting name="concurrency" value="none"/>

        <!-- Core simple_* dependencies -->
        <library name="simple_validation" location="$SIMPLE_EIFFEL/simple_validation/simple_validation.ecf"/>
        <library name="simple_yaml" location="$SIMPLE_EIFFEL/simple_yaml/simple_yaml.ecf"/>
        <library name="simple_json" location="$SIMPLE_EIFFEL/simple_json/simple_json.ecf"/>
        <library name="simple_cli" location="$SIMPLE_EIFFEL/simple_cli/simple_cli.ecf"/>
        <library name="simple_file" location="$SIMPLE_EIFFEL/simple_file/simple_file.ecf"/>

        <!-- Optional dependencies -->
        <library name="simple_toml" location="$SIMPLE_EIFFEL/simple_toml/simple_toml.ecf"/>
        <library name="simple_template" location="$SIMPLE_EIFFEL/simple_template/simple_template.ecf"/>
        <library name="simple_logger" location="$SIMPLE_EIFFEL/simple_logger/simple_logger.ecf"/>
        <library name="simple_env" location="$SIMPLE_EIFFEL/simple_env/simple_env.ecf"/>

        <!-- ISE dependencies -->
        <library name="base" location="$ISE_LIBRARY/library/base/base.ecf"/>

        <!-- Application source -->
        <cluster name="src" location=".\src\" recursive="true"/>
    </target>

    <target name="config_guardian_tests" extends="config_guardian">
        <root class="TEST_APP" feature="make"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL/simple_testing/simple_testing.ecf"/>
        <cluster name="tests" location=".\tests\" recursive="true"/>
    </target>
</system>
```

## Library Synergy Benefits

| Combination | Synergy |
|-------------|---------|
| simple_validation + simple_yaml | Validate YAML values with fluent rules |
| simple_validation + simple_json | Validate JSON and output structured reports |
| simple_cli + simple_file | Clean arg parsing with file validation |
| simple_yaml + simple_json | Rule files in YAML, output in JSON |
| simple_template + simple_json | Rich HTML/Markdown reports |
