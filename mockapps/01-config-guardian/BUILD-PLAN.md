# ConfigGuardian - Build Plan

## Phase Overview

| Phase | Deliverable | Effort | Dependencies |
|-------|-------------|--------|--------------|
| Phase 1 | MVP CLI | 5 days | simple_validation, simple_yaml, simple_cli |
| Phase 2 | Multi-format + Rules | 4 days | Phase 1, simple_json, simple_toml |
| Phase 3 | Polish + CI/CD | 3 days | Phase 2, simple_template |

## Phase 1: MVP

### Objective

Demonstrate core value: validate a YAML config file against a rule file and output errors. This proves the concept works and provides immediate value for basic use cases.

### Deliverables

1. **CONFIG_GUARDIAN_CLI** - Basic command-line entry point with validate command
2. **CONFIG_GUARDIAN_ENGINE** - Core orchestration: load config, load rules, validate
3. **RULE_COMPILER** - Parse YAML rule definitions into GUARDIAN_RULE objects
4. **GUARDIAN_RULE** - Data class for single validation rule
5. **GUARDIAN_RESULT** - Validation result with errors list
6. **GUARDIAN_REPORTER** - Text output format

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T1.1 | Create ECF with dependencies | Compiles with simple_validation, simple_yaml, simple_cli |
| T1.2 | Implement GUARDIAN_RULE class | Holds path, rule_type, constraints, message |
| T1.3 | Implement GUARDIAN_RESULT class | Tracks validity, errors list, error_count |
| T1.4 | Implement RULE_COMPILER | Parses YAML rules into GUARDIAN_RULE list |
| T1.5 | Implement CONFIG_GUARDIAN_ENGINE | Loads YAML config, applies rules, returns result |
| T1.6 | Implement GUARDIAN_REPORTER.to_text | Formats errors as readable text |
| T1.7 | Implement CONFIG_GUARDIAN_CLI | Parses args, calls engine, outputs report |
| T1.8 | Write MVP tests | All basic scenarios pass |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Valid config | server.yaml with port=8080, rules require port 1024-65535 | Exit 0, "Valid" |
| Missing required | server.yaml without host, rules require host | Exit 1, "host: This field is required" |
| Invalid format | server.yaml with port="abc", rules require integer | Exit 1, "port: Must be a valid integer" |
| Out of range | server.yaml with port=80, rules require 1024+ | Exit 1, "port: Must be at least 1024" |
| Rule syntax error | malformed rules.yaml | Exit 4, "Invalid rule syntax" |
| File not found | nonexistent.yaml | Exit 2, "Config file not found" |

### MVP Rule Types

| Rule Type | Validation |
|-----------|------------|
| required | Field must exist and be non-empty |
| type | Field must be: string, integer, number, boolean |
| min/max | Numeric range constraints |
| min_length/max_length | String length constraints |
| pattern | Regex pattern match |
| one_of | Value in allowed list |

## Phase 2: Multi-Format + Advanced Rules

### Objective

Support JSON and TOML configs. Add conditional rules, forbidden fields, and field groups. Generate JSON and JUnit output for CI/CD integration.

### Deliverables

1. **CONFIG_PARSER** - Multi-format parsing (YAML, JSON, TOML)
2. **Extended RULE_COMPILER** - Conditional rules, forbidden, groups
3. **GUARDIAN_REPORTER.to_json** - JSON output format
4. **GUARDIAN_REPORTER.to_junit** - JUnit XML for CI/CD
5. **Built-in rule sets** - Kubernetes, Docker Compose templates

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T2.1 | Add JSON parsing to CONFIG_PARSER | JSON files validate correctly |
| T2.2 | Add TOML parsing to CONFIG_PARSER | TOML files validate correctly |
| T2.3 | Implement auto-format detection | Correct format detected from extension |
| T2.4 | Implement conditional rules (when) | Rules apply only when condition met |
| T2.5 | Implement forbidden rules | Fields that must NOT exist |
| T2.6 | Implement field groups | Related fields validated together |
| T2.7 | Implement JSON reporter | Valid JSON output |
| T2.8 | Implement JUnit reporter | CI/CD compatible XML |
| T2.9 | Create kubernetes-base ruleset | Common K8s manifest rules |
| T2.10 | Create docker-compose ruleset | Common Docker Compose rules |
| T2.11 | Implement rule inheritance (extends) | Rulesets can extend others |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| JSON config | config.json, same rules | Same validation as YAML |
| TOML config | config.toml, same rules | Same validation as YAML |
| Format detection | config.yaml without --type | Correctly detected as YAML |
| Conditional rule | env=production, rule when env=production | Rule applied |
| Conditional skip | env=dev, rule when env=production | Rule skipped |
| Forbidden field | password in config, forbidden rule | Exit 1, "password must not be in config" |
| Field group | partial database config | "All database fields required" |
| JSON output | --format json | Valid JSON, same data |
| JUnit output | --format junit | Valid JUnit XML |

