# APIContractSentinel - Ecosystem Integration

## simple_* Dependencies

### Required Libraries

| Library | Purpose | Integration Point |
|---------|---------|-------------------|
| simple_validation | Core validation engine | Schema-based payload validation |
| simple_json | JSON payload parsing | Load payloads, generate JSON output |
| simple_yaml | OpenAPI spec parsing | Load YAML specs |
| simple_cli | Command-line interface | Argument parsing, help |
| simple_file | File operations | Read specs and payloads |

### Optional Libraries

| Library | Purpose | When Needed |
|---------|---------|-------------|
| simple_http | Live API testing | validate-live command |
| simple_template | Report generation | Markdown/HTML reports |
| simple_logger | Debug logging | When --verbose enabled |
| simple_uuid | UUID validation/generation | UUID format, mock data |

## Integration Patterns

### simple_validation Integration

**Purpose:** Core validation logic for schema-based validation

**Usage:**
```eiffel
class SCHEMA_COMPILER

feature -- Schema Compilation

    compile_schema (a_schema: OPENAPI_SCHEMA): SIMPLE_VALIDATOR
            -- Convert OpenAPI schema to validator chain.
        local
            l_validator: SIMPLE_VALIDATOR
        do
            create l_validator.make

            -- Handle required
            if a_schema.is_required then
                l_validator := l_validator.required
            end

            -- Handle type-specific constraints
            inspect a_schema.type_name
            when "string" then
                l_validator := compile_string_schema (l_validator, a_schema)
            when "integer", "number" then
                l_validator := compile_numeric_schema (l_validator, a_schema)
            else
                -- Object/array types handled recursively
            end

            Result := l_validator
        end

    compile_string_schema (a_validator: SIMPLE_VALIDATOR; a_schema: OPENAPI_SCHEMA): SIMPLE_VALIDATOR
            -- Add string-specific validation rules.
        do
            Result := a_validator

            -- Length constraints
            if attached a_schema.min_length as ml then
                Result := Result.min_length (ml)
            end
            if attached a_schema.max_length as ml then
                Result := Result.max_length (ml)
            end

            -- Pattern constraint
            if attached a_schema.pattern as p then
                Result := Result.pattern (p)
            end

            -- Format constraints
            if attached a_schema.format as f then
                inspect f
                when "email" then Result := Result.email
                when "uri", "url" then Result := Result.url
                when "uuid" then Result := Result.uuid
                else
                    -- Unknown format, skip
                end
            end

            -- Enum constraint
            if attached a_schema.enum_values as ev then
                Result := Result.one_of (ev)
            end
        end

end
```

**Data flow:** OpenAPI Schema -> SIMPLE_VALIDATOR chain -> VALIDATION_RESULT

### simple_json Integration

**Purpose:** Parse JSON payloads and generate structured output

**Usage:**
```eiffel
class PAYLOAD_VALIDATOR

feature -- Payload Validation

    validate_json_payload (a_json: STRING; a_schema: OPENAPI_SCHEMA): SENTINEL_RESULT
            -- Validate JSON payload against compiled schema.
        local
            l_parser: SIMPLE_JSON
            l_obj: SIMPLE_JSON_OBJECT
            l_validator: SIMPLE_VALIDATOR
            l_result: VALIDATION_RESULT
        do
            create Result.make

            -- Parse JSON
            create l_parser.make
            if attached l_parser.parse (a_json) as parsed then
                if attached {SIMPLE_JSON_OBJECT} parsed as obj then
                    -- Validate each property
                    across a_schema.properties as prop loop
                        l_validator := schema_compiler.compile_schema (prop)
                        l_validator := l_validator.with_field_name (prop.key)

                        if obj.has (prop.key) then
                            l_result := l_validator.validate (obj.item (prop.key))
                        elseif prop.is_required then
                            l_result := l_validator.validate (Void)
                        end

                        if not l_result.is_valid then
                            Result.add_violations (l_result.errors)
                        end
                    end
                end
            else
                Result.add_parse_error (l_parser.error_message)
            end
        end

end
```

**Data flow:** JSON string -> SIMPLE_JSON parser -> Nested values -> Validation

### simple_yaml Integration

**Purpose:** Parse OpenAPI specification files

**Usage:**
```eiffel
class OPENAPI_PARSER

feature -- Spec Parsing

    parse_spec (a_path: STRING): OPENAPI_SPEC
            -- Parse OpenAPI specification file.
        local
            l_yaml: SIMPLE_YAML
            l_content: STRING
            l_doc: YAML_DOCUMENT
        do
            l_content := file_reader.read_all (a_path)
            create l_yaml.make
            l_doc := l_yaml.parse (l_content)

            create Result.make

            -- Extract spec version
            Result.set_version (l_doc.string_at ("openapi"))

            -- Extract paths
            if attached l_doc.map_at ("paths") as paths then
                across paths as path loop
                    Result.add_path (parse_path (path.key, path))
                end
            end

            -- Extract schemas
            if attached l_doc.map_at ("components/schemas") as schemas then
                across schemas as schema loop
                    Result.add_schema (schema.key, parse_schema (schema))
                end
            end
        ensure
            valid_spec: Result.is_valid
        end

end
```

