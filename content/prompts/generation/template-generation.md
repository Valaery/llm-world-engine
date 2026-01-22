---
tags: [prompt, generation, meta-prompt, template, procedural-generation]
category: generation
technique: meta-prompting
status: proven
contributor: [[User-veritasr]]
models_tested: [Claude, GPT-4, Gemma 2, Stheno]
date_created: 2024-07-11
date_updated: 2024-07-12
---

# Prompt: Template Generation (Meta-Prompt)

## Metadata
- **Category**: Generation
- **Technique**: Meta-prompting (prompts that generate prompts)
- **Model Tested**: Claude, GPT-4, Gemma 2, Stheno, EstopianMaid 13B
- **Contributor**: [[User-veritasr]]
- **Status**: Proven (production in ReallmCraft)
- **Implementation**: ReallmCraft world generation system

## Purpose

Generate structured JSON templates that define procedural generation rules for game content (locations, characters, items, events). This meta-prompt creates templates that include attribute lists, generation prompts with placeholders, and hierarchical relationships. It's a "generator that creates generators."

**Key Innovation**: Instead of manually writing templates for every location type, item category, or character archetype, this prompt generates complete template specifications that can be used for procedural content creation.

## Template

```
Create a JSON template for generating {{entity_type}} in a {{genre}} setting.
The template should be designed for {{scope_level}} level content.

Requirements:

1. **template_name**: A descriptive name for this type of {{entity_type}}

2. **name_generation**: Array of 3-5 naming patterns using placeholders like {{descriptor}}, {{material}}, {{concept}}, etc.

3. **Core Classifications**:
   - location_type or entity_type (urban/rural/wilderness/dungeon, etc.)
   - location_subtype or entity_subtype (specific variants)
   - location_label or entity_label (what it's commonly called)

4. **node_lod**: Level of detail - {{lod_level}} (World/Region/Location/Detail)

5. **Hierarchical Rules**:
   - requires_children: boolean (does this need sub-locations/components?)
   - minimum_children: number
   - maximum_children: number
   - connection_types: how this connects to other entities

6. **Attribute Arrays**: For each major characteristic, provide:
   - 5-10 possible values
   - Thematic consistency with template name
   - Values that work in combination

   Include arrays for:
   - overall_wealth
   - group_affiliations
   - common_events
   - common_patrons
   - common_items
   - ambiance
   - reputation
   - size
   - opening_hours
   - security_level
   - services (if applicable)

7. **Randomization Rules**:
   - min_affiliations and max_affiliations
   - ambiance_count (how many adjectives to combine)
   - Any constraints on value combinations

8. **Special Features**:
   - major_landmarks (for locations)
   - unique_mechanics (for special behaviors)
   - governing_body (for settlements)
   - economic_specialties (for trade hubs)

9. **Name Component Arrays**:
   Define the building blocks for name_generation patterns:
   - settlement (city, town, village, haven, etc.)
   - descriptor (grand, ancient, forgotten, etc.)
   - resource (gold, iron, silk, etc.)
   - creature (dragon, wolf, raven, etc.)
   - material (stone, wood, crystal, etc.)

10. **generation_prompt**: Create a tailored prompt template for this specific type of {{entity_type}}. The prompt should:

    a) Use placeholders that correspond to the fields in this template (e.g., {{location_label}}, {{size}}, {{ambiance}})

    b) Be specific to the type of content being described, highlighting its unique features and purpose

    c) Remain flexible enough to accommodate any combination of options present in the template fields

    d) Guide the description to include relevant details without being overly prescriptive

    e) Specify the {{lod_level}} perspective (e.g., World = continental scope, Region = city overview, Location = specific building, Detail = room)

    Example structure:
    "Describe a {{size}} {{location_label}} called {{name}} in a {{location_subtype}} {{location_type}} setting. As a {{node_lod}}-level location, focus on {{appropriate_scope}}.

    Include:
    * Key feature 1 using {{placeholder1}}
    * Key feature 2 using {{placeholder2}}
    * Atmosphere from {{ambiance}} descriptors
    * Reputation as {{reputation}}
    * {{Other relevant attributes}}

    Your description should bring this {{location_label}} to life, emphasizing {{core_theme}}, and making it a vivid and memorable part of the {{scope}} setting."

Output valid JSON only. No additional commentary.
```

## Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{entity_type}} | What kind of content to generate | "tavern", "trade city", "magic shop", "NPC archetype" |
| {{genre}} | Setting type | "dark fantasy", "sci-fi", "steampunk", "historical" |
| {{scope_level}} | Level of detail | "Region", "Location", "Detail", "World" |
| {{lod_level}} | Specific LOD instruction | "Region" means city-level, "Location" means building-level |

## Usage Example

### Input to Meta-Prompt

```
Create a JSON template for generating trade city in a dark fantasy setting.
The template should be designed for Region level content.

[... full template requirements as above ...]
```

### Expected Output (Abbreviated)

```json
{
  "template_name": "Trade City",
  "name_generation": [
    "{{descriptor}} {{settlement}} of {{resource}}",
    "{{settlement}} of the {{creature}} {{resource}}",
    "{{material}} {{settlement}}"
  ],
  "location_type": "urban",
  "location_subtype": ["coastal", "river", "crossroads", "mountain pass"],
  "location_label": ["metropolis", "trade hub", "merchant city", "market capital"],
  "node_lod": "Region",
  "requires_children": true,
  "minimum_children": 5,
  "maximum_children": 15,
  "connection_types": ["road", "sea route", "river", "portal"],
  "overall_wealth": ["affluent", "prosperous", "wealthy", "opulent"],
  "group_affiliations": ["Merchant Guild", "Artisan Union", "Trade Council", "Foreign Embassy"],
  "min_affiliations": 2,
  "max_affiliations": 5,
  "common_events": [
    "trade fair",
    "auction",
    "merchant caravan arrival",
    "diplomatic summit",
    "festival of commerce"
  ],
  "common_patrons": [
    "merchant",
    "trader",
    "diplomat",
    "artisan",
    "guild master",
    "foreign visitor"
  ],
  "common_items": [
    "rare spices",
    "exotic fabrics",
    "precious gems",
    "magical artifacts",
    "foreign currencies"
  ],
  "ambiance": [
    "bustling",
    "cosmopolitan",
    "diverse",
    "vibrant",
    "multicultural",
    "lively"
  ],
  "ambiance_count": 2,
  "reputation": [
    "center of commerce",
    "melting pot of cultures",
    "economic powerhouse",
    "trading mecca"
  ],
  "size": ["large", "sprawling", "expansive", "vast"],
  "opening_hours": ["always open", "day and night", "round the clock"],
  "security_level": ["moderate", "high", "vigilant", "well-guarded"],
  "services": [
    "currency exchange",
    "trade brokerage",
    "caravan outfitting",
    "shipping",
    "warehousing",
    "legal arbitration"
  ],
  "major_landmarks": [
    "grand bazaar",
    "merchant's guild hall",
    "customs house",
    "trade embassy quarter",
    "international harbor"
  ],
  "governing_body": [
    "merchant council",
    "trade consortium",
    "elected commerce board",
    "guild alliance"
  ],
  "economic_specialties": [
    "textile trade",
    "spice market",
    "magical item exchange",
    "rare metal trading",
    "livestock auctions"
  ],
  "settlement": ["city", "metropolis", "port", "haven"],
  "descriptor": ["golden", "shining", "bustling", "grand"],
  "resource": ["gold", "silk", "spice", "gem"],
  "creature": ["dragon", "phoenix", "griffon", "unicorn"],
  "material": ["marble", "crystal", "ivory", "jade"],
  "generation_prompt": "Describe a {{size}} {{location_label}} known as {{name}}, located in a {{location_subtype}} {{location_type}} setting. As a Region-level location, focus on the city's overall layout, major districts, and its role in the broader world.\n\nInclude:\n* The city's primary function as a trade hub and its economic importance\n* Key features that make it a center of commerce (e.g., {{major_landmarks}})\n* The general atmosphere, drawing from {{ambiance}} descriptors\n* Its reputation as {{reputation}}\n* The {{governing_body}} and how it impacts the city's operations\n* Notable {{economic_specialties}} that drive the city's prosperity\n* How the city's {{security_level}} security level is maintained\n* The constant activity implied by its {{opening_hours}} nature\n\nMention any significant {{group_affiliations}} and their influence on the city. Describe the diverse population, including {{common_patrons}} and the multitude of visitors from far and wide.\n\nYour description should bring this {{location_label}} to life, emphasizing its role as a nexus of trade and cultural exchange, and making it a vivid and memorable part of the larger setting."
}
```

