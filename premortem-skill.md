---
name: premortem
description: "Run a premortem on any plan, launch, product, hire, strategy, or decision. Assumes it already failed 6 months from now and works backward to find every reason why. Produces a revised plan with blind spots exposed. MANDATORY TRIGGERS: 'premortem this', 'premortem my', 'run a premortem', 'what could kill this', 'future-proof this', 'stress test this plan', 'what am i missing here', 'find the blind spots'. STRONG TRIGGERS: 'what could go wrong', 'am i missing anything', 'poke holes in this', 'where will this break', 'devil's advocate this'. Do NOT trigger on simple feedback requests, factual questions, or LLM Council requests. DO trigger when someone has a plan or commitment where the cost of being wrong is high."
---

# Premortem

A premortem is the opposite of a postmortem. Instead of figuring out what went wrong after something fails, you imagine it already failed and figure out why before you start.

The method comes from psychologist Gary Klein. He published it in Harvard Business Review. Daniel Kahneman called it his single most valuable decision-making technique. Google, Goldman Sachs, and Procter & Gamble all use it before major decisions.

The core insight: when you ask people "what could go wrong?" they give you cautious, hedged answers. When you say "this already failed, tell me why," their brains switch into narrative mode and generate way more specific, creative, honest reasons.

The reason this matters for AI-assisted decisions: Claude defaults to agreeable, optimistic responses. If you ask "is this a good plan?" it will find reasons to say yes. The premortem breaks this pattern by forcing the frame into "this is dead, explain how it died."

---

## when to run a premortem

Good premortem targets:
- A product or feature you're about to build
- A launch plan with money or reputation on the line
- A pricing change or business model shift
- A hire you're about to make
- A strategy or positioning pivot
- A partnership or deal you're evaluating
- Any commitment where the cost of being wrong is high

Bad premortem targets:
- Vague ideas with no concrete plan yet
- Questions with one right answer
- Requests for creative feedback on a draft
- Decisions that are already made and irreversible

---

## context gathering (the minimum bar)

A premortem is only as good as the context it runs on. Before running the premortem, hit the minimum context threshold.

### step 1: scan for existing context
Look for context already available in the conversation, memory, or referenced files. Don't spend more than 30 seconds.

### step 2: evaluate context sufficiency
You need three things:
1. What is it? (the thing being premortemed)
2. Who is it for / who does it affect?
3. What does success look like?

### step 3: fill gaps conversationally
If missing pieces, ask for the most important one first. One question at a time. Conversational, not interrogative.

---

## how a premortem session works

### step 1: set the frame
"It's 6 months from now. The plan has failed. We're looking back to understand what went wrong."

### step 2: generate failure reasons (raw premortem)
Run as a single comprehensive analysis. No prescribed categories. Each reason: specific, grounded in actual details, a genuine threat.

### step 3: deep-dive agents (one per failure reason, all in parallel)
Each agent receives the failure reason and produces:
1. THE FAILURE STORY: 2-3 paragraph narrative of how it played out
2. THE UNDERLYING ASSUMPTION: one sentence
3. EARLY WARNING SIGNS: 1-2 observable signals

Keep under 300 words. Be direct. Don't hedge.

### step 4: synthesis
PREMORTEM REPORT:
1. The Most Likely Failure
2. The Most Dangerous Failure
3. The Hidden Assumption
4. The Revised Plan (concrete changes)
5. The Pre-Launch Checklist (3-5 items)

### step 5: generate the premortem report
Save `premortem-report-[timestamp].html` — single self-contained HTML, dark background, synthesis at top, one card per failure reason.

### step 6: save the transcript
Save `premortem-transcript-[timestamp].md` with full context, raw failure reasons, agent deep-dives, and synthesis.

---

## important notes

- Always spawn all failure agents in parallel.
- Always set the premortem frame explicitly. "This has already failed" is the mechanism that makes this work.
- Be comprehensive but not padded.
- The synthesis is the product.
- Don't sugarcoat.
- The revised plan must be concrete.
- Respect the minimum context threshold.
- This is not the LLM Council.
