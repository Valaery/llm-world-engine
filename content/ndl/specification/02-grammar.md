---
tags: [ndl, specification, grammar, syntax]
date: 2026-01-16
source: Discord - LLM World Engine Channel
status: complete
---

# NDL Grammar Specification

## Overview

This document provides a formal grammar specification for NDL (Natural Description Language) in Extended Backus-Naur Form (EBNF).

## Notation

- `::=` - Definition
- `|` - Alternative
- `()` - Grouping
- `[]` - Optional (zero or one)
- `{}` - Repetition (zero or more)
- `""` - Literal terminal
- `<>` - Non-terminal

## Complete Grammar

```ebnf
<ndl_statement> ::= <action_chain>

<action_chain> ::= <action_element> { "->" <action_element> }

<action_element> ::= <action> [ <modifiers> ]
                   | <control_statement>
                   | <meta_statement>

<action> ::= "do" "(" <entity_ref> "," <string> ")"
           | "search" "(" <string> [ "," <properties> ] ")"
           | "talk" "(" <string> "," <entity_list> ")"
           | "convey" "(" <string> [ "," <properties> ] ")"
           | "target" "(" <entity_ref> ")"
           | "result" "(" [ <string> ] ")"
           | "describe" "(" ")"
           | "roll" "(" <string> ")"
           | "damage" "(" <integer> ")"

<control_statement> ::= "wait" "(" <string> ")"

<meta_statement> ::= "system_response" "(" <string> ")"

<modifiers> ::= <manner> [ <properties> ]
              | <properties> [ <manner> ]

<manner> ::= "~" <string>

<properties> ::= <property> { "," <property> }

<property> ::= <identifier> "=" <string>

<entity_ref> ::= "$" <string>
               | "$"

<entity_list> ::= <string> { "^" <string> }

<identifier> ::= "intention" | "perspective" | <custom_identifier>

<custom_identifier> ::= [a-z_][a-z0-9_]*

<string> ::= '"' <any_char>* '"'

<integer> ::= [0-9]+

<any_char> ::= /* any character except unescaped " */
```

## Production Rules Explained

### Statement Level

#### `<ndl_statement>`
Top-level construct representing a complete NDL expression.

**Examples**:
```ndl
do($"player", "attack")
do($"player", "attack") -> result("success")
```

#### `<action_chain>`
Sequence of action elements connected by `->` operator.

**Examples**:
```ndl
action1
action1 -> action2
action1 -> action2 -> action3
```

### Action Level

#### `<action_element>`
Individual component in an action chain.

**Types**:
1. Regular actions (`do`, `search`, `talk`, etc.)
2. Control statements (`wait`)
3. Meta statements (`system_response`)

#### `<action>` - Primary Actions

**do() Action**:
```ebnf
"do" "(" <entity_ref> "," <string> ")"
```
**Example**: `do($"player", "attack with sword")`

**search() Action**:
```ebnf
"search" "(" <string> [ "," <properties> ] ")"
```
**Example**: `search("pile of wood", intention="find weapon")`

**talk() Action**:
```ebnf
"talk" "(" <string> "," <entity_list> ")"
```
**Example**: `talk("Mark", "John"^"Sue")`

**convey() Action**:
```ebnf
"convey" "(" <string> [ "," <properties> ] ")"
```
**Example**: `convey("desire to explore", perspective="adventurous")`

### Modifiers

#### `<manner>` - How Modifier
Specifies how an action is performed.

**Syntax**:
```ebnf
"~" <string>
```

**Example**: `do($"player", "walk") ~ "carefully"`

#### `<properties>` - Key-Value Attributes
Additional context for actions.

**Syntax**:
```ebnf
<identifier> "=" <string>
```

**Examples**:
- `intention="intimidate"`
- `perspective="hopeful"`

### Entity References

#### `<entity_ref>` - Entity Specification
References to game entities.

**Forms**:
1. Named entity: `$"entity_name"`
2. Implicit entity: `$`

**Examples**:
```ndl
$"player"
$"goblin"
$"guard"
$
```

#### `<entity_list>` - Multiple Entities
Multiple entities connected with `^` (conjunction).

**Syntax**:
```ebnf
<string> { "^" <string> }
```

**Example**: `"John"^"Sue"^"Bob"`

### Control Flow

#### `<control_statement>` - Wait
Introduces pause or delay.

**Syntax**:
```ebnf
"wait" "(" <string> ")"
```

**Examples**:
```ndl
wait("response validation")
wait("1s")
wait("player input")
```

### Meta Elements

#### `<meta_statement>` - System Response
System-level messages.

**Syntax**:
```ebnf
"system_response" "(" <string> ")"
```

**Example**: `system_response("something goes wrong when you try to perform this action")`

## Operator Precedence

From highest to lowest:

1. **Function/Action calls** - `do(...)`, `wait(...)`, etc.
2. **Manner modifier** - `~`
3. **Property assignment** - `=`
4. **Sequence** - `->`
5. **Conjunction** - `^`

## Associativity

| Operator | Associativity |
|----------|---------------|
| `->` | Left-to-right |
| `^` | Left-to-right |
| `~` | Postfix |
| `=` | N/A (not chained) |

