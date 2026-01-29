# APIContractSentinel - Build Plan

## Phase Overview

| Phase | Deliverable | Effort | Dependencies |
|-------|-------------|--------|--------------|
| Phase 1 | MVP CLI | 6 days | simple_validation, simple_json, simple_yaml, simple_cli |
| Phase 2 | Live Testing + Mock | 5 days | Phase 1, simple_http, simple_uuid |
| Phase 3 | Polish + Reports | 4 days | Phase 2, simple_template |

## Phase 1: MVP

### Objective

Demonstrate core value: validate a JSON payload against an OpenAPI schema and report violations. This proves the concept works and provides immediate value for API developers.

### Deliverables

1. **CONTRACT_SENTINEL_CLI** - Basic CLI with validate command
2. **CONTRACT_SENTINEL_ENGINE** - Core orchestration
3. **OPENAPI_PARSER** - Parse OpenAPI 3.0/3.1 specs
4. **OPENAPI_SPEC** - Parsed specification data structure
5. **OPENAPI_SCHEMA** - Individual schema representation
6. **SCHEMA_COMPILER** - Convert schemas to validation rules
7. **PAYLOAD_VALIDATOR** - Validate JSON against schemas
8. **SENTINEL_RESULT** - Validation result with violations
9. **SENTINEL_REPORTER** - Text output format

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T1.1 | Create ECF with dependencies | Compiles with all required libraries |
| T1.2 | Implement OPENAPI_SCHEMA class | Holds type, properties, constraints |
| T1.3 | Implement OPENAPI_SPEC class | Holds paths, schemas, operations |
| T1.4 | Implement OPENAPI_PARSER | Parses YAML OpenAPI specs |
| T1.5 | Implement $ref resolution | Resolves schema references |
| T1.6 | Implement SCHEMA_COMPILER | Converts schemas to validators |
| T1.7 | Implement PAYLOAD_VALIDATOR | Validates JSON against schemas |
| T1.8 | Implement SENTINEL_RESULT | Tracks violations |
| T1.9 | Implement SENTINEL_REPORTER.to_text | Text violation report |
| T1.10 | Implement CONTRACT_SENTINEL_CLI | CLI with validate command |
| T1.11 | Write MVP tests | Core scenarios pass |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Valid payload | User JSON matching schema | Exit 0, "Valid" |
| Missing required | User JSON without email | Exit 1, "email: This field is required" |
| Invalid type | User JSON with id="abc" | Exit 1, "id: Must be a valid integer" |
| Invalid format | User JSON with email="invalid" | Exit 1, "email: Must be a valid email" |
| Out of range | User JSON with age=200 | Exit 1, "age: Must be at most 150" |
| Invalid enum | User JSON with role="superuser" | Exit 1, "role: Must be one of: admin, user, guest" |
| Spec not found | nonexistent.yaml | Exit 2, "OpenAPI spec not found" |
| Invalid JSON | malformed payload | Exit 3, "Invalid JSON payload" |

### Supported Schema Features (MVP)

| OpenAPI Feature | Validation |
|-----------------|------------|
| type: string | String type check |
| type: integer | Integer validation |
| type: number | Number validation |
| type: boolean | Boolean check |
| required | Required field check |
| minLength/maxLength | String length |
| minimum/maximum | Numeric range |
| pattern | Regex pattern |
| format: email/uri/uuid | Format validation |
| enum | Enumeration |
| $ref | Schema reference resolution |

## Phase 2: Live Testing + Mock Generation

### Objective

Add live API validation and mock payload generation. Enable testing against real APIs and generating test data from schemas.

### Deliverables

1. **validate-live command** - Validate live API responses
2. **mock command** - Generate mock payloads from schemas
3. **MOCK_GENERATOR** - Generate realistic fake data
4. **diff command** - Compare two spec versions
5. **JSON output format** - Structured JSON reports
6. **JUnit output format** - CI/CD integration

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T2.1 | Implement validate-live command | Validates live API responses |
| T2.2 | Add simple_http integration | HTTP GET/POST requests work |
| T2.3 | Implement MOCK_GENERATOR | Generates valid mock data |
| T2.4 | Add realistic data generation | Emails, names, UUIDs look real |
| T2.5 | Implement diff command | Detects breaking changes |
| T2.6 | Implement JSON reporter | Valid JSON output |
| T2.7 | Implement JUnit reporter | CI/CD compatible XML |
| T2.8 | Add operation filtering | --operation flag works |
| T2.9 | Add response code filtering | --response flag works |
| T2.10 | Handle nested objects | Recursive schema validation |
| T2.11 | Handle arrays | Array item validation |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Live valid | GET https://api.example.com/users/1 | Exit 0 if matches spec |
| Live invalid | GET with response violating spec | Exit 1, violations listed |
| Mock single | mock "POST /users" | Valid JSON matching schema |
| Mock multiple | mock "POST /users" --count 10 | 10 unique valid payloads |
| Diff compatible | v1.yaml vs v2.yaml (compatible) | Exit 0, "No breaking changes" |
| Diff breaking | v1.yaml vs v2.yaml (breaking) | Exit 1, breaking changes listed |
| JSON output | --format json | Valid JSON report |
| JUnit output | --format junit | Valid JUnit XML |

