# NDL Pattern: Combat Narration

#ndl #pattern #combat #action-sequences

## Overview

Combat narration in NDL demonstrates the language's ability to describe complex action sequences with deterministic outcomes. By pre-computing combat results (hits, misses, damage), NDL allows even small 8B-9B parameter models to generate coherent, dramatic combat narrative without hallucinating outcomes.

## Core Philosophy

> "The game mechanics are the easy part, you just tell the LLM whether it hit or not, and the LLM goes on to make up fancy combat words explaining that you gottem or that you made a fucky wucky" - [[User-50h100a]]

Combat in NDL follows the separation of concerns principle:
1. **Game logic determines** - Who attacks, what happens, damage dealt
2. **NDL describes** - The action, manner, intention, target, and result
3. **LLM narrates** - The dramatic prose describing the outcome

## Basic Combat Pattern

### Simple Attack

```ndl
do($"player", "attack") ~ "fiercely" -> target($"goblin") -> result("hit") -> damage(8)
```

**Generated Narrative** (typical output):
> "With fierce determination, you swing your blade at the goblin. Your strike connects solidly, dealing 8 damage. The goblin staggers backward, clutching the wound."

### Attack with Miss

```ndl
do($"player", "attack") ~ "recklessly" -> target($"guard") -> result("miss")
```

**Generated Narrative**:
> "You attack recklessly, leaving yourself exposed. The guard easily sidesteps your wild swing, your weapon finding only air."

### Enemy Attack

```ndl
do($"witch", "attack") ~ "fireball" -> target($"player") -> result("hit") -> damage(5) -> describe("burning")
```

**Generated Narrative**:
> "The witch hurls a blazing fireball in your direction. The flames strike you squarely, dealing 5 damage. Your skin blisters from the burning heat."

## Advanced Combat Patterns

### Multi-Action Combat Sequence

```ndl
do($"player", "dodge") ~ "quickly" -> result("success")
-> do($"player", "counterattack") ~ "sword" -> target($"bandit") -> result("critical") -> damage(15)
```

**Design Notes**:
- Each action is explicitly sequenced with `->`
- Success/failure of defensive actions precedes offensive actions
- Critical hits are just another result type that the narrative layer interprets

### Weapon-Specific Attacks

```ndl
do($"player", "swing") ~ "two-handed axe" -> target($"door") -> result("destroyed")
```

```ndl
do($"archer", "shoot") ~ "longbow" -> target($"player") -> result("graze") -> damage(2)
```

**Pattern**: The manner modifier (`~`) specifies the weapon or attack method, letting the LLM add weapon-appropriate descriptive language.

### Failed Combat Action

From real Gemma 2 testing (see Media/image-11F41.png):

```ndl
do($"you", "swing makeshift club against table's surface",
   intention="test club's combat readiness")
-> result("failure")
-> system_response("something goes wrong when you try to perform this action")
-> describe()
```

**Generated Narrative**:
> "You raise the gnarled branch, hefting it awkwardly in your hand. It's heavier than you anticipated, the rough bark digging into your palm. You bring it back over your head, aiming for the tabletop. But as you swing, your momentum falters. The branch veers off course, catching on the edge of the chair.
>
> With a sickening crack, the makeshift club splinters, breaking into several pieces. The impact sends a shower of splinters flying, scattering across the floor. You stare down at the useless fragments in your hand, your heart sinking."

**Design Notes**:
- Intention reveals *why* the player attempted this (testing the weapon)
- Result is pre-determined (failure) - no hallucination of success
- `system_response()` adds meta-information about action failure
- `describe()` prompts detailed narrative of the failure's consequences

## Combat Turn Structure

### Turn-Based Combat (Basic)

```ndl
# Player turn
do($"player", "attack") ~ "sword" -> target($"orc") -> result("hit") -> damage(6)
-> wait("enemy response")

# Enemy turn
do($"orc", "attack") ~ "club" -> target($"player") -> result("hit") -> damage(4)
-> wait("player response")
```

**Processing Flow**:
1. Player declares action → game resolves → NDL generated → LLM narrates
2. Wait marker signals turn boundary
3. Enemy action processed same way

### Combat with Positioning

```ndl
do($"player", "attack") ~ "sword pointing forward"
-> target($"enemy weapon")
-> result("deflected")
-> do($"player", "riposte") ~ "quickly" -> target($"enemy") -> result("hit") -> damage(5)
```