## Grammar Examples

### Simple Action
```ndl
do($"player", "walk")
```

**Parse Tree**:
```
<ndl_statement>
  <action_chain>
    <action_element>
      <action>
        do ( $"player" , "walk" )
```

### Action with Manner
```ndl
do($"player", "walk") ~ "carefully"
```

**Parse Tree**:
```
<ndl_statement>
  <action_chain>
    <action_element>
      <action>
        do ( $"player" , "walk" )
      <modifiers>
        <manner>
          ~ "carefully"
```

### Action with Properties
```ndl
do($"player", "attack") intention="intimidate"
```

**Parse Tree**:
```
<ndl_statement>
  <action_chain>
    <action_element>
      <action>
        do ( $"player" , "attack" )
      <modifiers>
        <properties>
          <property>
            intention = "intimidate"
```

### Chained Actions
```ndl
do($"player", "attack") -> roll("hit") -> result("success")
```

**Parse Tree**:
```
<ndl_statement>
  <action_chain>
    <action_element>
      <action>
        do ( $"player" , "attack" )
    -> <action_element>
      <action>
        roll ( "hit" )
    -> <action_element>
      <action>
        result ( "success" )
```

### Complete Complex Example
```ndl
do($"you", "swing makeshift club against table's surface", intention="test club's combat readiness")
-> result("failure")
-> system_response("something goes wrong when you try to perform this action")
-> describe()
```

**Parse Tree**:
```
<ndl_statement>
  <action_chain>
    <action_element>
      <action>
        do ( $"you" , "swing makeshift club against table's surface" )
      <modifiers>
        <properties>
          <property>
            intention = "test club's combat readiness"
    -> <action_element>
      <action>
        result ( "failure" )
    -> <action_element>
      <meta_statement>
        system_response ( "something goes wrong when you try to perform this action" )
    -> <action_element>
      <action>
        describe ( )
```

## Syntactic Constraints

### 1. Entity Reference Placement
Entity references (`$`) can appear:
- As first parameter to `do()` action
- In `target()` action
- Not in manner modifiers or property values (string literals only)

### 2. Manner Modifier Position
`~` operator must immediately follow an action element:
```ndl
✓ do($"player", "walk") ~ "carefully"
✗ ~ "carefully" do($"player", "walk")
```

### 3. Property Position
Properties can appear:
- After actions
- After manner modifiers
- Not standalone

### 4. Sequence Operator Rules
`->` connects complete action elements:
```ndl
✓ action1 -> action2
✗ action1 -> -> action2  # No empty elements
✗ -> action1              # Must have left operand
```

## Syntactic Ambiguities

### 1. String Content
Strings can contain any character, including operators:
```ndl
do($"player", "move -> next room")  # -> inside string is literal
```

### 2. Property Names
Property names can be:
- Reserved keywords (`intention`, `perspective`)
- Custom identifiers (user-defined)

Context determines interpretation:
```ndl
intention="test"     # 'intention' is property name
"intention"          # 'intention' is string value
```

## Whitespace Rules

Whitespace is **not significant** except:
1. To separate tokens
2. Within string literals (preserved)

Equivalent statements:
```ndl
do($"player", "attack") -> result("success")
do($"player","attack")->result("success")
do  (  $"player"  ,  "attack"  )  ->  result  (  "success"  )
```

## Comment Syntax

Comments are not explicitly defined in the transcripts. If implemented:

```ndl
# This could be a comment
do($"player", "attack")  # End-of-line comment
```

## Grammar Extensions

### Observed But Not Fully Specified

1. **Duration Literals**: `wait(1s)`, `wait(500ms)`
2. **Numeric Parameters**: `damage(8)`
3. **Multiple Property Syntax**: Not fully clear if comma-separated or repeated

### Future Expansion

As veritasr noted, NDL vocabulary is growing:
- 43+ social actions
- 36 physical actions
- 35 mental actions

Grammar is extensible to accommodate new action types without structural changes.

## Validation Rules

A valid NDL statement must:
1. Start with an action, control, or meta statement
2. Use `->` only to connect complete action elements
3. Apply `~` only to actions (not control/meta statements)
4. Use proper entity reference syntax (`$"name"`)
5. Enclose all string values in double quotes

## Error Cases

### Syntax Errors

```ndl
❌ do()                              # Missing parameters
❌ do($"player")                     # Incomplete parameters
❌ do($"player", "walk" -> action2)  # -> inside parameter
❌ ~ "carefully"                     # Manner without action
❌ intention="test"                  # Property without action
```

### Semantic Errors (Valid Syntax, Invalid Meaning)

```ndl
⚠️ wait("player")                    # Waiting for entity (not condition)
⚠️ do($"nonexistent", "act")        # Unknown entity
⚠️ roll("invalid_stat")             # Invalid stat name
```

## Related Documentation

- [[01-lexical-elements|Lexical Elements]]
- [[03-semantics|Semantic Rules]]
- [[04-parsing|Parsing Implementation]]
- [[do-action|do() Construct]]

---

#ndl #specification #grammar #syntax
