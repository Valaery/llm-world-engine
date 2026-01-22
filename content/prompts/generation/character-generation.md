---
tags: [prompt, generation, characters, content-creation]
category: generation
technique: template-based
model-tested: [GPT-4, GPT-3.5, local-models]
contributor: appl2613
status: production-ready
---

# Prompt: Character Generation

## Metadata
- **Category**: Generation
- **Technique**: Template-based generation with validation
- **Model Tested**: GPT-4, GPT-3.5, various local models
- **Contributor**: [[User-appl2613]] (monkeyrithms)
- **Status**: Production-ready (used in ChatBot RPG)

## Purpose

Generate complete NPC character sheets on-the-fly with attributes, personality, equipment, and backstory that fit the game's setting and genre. Characters are immediately usable for roleplay and interaction.

This technique enables procedural character population of game worlds without manual content creation.

## Template

```
Generate a character for a {{GENRE}} setting.

Setting: {{SETTING_NAME}}
Setting Description: {{SETTING_DESCRIPTION}}
Location: {{CHARACTER_LOCATION}}
Role/Occupation: {{OCCUPATION}}

Create a character with the following sections:

NAME:
[Generate an appropriate name for the setting]

DESCRIPTION (2-3 sentences):
[Brief overview of who they are]

PERSONALITY (3-4 traits):
[Key personality characteristics]

APPEARANCE:
[Physical description including build, features, clothing]

GOALS:
[What they want, what motivates them]

EQUIPMENT:
[Items they own or carry, appropriate to setting and role]

ABILITIES:
[Skills and capabilities, realistic for their background]

STORY (2-3 sentences):
[Brief backstory explaining how they ended up in current role]

Format the output as parseable sections with clear headers.
```

## Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{GENRE}} | Game genre/setting type | "medieval fantasy", "cyberpunk", "post-apocalyptic" |
| {{SETTING_NAME}} | Name of the specific setting | "Village of Millhaven" |
| {{SETTING_DESCRIPTION}} | Brief setting context | "A quiet farming village near the border" |
| {{CHARACTER_LOCATION}} | Where they're located | "The village tavern" |
| {{OCCUPATION}} | Their job or role | "Bartender", "Merchant", "Guard", "Alchemist" |

## Usage Example

### Full Prompt
```
Generate a character for a medieval fantasy setting.

Setting: Village of Millhaven
Setting Description: A quiet farming village known for its apple orchards and proximity to the Darkwood Forest. The village has about 200 residents and serves as a rest stop for travelers heading to the capital.
Location: The Brass Lantern Inn (village tavern)
Role/Occupation: Innkeeper

Create a character with the following sections:

NAME:
[Generate an appropriate name for the setting]

DESCRIPTION (2-3 sentences):
[Brief overview of who they are]

PERSONALITY (3-4 traits):
[Key personality characteristics]

APPEARANCE:
[Physical description including build, features, clothing]

GOALS:
[What they want, what motivates them]

EQUIPMENT:
[Items they own or carry, appropriate to setting and role]

ABILITIES:
[Skills and capabilities, realistic for their background]

STORY (2-3 sentences):
[Brief backstory explaining how they ended up in current role]

Format the output as parseable sections with clear headers.
```

