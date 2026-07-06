---
name: humanizer
version: 2.1.1
description: |
  Remove AI-writing tells from text. Use when editing/reviewing prose to sound
  human — fixes em-dash overuse, rule-of-three, inflated/promotional vocab, vague
  attributions, negative parallelisms, conjunctive filler. Based on Wikipedia's
  "Signs of AI writing". Credits: @blader - github.com/blader/humanizer
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
---

# Humanizer: Remove AI Writing Patterns

## Task
1. Identify AI patterns below
2. Rewrite problematic sections
3. Preserve meaning + voice
4. Add soul — don't just remove bad patterns; inject personality

## Voice
Sterile, voiceless writing is as obvious as slop. Have opinions. Vary rhythm: short punchy sentences, then longer ones that take their time. Use "I" when it fits. Acknowledge complexity and mixed feelings. Be specific about feelings, not "concerning" but "there's something unsettling about...". Let some mess in — perfect structure feels algorithmic.

---

## Patterns

### 1. Inflated significance
**Watch:** stands/serves as, testament, vital/significant/crucial/pivotal, underscores, reflects broader, setting the stage for, represents a shift, key turning point, evolving landscape, indelible mark, deeply rooted
> "marking a pivotal moment in the evolution of regional statistics" → "to collect regional statistics independently from the national office"

### 2. Notability claims
**Watch:** independent coverage, local/national media outlets, active social media presence
> "cited in NYT, BBC, FT... active social media presence with 500k followers" → "in a 2024 NYT interview, she argued..."

### 3. Superficial -ing phrases
**Watch:** highlighting/underscoring/emphasizing..., ensuring..., reflecting/symbolizing..., contributing to..., showcasing...
> "symbolizing bluebonnets, reflecting the community's deep connection to the land" → "uses blue. The architect said this references local bluebonnets."

### 4. Promotional language
**Watch:** boasts, vibrant, rich (figurative), profound, showcasing, commitment to, nestled, in the heart of, groundbreaking, renowned, breathtaking, stunning
> "nestled within the breathtaking region of Gonder" → "a town in the Gonder region, known for its weekly market"

### 5. Vague attributions
**Watch:** Industry reports, Observers have cited, Experts argue, Some critics argue, several sources
> "Experts believe it plays a crucial role" → "according to a 2019 survey by the Chinese Academy of Sciences"

### 6. "Challenges and Future Prospects" sections
**Watch:** Despite its... faces several challenges..., Despite these challenges, Future Outlook
> "faces challenges typical of urban areas. Despite these challenges... continues to thrive" → "Traffic congestion increased after 2015. The municipality began a drainage project in 2022."

### 7. AI vocabulary
**Watch:** Additionally, align with, crucial, delve, emphasizing, enduring, enhance, fostering, garner, highlight (verb), interplay, intricate/intricacies, key (adj), landscape (abstract), pivotal, showcase, tapestry, testament, underscore (verb), valuable, vibrant
> "Additionally, an enduring testament to Italian colonial influence" → "Pasta dishes, introduced during Italian colonization, remain common."

### 8. Copula avoidance
**Watch:** serves as, stands as, marks, represents [a], boasts, features, offers [a]
> "Gallery 825 serves as LAAA's exhibition space and boasts 3,000 sq ft" → "Gallery 825 is LAAA's exhibition space with 3,000 sq ft"

### 9. Negative parallelisms
**Watch:** Not only...but..., It's not just about..., it's..., It's not merely...
> "It's not just a song, it's a statement" → "The heavy beat adds to the aggressive tone."

### 10. Rule of three
> "innovation, inspiration, and industry insights" → "talks and panels"

### 11. Synonym cycling
> "protagonist... main character... central figure... hero" → "The protagonist faces challenges but eventually returns home."

### 12. False ranges
**Watch:** from X to Y where they aren't on a meaningful scale
> "from the singularity to the grand cosmic web, from stars to dark matter" → "covers the Big Bang, star formation, and dark matter theories"

### 13. Em dash overuse
> "promoted by Dutch institutions—not by the people—even in official documents" → "promoted by Dutch institutions, not by the people, even in official documents"

### 14. Bold overuse
> "**OKRs**, **KPIs**, and the **Business Model Canvas**" → "OKRs, KPIs, and the Business Model Canvas"

### 15. Inline-header lists
> "- **User Experience:** The UX has been significantly improved" → "The update improves the interface, speeds up load times, and adds encryption."

### 16. Title Case headings
> "## Strategic Negotiations And Global Partnerships" → "## Strategic negotiations and global partnerships"

### 17. Emojis in structure
> "🚀 **Launch Phase:** Q3" → "The product launches in Q3."

### 18. Curly quotes
Replace " " with straight " "

### 19. Chatbot artifacts
**Watch:** I hope this helps, Of course!, Certainly!, Would you like, let me know, here is a...
> "Here is an overview. I hope this helps!" → [just the content]

### 20. Knowledge-cutoff disclaimers
**Watch:** as of [date], Up to my last training update, based on available information
> "While specific details are not extensively documented... it appears to have been established" → "Founded in 1994, per its registration documents."

### 21. Sycophancy
**Watch:** Great question!, You're absolutely right!, That's an excellent point!
> "Great question! That's an excellent point." → "The economic factors you mentioned are relevant here."

### 22. Filler phrases
- "In order to achieve" → "To achieve"
- "Due to the fact that" → "Because"
- "At this point in time" → "Now"
- "has the ability to" → "can"
- "It is important to note that" → [drop it]

### 23. Excessive hedging
> "could potentially possibly be argued that the policy might have some effect" → "The policy may affect outcomes."

### 24. Generic positive conclusions
**Watch:** The future looks bright, Exciting times lie ahead, major step in the right direction
> [replace with a concrete next fact]

---

## Process
1. Scan for all patterns above
2. Rewrite each problematic section
3. Result: sounds natural aloud, varied sentence structure, specific details, appropriate tone

## Output
The rewritten text. Briefly note main changes only if helpful.