## Phase 3: Production Polish

### Objective

Production-ready with help documentation, performance optimization, and integration guides. Add markdown reports and verbose diagnostics.

### Deliverables

1. **Help system** - Detailed help for all commands
2. **init command** - Generate starter rule files
3. **list-rules command** - Show built-in rule types
4. **check-rules command** - Validate rule syntax
5. **GUARDIAN_REPORTER.to_markdown** - Markdown reports
6. **Performance optimization** - Large file handling
7. **Documentation** - README, integration guides

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T3.1 | Implement init command | Generates working starter rules |
| T3.2 | Implement list-rules command | Shows all rule types with examples |
| T3.3 | Implement check-rules command | Validates rule file before use |
| T3.4 | Add --verbose mode | Debug output for troubleshooting |
| T3.5 | Implement markdown reporter | Clean markdown output |
| T3.6 | Optimize large file handling | <5s for 10K line configs |
| T3.7 | Add glob pattern support | Validate multiple files |
| T3.8 | Write README.md | Installation, usage, examples |
| T3.9 | Write CI/CD guide | GitHub Actions, GitLab CI examples |
| T3.10 | Final testing | All edge cases covered |

## ECF Target Structure
```xml
<!-- Library target (reusable) -->
<target name="config_guardian">
    <root class="CONFIG_GUARDIAN_CLI" feature="make"/>
    <library name="simple_validation" location="..."/>
    <library name="simple_yaml" location="..."/>
    <library name="simple_json" location="..."/>
    <library name="simple_cli" location="..."/>
    <library name="simple_file" location="..."/>
    <cluster name="src" location=".\src\"/>
</target>

<!-- Test target -->
<target name="config_guardian_tests" extends="config_guardian">
    <root class="TEST_APP" feature="make"/>
    <library name="simple_testing" location="..."/>
    <cluster name="tests" location=".\tests\"/>
</target>
```

## Build Commands
```bash
# Compile CLI (workbench for development)
/d/prod/ec.sh -batch -config config_guardian.ecf -target config_guardian -c_compile

# Run tests
/d/prod/ec.sh -batch -config config_guardian.ecf -target config_guardian_tests -c_compile
./EIFGENs/config_guardian_tests/W_code/config_guardian.exe

# Compile finalized (production)
/d/prod/ec.sh -batch -config config_guardian.ecf -target config_guardian -finalize -c_compile
```

## Success Criteria

| Criterion | Measure | Target |
|-----------|---------|--------|
| Compiles | Zero errors | 100% |
| Tests pass | All tests | 100% |
| MVP commands | validate works | Functional |
| All commands | All 4 commands work | Functional |
| Performance | 10K line config | <5 seconds |
| Documentation | README complete | Yes |

## File Structure
```
config_guardian/
    config_guardian.ecf
    README.md
    CHANGELOG.md
    src/
        config_guardian_cli.e
        config_guardian_engine.e
        rule_compiler.e
        config_parser.e
        guardian_rule.e
        guardian_ruleset.e
        guardian_result.e
        guardian_reporter.e
    tests/
        test_app.e
        test_rule_compiler.e
        test_engine.e
        test_reporter.e
    rulesets/
        kubernetes-base.yaml
        docker-compose.yaml
        github-actions.yaml
    examples/
        server-config.yaml
        .config-guardian.yaml
```
