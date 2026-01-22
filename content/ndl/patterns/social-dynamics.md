# NDL Pattern: Social Dynamics

#ndl #pattern #social #relationships #interaction #npc-behavior

## Overview

Social dynamics in NDL handle the complex web of relationships, emotional states, reputation systems, and interpersonal interactions that make NPCs feel alive. By tracking relationship states, social contexts, and interaction history, NDL enables sophisticated social simulation without requiring the LLM to hallucinate or maintain complex interpersonal logic.

## Core Philosophy

Social interaction is **stateful** and **deterministic**:
1. **Game logic tracks** - Relationships, reputation, emotional states, social hierarchies
2. **NDL describes** - Social context, emotional cues, power dynamics
3. **LLM generates** - Appropriate dialogue, body language, and reactions

From [[User-veritasr]]:
> "this is why the character card shouldn't say, 'so-and-so is angry or prone to anger.' The model will tend to write that character _always_ angry, so it would be better to make emotions happen on some other mechanic instead"

**Key Insight**: Emotions and social states are **data**, not personality traits.

## The 43+ Social Actions

From the transcript analysis, NDL includes an extensive vocabulary of social actions:

### Categories (100+ Total Actions)

- **Physical Actions**: 36 documented
- **Mental Actions**: 35 documented
- **Social Actions**: 43+ documented (and growing)

**Design Note**: The social action vocabulary continues expanding as new game mechanics are added.

### Social Action Examples

```ndl
# Communication
talk($"npc", $"player") -> convey("information")
whisper($"rogue", $"player") -> convey("secret")
shout($"guard", $"crowd") -> convey("warning")

# Emotional Expression
laugh($"npc") ~ "mockingly" -> target($"player")
cry($"child") ~ "desperately"
smile($"merchant") ~ "falsely" -> target($"customer")

# Social Positioning
bow($"servant") ~ "deeply" -> target($"lord")
intimidate($"warrior") -> target($"coward") -> result("success")
charm($"bard") -> target($"noble") -> result("enamored")

# Relationship Actions
befriend($"npc") -> target($"player") -> result("friendship +10")
insult($"rival") -> target($"player") -> result("reputation -5")
seduce($"npc") -> target($"player") -> result("attracted")
```

## Relationship Tracking

### Basic Relationship States

```ndl
system_response("relationship: player-guard = hostile")
-> talk($"guard", $"player") ~ "aggressively" -> convey("move along")
```

**Generated Narrative**:
> The guard glares at you. "Move along," he snaps. "We don't want your kind here."

### Relationship Progression

```ndl
# Initial meeting (neutral)
talk($"player", $"npc") -> convey("greeting")
-> result("neutral response")
-> system_response("relationship: player-npc = neutral (50)")

# Positive interaction
do($"player", "help") -> target($"npc") -> result("grateful")
-> system_response("relationship: player-npc = friendly (70)")

# After multiple positive interactions
-> system_response("relationship: player-npc = trusted friend (90)")
```

**Pattern**: Relationship value affects perspective modifiers in dialogue and actions.

### Relationship-Dependent Behavior

```python
def generate_social_ndl(npc: Entity, player: Entity,
                       action: str) -> str:
    """Generate NDL based on relationship state."""

    relationship = get_relationship(npc, player)

    if relationship >= 80:  # Trusted
        perspective = "warm"
        manner = "openly"
    elif relationship >= 50:  # Friendly
        perspective = "friendly"
        manner = "casually"
    elif relationship >= 20:  # Neutral
        perspective = "businesslike"
        manner = "politely"
    else:  # Hostile
        perspective = "cold"
        manner = "curtly"

    ndl = f'talk(${npc.id}, ${player.id}) ~ "{manner}"'
    ndl += f' -> convey("{action}", perspective="{perspective}")'

    return ndl
```

## Reputation Systems

### Guild Reputation

```ndl
do($"player", "complete guild quest") -> result("success")
-> system_response("Guild of Thieves reputation +25, now: 75 (Trusted)")
-> talk($"guild_master", $"player") -> convey("new opportunities available", perspective="respectful")
```

### Faction Dynamics