### Expected Output
```
NAME:
Gareth Thornwood

DESCRIPTION:
Gareth is a weathered innkeeper in his fifties who has run the Brass Lantern Inn for over two decades. He knows everyone in Millhaven and most of the regular travelers who pass through. Despite his age, he remains sharp-witted and surprisingly strong from years of hauling barrels and breaking up tavern fights.

PERSONALITY:
- Hospitable but cautious with strangers
- Protective of his regular customers
- Good-natured and quick with a joke
- Observant and remembers faces

APPEARANCE:
Build: Stocky and broad-shouldered, about 5'10"
Features: Salt-and-pepper beard, laugh lines around his eyes, calloused hands
Clothing: Simple linen tunic with a leather apron, practical wool trousers, comfortable boots
Notable: A small scar above his left eyebrow from a bar fight years ago

GOALS:
- Keep the inn profitable and maintain its reputation
- Protect his daughter who helps run the kitchen
- Stay informed about events affecting the village
- Eventually retire and pass the inn to his daughter

EQUIPMENT:
- Ring of keys for the inn's rooms
- Sturdy oak cudgel kept behind the bar
- Leather coin purse
- Small knife for opening casks

ABILITIES:
- Skilled at reading people and detecting lies
- Competent bartender and host
- Basic knowledge of local history and gossip
- Can handle himself in a brawl if needed
- Knows the surrounding area well

STORY:
Gareth inherited the Brass Lantern from his father and has expanded it from a simple tavern to a proper inn with rooms. He's seen the village through good times and bad, including a bandit raid ten years ago that left him with his scar. He's a pillar of the community and a reliable source of local information for travelers willing to buy a drink and listen.
```

## Effectiveness Notes

### What Works Well
- ✅ Generates complete, usable characters quickly
- ✅ Characters fit the setting when given good context
- ✅ Output is largely parseable (with post-processing)
- ✅ Creates consistent character archetypes
- ✅ Backstories are coherent and tie to location
- ✅ Equipment and abilities are setting-appropriate

### Known Limitations
- ⚠️ Format can drift (e.g., "Elara is an alchemist" instead of just "Alchemist")
- ⚠️ Need post-processing validation
- ⚠️ Creative models may add abilities not in the template
- ⚠️ Sometimes too generic without specific prompting
- ⚠️ May need multiple generations to get desired result

### Community Feedback

**monkeyrithms**: "Ive been working on my own, still blazing that trail in a sense and today I just implemented procedural generation of characters (so it writes and adds new characters to the game on-the-fly). Its suuuper cool and verifies that this whole idea has a lot of legs to it"

**monkeyrithms**: "it created a new character (who is an alchemist) named Elara Fairweather. It filled in the entire character sheet with generated information and it is a medieval-fantasy character with equipment that fits that setting"

**monkeyrithms**: "it is mostly good at following formats (this char sheet is mostly parsable to my engine). There is 1 formatting issue under Occupation -- It put 'Elara is an alchemist.' -- so for generated content you will have to account for edge-cases"

**Solution**: "for this case, I will have a list of pre-existing occupations and check of those strings are in the occupation, first -- and f they are I will correct it, so this would become 'Alchemist'"

## Post-Processing and Validation

### Format Normalization
```python
def normalize_character_sheet(generated_text):
    """Clean up common formatting issues"""

    # Extract occupation from sentence
    occupation_match = re.search(r'(\w+) is an? (\w+)', generated_text)
    if occupation_match:
        generated_text = occupation_match.group(2)

    # Standardize section headers
    generated_text = re.sub(r'name:', 'NAME:', generated_text, flags=re.IGNORECASE)

    # Remove markdown if present
    generated_text = re.sub(r'```.*?```', '', generated_text, flags=re.DOTALL)

    return generated_text
```

### Equipment Validation
```python
def validate_equipment(equipment_list, setting_rules):
    """Ensure equipment fits setting rules"""

    allowed_items = setting_rules.get_allowed_items()
    validated = []

    for item in equipment_list:
        if item.lower() in allowed_items or \
           any(allowed in item.lower() for allowed in allowed_items):
            validated.append(item)
        else:
            print(f"Warning: Generated invalid item '{item}'")

    return validated
```

### Ability Checking
```python
def validate_abilities(abilities, character_class):
    """Check abilities are appropriate for class"""

    class_abilities = get_class_abilities(character_class)

    for ability in abilities:
        if not is_reasonable_ability(ability, character_class):
            print(f"Warning: '{ability}' may not fit '{character_class}'")

    return abilities
```

## Variations

