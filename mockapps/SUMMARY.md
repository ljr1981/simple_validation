# Mock Apps Summary: simple_validation

## Generated: 2026-01-24

## Library Analyzed

- **Library:** simple_validation
- **Core capability:** Fluent data validation with chainable rules
- **Ecosystem position:** Data quality and input validation foundation

## Mock Apps Designed

### 1. ConfigGuardian

- **Purpose:** Validates configuration files (YAML, JSON, TOML, INI) against custom rule sets before deployment
- **Target:** DevOps teams, platform engineers, SRE teams managing infrastructure-as-code
- **Revenue:** Free tier + Pro $49/mo + Enterprise $199/mo
- **Ecosystem:** simple_validation, simple_yaml, simple_json, simple_toml, simple_cli, simple_file
- **Status:** Design complete
- **Build effort:** 12 days (3 phases)

### 2. APIContractSentinel

- **Purpose:** Validates API request/response payloads against OpenAPI schemas with rich error reporting
- **Target:** API developers, integration teams, QA engineers testing microservices
- **Revenue:** Free tier + Team $99/mo + Enterprise $399/mo
- **Ecosystem:** simple_validation, simple_json, simple_yaml, simple_http, simple_cli, simple_file, simple_template
- **Status:** Design complete
- **Build effort:** 15 days (3 phases)

### 3. CSVIntakeValidator

- **Purpose:** Batch validates CSV data imports against schema rules with detailed error reports
- **Target:** Data engineers, analysts, integration specialists handling data migrations
- **Revenue:** Free tier + Pro $79/mo + Enterprise $299/mo
- **Ecosystem:** simple_validation, simple_csv, simple_json, simple_cli, simple_file, simple_template
- **Status:** Design complete
- **Build effort:** 12 days (3 phases)

## Ecosystem Coverage

| simple_* Library | Used In |
|------------------|---------|
| simple_validation | ConfigGuardian, APIContractSentinel, CSVIntakeValidator |
| simple_cli | ConfigGuardian, APIContractSentinel, CSVIntakeValidator |
| simple_file | ConfigGuardian, APIContractSentinel, CSVIntakeValidator |
| simple_json | ConfigGuardian, APIContractSentinel, CSVIntakeValidator |
| simple_yaml | ConfigGuardian, APIContractSentinel |
| simple_csv | CSVIntakeValidator |
| simple_http | APIContractSentinel |
| simple_template | APIContractSentinel, CSVIntakeValidator |
| simple_toml | ConfigGuardian |
| simple_uuid | APIContractSentinel |
| simple_datetime | CSVIntakeValidator |
| simple_logger | ConfigGuardian, APIContractSentinel, CSVIntakeValidator |
| simple_env | ConfigGuardian |

**Total unique libraries leveraged:** 13

## Market Research Summary

### Validated Demand

| Market | Evidence | Opportunity |
|--------|----------|-------------|
| Config Validation | Boeing's config-file-validator, Stelligent config-lint | Business-tier features, richer rules |
| API Contract Testing | PactFlow ($$$), Swagger Hub | Lightweight CLI alternative |
| CSV Data Quality | Osmos, CSVLinter, Datograde | Developer-focused CLI tool |

### Commercial Competitors Analyzed

| Competitor | Category | Gap We Fill |
|------------|----------|-------------|
| Great Expectations | Data Quality | Simpler CLI, Eiffel DBC |
| PactFlow | API Contracts | Lightweight, no setup |
| Boeing config-file-validator | Config Validation | Business rules, audit |
| Informatica DQ | Enterprise DQ | Affordable, focused |
| ZeroBounce | Email Validation | Offline, batch |
| Osmos | CSV Transformation | Developer CLI |

### Target Personas

| Persona | Apps That Serve Them |
|---------|---------------------|
| DevOps Engineer | ConfigGuardian |
| Data Engineer | CSVIntakeValidator |
| API Developer | APIContractSentinel |
| QA Engineer | APIContractSentinel, CSVIntakeValidator |
| Integration Specialist | CSVIntakeValidator |

## Total Build Effort

| Phase | ConfigGuardian | APIContractSentinel | CSVIntakeValidator | Total |
|-------|----------------|---------------------|--------------------|-------|
| Phase 1 (MVP) | 5 days | 6 days | 5 days | 16 days |
| Phase 2 (Features) | 4 days | 5 days | 4 days | 13 days |
| Phase 3 (Polish) | 3 days | 4 days | 3 days | 10 days |
| **Total** | **12 days** | **15 days** | **12 days** | **39 days** |

## Next Steps

1. **Select Mock App for implementation**
   - ConfigGuardian: Best for DevOps market penetration
   - APIContractSentinel: Best for API-first developer market
   - CSVIntakeValidator: Best for data engineering market

2. **Add app target to simple_validation.ecf** (or create standalone project)

3. **Implement Phase 1 (MVP)** following BUILD-PLAN.md

4. **Run /eiffel.verify** for contract validation

5. **Iterate through phases** based on user feedback

## Files Generated

```
mockapps/
    00-MARKETPLACE-RESEARCH.md
    01-config-guardian/
        CONCEPT.md
        DESIGN.md
        ECOSYSTEM-MAP.md
        BUILD-PLAN.md
    02-api-contract-sentinel/
        CONCEPT.md
        DESIGN.md
        ECOSYSTEM-MAP.md
        BUILD-PLAN.md
    03-csv-intake-validator/
        CONCEPT.md
        DESIGN.md
        ECOSYSTEM-MAP.md
        BUILD-PLAN.md
    SUMMARY.md
```

## Recommendation

**Start with CSVIntakeValidator** for these reasons:

1. **Universal pain point** - Every organization imports CSV data
2. **Clear ROI** - Gartner says poor data quality costs $15M/year average
3. **Simpler MVP** - Fewer dependencies than APIContractSentinel
4. **Ecosystem showcase** - Uses 6+ simple_* libraries
5. **CLI-first natural** - Batch processing is inherently CLI-oriented

However, if targeting DevOps market specifically, ConfigGuardian is the better choice due to its CI/CD integration value.

---

*Generated by /eiffel.mockapp for simple_validation*
