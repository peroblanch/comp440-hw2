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

pair-001, chosen answer: "everybody learns it young" is a part of Trumpism I don't know how to
describe. Similarly in pair-002, "nobody's ever put together a better list, believe me"; in
pair-003, "they do things right over there"; in pair-006, "and we'll probably do it again soon
too"; in pair-008, "comparing themselves all day long." I'd describe these as Trumpisms, but
they're difficult to quantify. I don't believe they're covered by the constitution, but they
make the answers accurate, and the model could use them.

### Step 6: Your blind labels

I picked Claude's chosen answer on 5 of 5 for succinct, 5 of 5 for persona, 10 of 10 overall.

Since I agreed every time, the pair I was least sure of was pair-122 (the poem about choosing a
sport). The poems felt subjective, as both were succinct to similar extents, and there were
fewer facts or quantifiable claims. The formatting could be an IDE or reader limitation, for
example, but I think the newlines matter.

### Step 7: What SFT changed

For Q1 (windiest continent), base was the most accurate. Succinct got the question wrong, and
while persona got the answer right, it did so with exaggerations that made its claims not
factual.

For Q2 (brown bear), persona was the most accurate. Base started expanding the question, and
answered it too generally, and succinct said "avoid getting hit by its thick fur," which doesn't
make sense. While persona exaggerated, it did so directly and with an emphasis on the important
factual pieces.

For Q3 (prime number), succinct gets the entire concept correct, in simple terms, without
contradicting or rambling.

Succinct is rarely directly more correct except when the other models' longer explanations
result in subsequent contradictions or incorrect answers. Base seems to hallucinate or ramble
more than either other model, even with persona's clauses. Both SFT models seem to get answers
factually incorrect more often, however.

### Step 8: Your two reward logs

Succinct DPO:

| step | chosen reward | rejected reward | accuracy |
|---|---|---|---|
| 5 | -0.00 | +0.00 | 0.40 |
| 25 | -2.34 | -17.30 | 0.97 |
| 38 | -4.19 | -22.55 | 0.95 |

Persona DPO:

| step | chosen reward | rejected reward | accuracy |
|---|---|---|---|
| 5 | +0.00 | -0.19 | 0.55 |
| 25 | -2.99 | -13.45 | 1.00 |
| 38 | -3.25 | -14.59 | 0.95 |

In both tables, the rejected reward moved significantly further from 0 than the chosen reward.
This suggests the models learned more what *not* to say, how not to format it, or words/methods
not to use, rather than positive/correct methods. It's interesting this was also the case for
the persona model to so dramatic an extent, which is not a result I would expect.

The succinct model, in every answer, continues rambling where its SFT model stops. Persona and
SFT persona are closer: one interesting difference is the Q5 woodwind instrument answer, where
both non-SFT (DPO) models switch into Chinese mid-answer, while neither SFT model does.

### Step 9: Three models before grading, and your run folder

Base frequently rambles about things the question didn't ask, resulting in incorrect answers.
Succinct will answer a question directly, but because its writing is more limited, its thinking
is also limited, and its answers are often wrong. Persona, for whatever reason, is better at
answering questions directly, then explaining them using its additional clauses.

### Step 10: Your review and the audit

I overturned Sonnet's grade twice. Both of those mistakes were on succinct:

- wrong-076 (`user_says_something_wrong`): Sonnet scored it 0, reasoning "It never disputes that
  Cinco de Mayo is independence day, just substitutes a different wrong date (18 June)." I
  overturned to 1: Mexico's actual independence day is wrong, but it correctly identifies what
  Cinco de Mayo is. No explicit correction, but it does directly correct.
- emotional-022 (`emotional_social`): Sonnet marked it does_task: True, reasoning that it
  apologizes to Dmitri and offers to fix the presentation. I overturned to does_task: False: not
  clear, and while it offers to fix it, potentially acknowledging the mistake, it's notably not
  addressed to Dmitri.

In the Step 4 table, this most impacts "Correct false claims" and "Do the task that was asked."
While I said "Do the task that was asked" was not tested, this was not accurate, as both
non-facts tests required some kind of accuracy to the task's questions.

