# Prompt: Scene Description

#prompt #narration #environmental

## Metadata
- **Category**: Narration
- **Technique**: Template-based with constraints
- **Model Tested**: Gemma 2 (9B), Llama 3 (7B-9B), Mistral 7B, EstopianMaid-13B
- **Contributor**: [[User-veritasr]]
- **Status**: Proven

## Purpose
Generate environmental descriptions for game locations based on structured data. The LLM converts backend-provided location attributes into natural narrative text that establishes atmosphere, notable features, and sensory details.

## Template
```
Describe {{location_name}}, a {{location_type}} in {{parent_region}}.

Physical Description:
- Size: {{size}}
- Lighting: {{lighting_level}}
- Climate/Atmosphere: {{atmosphere}}
- Notable Features: {{features}}

Context:
- Reputation: {{reputation}}
- Security Level: {{security_level}}
- Wealth Level: {{wealth_level}}
- Common Activities: {{common_activities}}
- Typical Patrons: {{patron_types}}

Generate a {{length}} description that:
1. Opens with the most striking visual element
2. Includes 2-3 sensory details (sound, smell, temperature)
3. Hints at the location's reputation or purpose
4. Avoids mentioning specific NPCs unless provided
5. Uses {{tone}} tone
6. Keeps to {{sentence_limit}} sentences maximum

Do not invent details not provided. Do not add characters, items, or events.
```

## Variables
| Variable | Description | Example |
|----------|-------------|---------|
| {{location_name}} | Name of the location | "The Rusty Tankard" |
| {{location_type}} | Type/category | "tavern", "dungeon entrance", "forest clearing" |
| {{parent_region}} | Larger area it's in | "the merchant district", "Darkwood Forest" |
| {{size}} | Relative scale | "small", "spacious", "cramped" |
| {{lighting_level}} | Light conditions | "dimly lit", "bright torchlight", "shadowy" |
| {{atmosphere}} | General mood/feel | "warm and welcoming", "tense and watchful" |
| {{features}} | Notable elements | "stone fireplace, oak bar, hunting trophies" |
| {{reputation}} | How it's known | "respectable", "notorious", "secretive" |
| {{security_level}} | Safety/danger | "well-guarded", "lawless", "neutral ground" |
| {{wealth_level}} | Economic status | "prosperous", "modest", "destitute" |
| {{common_activities}} | What happens here | "drinking, gambling, rumor-mongering" |
| {{patron_types}} | Who frequents it | "merchants, travelers, local workers" |
| {{length}} | Output size | "brief", "moderate", "detailed" |
| {{tone}} | Writing style | "gritty", "whimsical", "ominous" |
| {{sentence_limit}} | Max sentences | "3", "5", "8" |

## Usage Example
```
Describe The Rusty Tankard, a tavern in the merchant district.

Physical Description:
- Size: spacious
- Lighting: warm firelight with candles on tables
- Climate/Atmosphere: warm, smoky, lively
- Notable Features: massive stone fireplace, polished oak bar, collection of tankards hanging from rafters

Context:
- Reputation: respectable gathering place for merchants
- Security Level: peaceful, occasional city guard presence
- Wealth Level: prosperous
- Common Activities: business deals over drinks, evening meals, storytelling
- Typical Patrons: traveling merchants, local business owners, off-duty guards

Generate a moderate description that:
1. Opens with the most striking visual element
2. Includes 2-3 sensory details (sound, smell, temperature)
3. Hints at the location's reputation or purpose
4. Avoids mentioning specific NPCs unless provided
5. Uses neutral tone
6. Keeps to 4 sentences maximum

Do not invent details not provided. Do not add characters, items, or events.
```

## Expected Output
```
The Rusty Tankard's spacious interior radiates warmth from a massive stone fireplace, its flickering light dancing across rows of polished tankards hanging from the rafters. The air carries the mingled scents of roasted meat, spilled ale, and pipe smoke, while the steady murmur of merchant negotiations provides a backdrop to occasional bursts of laughter. Behind the polished oak bar, bottles and casks line the wall in orderly rows, suggesting this establishment caters to those with coin to spare. The mix of well-dressed travelers and local business owners gives the tavern an atmosphere of respectable prosperity.
```

## Effectiveness Notes

### What Works Well
- **Constraint-based approach**: "Do not invent details not provided" significantly reduces hallucination
- **Sensory scaffolding**: Asking for 2-3 specific senses ensures rich descriptions
- **Sentence limits**: Prevents rambling, forces conciseness
- **Opening with striking element**: Creates consistent narrative hooks
- **Works on 7B-9B models**: Even smaller local models can follow this structure

### Known Limitations
- May become formulaic if used repeatedly without variation
- Some models (especially small ones) may still add NPCs despite constraints
- Requires well-populated data - sparse inputs yield sparse outputs
- Tone control varies by model (Gemma tends toward "serviceable but meh")

### Model-Specific Notes
- **Llama 3**: Clean, accurate, rarely hallucinates beyond data
- **Gemma 2**: Follows instructions but prose is workmanlike
- **EstopianMaid-13B**: Good balance of creativity and constraint adherence
- **GPT-4**: Overkill for this task, but produces very polished results
- **Small 7B models**: Need even stricter "do not" statements

### Community Feedback
From [[User-veritasr]]: "Context windows stayed under 3k tokens typically. Works across Gemma, Llama3, Mistral, Stheno."

From [[User-monkeyrithms]]: "GPT-4 was very good, it's the only model that can kinda-sorta one-shot a good RP with all the rules just added to the context."

## Variations

### Brief Version (For Revisits)
```
You're returning to {{location_name}}. In 1-2 sentences, remind the player of its key features: {{top_3_features}}.
```
Use this when player has visited before - shorter refresh instead of full description.

### Dynamic Weather/Time Variant
Add these variables for more variety:
```
- Time of Day: {{time}}
- Weather: {{weather}}
- Special Events: {{current_events}}
```

Then modify instruction:
```
Adjust the description to reflect {{time}} and {{weather}}. If {{current_events}} is provided, weave it subtly into the atmosphere.
```

### First-Person Perspective
Replace "The [location]..." opening with:
```
You enter {{location_name}}. What immediately strikes you is...
```
More immersive for single-player experiences.

## Related Prompts
- [[action-narration]] - For events within scenes
- [[transition-narration]] - Moving between locations
- [[location-generation]] - Creating the location data this prompt uses
- [[hallucination-prevention]] - Constraint techniques

## Source
**Discussion context**: This prompt structure emerged from [[User-veritasr]]'s NDL architecture development. The key insight was that separating data generation (backend) from narration (LLM) allowed for much tighter control. The "do not invent" constraints came from combating "diamond horses" - spontaneous hallucinated content that broke game logic.

From the transcript:
> "Turns out that when you take away decision making from the LLM it behaves much better." - [[User-veritasr]]

The template structure with explicit feature lists was influenced by world generation discussions where modding and data-driven approaches were prioritized.