```ndl
do($"player", "betray faction A") -> result("exposed")
-> system_response("Faction A: hostile, Faction B: friendly")
-> system_response("Faction A members will attack on sight")
```

### Social Class and Status

```ndl
system_response("player status: commoner")
-> talk($"player", $"noble") ~ "casual greeting"
-> result("offended by informality")
-> talk($"noble", $"player") ~ "condescending" -> convey("dismissal")
-> system_response("reputation with nobility -10")
```

**Pattern**: Status mismatches create social friction.

## Emotional State Management

### Temporary Emotional States

```ndl
# NPC becomes angry
do($"player", "insult") -> target($"npc")
-> result("provoked")
-> system_response("npc emotion: angry (30 second duration)")

# Angry NPC behavior
-> talk($"npc", $"player") ~ "shouting" -> convey("offense taken", perspective="furious")

# Emotion fades
-> wait("30 seconds")
-> system_response("npc emotion returned to: neutral")
```

### Persistent Moods

```ndl
system_response("npc mood: depressed (due to quest failure)")
-> talk($"npc", $"player") -> convey("reluctant greeting", perspective="melancholy")
```

### Emotional Contagion

```ndl
# Player inspires courage in allies
do($"player", "rally") -> target($"party") -> result("morale boosted")
-> system_response("party emotion: inspired, courage +2 for 10 minutes")
-> talk($"ally", $"player") -> convey("readiness for battle", perspective="determined")
```

## Social Context and Hierarchy

### Power Dynamics

```ndl
# Subordinate to superior
system_response("social context: servant addressing master")
-> talk($"servant", $"lord") ~ "deferential" -> convey("report", perspective="respectful")

# Superior to subordinate
-> talk($"lord", $"servant") ~ "commanding" -> convey("orders", perspective="authoritative")

# Between equals
system_response("social context: peer conversation")
-> talk($"merchant1", $"merchant2") ~ "familiar" -> convey("business discussion", perspective="collegial")
```

### Public vs. Private Interaction

```ndl
# Public (formal)
system_response("context: throne room, court in session")
-> talk($"ambassador", $"king") ~ "formally" -> convey("diplomatic message", perspective="official")

# Private (informal)
system_response("context: king's private chambers")
-> talk($"ambassador", $"king") ~ "relaxed" -> convey("real concerns", perspective="candid")
```

## Social Skill Checks

### Persuasion

```ndl
talk($"player", $"guard") -> convey("bribe offer", intention="gain entry")
-> roll("persuasion") -> result("success")
-> talk($"guard", $"player") ~ "quietly" -> convey("acceptance", perspective="conflicted")
-> system_response("guard reputation: corruptible (known to player)")
```

### Deception

```ndl
talk($"player", $"suspicious_npc") -> convey("lie", intention="avoid suspicion")
-> roll("deception") -> result("failure")
-> talk($"npc", $"player") ~ "accusatory" -> convey("disbelief", perspective="hostile")
-> system_response("npc alerted, guards called")
```

### Intimidation

```ndl
do($"player", "intimidate") ~ "weapon drawn" -> target($"informant")
-> roll("intimidation") -> result("success")
-> talk($"informant", $"player") ~ "stammering" -> convey("information", perspective="terrified")
-> system_response("informant will remember this threat")
```

### Insight

```ndl
do($"player", "observe") ~ "carefully" -> target($"negotiator")
-> roll("insight") -> result("success")
-> system_response("revealed: negotiator is lying about price")
-> talk($"player", $"negotiator") -> convey("call out deception", perspective="confident")
```

## Relationship Memory and History

### Interaction History

```ndl
# First meeting
system_response("npc has no prior knowledge of player")
-> talk($"npc", $"player") -> convey("curious greeting", perspective="cautious")

# Subsequent meeting (player helped before)
system_response("npc remembers: player helped with quest")
-> talk($"npc", $"player") -> convey("warm greeting", perspective="grateful")
-> system_response("discount offered at shop")

# After betrayal
system_response("npc remembers: player stole from them")
-> talk($"npc", $"player") ~ "coldly" -> convey("refusal to engage", perspective="bitter")
```

### Long-Term Relationship Arcs