## Multi-Template Generation Workflow

**Production Pattern** from [[User-veritasr]] (July 2024):

```python
# Generate multiple templates in batch
template_types = [
    "tavern",
    "trade city",
    "wizard tower",
    "dungeon entrance",
    "forest grove",
    "haunted mansion"
]

templates = []
for template_type in template_types:
    prompt = create_template_generation_prompt(
        entity_type=template_type,
        genre="dark fantasy",
        scope_level="Location"
    )

    template_json = llm.generate(prompt, temperature=0.7)
    templates.append(json.loads(template_json))

    # Save for later use
    save_template(template_type, template_json)

print(f"Generated {len(templates)} templates")
```

**[[User-veritasr]]'s Note**:
> "essentially a generator that creates generators. If it works, then I'll just let it run in the background on a loop, then come back and tweak the results."

## Just-In-Time Generation (JITG) Pattern

Once you have templates, use them for JIT generation:

```python
def generate_location(template, parent_context=None):
    """
    Generate a specific location from a template.
    Only generates when needed, not up-front.
    """
    # 1. Roll on random tables to select values
    selected_values = {}
    for field, options in template.items():
        if isinstance(options, list):
            selected_values[field] = random.choice(options)

    # 2. Generate name
    name_pattern = random.choice(template['name_generation'])
    name = fill_placeholders(name_pattern, selected_values)

    # 3. Use template's generation_prompt
    gen_prompt = template['generation_prompt']
    filled_prompt = fill_placeholders(gen_prompt, {**selected_values, 'name': name})

    # 4. Generate description
    description = llm.generate(filled_prompt, temperature=0.6)

    # 5. Store as fixed entity
    location = {
        'name': name,
        'description': description,
        'attributes': selected_values,
        'template': template['template_name']
    }

    save_to_db(location)
    return location
```

**[[User-veritasr]]'s Philosophy**:
> "basically limitless potential until it exists as a fact. Once it becomes a fact, though it needs to be set in stone."

## Effectiveness Notes

### What Works Well
- **Rapid iteration**: Generate dozens of templates quickly
- **Consistent structure**: All templates follow same schema
- **Modding-friendly**: Non-programmers can understand template JSON
- **Reduces manual work**: One meta-prompt replaces hours of template writing
- **Hierarchical support**: Templates can reference parent/child relationships
- **Placeholder system**: Clear variable substitution
- **Self-documenting**: generation_prompt explains how to use the template

### Known Limitations
- **Requires refinement**: First-pass templates need human review
- **Quality varies by model**: GPT-4/Claude best, local models struggle
- **JSON validation needed**: Output may have syntax errors
- **Combinatorial explosions**: Some attribute combinations don't make sense
- **Over-generic**: Templates can be too general without specific examples
- **Context limits**: Very complex templates may exceed context windows

### Model-Specific Notes
- **GPT-4/Claude**: Excellent at generating coherent templates, understands complex requirements
- **Gemma 2 9B**: Can generate simple templates but struggles with nested complexity
- **Local 7B models**: Generally insufficient for meta-prompting
- **Stheno**: Decent for creative template variations

### Community Feedback

**[[User-veritasr]]** (July 11, 2024):
> "Think I have my prompt. Time to test it out. Essentially takes a scope and a type of location trying to be designed... essentially a generator that creates generators."

**After Testing** (July 11, 2024):
> "lol. it works. Should save a bit of time."

**On Output Quality**:
> "Obviously not perfect in it's current state, but it's a solid start."

## Temperature Settings