### Quick NPC (Minimal Detail)
```
Generate a brief NPC for {{SETTING}}:
Name: [appropriate name]
Role: [their job]
One-sentence description: [who they are]
Notable trait: [one memorable characteristic]
```

For background NPCs who don't need full detail.

### Companion Character (Detailed)
```
Generate a detailed companion character for {{SETTING}}.

This character will be a party member, so include:
- Combat abilities and tactics
- Questline hooks in their backstory
- Relationship dynamics (how they interact with others)
- Character arc potential (how they could grow)
- Unique personality quirks
- Dialogue examples

[Rest of template...]
```

### Antagonist Generation
```
Generate an antagonist for {{SETTING}}.

Include:
- Motivation for opposing the player
- Resources and capabilities
- Weaknesses or vulnerabilities
- Relationship to the setting
- Potential redemption arc or reason they're not purely evil

[Rest of template...]
```

### Context Inheritance (Advanced)
For characters in specific locations, inherit thematic properties:

```
Generate a character for {{LOCATION_NAME}}.

Location type: {{LOCATION_TYPE}}
Location characteristics: {{LOCATION_THEMES}}
Common activities: {{LOCATION_ACTIVITIES}}
Typical residents: {{TYPICAL_RESIDENTS}}

The character should reflect the location's themes and fit naturally into this environment.

[Rest of template...]
```

**Example**: Character generated for "Swamp Village" inherits swamp-related themes, equipment, and backstory elements.

## Integration with Game Loop

### Just-In-Time Generation
```python
def get_or_generate_npc(npc_id, location):
    # Check if NPC already exists
    if npc_id in character_database:
        return character_database[npc_id]

    # Generate new NPC
    prompt = build_character_prompt(
        location=location,
        occupation=select_appropriate_occupation(location),
        setting=game_world.setting
    )

    generated_char = llm.generate(prompt)
    validated_char = validate_and_normalize(generated_char)

    # Cache for future use
    character_database[npc_id] = validated_char

    return validated_char
```

### Batch Generation
```python
def populate_location(location, num_npcs=5):
    """Generate multiple NPCs for a location"""

    occupations = get_appropriate_occupations(location)
    characters = []

    for occupation in random.sample(occupations, num_npcs):
        char_prompt = build_character_prompt(location, occupation)
        character = llm.generate(char_prompt)
        characters.append(validate_and_normalize(character))

    location.npcs = characters
    return location
```

## Temperature and Sampling Settings

**Recommended**:
- Temperature: 0.7 - 0.9 (want variety and creativity)
- Top-p: 0.9 - 0.95
- Repetition Penalty: 1.15 - 1.25

Higher creativity is good for character generation - you want variety.

## Quality Control

### Reroll System
Allow multiple generations and let user/system pick best:

```python
def generate_character_with_rerolls(prompt, num_attempts=3):
    candidates = []

    for _ in range(num_attempts):
        char = llm.generate(prompt)
        score = rate_character_quality(char)
        candidates.append((char, score))

    # Return highest-scoring character
    return max(candidates, key=lambda x: x[1])[0]
```

### Quality Scoring
```python
def rate_character_quality(character):
    score = 0

    # Check completeness
    required_sections = ['NAME', 'DESCRIPTION', 'PERSONALITY', 'EQUIPMENT']
    for section in required_sections:
        if section in character:
            score += 1

    # Check for formatting issues
    if not contains_format_errors(character):
        score += 2

    # Check for setting appropriateness
    if fits_setting_theme(character):
        score += 2

    return score
```

## Related Prompts
- [[location-generation]] - Generate places for characters
- [[template-generation]] - Meta-prompts for character templates
- [[world-generation]] - High-level world context

## Source
- Discussion context: Mid-2024 procedural generation implementation
- Implemented in: ChatBot RPG by appl2613
- Referenced in: [[04-World-Generation]]
- Character editor shown in: Media/ChatbotRPG5-64BE7.jpg
