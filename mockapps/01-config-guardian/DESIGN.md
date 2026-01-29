# ConfigGuardian - Technical Design

## Architecture

### Component Overview
```
+-------------------------------------------------------------+
|                      ConfigGuardian                          |
+-------------------------------------------------------------+
|  CLI Interface Layer                                         |
|    - Argument parsing (simple_cli)                          |
|    - Command routing (validate, init, list-rules)           |
|    - Output formatting (text, json, junit)                  |
+-------------------------------------------------------------+
|  Business Logic Layer                                        |
|    - CONFIG_GUARDIAN_ENGINE: Orchestration                  |
|    - RULE_COMPILER: Parse rule definitions                  |
|    - CONFIG_PARSER: Multi-format config loading             |
|    - VALIDATION_EXECUTOR: Apply rules to config             |
+-------------------------------------------------------------+
|  Integration Layer                                           |
|    - simple_validation: Core validation engine              |
|    - simple_yaml: YAML parsing                              |
|    - simple_json: JSON parsing                              |
|    - simple_toml: TOML parsing                              |
|    - simple_file: File operations                           |
+-------------------------------------------------------------+
```

### Class Design

| Class | Responsibility | Key Features |
|-------|----------------|--------------|
| CONFIG_GUARDIAN_CLI | Command-line interface | parse_args, execute, format_output |
| CONFIG_GUARDIAN_ENGINE | Core orchestration | load_config, load_rules, validate, report |
| RULE_COMPILER | Parse rule definitions | compile_ruleset, validate_rule_syntax |
| CONFIG_PARSER | Multi-format parsing | parse_yaml, parse_json, parse_toml, parse_ini |
| VALIDATION_EXECUTOR | Rule execution | apply_rules, collect_errors, generate_report |
| GUARDIAN_RULE | Single validation rule | field_path, rule_type, constraint, message |
| GUARDIAN_RULESET | Collection of rules | rules, metadata, inheritance |
| GUARDIAN_RESULT | Validation outcome | is_valid, errors, warnings, stats |
| GUARDIAN_REPORTER | Output generation | to_text, to_json, to_junit, to_markdown |

### Command Structure
```bash
config-guardian <command> [options] [arguments]

Commands:
  validate <file>     Validate config file against rules
  init                Initialize rule file in current directory
  list-rules          List available built-in rule types
  check-rules         Validate rule file syntax
  version             Show version information

Global Options:
  --rules FILE        Rule file (default: .config-guardian.yaml)
  --format FORMAT     Output format: text, json, junit, markdown
  --strict            Treat warnings as errors
  --quiet             Only output errors
  --no-color          Disable colored output
  --verbose           Verbose output with debug info
  --help              Show help

Validate Options:
  --type TYPE         Force config type: yaml, json, toml, ini
  --ignore PATTERN    Ignore fields matching pattern
  --only PATTERN      Only validate fields matching pattern

Examples:
  config-guardian validate config.yaml
  config-guardian validate deploy/*.yaml --rules ci-rules.yaml
  config-guardian validate config.json --format junit > results.xml
  config-guardian init --type kubernetes
```

### Data Flow
```
Config File  -->  CONFIG_PARSER  -->  Parsed Config (nested structure)
                       |
Rule File    -->  RULE_COMPILER  -->  Compiled Ruleset
                       |
                       v
            VALIDATION_EXECUTOR  -->  Apply rules to config paths
                       |
                       v
              GUARDIAN_RESULT    -->  Errors, warnings, stats
                       |
                       v
            GUARDIAN_REPORTER    -->  Formatted output
```

### Rule Definition Schema
```yaml
# .config-guardian.yaml
version: "1.0"
name: "Production Config Rules"
description: "Validation rules for production deployments"

# Inherit from built-in rulesets
extends:
  - kubernetes-base
  - security-best-practices

# Custom rules
rules:
  - path: "server.port"
    required: true
    type: integer
    min: 1024
    max: 65535
    message: "Server port must be between 1024 and 65535"

  - path: "database.host"
    required: true
    pattern: "^[a-z0-9.-]+$"
    message: "Database host must be valid hostname"

  - path: "database.password"
    forbidden: true
    message: "Passwords must not be in config files"

  - path: "replicas"
    required: true
    type: integer
    min: 1
    when:
      path: "environment"
      equals: "production"
    message: "Production must have at least 1 replica"

  - path: "logging.level"
    required: true
    one_of: ["debug", "info", "warn", "error"]
    message: "Logging level must be valid"

# Field groups for related validation
groups:
  database:
    fields: ["database.host", "database.port", "database.name"]
    all_required: true
    message: "All database fields required if any specified"
```

### Configuration Schema
```json
{
  "config_guardian": {
    "default_format": "text",
    "color": true,
    "strict": false,
    "rule_paths": [
      ".config-guardian.yaml",
      "~/.config/config-guardian/rules/"
    ],
    "ignore_patterns": [
      "*.example.*",
      "test/*"
    ],
    "built_in_rules": {
      "kubernetes": true,
      "docker": true,
      "github_actions": true
    }
  }
}
```

### Error Handling

| Error Type | Handling | User Message |
|------------|----------|--------------|
| File not found | Exit 2 | "Config file not found: {path}" |
| Parse error | Exit 3 | "Failed to parse {format}: {details}" |
| Rule syntax error | Exit 4 | "Invalid rule at line {n}: {details}" |
| Validation failure | Exit 1 | Detailed error report |
| Unknown format | Exit 5 | "Cannot detect format, use --type" |

### Exit Codes
| Code | Meaning |
|------|---------|
| 0 | Validation passed |
| 1 | Validation failed (errors found) |
| 2 | File not found |
| 3 | Parse error (invalid config syntax) |
| 4 | Rule error (invalid rule syntax) |
| 5 | Unknown format |
| 255 | Internal error |

## GUI/TUI Future Path

**CLI foundation enables:**
- All validation logic in reusable classes (CONFIG_GUARDIAN_ENGINE)
- Format-agnostic result objects (GUARDIAN_RESULT)
- Multiple output formats already supported

**What would change for TUI:**
- Add simple_tui for interactive rule browser
- Real-time validation as user edits rules
- Visual diff of config changes

**What would change for GUI:**
- Web dashboard for rule management
- Visual rule builder (drag-drop)
- Historical validation reports
- Team collaboration features

**Shared components between CLI/GUI:**
- CONFIG_GUARDIAN_ENGINE (100% reusable)
- RULE_COMPILER (100% reusable)
- GUARDIAN_RESULT (100% reusable)
- GUARDIAN_REPORTER (extend for HTML output)