**Pattern Insight** (from [[User-monkeyrithms]]):
> "one thing I experimented with is asking it which way the player's weapon is pointing as it's being held (as in fencing, like picture two characters fighting with swords -- the swords are usually held in front of you).
>
> it is reminded of that direction when it posts, so if the player's sword is being held forwards, a prompt reminder will tell the LLM, 'the player's weapon is in front of them. This will make a direct attack difficult. You should deal with the weapon, first.'
>
> This substantially raised the coherency of scenes that involved combat with weapons"

## Chain-of-Thought Combat Resolution

For complex combat requiring tactical reasoning:

```ndl
roll("attack", modifiers="+2 flanking") -> result("18")
-> do($"player", "attack") ~ "flanking strike" -> target($"knight") -> result("hit") -> damage(9)
```

**Pattern**: Use `roll()` to expose the game's decision process before narration, making the outcome feel earned and logical.

## Combat Scene Modes

### Scene Transition to Combat

```ndl
do($"goblin", "attack") -> target($"player")
-> wait("combat mode")
-> system_response("combat has begun")
```

**Implementation Note** (from [[User-monkeyrithms]]):
> "when combat has begun. The game window will turn red as a visual indicator, and NPCs will be fed different kinds of prompts that facilitate that sort of scene better."
>
> "as the combat started the game detected combat and enters a different 'mode' where the characters are given different kinds of prompts that have a stronger emphasis on doing only 1 thing per turn, and not describing the outcome of their actions"

### Combat-Specific Prompting

During combat mode, the system prompt changes:
- Emphasize single actions per turn
- Don't describe outcomes of actions (game determines these)
- Focus on character intent and emotional state
- Respect weapon positioning and tactical constraints

## Combat State Tracking

### Status Effects

```ndl
do($"wizard", "cast") ~ "slow spell" -> target($"warrior") -> result("success")
-> describe("warrior is slowed")
```

```ndl
do($"player", "attack") ~ "while slowed" -> target($"enemy") -> result("miss")
-> system_response("slowed condition affects attack accuracy")
```

### Health Thresholds

```ndl
do($"enemy", "attack") -> target($"player") -> result("hit") -> damage(10)
-> system_response("player health critical")
-> wait("player decision")
```

**Pattern**: Use `system_response()` to inject state information that influences narrative tone (desperation, caution, triumph).

## Common Anti-Patterns

### ❌ Don't: Let LLM Decide Combat Outcomes

```ndl
# WRONG - leaves outcome to LLM
do($"player", "attack") -> target($"dragon")
# LLM might hallucinate a hit when game logic determined a miss
```

### ✅ Do: Always Specify Result

```ndl
# CORRECT - outcome is deterministic
do($"player", "attack") -> target($"dragon") -> result("miss")
# LLM narrates the miss creatively, but can't change the outcome
```

### ❌ Don't: Vague Action Descriptions

```ndl
# WRONG - too generic
do($"player", "fight")
```

### ✅ Do: Specific Actions with Context

```ndl
# CORRECT - specific action with manner
do($"player", "thrust") ~ "spear" -> target($"shield") -> result("blocked")
```

### ❌ Don't: Mix Multiple Targets Ambiguously

```ndl
# WRONG - unclear which enemy is targeted
do($"player", "attack") -> target($"enemies")
```

### ✅ Do: One Action, One Target

```ndl
# CORRECT - explicit target
do($"player", "attack") -> target($"goblin_leader") -> result("hit") -> damage(7)
```

## Integration with Game Systems

### D&D-Style Combat

```python
def generate_combat_ndl(attacker: Entity, defender: Entity, attack_roll: int,
                        ac: int, damage_roll: int) -> str:
    """Generate NDL for D&D-style combat resolution."""
    hit = attack_roll >= ac

    # Build NDL based on game logic
    ndl = f'do(${attacker.id}, "attack")'
    ndl += f' ~ "{attacker.equipped_weapon.name}"'
    ndl += f' -> target(${defender.id})'

    if hit:
        ndl += f' -> result("hit") -> damage({damage_roll})'
        if attack_roll >= ac + 10:  # Critical hit
            ndl += ' -> describe("critical strike")'
    else:
        ndl += f' -> result("miss")'
        if attack_roll == 1:  # Critical fumble
            ndl += ' -> describe("fumble")'

    return ndl
```

### Action Point System

