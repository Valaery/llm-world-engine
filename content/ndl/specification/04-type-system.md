# NDL Type System

#ndl #specification #types

[[00-NDL-INDEX|← Back to Index]]

## Overview

While NDL is not statically typed in the traditional programming sense, it has implicit type expectations for different contexts. This document defines the type categories and validation rules for NDL constructs.

## Type Categories

### Action Type

**Domain**: Verb identifiers representing actions

**Syntax**: `<identifier>`

**Validation**:
- Must be lowercase
- Alphanumeric + underscore
- Should represent valid game action

**Examples**:
```ndl
do(walk)
do(attack)
do(speak)
do(look_away)
do(open_door)
```

**Invalid**:
```ndl
do(Walk)          # Wrong case
do(do-something)  # Invalid character (-)
do(123action)     # Starts with number
```

### Manner Type

**Domain**: Adverbs or adverbial phrases

**Syntax**: `<identifier>` or `<identifier>_<identifier>...`

**Validation**:
- Describes how action is performed
- Usually adverbs or adverbial phrases
- Snake_case for multi-word

**Examples**:
```ndl
~ slowly
~ carefully
~ with_determination
~ in_a_panic
~ menacingly
```

**Semantic Constraints**:
- Should be applicable to the action
- `do(think) ~ loudly` is semantically odd but syntactically valid

### Duration Type

**Domain**: Time measurements

**Syntax**: `<number><unit>`

**Units**:
- `s` - seconds
- `ms` - milliseconds

**Validation**:
- Number must be positive
- Number can be integer or decimal
- Unit required

**Examples**:
```ndl
wait(1s)
wait(500ms)
wait(2.5s)
wait(0.1s)
```

**Invalid**:
```ndl
wait(-1s)      # Negative duration
wait(1)        # Missing unit
wait(s)        # Missing number
wait(1 s)      # Space not allowed
```

### String Type

**Domain**: Arbitrary text content

**Syntax**: `"<characters>"`

**Validation**:
- Enclosed in double quotes
- Escape sequences for special characters
- Can be empty

**Examples**:
```ndl
text="Hello there"
intention="intimidate"
location="tavern"
text=""
```

**Escape Sequences** (if supported):
```ndl
text="He said \"hello\""
text="Line one\nLine two"
text="Path: C:\\Users"
```

### Number Type

**Domain**: Numeric values

**Syntax**: `<digits>` or `<digits>.<digits>`

**Validation**:
- Integer or floating-point
- No scientific notation (typically)
- Context-dependent interpretation

**Examples**:
```ndl
damage=15
health=87.5
distance=100
chance=0.75
```

**Invalid**:
```ndl
value=1e10       # Scientific notation (not standard)
value=1,000      # Comma separator (not allowed)
```

### Identifier Type

**Domain**: Property names, bare values

**Syntax**: `<letter>(<letter>|<digit>|_)*`

**Validation**:
- Starts with letter
- Alphanumeric + underscore
- Case-sensitive

**Examples**:
```ndl
intention="value"
target="goblin"
my_custom_property="data"
```

## Property Types

Common properties and their expected types:

| Property | Type | Description | Example |
|----------|------|-------------|---------|
| `text` | String | Literal text content | `text="Hello"` |
| `intention` | String | Purpose/intent | `intention="intimidate"` |
| `emotion` | String | Emotional state | `emotion="fear"` |
| `target` | String | Action object | `target="enemy"` |
| `location` | String | Place name | `location="tavern"` |
| `damage` | Number | Numeric damage | `damage=15` |
| `health` | Number | Health value | `health=87.5` |
| `weapon` | String | Weapon identifier | `weapon="sword"` |
| `duration` | Duration | Time span | Built into wait() |

## Type Validation

### Compile-Time Validation
When generating NDL:
```python
# Valid
generate_ndl(action="walk", manner="slowly")

# Type error (should catch)
generate_ndl(action=123)  # Action must be string
generate_ndl(duration=-1)  # Duration must be positive
```

### Runtime Validation
When parsing NDL:
```python
parse("do(walk) ~ slowly")  # Valid
parse("do(walk) ~ 123")      # Invalid: manner not identifier
parse("wait(-1s)")           # Invalid: negative duration
```

### LLM Validation
The LLM should not validate NDL - it should trust it as authoritative.

## Type Coercion

NDL typically does not perform implicit type coercion:

```ndl
# NO implicit coercion
damage="15"    # String, not number
damage=15      # Number

# Explicit in generation code
damage=${damageValue}   # Use string interpolation
```