```python
class RelationshipArc:
    """Track relationship progression over time."""

    def __init__(self, npc_id: str, player_id: str):
        self.npc_id = npc_id
        self.player_id = player_id
        self.value = 50  # Neutral start
        self.history: list[str] = []
        self.milestones = {
            "first_meeting": False,
            "became_friends": False,
            "romantic_interest": False,
            "betrayal": False
        }

    def generate_greeting_ndl(self) -> str:
        """Generate appropriate greeting based on history."""

        if self.milestones["betrayal"]:
            manner = "hostile"
            perspective = "unforgiving"
        elif self.value >= 80:
            manner = "warmly"
            perspective = "affectionate"
        elif self.value >= 50:
            manner = "friendly"
            perspective = "welcoming"
        else:
            manner = "coolly"
            perspective = "distant"

        ndl = f'talk(${self.npc_id}, ${self.player_id}) ~ "{manner}"'
        ndl += f' -> convey("greeting", perspective="{perspective}")'

        return ndl
```

## Group Dynamics

### Party Cohesion

```ndl
# High morale party
system_response("party morale: high")
-> talk($"warrior", $"party") -> convey("encouragement", perspective="optimistic")
-> talk($"mage", $"party") -> convey("agreement", perspective="confident")

# Low morale party
system_response("party morale: low, recent losses")
-> talk($"warrior", $"party") -> convey("concern", perspective="worried")
-> talk($"mage", $"party") -> convey("suggestion to retreat", perspective="fearful")
```

### Internal Conflict

```ndl
system_response("party tension: warrior and mage disagree on strategy")
-> talk($"warrior", $"mage") -> convey("aggressive plan", perspective="confrontational")
-> talk($"mage", $"warrior") -> convey("cautious alternative", perspective="defensive")
-> do($"player", "mediate") -> result("tension reduced")
-> system_response("party cohesion +5")
```

### Leadership Dynamics

```ndl
# Player as leader
system_response("party recognizes player as leader")
-> talk($"player", $"party") -> convey("strategy")
-> talk($"party_members", $"player") -> convey("acceptance", perspective="trusting")

# Contested leadership
system_response("npc challenges player's authority")
-> talk($"rival", $"party") -> convey("alternative plan", intention="undermine player")
-> wait("party members choose sides")
```

## Romance and Attraction

### Initial Attraction

```ndl
do($"npc", "flirt") ~ "subtly" -> target($"player")
-> result("player notices")
-> system_response("romance flag: npc interested in player")
```

### Romance Progression

```ndl
# Gift giving
do($"player", "give gift") -> target($"romance_interest") -> result("delighted")
-> system_response("romance level: interested → enamored")
-> talk($"npc", $"player") -> convey("gratitude", perspective="affectionate")

# Declaration
talk($"player", $"npc") -> convey("romantic feelings", intention="confess attraction")
-> roll("charisma") -> result("reciprocated")
-> talk($"npc", $"player") -> convey("mutual feelings", perspective="tender")
-> system_response("relationship status: romantic partners")
```

### Jealousy and Rivalry

```ndl
# Player flirts with rival's love interest
do($"player", "flirt") -> target($"npc")
-> system_response("observed by: rival")
-> talk($"rival", $"player") ~ "angrily" -> convey("challenge", perspective="jealous")
-> system_response("rivalry intensified")
```

## Social Stealth and Manipulation

### Blending In

```ndl
do($"player", "adopt disguise") ~ "noble attire"
-> do($"player", "enter") -> target($"noble party")
-> roll("deception") -> result("accepted as noble")
-> system_response("social infiltration successful")
```

### Social Engineering

```ndl
talk($"player", $"guard") -> convey("fake authority", intention="gain access")
-> roll("performance") -> result("believed")
-> talk($"guard", $"player") ~ "deferential" -> convey("permission granted")
```

### Rumor Spreading

```ndl
talk($"player", $"gossip") -> convey("false rumor about rival", intention="damage reputation")
-> wait("rumor spreads")
-> system_response("rival reputation -20 in this town")
```

## Social Consequences

### Reputation Cascade

```ndl
do($"player", "public heroism") -> result("witnessed by crowd")
-> system_response("reputation +30 in city")
-> system_response("strangers recognize player")
-> talk($"citizen", $"player") -> convey("admiration", perspective="starstruck")
```

