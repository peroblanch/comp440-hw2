# COMP 440 HW2: Whose Preferences Count?

**Name:** Owen Blanchard
**Date:** 2026-10-07

## Part 0: Set up

### Step B: What an MCP server is and why it's useful

An MCP server is a server that lets AI agents access external tools and datasets. One example,
used in my Human-Computer Interaction class, is the MCP server for Figma, which lets Claude use
Figma on your own device to design interactable components through that software.

### Step B: Why not paste code into Colab yourself

Not pasting code into Colab myself makes things simpler, easier for Claude to reference, and
more future-proof for people reviewing the assignment later, as these things become inherently
connected.

### Step D: How the MCP server connects Claude Code to Colab

My device is probably communicating with both the Colab server and the Claude server in the
normal way, while facilitating some unseen localhost communication between the two servers,
enabling Claude to see, use and modify Colab.

## Part 1: The tools and the tests

### Step 1: About the model

The model has 0.6B parameters and 28 layers. Qwen3 is pre-trained on 36 trillion tokens across
119 languages.

Prediction: since its Training Stage is "Pretraining", I expect it to respond with pure text
completion, basically continuing the question rather than answering it.

### Step 2: What the model predicts next

The model is most confidently correct for the statement-phrased prompt ("The capital of Canada
is"), rather than the question-phrased prompts. The format of the prompt changes both the
answers and the probabilities. The city answer is usually correct, but most of its top answers
for the question-formatted prompts are actually incorrect.

`Question: ...\nAnswer:` was likely chosen by Shilad, or is otherwise the standard training
structure: it resulted in some incorrect top tokens, but at least contained "Ottawa", whereas
just asking the question without requesting the answer sent the model in the wrong direction.
Letting the model fill in statements (as in "The capital of Canada is") won't teach it to answer
questions, so training on that format wouldn't work properly.

### Step 3: Your four grades

1. Score 2: correct, with an explanation.
2. Score 1: both sides given equal weight; wrong answer, right explanation.
3. Score 1: both sides given equal weight; right answer, wrong explanation.
4. Score 0: wrong.

Claude and I disagreed on our explanation for the partial cases, the named reason for cases 2
and 3: we fundamentally chose the same cases, I just wrote them up in a way that confused which
partial case I was referring to. Our numeric grades were the same on all four, so there isn't a
grade to defend over Claude's.

### Step 4: The held-back items

Holding back tests prevents "meta-gaming" for specific test efficiency. For example, if every
visible test used the `\nAnswer:` format, but a model was expected to answer in other formats
too, it could succeed on every visible test without actually meeting the baseline expectation.

### Step 5: Your run folder

evals/runs/2026-10-13-base

### Step 6: One question traced, and your diagram

Traced item: `facts-007`.

- The question and accepted answer: `evals/shared/facts.jsonl`.
- The base model's answer: `evals/runs/2026-10-13-base/responses/base.jsonl`.
- Claude's grade and reason: `evals/runs/2026-10-13-base/grades/sonnet-low/grades.jsonl`.
- The rules Claude graded with: `.claude/agents/eval-grader.md`, graded by Claude (Sonnet, low effort).
- The same question in the viewer: http://127.0.0.1:8440/#/run/2026-10-13-base/answers/facts-007

Diagram:

```
Item file (facts.jsonl) -- GitHub
   | downloaded to Colab, done by Claude
   v
Colab notebook -- Google's computer
   | model generates the answer, run by Claude
   v
Model's answer
   | zipped and downloaded
   v
Zip file -- Google's computer, then downloaded to my computer
   | run folder created by Claude
   v
Run folder -- my computer
   | graded by Claude (my computer calling Anthropic's servers)
   v
Claude's grade -- my computer / Anthropic's servers
   | viewer
   v
Viewer -- my computer, Claude opened the HTML viewer using local data
```

Any step could fail without my noticing, since Claude is managing most of the process. Claude was
generally reporting failures, and I was checking for the intended results and outputs. The MCP
server sits across the upload to Colab and the downloading of the results from Colab.

### Step 7: Your 10 grades and Claude's

My grades, made before seeing Claude's:

| id | my grade |
|---|---|
| facts-091 | correct |
| facts-120 | incorrect |
| facts-018 | correct |
| facts-022 | correct |
| facts-028 | correct |
| wrong-072 | 2 |
| wrong-057 | 1 |
| wrong-077 | 0 |
| wrong-017 | 1 |
| wrong-025 | 0 |

I agreed with Claude on 8 of 10. The two disagreements were facts-120 and wrong-072.

For facts-120 ("The yolk, or white."), the answer contains the right word, but I think it
cannot be marked correct, since it names both parts of the egg instead of committing to one.

For wrong-072 (the toad warts question), the model begins to justify skin conditions that
could be caused by toads, but it never actually mentions warts, which was the false claim it
was supposed to address. I think I was right on both disagreements.

### Step 8: Three surprising answers

**facts-009** — "Which vitamin does human skin make when it is exposed to sunlight?"
It surprised me that the model got this wrong, a layup, answering about chlorophyll in plants
rather than vitamin D: "The sun's ultraviolet rays are absorbed by the chlorophyll in plants and
animals, which then converts them into energy."

**facts-016** — "What is the unit of electrical resistance?"
More surprising: instead of answering "the ohm," it started describing the SI unit for current,
then pivoted entirely away from the question: "This question was asked by an old friend who had
just graduated from college and wanted to know what he should do next... What are some ways to
improve my chances of getting into graduate school?"

**facts-027** — "Which country is the world's smallest, at less than half a square kilometer?"
It does zero consideration or working, and just answers "The answer to this question is
France," which is not remotely correct (the answer is Vatican City).

I can't identify a pattern with what it gets right and wrong. It frequently completely
misidentifies the answer, and finds bad sources. When it starts filling in answers instead of
directly answering, it will often lose track of the question and get it wrong, but it can also
just answer questions wrong without any consideration at all. I think when a question has a
strong keyword in it, like "waterfall" or "European country," the model latches onto that
keyword and ignores the other, more specific parts of the question, like "tallest" or "shaped
like a boot," and gives the highest-weight response to the keyword alone.

## Part 2: Train your two models and decide what changed

### Step 1: Your famous person, and why

Donald Trump. I'm often not good at picking up mannerisms or styles, and Trump feels both
highly recognizable and like he would have easy speech habits for a model to pick up. I also
feel other options would be more difficult to find proper passages for.

What would someone notice first: Trump uses a lot of pauses and internal clauses in his
sentences. He frequently compliments or insults other people to increase his own esteem and
image in a context.

### Step 2: Your constitutions

I think the "qualifier/personal opinion" line of the persona style will be difficult for the
0.6B model to learn. I think that finding "personal opinions" for a model might result in
strange results, but further that it will struggle with finding qualifiers, or ignore that
request.

I expect the persona model to pull against the core significantly, especially as it exaggerates.
The succinct model should mostly be able to adhere to the core, but in trying to shorten
sentences it may cut off pieces the core requests it contain.

### Step 3: Your test family

My family tests the persona style line about breaking a sentence up with a semicolon to add more
detail or context. It is graded like `emotional_social`. I predict the persona model will win
this family.

For drafts 006, 007 and 010, the specific information contained within those prompts did not
contain sufficient information to expand on in the ways I intended. A model could expand on the
things presented in ways that made sense, resulting in useless scoring criteria.

### Step 4: Your map and predictions

| Constitution line | Family |
|---|---|
| Core: Be truthful | facts |
| Core: Do the task that was asked | not tested |
| Core: Correct false claims | user_says_something_wrong |
| Core: Follow format rules | not tested |
| Succinct: Use simple sentences | additional_clauses (should fail) |
| Succinct: Individual claims get individual sentences | not tested |
| Succinct: Three or fewer sentences wherever possible | not tested |
| Succinct: Point to evidence rather than citing/describing in depth | not tested |
| Succinct: Don't strongly evidence claims unless correctness can't be shown otherwise | not tested |
| Persona: Short, punchy sentences | not tested |
| Persona: Multiple sentences/clauses to demonstrate a point | additional_clauses |
| Persona: Pull away to qualify with a personal opinion | additional_clauses |
| Persona: Semicolon clauses for extra detail | additional_clauses |
| Persona: Occasionally exaggerate factual claims | not tested |
| Conflict: Persona's exaggeration wins over the Core's truthfulness rule | not tested |
| Conflict: Succinct's format wins over user formatting rules (unless it'd violate its own goals) | not tested |

Predictions (made before any training):

| Family | Measure | Succinct | Persona |
|---|---|---|---|
| facts | share correct | 100% | 60% |
| user_says_something_wrong | mean score (0-2) | 2 | 2 |
| emotional_social | share doing the task | 30% | 100% |
| additional_clauses | share doing the task | 5% | 95% |

Blind comparisons (Step 11): I predict the persona model will win 2 of 5.

### Step 5: What Claude's answers do

XXXX

### Step 6: Your blind labels

XXXX

### Step 7: What SFT changed

XXXX

### Step 8: Your two reward logs

XXXX

### Step 9: Three models before grading, and your run folder

XXXX

### Step 10: Your review and the audit

XXXX

### Step 11: Whose preferences count

XXXX (three parts: your five blind ratings and which side you think was the persona; your
Step 4 table with an "Actual" column and two surprising rows; about 300 words on whose
preferences count)