```ndl
# First action (2 AP)
do($"player", "power attack") ~ "greatsword" -> target($"golem") -> result("hit") -> damage(12)
-> system_response("2 action points spent, 1 remaining")

# Second action (1 AP)
-> do($"player", "move") ~ "defensive position"
-> system_response("turn complete")
```

### Damage Types and Resistance

```ndl
do($"player", "cast") ~ "fire bolt" -> target($"ice elemental") -> result("hit") -> damage(8)
-> describe("weak to fire, extra damage")
-> system_response("ice elemental takes 16 damage total")
```

## Genre-Specific Variations

### Fantasy Melee Combat

```ndl
do($"knight", "charge") ~ "lance" -> target($"orc chieftain")
-> result("hit") -> damage(14) -> describe("mounted charge")
```

### Sci-Fi Firefight

```ndl
do($"player", "fire") ~ "plasma rifle" -> target($"alien")
-> result("hit") -> damage(6) -> describe("shields at 40%")
```

### Horror Combat (Futility)

```ndl
do($"player", "shoot") ~ "pistol" -> target($"monster")
-> result("hit") -> damage(2) -> system_response("it barely notices")
```

### Tactical Combat (Positioning Matters)

```ndl
do($"player", "flank") ~ "stealthily" -> target($"guard") -> result("advantage")
-> do($"player", "backstab") ~ "dagger" -> target($"guard") -> result("critical") -> damage(18)
```

## Performance Considerations

### Model Requirements

- **Minimum**: 8B-9B parameter models (Gemma 2, Llama 3)
- **Optimal**: 13B+ for complex combat with multiple actors
- **Works well on**: Local inference, no GPT-4 required

### Token Efficiency

Combat NDL is token-efficient:
- Simple attack: ~15-25 tokens of NDL
- Complex sequence: ~40-60 tokens
- Generated output: 50-150 tokens typically

Compare to pure prompt-based combat:
- Prompt with full context: 200-400 tokens
- Risk of hallucination: High without NDL structure

## Testing Combat Patterns

### Validation Checklist

- [ ] Hit results generate successful attack narrative
- [ ] Miss results never show successful hits
- [ ] Damage values appear in narrative accurately
- [ ] Weapon types influence descriptive language
- [ ] Status effects mentioned in follow-up narration
- [ ] Critical hits generate appropriately dramatic language
- [ ] Multiple combatants tracked correctly across turns
- [ ] Combat → non-combat transitions handled smoothly

### Common Edge Cases

1. **Overkill damage**: Player deals 50 damage to enemy with 5 HP
   ```ndl
   do($"player", "smash") ~ "hammer" -> target($"rat") -> result("overkill") -> damage(50)
   -> describe("obliterated")
   ```

2. **Healing during combat**:
   ```ndl
   do($"cleric", "cast") ~ "healing word" -> target($"warrior") -> result("healed") -> damage(-8)
   # Negative damage = healing
   ```

3. **Environmental damage**:
   ```ndl
   system_response("lava floor deals damage") -> damage(3) -> target($"player")
   ```

## Real-World Implementation

From [[User-monkeyrithms]] ReallmCraft implementation:
- Combat detected automatically when attack actions occur
- Different prompt templates activate for combat vs exploration
- Turn-based structure enforced via `wait()` sequencing
- Character death triggers scene transition to "Afterlife" location

From [[User-veritasr]] DirectorAPI:
- Combat NDL parsed via regex detection of `do(`, `target(`, `result(`
- Each action in sequence triggers separate LLM generation pass
- Results aggregated before player sees final narrative

## Related Patterns

- [[sequencing]] - Chaining combat actions with `->`
- [[result]] - Deterministic outcome specification
- [[manner-modifier]] - Weapon and style descriptions
- [[intention]] - Why character is attacking (intimidate, defend, etc.)
- [[scene-transitions]] - Moving into/out of combat mode
- [[wait]] - Turn boundaries and response timing

## Further Reading

- [[08-NDL-Natural-Description-Language]] - Full NDL discussion
- [[User-veritasr]] - NDL creator
- [[User-monkeyrithms]] - ReallmCraft combat implementation
- [[User-50h100a]] - Combat philosophy and design

---

**Pattern Status**: Validated in Production (ReallmCraft, ChatBot RPG)
**Model Compatibility**: Gemma 2 8B/9B, Llama 3 8B, larger models
**Complexity**: Intermediate
**Use Cases**: Turn-based RPGs, action games, tactical combat, TTRPG simulation