### Social Exile

```ndl
do($"player", "betray guild") -> result("discovered")
-> system_response("exiled from guild, marked as traitor")
-> talk($"guild_member", $"player") ~ "hostile" -> convey("challenge to leave or fight")
-> system_response("guild members will attack on sight")
```

### Redemption Arc

```ndl
# After exile, perform major service
do($"player", "save guild from threat") -> result("success")
-> talk($"guild_master", $"player") -> convey("grudging acknowledgment", perspective="conflicted")
-> system_response("exile lifted, reputation reset to neutral")
```

## NPC Social AI

### Schedule-Based Social State

From [[User-monkeyrithms]]:
> "as the player follows an NPC throughout the day and observes their schedule living life in the castle, the player comes across a number of scripted events or conversations as they observe them interacting with other characters"

```ndl
# Morning: NPC at breakfast
system_response("time: 8am, npc: $cook at kitchen")
-> talk($"cook", $"servants") -> convey("daily tasks", perspective="businesslike")

# Afternoon: NPC working
system_response("time: 2pm, npc: $cook at market")
-> talk($"cook", $"vendor") -> convey("negotiate supplies", perspective="shrewd")

# Evening: NPC relaxing
system_response("time: 8pm, npc: $cook at tavern")
-> talk($"cook", $"friends") -> convey("gossip", perspective="relaxed")
```

### Mood-Affected Social Behavior

```python
def generate_npc_social_behavior(npc: Entity,
                                 time_of_day: str,
                                 recent_events: list[str]) -> str:
    """Generate NPC social behavior based on state."""

    # Determine mood from recent events
    if "lost_money" in recent_events:
        mood = "frustrated"
        perspective = "irritable"
    elif "received_compliment" in recent_events:
        mood = "pleased"
        perspective = "friendly"
    else:
        mood = "neutral"
        perspective = "businesslike"

    ndl = f'system_response("npc mood: {mood}")'

    # Behavior varies by time and mood
    if time_of_day == "morning":
        if mood == "frustrated":
            ndl += f' -> talk(${npc.id}, $nearby) ~ "grumbling"'
            ndl += f' -> convey("complaints", perspective="{perspective}")'
        else:
            ndl += f' -> do(${npc.id}, "greet") ~ "cheerfully"'

    return ndl
```

## Social Information Economy

### Knowledge as Social Currency

```ndl
# Player has valuable information
talk($"player", $"information_broker") -> convey("secret knowledge", intention="trade for reward")
-> talk($"broker", $"player") -> convey("payment offer", perspective="interested")
-> system_response("gold +100, reputation with brokers +10")
```

### Secrets and Blackmail

```ndl
# Discover secret
search("noble's desk", intention="find incriminating evidence") -> result("letter discovered")
-> system_response("knowledge acquired: noble's affair")

# Use for leverage
talk($"player", $"noble") -> convey("blackmail threat", intention="extort cooperation")
-> roll("intimidation") -> result("success")
-> talk($"noble", $"player") ~ "fearful" -> convey("cooperation", perspective="coerced")
```

### Trust and Betrayal

```ndl
# NPC shares secret with trusted player
system_response("relationship: player-npc = 85 (trusted)")
-> talk($"npc", $"player") ~ "quietly" -> convey("dangerous secret", perspective="confiding")

# If player betrays trust
-> talk($"player", $"enemy") -> convey("reveal npc's secret")
-> system_response("betrayal detected by npc")
-> system_response("relationship: player-npc = 10 (betrayed)")
-> talk($"npc", $"player") ~ "cold" -> convey("I trusted you", perspective="devastated")
```

## Social Learning and Adaptation

### NPC Learning from Interaction

```ndl
# First time player lies to NPC
talk($"player", $"npc") -> convey("lie")
-> roll("deception") -> result("believed")
-> system_response("npc notes: player is deceptive")

# Second time
-> talk($"player", $"npc") -> convey("another lie")
-> roll("deception", modifiers="-2 (npc is suspicious)") -> result("failure")
-> talk($"npc", $"player") ~ "accusatory" -> convey("caught your lie", perspective="distrustful")
```