## Custom Types

Implementations may extend with custom types:

### Enum Types
```ndl
# If implementation defines emotion enum
emotion="fear"    # Valid
emotion="hungry"  # Valid if in enum
emotion="xyzzy"   # Invalid if not in enum
```

### Structured Types
```ndl
# Future extension possibility
equipment={weapon: "sword", armor: "plate"}
position=[10, 20, 30]
```

## Type Safety in Generation

### Recommended Approach
Use typed data structures when generating NDL:

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class NDLAction:
    verb: str
    manner: Optional[str] = None
    intention: Optional[str] = None
    text: Optional[str] = None

    def to_ndl(self) -> str:
        parts = [f"do({self.verb})"]
        if self.manner:
            parts.append(f"~ {self.manner}")
        if self.intention:
            parts.append(f'intention="{self.intention}"')
        if self.text:
            parts.append(f'text="{self.text}"')
        return " ".join(parts)

# Usage
action = NDLAction(
    verb="speak",
    manner="nervously",
    text="I didn't mean to"
)
ndl = action.to_ndl()
# Result: do(speak) ~ nervously text="I didn't mean to"
```

## Type Compatibility

### Compatible Combinations
```ndl
# String property values
do(speak) text="hello"           ✓

# Number property values
do(attack) damage=15              ✓

# Mixed types
do(attack) ~ fiercely damage=15   ✓
```

### Incompatible Combinations
```ndl
# Manner with wrong type
do(walk) ~ 123                    ✗

# Duration without unit
wait(5)                           ✗

# Property without value
do(walk) intention=               ✗
```

## Semantic Type Checking

Beyond syntax, semantic validation:

### Action-Manner Compatibility
```ndl
do(walk) ~ slowly      # Sensible
do(think) ~ loudly     # Odd but syntactically valid
do(sleep) ~ quickly    # Semantically questionable
```

Recommendation: Warn but allow semantic oddities.

### Property Relevance
```ndl
do(walk) damage=15     # Walk with damage? Unusual
do(attack) damage=15   # Attack with damage - sensible
```

Recommendation: Game logic determines relevance.

## Type Inference

NDL does not require type declarations:

```ndl
# Type inferred from context
intention="intimidate"  # String (quotes)
damage=15               # Number (no quotes)
wait(1s)                # Duration (unit suffix)
```

## Null and Optional Values

### Missing Properties
Properties are optional:
```ndl
do(walk)                # No properties - valid
do(walk) ~ slowly       # Optional manner - valid
```

### Empty Values
```ndl
text=""                 # Empty string - valid
intention=""            # Empty intention - questionable
```

Recommendation: Omit property rather than empty string.

## Type Extensions

### Adding New Types
Implementations can extend:

```ndl
# Hypothetical color type
do(glow) color=#FF0000

# Hypothetical coordinate type
do(move) position=(10,20)

# Hypothetical list type
do(affect) targets=["goblin1", "goblin2"]
```

### Versioning Considerations
- Keep core types stable
- Document extensions clearly
- Namespace custom types if needed

## Type Documentation

When defining custom properties, document expected types:

```yaml
# properties.yaml
properties:
  text:
    type: string
    description: "Literal text to include in narration"
    required: false

  damage:
    type: number
    description: "Numeric damage value"
    required: false
    minimum: 0

  intention:
    type: string
    description: "Purpose behind the action"
    required: false
    enum: ["intimidate", "persuade", "deceive", "help"]
```

## Error Messages

Good type error messages:

```
Error: Invalid manner type
  at: do(walk) ~ 123
           ^^^
  Expected: identifier or snake_case phrase
  Got: number literal

Error: Missing duration unit
  at: wait(5)
           ^
  Expected: duration with unit (e.g., "5s" or "500ms")
  Got: number without unit

Error: Negative duration
  at: wait(-1s)
           ^^^
  Duration must be positive
```

## Design Rationale

### Why No Static Types?
- Simplicity: NDL is generated, not hand-written
- Flexibility: Easy to extend
- Context: Game logic already knows types

### Why Some Type Structure?
- Validation: Catch generation errors
- Clarity: Clear expectations
- Parsing: Unambiguous interpretation

### Why String-Based?
- Human-readable
- LLM-friendly
- Easy to embed in prompts

## Related

- [[01-lexical-elements]] - Token definitions
- [[02-grammar]] - Syntactic rules
- [[03-semantics]] - Meaning of constructs
- [[game-state-to-ndl]] - Type-safe generation

---

*Synthesized from Discord discussions by [[User-veritasr]] and community members.*