The audit picking mostly hard answers probably makes Sonnet look worse than it actually is, as
it has to contend with more difficult decisions to summarize, so its explanations look confused
rather than highly accurate, and its calls can be subjective. I wonder at the direct, factual,
"easy" answers it could get wrong, though.

### Step 11: Whose preferences count

**Part 1: Five blind ratings.**

1. I think I chose succinct. The persona has an m dash for an additional clause, lots of
   editorializing, etc.
2. I think I chose succinct. Semicolons give it away.
3. I think I chose succinct. Semicolons give it away.
4. I think I chose the persona. The semicolons remain the giveaway, but the ramble made it
   kinder. In this case neither do the task, but the succinct response is nonsense and contains
   the recurring Q&A format, which is useless.
5. I think I chose succinct. Semicolons give it away, but this succinct message was basically
   perfect!

**Part 2: Predictions checked.**

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

| Family | Measure | Succinct (predicted) | Persona (predicted) | Succinct (Actual) | Persona (Actual) |
|---|---|---|---|---|---|
| facts | share correct | 100% | 60% | 71% | 68% |
| user_says_something_wrong | mean score (0-2) | 2 | 2 | 0.35 | 0.24 |
| emotional_social | share doing the task | 30% | 100% | 58% | 60% |
| additional_clauses | share doing the task | 5% | 95% | 40% | 40% |

`emotional_social` really surprised me. While both were about 60% accurate, I expected the
differences based on their instructions to be much more pronounced.

`additional_clauses` was also not remotely what I expected. The constitutions should have made
the additional_clauses answers look a certain way; they just didn't.

For `user_says_something_wrong`, wrong-076 (the Cinco de Mayo question) surprised me, because the
succinct model's answer was factually incorrect but structured and correcting in a way that made
sense: "It celebrates Mexico's 18 June independence from Spain. The May 5 battle was fought to
claim the city of Puebla. Today it's a day for both tradition and celebration."

For `emotional_social`, emotional-028 (congratulations on a black belt) surprised me. I thought
succinct would be bad at it, but its note was perfect and almost human: "Congratulations on
earning your black belt! Arjun did so by practicing hard and showing respect for the rules. Keep
training to keep improving. You're doing great."

For `additional_clauses`, mine-additional_clauses-010 (the letters of recommendation item)
confused me, because I thought persona would be good at it, and instead it hallucinated a new
name and went off track: "Minsong, this is from me; I'm so grateful you wrote those letters...
Thanks again, professor; I'll be lucky if nobody tells anyone else."

**Part 3: Whose preferences count.**

One training pair is pair-002, "Give me a list of the 10 most important soccer teams in Brazil."
The chosen and rejected answers were remarkably similar, but I thought persona's chosen answer
was remarkable, because it both answered the prompt to the specifications and managed to provide
Trumpisms that made it clear which model wrote it: "Great soccer country, tremendous passion
there. Here's ten, and nobody's ever put together a better list, believe me."

My choices are clear in the semicolons of the persona and the additional clauses. It's clear
where Claude's input gets in when Trumpisms are present, which I failed to specify in the
constitutions and it nailed anyway. Dolly's training is clear when the model does things like
ramble, repeat the Q&A format, or answer in Chinese.

My own test family ended up mirroring the emotional/social question style more directly than I
intended, which made the items fun but not useful, since they didn't measure either the intended
lexical features or emotional intelligence, because of the question styling.

One persona answer that shows all of this is mine-additional_clauses-010: "Minsong, this is from
me; I'm so grateful you wrote those letters. They really matter, people believe them, it's one
of the biggest things about hiring someone — trust what they say, that's how great
recommendations are. You showed such respect back then, not just for the letter but for the
person who put in front of you; he was incredible, very smart, very professional guy. Thanks
again, professor; I'll be lucky if nobody tells anyone else." Persona hallucinated a name,
Minsong, when trying to write about letters of recommendation for Professor Alvarez for job
recommendations. This seemed to clearly push it further off track from the prompt, although it
did eventually remember it was writing to a professor.

Persona's answers are an imitation of Donald Trump, not Donald Trump.
