<p align="center">
  <img src="https://raw.githubusercontent.com/simple-eiffel/.github/main/profile/assets/logo.svg" alt="simple_ library logo" width="400">
</p>

# simple_validation

**[Documentation](https://simple-eiffel.github.io/simple_validation/)**

Fluent data validation library for Eiffel with chainable rules inspired by HTML5 Constraint Validation API and JSON Schema.

## Features

- **Fluent API** - Chain validation rules naturally
- **HTML5 Constraints** - Required, min/max length, pattern matching
- **Numeric Validation** - Min/max values, integer checks
- **Format Validation** - Email, URL, alpha, alphanumeric
- **Custom Rules** - Add your own validation logic
- **Rich Error Messages** - Detailed error information with field names
- **Design by Contract** - Full preconditions/postconditions

## Installation

Set the ecosystem environment variable (one-time setup for all simple_* libraries):
```
SIMPLE_EIFFEL=D:\prod
```

Add to your ECF:

```xml
<library name="simple_validation" location="$SIMPLE_EIFFEL/simple_validation/simple_validation.ecf"/>
```

## Quick Start (Zero-Configuration)

Use `SIMPLE_VALIDATION_QUICK` for the simplest possible validation:

```eiffel
local
    v: SIMPLE_VALIDATION_QUICK
do
    create v.make

    -- One-liner validators that return BOOLEAN
    if v.email ("user@example.com") then ...
    if v.url ("https://example.com") then ...
    if v.required (input) then ...
    if v.not_blank (input) then ...

    -- Length validation
    if v.min_length (input, 3) then ...
    if v.max_length (input, 20) then ...
    if v.length_ok (input, 3, 20) then ...

    -- Format validators
    if v.alpha (input) then ...           -- letters only
    if v.alphanumeric (input) then ...    -- letters and digits
    if v.numeric (input) then ...         -- digits only
    if v.is_integer (input) then ...
    if v.is_number (input) then ...

    -- Numeric range
    if v.in_range (age, 18, 65) then ...
    if v.positive (count) then ...

    -- Comparison
    if v.equals (password, confirm_password) then ...
    if v.one_of (status, <<"active", "pending", "closed">>) then ...

    -- Error message from last failed validation
    if not v.email (input) then
        print ("Error: " + v.last_error)
    end
end
```

## Standard API (Full Control)

### Basic Validation

```eiffel
local
    validator: SIMPLE_VALIDATOR
    result: VALIDATION_RESULT
do
    create validator.make

    result := validator.required.email.for_field ("email").validate ("user@example.com")

    if result.is_valid then
        io.put_string ("Valid!%N")
    else
        io.put_string (result.to_string)
    end
end
```

### Chaining Rules

```eiffel
local
    validator: SIMPLE_VALIDATOR
    result: VALIDATION_RESULT
do
    create validator.make

    -- Username: 3-20 chars, alphanumeric
    result := validator.required
        .length_between (3, 20)
        .alpha_numeric
        .for_field ("username")
        .validate ("john_doe123")
end
```

### Custom Messages

```eiffel
validator.required.with_message ("This field cannot be empty")
```

### Custom Rules

```eiffel
validator.custom (agent my_validation_function)

my_validation_function (value: STRING): BOOLEAN
    do
        Result := value.count > 3 and value.starts_with ("user_")
    end
```

## API Reference

### String Validation

| Rule | Description |
|------|-------------|
| `required` | Value must not be empty |
| `min_length (n)` | Minimum character count |
| `max_length (n)` | Maximum character count |
| `length_between (min, max)` | Length within range |
| `pattern (regex)` | Match regular expression |

### Format Validation

| Rule | Description |
|------|-------------|
| `email` | Valid email format |
| `url` | Valid URL format |
| `alpha` | Letters only (A-Za-z) |
| `alpha_numeric` | Letters and digits only |

### Numeric Validation

| Rule | Description |
|------|-------------|
| `min_value (n)` | Minimum numeric value |
| `max_value (n)` | Maximum numeric value |
| `value_between (min, max)` | Value within range |
| `is_integer` | Must be a whole number |

## Dependencies

- EiffelBase
- Gobo Regexp (for pattern matching)

## License

MIT License - Copyright (c) 2024-2025, Larry Rix