**Recommended Settings**:
- **For meta-prompt**: Temperature 0.7, Top-p 0.9
  - Need creativity for varied templates
  - But not so much that JSON breaks
- **For using generated templates**: Temperature 0.5-0.7, Top-p 0.9
  - Consistent with template constraints
  - Some variety in actual content

## Related Prompts

- [[generation/location-generation|Location Generation]] - Uses templates created by this prompt
- [[generation/character-generation|Character Generation]] - Similar template approach
- [[narration/scene-description|Scene Description]] - Consumes template-generated data
- [[techniques/procedural-generation|Procedural Generation]] - Broader context

## Variations

### 1. Item Template Generator

```
Create a JSON template for generating {{item_category}} in a {{genre}} setting.

Include:
- item_type, item_subtype
- rarity levels
- material_types
- enchantment_categories (if magical)
- value_range
- weight_class
- damage_type (if weapon)
- armor_rating (if armor)
- effect_types (if consumable)
- name_generation patterns
- description_prompt template
```

### 2. Character Archetype Generator

```
Create a JSON template for generating {{character_archetype}} NPCs.

Include:
- personality_traits (5-10 options)
- motivations
- fears
- speech_patterns
- typical_occupations
- relationships_potential
- quest_giving_likelihood
- dialogue_prompt template
- backstory_generation_prompt
```

### 3. Quest Template Generator

```
Create a JSON template for generating {{quest_type}} quests.

Include:
- quest_giver_types
- objective_types
- reward_categories
- difficulty_levels
- estimated_duration
- required_party_size
- location_requirements
- prerequisite_quests
- branching_paths
- failure_conditions
- quest_prompt template
```

## Hierarchical Template Chaining

Templates can reference each other:

```python
# World -> Region -> Location -> Detail

world_template = generate_template("continent", lod="World")
region_template = generate_template("kingdom", lod="Region", parent=world_template)
location_template = generate_template("castle", lod="Location", parent=region_template)
detail_template = generate_template("throne room", lod="Detail", parent=location_template)

# Context inherits from parent to child
for template in [world_template, region_template, location_template, detail_template]:
    template['parent_context'] = get_parent_attributes(template)
```

**[[User-veritasr]]'s Note**:
> "Then it's just rinse and repeat for n locations within {{parent}} and optionally generate n children per entity, if the template calls for it."

## Post-Generation Refinement

After generating templates, refine them:

```python
def refine_template(template_json):
    """
    Post-process generated template for quality.
    """
    # 1. Validate JSON structure
    validate_json_schema(template_json)

    # 2. Check for contradictory attributes
    remove_contradictions(template_json)

    # 3. Ensure placeholder consistency
    verify_placeholders(template_json)

    # 4. Test generation_prompt
    test_prompt_output(template_json['generation_prompt'])

    # 5. Save validated template
    return template_json
```

## Testing Checklist

- [ ] Generate 5-10 templates of same type, compare quality
- [ ] Validate all JSON outputs parse correctly
- [ ] Test generated generation_prompts with actual data
- [ ] Check placeholder consistency across template
- [ ] Verify hierarchical relationships make sense
- [ ] Test with different models (GPT-4, Claude, local)
- [ ] Generate actual content from templates
- [ ] Check for contradictory attribute combinations
- [ ] Validate name_generation patterns produce good names
- [ ] Test edge cases (minimal children, maximum children)

## Source

**Original Discussion**: Discord transcript lines 32000-32199 (July 11-12, 2024)
**Context**: [[User-veritasr]] demonstrating meta-prompt system for ReallmCraft's world generation
**Key Quotes**:
> "placeholder serve two functions, to act as state flags and to assist in generation, when called upon."

> "in the case of the tavern example above, the children represent important / rooms of note. Sort of working in a JITG (Just In Time Generation) mindset, where things don't need to exist for the most part until they're needed. But once they're needed they need to be generated with fixed values, so rules can be applied to them."

## Related Documentation

- [[04-World-Generation]] - Broader world-building context
- [[02-Prompt-Engineering]] - Prompting techniques used here
- [[User-veritasr]] - Creator's other contributions
- [[08-NDL-Natural-Description-Language]] - How NDL integrates with generated content