### Adaptive Social Strategies

```python
class AdaptiveNPC:
    """NPC that learns from social interactions."""

    def __init__(self):
        self.interaction_history: dict[str, list[str]] = {}
        self.personality_model: dict[str, float] = {
            "player_honesty": 0.5,  # 0 = always lies, 1 = always truthful
            "player_generosity": 0.5,
            "player_aggression": 0.5
        }

    def update_model(self, player_id: str, action: str, outcome: str):
        """Update personality model based on observation."""
        if action == "lie" and outcome == "detected":
            self.personality_model["player_honesty"] -= 0.1
        elif action == "gift" and outcome == "given":
            self.personality_model["player_generosity"] += 0.1

    def generate_response_ndl(self, player_id: str, player_action: str) -> str:
        """Generate response based on learned model."""

        honesty = self.personality_model["player_honesty"]

        if player_action == "promise" and honesty < 0.3:
            # NPC doesn't trust player's promises
            perspective = "skeptical"
            manner = "doubtfully"
        else:
            perspective = "trusting"
            manner = "openly"

        return f'talk($npc, ${player_id}) ~ "{manner}" -> convey("response to {player_action}", perspective="{perspective}")'
```

## Common Anti-Patterns

### ❌ Don't: Forget Relationship State

```ndl
# WRONG - previous hostile encounter not reflected
# Last time: player insulted NPC
talk($"npc", $"player") -> convey("friendly greeting")
# Inconsistent!
```

### ✅ Do: Maintain Social Memory

```ndl
# CORRECT - relationship state affects behavior
system_response("relationship: player-npc = 20 (offended)")
-> talk($"npc", $"player") ~ "coldly" -> convey("curt greeting", perspective="still angry")
```

### ❌ Don't: Static Social Responses

```ndl
# WRONG - same response regardless of relationship
talk($"npc", $"player") -> convey("generic dialogue")
# Boring and unrealistic
```

### ✅ Do: Dynamic Relationship-Based Responses

```ndl
# CORRECT - response varies with relationship
system_response("relationship: {relationship_value}")
-> talk($"npc", $"player") -> convey("contextual dialogue", perspective="{relationship_perspective}")
```

## Testing Social Systems

### Validation Checklist

- [ ] Relationship values affect NPC behavior consistently
- [ ] Emotional states have appropriate duration
- [ ] Reputation changes reflected in NPC attitudes
- [ ] Social hierarchy respected in interactions
- [ ] Trust/betrayal tracked and remembered
- [ ] Group dynamics affect individual behavior
- [ ] Social skill checks have meaningful consequences
- [ ] Romance progression feels natural and earned

## Real-World Implementation

From [[User-monkeyrithms]] ReallmCraft:
- Quest stage affects NPC dialogue dynamically
- Schedule-based NPC behavior and conversations
- Relationship tracking influences interactions

From [[User-veritasr]]'s approach:
> "this is basically to keep people from getting too friendly too quickly" - Reputation gates affect relationship progression rates

**23 Exposed States for Social Interaction**:
The implementation tracks 23 different character states that can influence social dynamics (exact list pending deeper transcript analysis).

## Related Patterns

- [[dialogue-generation]] - How social dynamics manifest in conversation
- [[intention]] - Social motivations behind actions
- [[manner-modifier]] - How social context affects delivery
- [[talk]] - Core dialogue construct (to be documented)
- [[convey]] - Message and emotional content (to be documented)

## Further Reading

- [[08-NDL-Natural-Description-Language]] - Full NDL discussion
- [[User-veritasr]] - NDL creator, emotion-as-data philosophy
- [[User-monkeyrithms]] - NPC AI and social systems
- [[02-Prompt-Engineering]] - Social context prompting

---

**Pattern Status**: Validated in Production (ReallmCraft)
**Model Compatibility**: 8B+ models for basic social dynamics, 13B+ for complex relationship modeling
**Complexity**: Advanced
**Use Cases**: RPGs with deep social systems, dating sims, political intrigue games, social stealth games

**Scale**: 43+ social actions documented, 23 exposed character states, 100+ total actions across all categories
