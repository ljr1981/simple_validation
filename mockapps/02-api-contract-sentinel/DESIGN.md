# APIContractSentinel - Technical Design

## Architecture

### Component Overview
```
+-------------------------------------------------------------+
|                    APIContractSentinel                       |
+-------------------------------------------------------------+
|  CLI Interface Layer                                         |
|    - Argument parsing (simple_cli)                          |
|    - Command routing (validate, mock, diff, report)         |
|    - Output formatting (text, json, junit, markdown)        |
+-------------------------------------------------------------+
|  Business Logic Layer                                        |
|    - CONTRACT_SENTINEL_ENGINE: Orchestration                |
|    - OPENAPI_PARSER: Parse OpenAPI 3.x specs                |
|    - SCHEMA_COMPILER: Convert OpenAPI schemas to rules      |
|    - PAYLOAD_VALIDATOR: Validate JSON against schema        |
|    - MOCK_GENERATOR: Generate mock payloads from schema     |
+-------------------------------------------------------------+
|  Integration Layer                                           |
|    - simple_validation: Core validation engine              |
|    - simple_json: JSON payload parsing                      |
|    - simple_yaml: OpenAPI spec parsing                      |
|    - simple_http: Live API testing                          |
|    - simple_template: Report generation                     |
+-------------------------------------------------------------+
```

### Class Design

| Class | Responsibility | Key Features |
|-------|----------------|--------------|
| CONTRACT_SENTINEL_CLI | Command-line interface | parse_args, execute, format_output |
| CONTRACT_SENTINEL_ENGINE | Core orchestration | load_spec, validate_payload, generate_report |
| OPENAPI_PARSER | Parse OpenAPI specs | parse_3_0, parse_3_1, resolve_refs |
| OPENAPI_SPEC | Parsed specification | paths, schemas, operations |
| OPENAPI_SCHEMA | JSON Schema from spec | type, properties, required, format |
| SCHEMA_COMPILER | Schema to validation rules | compile_schema, build_validator |
| PAYLOAD_VALIDATOR | Validate payloads | validate_request, validate_response |
| MOCK_GENERATOR | Generate mock data | generate_from_schema, randomize |
| SENTINEL_RESULT | Validation outcome | is_valid, violations, coverage |
| SENTINEL_VIOLATION | Single contract violation | path, expected, actual, message |
| SENTINEL_REPORTER | Output generation | to_text, to_json, to_junit |

### Command Structure
```bash
api-contract-sentinel <command> [options] [arguments]

Commands:
  validate <spec> <payload>    Validate payload against spec
  validate-live <spec> <url>   Validate live API response
  mock <spec> <operation>      Generate mock payload from schema
  diff <spec1> <spec2>         Compare two spec versions
  report <spec>                Generate spec coverage report
  version                      Show version information

Global Options:
  --format FORMAT       Output format: text, json, junit, markdown
  --strict              Treat warnings as errors
  --quiet               Only output violations
  --no-color            Disable colored output
  --verbose             Verbose output with debug info
  --help                Show help

Validate Options:
  --operation PATH      Specific operation (e.g., "GET /users/{id}")
  --request             Validate as request payload
  --response CODE       Validate as response with status code
  --all                 Validate against all matching schemas

Mock Options:
  --count N             Generate N mock payloads
  --seed VALUE          Random seed for reproducibility
  --realistic           Use realistic fake data

Examples:
  api-contract-sentinel validate api.yaml response.json --response 200
  api-contract-sentinel validate api.yaml request.json --request --operation "POST /users"
  api-contract-sentinel validate-live api.yaml https://api.example.com/users/123
  api-contract-sentinel mock api.yaml "POST /users" --count 10
  api-contract-sentinel diff api-v1.yaml api-v2.yaml
```

### Data Flow
```
OpenAPI Spec  -->  OPENAPI_PARSER  -->  OPENAPI_SPEC (paths, schemas)
                         |
                         v
                  SCHEMA_COMPILER  -->  Validation rules per schema
                         |
JSON Payload  -->  PAYLOAD_VALIDATOR  -->  Apply schema rules
                         |
                         v
                 SENTINEL_RESULT  -->  Violations with field paths
                         |
                         v
               SENTINEL_REPORTER  -->  Formatted output
```

### Schema Mapping
```yaml
# OpenAPI Schema to simple_validation rules

OpenAPI Property        simple_validation Rule
-------------------     ----------------------
type: string            (base type check)
type: integer           is_integer rule
type: number            is_number rule
type: boolean           (boolean check)
minLength               min_length rule
maxLength               max_length rule
minimum                 min_value rule
maximum                 max_value rule
pattern                 pattern rule
format: email           email rule
format: uri             url rule
format: uuid            uuid rule
enum                    one_of rule
required                required rule
```

### OpenAPI Schema Example
```yaml
# From OpenAPI spec
components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - name
      properties:
        id:
          type: integer
          minimum: 1
        email:
          type: string
          format: email
          maxLength: 254
        name:
          type: string
          minLength: 1
          maxLength: 100
        age:
          type: integer
          minimum: 0
          maximum: 150
        role:
          type: string
          enum: [admin, user, guest]
```

### Compiled Validation Rules
```eiffel
-- Generated from User schema
create validator.make
validator := validator
    .required.with_field_name ("id")
    .min_value (1)

validator := validator
    .required.with_field_name ("email")
    .email
    .max_length (254)

validator := validator
    .required.with_field_name ("name")
    .min_length (1)
    .max_length (100)

-- Optional fields
if has_field ("age") then
    validator := validator
        .min_value (0).max_value (150)
        .with_field_name ("age")
end

if has_field ("role") then
    validator := validator
        .one_of (<<"admin", "user", "guest">>)
        .with_field_name ("role")
end
```

### Error Handling

| Error Type | Handling | User Message |
|------------|----------|--------------|
| Spec not found | Exit 2 | "OpenAPI spec not found: {path}" |
| Invalid spec | Exit 3 | "Invalid OpenAPI spec: {details}" |
| Payload not found | Exit 2 | "Payload file not found: {path}" |
| Invalid JSON | Exit 3 | "Invalid JSON payload: {details}" |
| Validation failure | Exit 1 | Detailed violation report |
| Schema not found | Exit 4 | "Schema not found: {name}" |
| Network error | Exit 5 | "Failed to fetch: {url}" |

### Exit Codes
| Code | Meaning |
|------|---------|
| 0 | Validation passed |
| 1 | Validation failed (violations found) |
| 2 | File not found |
| 3 | Parse error (invalid spec or payload) |
| 4 | Schema not found |
| 5 | Network error |
| 255 | Internal error |

## GUI/TUI Future Path

**CLI foundation enables:**
- All validation logic in reusable classes (CONTRACT_SENTINEL_ENGINE)
- Format-agnostic result objects (SENTINEL_RESULT)
- Mock generation independent of output

**What would change for TUI:**
- Add simple_tui for interactive spec browser
- Real-time validation as user edits payloads
- Side-by-side spec vs payload view

**What would change for GUI:**
- Web dashboard for spec management
- Visual schema explorer
- Historical compliance reports
- Team collaboration features
- API change timeline

**Shared components between CLI/GUI:**
- CONTRACT_SENTINEL_ENGINE (100% reusable)
- OPENAPI_PARSER (100% reusable)
- PAYLOAD_VALIDATOR (100% reusable)
- MOCK_GENERATOR (100% reusable)
- SENTINEL_RESULT (100% reusable)
