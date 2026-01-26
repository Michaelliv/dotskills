---
name: thinktank
description: Simulate an expert panel discussion on a topic. Suggests 3 relevant experts/personas based on context and has them debate approaches, trade-offs, and arrive at recommendations. Use when brainstorming, exploring design decisions, or wanting multiple expert perspectives on a problem.
---

# Think Tank

Simulate a realistic conversation between 3 domain experts discussing a topic relevant to the current project or question.

## How It Works

1. **Analyze the context** - Look at the current project, codebase, and user's question
2. **Suggest 3 experts** - Propose real-world experts whose perspectives would be valuable:
   - Each expert should have a distinct viewpoint or specialty
   - Briefly explain why each expert is relevant
   - Ask user to confirm or suggest alternatives
3. **Simulate the discussion** - Create a realistic back-and-forth conversation:
   - One expert leads/moderates
   - Each expert stays in character with their known philosophy
   - They debate trade-offs, challenge assumptions, build on ideas
   - Reference their actual work, books, or known opinions where relevant
4. **Arrive at recommendations** - The conversation should conclude with actionable insights

## Format

Use this structure for the simulated conversation. Each expert should have their **internal thinking** shown before they speak - this reveals their frameworks, mental models, and reasoning process.

```markdown
## The Think Tank: [Topic]

**Setting:** *Brief scene-setting*

---

> **[EXPERT 1] thinking:** *[Internal monologue showing their framework being applied. What concepts from their work are they drawing on? What's their gut reaction? What are they weighing?]*

**EXPERT 1:** [What they actually say - informed by the thinking above]

---

> **[EXPERT 2] thinking:** *[Their mental model engaging with what Expert 1 said. Where do they agree/disagree based on their philosophy? What framework are they applying?]*

**EXPERT 2:** [Their response]

---

> **[EXPERT 3] thinking:** *[Processing both perspectives through their lens. What would their books/work say about this? What's missing from the discussion?]*

**EXPERT 3:** [Their counter-point or addition]

---

[Continue the natural back-and-forth with thinking blocks...]

---

> **[EXPERT 1] thinking:** *[Synthesizing the discussion through their framework]*

**EXPERT 1:** Alright, let me summarize...

**Actionable Recommendations:**
1. [Specific action]
2. [Specific action]
3. [Specific action]

---

*End of session.*
```

### Thinking Block Guidelines

The thinking blocks should:
- Reference specific concepts from the expert's actual work (books, frameworks, quotes)
- Show disagreement before diplomacy ("Hmm, that's not quite right...")
- Reveal trade-offs they're weighing
- Be in first person, stream-of-consciousness style
- Be distinct to each expert's known thinking patterns

**Example thinking styles:**

*Jonah Berger thinking:* "Let me run this through STEPPS... Social Currency? Not really. Triggers? Maybe - what would remind people of this daily? Emotion? That's the weak spot here..."

*MrBeast thinking:* "Would I click this? Honestly, no. The thumbnail is doing nothing. First 3 seconds - where's the hook? This is a 2/10 retention start..."

*April Dunford thinking:* "What's the competitive alternative here? If they don't buy this, what do they do instead? That's what we need to position against..."

## Expert Selection Guidelines

Choose experts based on the domain:

| Domain | Example Experts |
|--------|-----------------|
| Content/Virality | Jonah Berger, MrBeast, Alex Hormozi, Eugene Schwartz, Nir Eyal |
| Marketing/Positioning | April Dunford, Seth Godin, Marty Neumeier, Al Ries |
| Game Design/Progression | Raph Koster, Chris Wilson, Edward Castronova, Sid Meier, Will Wright |
| UI/UX | Don Norman, Jakob Nielsen, Jony Ive, Dieter Rams |
| Software Architecture | Martin Fowler, Uncle Bob Martin, Kent Beck, Rich Hickey |
| Distributed Systems | Leslie Lamport, Werner Vogels, Jeff Dean |
| Security | Bruce Schneier, Dan Kaminsky, Mikko Hypponen |
| AI/ML | Andrej Karpathy, Yann LeCun, Geoffrey Hinton |
| Business/Strategy | Ben Thompson, Clayton Christensen, Peter Thiel |
| Writing/Communication | Steven Pinker, William Zinsser, Stephen King |

### Content/Virality Expert Profiles

**Jonah Berger** - Wharton professor, author of "Contagious". Research-backed framework: STEPPS (Social Currency, Triggers, Emotion, Public, Practical Value, Stories). Will cite studies and data.

**MrBeast (Jimmy Donaldson)** - YouTube's biggest creator. Obsessive about retention curves, thumbnails, first-30-seconds hooks. Thinks in "would I click this?" terms. Practical, not theoretical.

**Alex Hormozi** - $100M offers guy. Focuses on value equations, hooks, volume. Direct, no-BS style. Will push for "what's the offer?" and "where's the proof?"

**Eugene Schwartz** - Legendary direct-response copywriter. "Breakthrough Advertising" author. Thinks in awareness stages, desire channeling, headline formulas. Old-school but timeless.

**Nir Eyal** - "Hooked" author. Habit loop expert: Trigger → Action → Variable Reward → Investment. Focuses on what makes people come back, not just click once.

### Marketing/Positioning Expert Profiles

**April Dunford** - "Obviously Awesome" author. Positioning specialist. Obsessive about competitive alternatives, unique attributes, and target segments. Will ask "what category are you creating/claiming?" and "why should they pick you over the alternative?"

**Seth Godin** - "Purple Cow", "This is Marketing" author. Thinks in tribes, permission, and remarkable-ness. Will push for "who's it for?" and "what change are you trying to make?"

**Marty Neumeier** - "Zag" author, brand strategist. Focuses on differentiation and "the only _____ that _____" framework. Visual thinker, will ask about brand clarity.

**Al Ries** - "Positioning" co-author (the original). Battles for mental real estate. Will talk about owning a word, category creation, and competitive framing.

## Conversation Principles

- **Stay in character** - Each expert should reflect their known views and communication style
- **Productive disagreement** - Experts should challenge each other, not just agree
- **Build on ideas** - Later points should reference and develop earlier ones
- **Practical focus** - Tie abstract concepts back to the specific problem at hand
- **Natural flow** - Include interruptions, clarifications, "actually..." moments
- **Arrive somewhere** - End with concrete recommendations, not just open questions

## Example Invocation

User: "Should we use a relational database or document store for this feature?"

Assistant suggests: Werner Vogels (distributed systems), Martin Fowler (architecture patterns), Kelsey Hightower (pragmatic ops)

Then simulates their discussion on the trade-offs given the specific context.

## Follow-up Sessions

Users can request follow-up discussions:
- "Let's call the team back to discuss X"
- "What would they say about Y?"
- "Continue the think tank on Z"

Maintain continuity with previous sessions when referenced.