**Data flow:** YAML file -> SIMPLE_YAML -> YAML_DOCUMENT -> OPENAPI_SPEC

### simple_http Integration

**Purpose:** Validate live API responses

**Usage:**
```eiffel
class CONTRACT_SENTINEL_ENGINE

feature -- Live Validation

    validate_live (a_spec: OPENAPI_SPEC; a_url: STRING; a_operation: STRING): SENTINEL_RESULT
            -- Validate live API response against spec.
        local
            l_http: SIMPLE_HTTP
            l_response: HTTP_RESPONSE
            l_schema: OPENAPI_SCHEMA
        do
            create l_http.make

            -- Make request
            l_response := l_http.get (a_url)

            -- Find matching schema
            l_schema := a_spec.response_schema (a_operation, l_response.status_code)

            -- Validate response body
            Result := payload_validator.validate_json_payload (
                l_response.body,
                l_schema
            )

            -- Add response metadata
            Result.set_status_code (l_response.status_code)
            Result.set_response_time (l_response.duration_ms)
        end

end
```

**Data flow:** URL -> SIMPLE_HTTP request -> Response -> Validation

### simple_cli Integration

**Purpose:** Command-line argument parsing and routing

**Usage:**
```eiffel
class CONTRACT_SENTINEL_CLI

feature {NONE} -- Initialization

    make
            -- Initialize CLI and parse arguments.
        local
            l_cli: SIMPLE_CLI
        do
            create l_cli.make ("api-contract-sentinel", "API contract validation tool")

            -- Commands
            l_cli.add_command ("validate", "Validate payload against spec")
            l_cli.add_command ("validate-live", "Validate live API response")
            l_cli.add_command ("mock", "Generate mock payload from schema")
            l_cli.add_command ("diff", "Compare two spec versions")
            l_cli.add_command ("report", "Generate coverage report")

            -- Options
            l_cli.add_option ("format", "f", "Output format", True)
            l_cli.add_option ("operation", "o", "Specific operation", True)
            l_cli.add_option ("response", "r", "Response status code", True)
            l_cli.add_flag ("request", "", "Validate as request")
            l_cli.add_flag ("strict", "s", "Strict mode")

            -- Parse and route
            l_cli.parse
            execute_command (l_cli)
        end

end
```

## Dependency Graph
```
api_contract_sentinel
    |
    +-- simple_validation (required)
    |       +-- simple_mml
    |       +-- gobo regexp
    |
    +-- simple_json (required)
    |
    +-- simple_yaml (required)
    |       +-- simple_json
    |
    +-- simple_cli (required)
    |
    +-- simple_file (required)
    |
    +-- simple_http (optional)
    |       +-- simple_json
    |
    +-- simple_template (optional)
    |       +-- simple_json
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
<system name="api_contract_sentinel" uuid="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX">
    <description>API contract validation against OpenAPI specifications</description>

    <target name="api_contract_sentinel">
        <root class="CONTRACT_SENTINEL_CLI" feature="make"/>

        <option warning="warning" syntax="provisional">
            <assertions precondition="true" postcondition="true" check="true" invariant="true"/>
        </option>

        <setting name="console_application" value="true"/>

        <!-- Core simple_* dependencies -->
        <library name="simple_validation" location="$SIMPLE_EIFFEL/simple_validation/simple_validation.ecf"/>
        <library name="simple_json" location="$SIMPLE_EIFFEL/simple_json/simple_json.ecf"/>
        <library name="simple_yaml" location="$SIMPLE_EIFFEL/simple_yaml/simple_yaml.ecf"/>
        <library name="simple_cli" location="$SIMPLE_EIFFEL/simple_cli/simple_cli.ecf"/>
        <library name="simple_file" location="$SIMPLE_EIFFEL/simple_file/simple_file.ecf"/>

        <!-- Optional dependencies -->
        <library name="simple_http" location="$SIMPLE_EIFFEL/simple_http/simple_http.ecf"/>
        <library name="simple_template" location="$SIMPLE_EIFFEL/simple_template/simple_template.ecf"/>
        <library name="simple_uuid" location="$SIMPLE_EIFFEL/simple_uuid/simple_uuid.ecf"/>
        <library name="simple_logger" location="$SIMPLE_EIFFEL/simple_logger/simple_logger.ecf"/>

        <!-- ISE dependencies -->
        <library name="base" location="$ISE_LIBRARY/library/base/base.ecf"/>

        <!-- Application source -->
        <cluster name="src" location=".\src\" recursive="true"/>
    </target>

    <target name="api_contract_sentinel_tests" extends="api_contract_sentinel">
        <root class="TEST_APP" feature="make"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL/simple_testing/simple_testing.ecf"/>
        <cluster name="tests" location=".\tests\" recursive="true"/>
    </target>
</system>
```

## Library Synergy Benefits

| Combination | Synergy |
|-------------|---------|
| simple_validation + simple_json | Validate JSON payloads with fluent rules |
| simple_yaml + simple_json | Parse YAML specs, validate JSON payloads |
| simple_http + simple_validation | Live API contract testing |
| simple_cli + simple_file | Clean arg parsing with file handling |
| simple_uuid + simple_validation | UUID format validation and mock generation |