## Phase 3: Production Polish

### Objective

Production-ready with comprehensive reports, documentation, and CI/CD integration guides.

### Deliverables

1. **report command** - Generate spec coverage report
2. **Markdown reporter** - Rich markdown output
3. **Array validation** - Complete array support
4. **anyOf/oneOf/allOf** - Complex schema combinations
5. **Documentation** - README, CI/CD guides
6. **Performance optimization** - Large spec handling

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T3.1 | Implement report command | Shows spec coverage |
| T3.2 | Implement markdown reporter | Clean markdown output |
| T3.3 | Add anyOf support | Validates against any matching schema |
| T3.4 | Add oneOf support | Validates against exactly one schema |
| T3.5 | Add allOf support | Validates against all schemas |
| T3.6 | Add nullable support | Null values where allowed |
| T3.7 | Add discriminator support | Polymorphic schemas |
| T3.8 | Optimize large specs | <5s for 100+ paths |
| T3.9 | Add --all flag | Validate against all operations |
| T3.10 | Write README.md | Installation, usage, examples |
| T3.11 | Write CI/CD guide | GitHub Actions, GitLab CI |
| T3.12 | Final testing | All edge cases covered |

## ECF Target Structure
```xml
<!-- Library target (reusable) -->
<target name="api_contract_sentinel">
    <root class="CONTRACT_SENTINEL_CLI" feature="make"/>
    <library name="simple_validation" location="..."/>
    <library name="simple_json" location="..."/>
    <library name="simple_yaml" location="..."/>
    <library name="simple_cli" location="..."/>
    <library name="simple_file" location="..."/>
    <library name="simple_http" location="..."/>
    <cluster name="src" location=".\src\"/>
</target>

<!-- Test target -->
<target name="api_contract_sentinel_tests" extends="api_contract_sentinel">
    <root class="TEST_APP" feature="make"/>
    <library name="simple_testing" location="..."/>
    <cluster name="tests" location=".\tests\"/>
</target>
```

## Build Commands
```bash
# Compile CLI (workbench for development)
/d/prod/ec.sh -batch -config api_contract_sentinel.ecf -target api_contract_sentinel -c_compile

# Run tests
/d/prod/ec.sh -batch -config api_contract_sentinel.ecf -target api_contract_sentinel_tests -c_compile
./EIFGENs/api_contract_sentinel_tests/W_code/api_contract_sentinel.exe

# Compile finalized (production)
/d/prod/ec.sh -batch -config api_contract_sentinel.ecf -target api_contract_sentinel -finalize -c_compile
```

## Success Criteria

| Criterion | Measure | Target |
|-----------|---------|--------|
| Compiles | Zero errors | 100% |
| Tests pass | All tests | 100% |
| MVP commands | validate works | Functional |
| All commands | All 5 commands work | Functional |
| OpenAPI 3.0 | Full support | Yes |
| OpenAPI 3.1 | Full support | Yes |
| Performance | 100 path spec | <5 seconds |
| Documentation | README complete | Yes |

## File Structure
```
api_contract_sentinel/
    api_contract_sentinel.ecf
    README.md
    CHANGELOG.md
    src/
        contract_sentinel_cli.e
        contract_sentinel_engine.e
        openapi_parser.e
        openapi_spec.e
        openapi_schema.e
        openapi_path.e
        openapi_operation.e
        schema_compiler.e
        payload_validator.e
        mock_generator.e
        sentinel_result.e
        sentinel_violation.e
        sentinel_reporter.e
    tests/
        test_app.e
        test_openapi_parser.e
        test_schema_compiler.e
        test_payload_validator.e
        test_mock_generator.e
    examples/
        petstore.yaml
        user-valid.json
        user-invalid.json
```
