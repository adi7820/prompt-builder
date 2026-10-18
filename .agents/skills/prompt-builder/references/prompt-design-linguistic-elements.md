# Prompt Design: Linguistic Elements

Read this file after the prompt's structure is in place. Structure decides which sections
a prompt has. Linguistic elements decide how the words inside those sections are written:
how specific the requirements are, how explicit the instruction style is, and how much
wording is used. Use this file both when writing a new prompt and when diagnosing output
that is generic, exploratory when a decision was needed, or no better than a one-line
request would have produced.

## Contents
1. The three primary elements
2. First decide: exploration or execution
3. All three at work: one task, two wordings
4. Precision
5. Directness
6. Brevity
7. Secondary refinements
8. Applying the elements when building a prompt
9. Diagnosing bad output
10. Failure modes to avoid
11. Pre-delivery checklist

---

## 1. The three primary elements

| Element | Controls | Too low looks like | Too high looks like |
|---|---|---|---|
| **Precision** | How specifically requirements are stated, and how narrowly the model may interpret them | Plausible but generic output, unrequested additions, important details missing | A long, rigid prompt; conflicting details that produce filler or failure; no room to explore |
| **Directness** | Whether the instruction is implicit (asks around the goal) or explicit (states the goal) | A discursive essay or "it depends" when a decision was needed | Forcing a premature verdict when the task needed open reasoning |
| **Brevity** | How much wording is used to get the needed output | Underspecified, so output turns generic | Filler, politeness, and restated instructions that add cost and widen the output's scope |

Each element points to a different cause of bad output. That is what makes them useful:
when output falls short, you adjust the one element responsible instead of rewriting by
trial and error.

## 2. First decide: exploration or execution

The right setting for each element depends on what the output is for. Decide this before
wording anything.

| | Execution (decide, build, ship) | Exploration (brainstorm, research, open options) |
|---|---|---|
| Precision | High: name the items, facts, quantities, and edge cases | Moderate to low: name the goal and the boundaries, and leave the rest open |
| Directness | Explicit: an imperative verb naming the deliverable | Implicit or question form is fine: invite reasoning |
| Brevity | Brief but complete: every concrete requirement stays in | Brief: a short framing leaves room for range |

Most prompts the user asks for are execution prompts. If the user's request mentions
brainstorming, ideas, options, or "help me think through", treat it as exploration.

For larger projects, one prompt doesn't have to serve both phases. Use moderate Precision
first to surface directions. Then raise Precision to turn the chosen direction into
something implementable. When the user's workflow fits this pattern, offer two prompts,
one per phase.

## 3. All three at work: one task, two wordings

Both versions below ask for the same thing in a similar structure. Only Precision,
Directness, and Brevity change.

**Version A: low Precision, hedged, wordy**
~~~
I was hoping you could help me come up with a database schema for a gym class booking
app. Something clean and well-designed would be great. I'm not really sure which tables
are needed, so whatever you think makes sense is fine.
~~~
Typical result: a plausible schema padded with tables nobody asked for (payments,
reviews, equipment). The rules that matter, such as capacity and double bookings, are not
enforced.

**Version B: high Precision, explicit, efficient**
~~~
Design a relational database schema for a gym class booking app. Include tables for
members, instructors, classes, class sessions, and bookings. Enforce these rules: each
session has a fixed capacity; a member can book a given session only once; cancellations
within 2 hours of the session start are recorded as late. List each table with its
columns, data types, primary key, and foreign keys.
~~~
Typical result: exactly the five tables, a capacity column, a uniqueness rule on member
plus session, and a field that records late cancellations. It is ready to review.

Version A leaves the model to decide the scope, so it fills the gap with generic
supporting systems. Version B narrows the result with specific requirements, states the
task decisively, and spends every sentence on a requirement.

## 4. Precision

Precision is how specifically you state requirements and how narrowly you want the model
to interpret them. Higher Precision narrows the range of plausible outputs and makes them
more consistent. Lower Precision leaves room for variation. That helps when brainstorming,
but it usually means more revision.

Match Precision to the task. Use high Precision when consistency, accuracy, or fit with an
existing system matters. Use lower Precision when exploration matters more than control.

**Ways to raise Precision:**
- Name the parts that must appear: entities, sections, actors, fields, components.
- State the facts the output must contain instead of letting the model fill them in.
- Put numbers on quantities: word ranges, counts, limits, dates.
- List the edge cases, branches, or failure states the output must cover. Low-Precision
  prompts lose these first.
- Replace vague quality words with concrete specifications (table below).

| Vague wording | Concrete replacement |
|---|---|
| "clean", "well-designed", "polished" | The specific parts, labels, or rules that would make it so |
| "comprehensive", "cover everything important" | The list of items to cover |
| "short", "concise" | A word or sentence range |
| "professional tone" | The audience plus the register ("for finance leads; formal, no slang") |
| "include the key details" | The details themselves |
| "handle errors properly" | The error cases and the expected behavior for each |

**Costs.** More detail means a longer prompt and more tokens. Requirements that are
unclear or contradict each other can produce filler or outright failure, so precise
requirements must also agree with each other. Clear Instructions, specific Context, and an
explicit Output Format all reinforce Precision.

**Sampling.** Output variance also depends on sampling settings. For the same prompt, a
higher temperature usually increases variance and a lower one reduces it. If the user
controls the temperature and needs consistent output, mention this.

### 4.1 Example: store closure email

Low Precision:
~~~
Write a closure notice.
~~~
Typical result: a generic "we're closing, thank you for your patience" message. It names
no store, no date, and nothing the customer should do.

High Precision:
~~~
Write a 90–120 word email to loyalty-program members announcing that the Elm Street
branch of Corner Books closes on March 14. Include: March 14 is the last trading day;
gift cards and loyalty points stay valid at the Riverside branch (2 miles away) and
online; existing pre-orders will ship free. Use a warm, plain tone with no sales
language.
~~~
Typical result: usable as written. Every fact a customer needs is present, and the length
and tone fit the channel.

### 4.2 Example: signup form test cases

Low Precision:
~~~
Write test cases for a signup form.
~~~
Typical result: the happy path plus a generic "invalid email" case. It misses the cases
that actually break in production.

High Precision:
~~~
Write test cases for a signup form with fields email, password, and date of birth.
Cover: valid signup; duplicate email; password under 10 characters; password with no
number; birth date making the user under 16; each required field left empty; an email
differing only in letter case from an existing account (must count as a duplicate). For
each case, give the input, the expected result, and the error message shown.
~~~
Typical result: one case per listed scenario, including the case-insensitive duplicate
that a generic list would miss.

### 4.3 When to use each level

| Choose high Precision when | Choose low Precision when |
|---|---|
| Output must fit existing systems, formats, or workflows | You are exploring options or new directions |
| Several people need consistent results | Variation is useful and a human will review |
| Clear success criteria already exist | Success criteria are still emerging |
| Revision cycles are costly or time-constrained | Iteration is cheap and part of the process |
| Accuracy and implementation readiness matter most | Novelty and idea generation matter most |

## 5. Directness

Directness controls whether the instruction is **implicit** (it describes the situation
and asks around the goal) or **explicit** (it states the goal and names the deliverable).
It is a separate control from Precision. Precision is about how specific the content is.
Directness is about how the request is phrased. It has nothing to do with being forceful
or rude.

Compare asking for the salt. "Could someone maybe pass that?" can be read several ways.
"Pass me the salt" is explicit and easy to follow.

Implicit prompts tend to produce discursive responses that weigh considerations. Explicit
prompts tend to produce structured decisions and actions. This is a reliable tendency, not
a rule. Recent models infer intent from implicit requests better than earlier ones, but
explicit instructions remain the safer default when you need consistent, ready-to-act
results.

**Ways to make an instruction explicit:**
- Lead with an imperative verb that names the deliverable: Decide, Choose one, Rank,
  Recommend, List, Write.
- Name the components of the answer: "State the decision, the main risk, and the first
  step."
- Remove hedges: "I was wondering if", "could you maybe", "share your thoughts on",
  "whatever seems right".

Use implicit phrasing on purpose when you want the model to reason through options or
surface considerations conversationally. Use explicit phrasing when the response must
centre on a decision, ownership, or next actions.

### 5.1 Example: pricing decision

The facts stay the same; only the instruction style changes.

Implicit:
~~~
We're wondering whether raising the Pro plan from $12 to $15 a month makes sense.
Churn is steady at 3%, two competitors charge $16–18, but support tickets about slow
sync are up this quarter. Could you share how you'd think about it?
~~~
Typical result: a balanced discussion of the trade-offs that ends near "it depends".

Explicit:
~~~
Decide whether to raise the Pro plan from $12 to $15 a month next quarter. Churn is
steady at 3%, two competitors charge $16–18, and support tickets about slow sync are up
this quarter. State the decision, the condition that would change it, and the first step
to take.
~~~
Typical result: a committed answer, such as raising the price once the sync fix ships,
with a clear trigger for reversing it and one concrete first step.

### 5.2 Example: choosing one approach

Implicit:
~~~
We're looking at ways to shorten how long new support agents take to get up to speed.
Any thoughts on shadowing, a recorded training library, or a scripted practice sandbox?
~~~
Typical result: a paragraph of pros and cons for each option, with no choice made.

Explicit:
~~~
Choose one approach to shorten new support agents' ramp-up time this quarter:
shadowing, a recorded training library, or a scripted practice sandbox. Give the final
choice, one key risk, and the first rollout step.
~~~
Typical result: three short labelled lines: the choice, the risk, and the step.

Explicit prompts can still return uncertainty when the evidence is thin. Implicit prompts
can still return concrete actions. Directness shifts the odds; it doesn't fix the outcome.

## 6. Brevity

Brevity is how much wording the prompt uses. Aim for the **minimum effective length**:
the shortest prompt that still carries every concrete requirement. A long prompt helps
only when the extra length adds constraints, facts, or success criteria. Otherwise it
costs tokens and changes nothing, and that cost compounds when a prompt runs many times in
a pipeline.

Wordiness can also widen the output's scope. Words like "comprehensive", "thorough", or
"capture everything" invite extra sections the user didn't need.

Shorter is not always better. Routine tasks with obvious requirements need little detail.
Specialized or high-stakes tasks need the detail that prevents rework.

| Cut | Keep |
|---|---|
| Politeness ("I'd really appreciate it if...") | The task verb and its object |
| Hedges ("maybe", "if possible", "I'm not sure but") | Facts the output must contain |
| Restated or self-cancelling instructions ("be thorough but concise") | Quantities, limits, and named items |
| Generic quality adjectives ("polished, premium, engaging") | Edge cases and required branches |
| Explanations of why you're asking that don't change the answer | Audience details that change the answer |

Start brief, then add only the details that close gaps you actually observe.

### 6.1 Example: meeting notes summary

Wordy:
~~~
I would be really grateful if you could carefully go through the notes from our planning
meeting below and write up a thorough summary that captures everything important. Please
be sure to include the decisions we made, who is doing what, and anything still open.
Try to be comprehensive but also keep it concise, since this is going to the whole team.

"Agreed to move launch to May 6. Priya to finalize pricing page by Apr 20. Dev blocked on
payment sandbox access; Leo chasing vendor. Still undecided: whether to include the
annual plan at launch."
~~~
Typical result: correct, but longer than needed. "Thorough" and "comprehensive" invite
background paragraphs and recommended next steps that nobody asked for.

Brief:
~~~
Summarize these planning-meeting notes as decisions, action items with owners, and open
questions:

"Agreed to move launch to May 6. Priya to finalize pricing page by Apr 20. Dev blocked on
payment sandbox access; Leo chasing vendor. Still undecided: whether to include the
annual plan at launch."
~~~
Typical result: the same three categories in compact form, at a fraction of the prompt
length.

### 6.2 Example: landing page hero copy

Wordy:
~~~
Please write a really polished, high-quality, professional and engaging hero section for
our landing page that feels modern and premium and will really resonate with visitors.
It's for our habit-tracking app. It should include a headline, a subheadline, and a call
to action, and please make it compelling.
~~~
Typical result: decent copy, but the adjectives constrained nothing. The headline length,
the featured benefit, and the button text are all left to chance.

Brief:
~~~
Write a hero section for a habit-tracking app's landing page: a headline under 8 words,
a one-sentence subheadline naming the streak-reminder feature, and a call-to-action
button label.
~~~
Typical result: copy that fits the layout and features the right benefit. If an extra
element matters, such as a social-proof line, name it as a specific requirement. More
adjectives won't produce it.

## 7. Secondary refinements

Precision, Directness, and Brevity are the main controls. Use the refinements below when
the output is functionally correct but misaligned in tone, framing, or format. Modern
models need them less than earlier ones did, but they still help in specialized cases.

**Positive versus negative framing.** Prefer stating what to do. Use negative
instructions sparingly and precisely: a specific prohibition, such as "do not include
code blocks", is fine and testable. A prompt built mostly from "don't" lists can draw
attention to exactly the things it forbids. Where possible, pair a prohibition with the
desired alternative: "Use plain language; spell out acronyms" works better than "No
jargon".

**Tone and register.** Models often infer tone, but stating it improves consistency.
Specify tone when the audience, documentation standard, or level of formality isn't
obvious, or when consistency across runs matters. Describe tone through the audience and
register ("for first-time customers; warm, plain, no slang"), not with bare adjectives.

**Salience words.** Words like "critical", "urgent", or "high-risk" change what the
model treats as important. Use them only where you intend that emphasis. Leave them out
when you want neutral, evidence-first reasoning, because they can push the model toward
alarm or overcaution.

**Visual content framing.** For diagrams, mockups, and other visuals, vague terms such as
"clean" or "production-ready" are especially unreliable. Specify the components, flows,
boundaries, and labels instead, and prefer positive specifications to lists of things to
avoid.

**Question versus statement form.** Questions suit brainstorming and research. Statements
suit implementation planning and decision support. This follows the exploration/execution
choice in section 2.

## 8. Applying the elements when building a prompt

Once the structure is drafted, make one pass through the wording in this order:

1. **Set the dials.** Decide exploration or execution (section 2).
2. **Precision pass.** Replace every vague quality word with a concrete specification.
   Add any missing facts, quantities, named items, and edge cases that the task's
   consistency depends on. Check that no two requirements conflict.
3. **Directness pass.** For execution prompts, make the Instructions start with an
   imperative verb that names the deliverable, and remove hedges. For exploration
   prompts, keep an open or question form on purpose.
4. **Brevity pass.** Delete politeness, hedges, restated instructions, and adjectives that
   don't constrain anything. Every remaining sentence should set the task, add a fact or
   constraint, or define the output.
5. **Refinements.** Add tone or register only if it isn't obvious. Rewrite heavy negative
   phrasing as positive specifications. Check that salience words are intended.

When rewriting a user's existing prompt, report the linguistic changes in one line each,
such as "replaced 'professional' with audience and register", so the user can learn the
pattern.

## 9. Diagnosing bad output

When a user reports that a prompt's output is wrong, match the symptom to an element and
change only that element.

| Symptom | Likely cause | Adjustment |
|---|---|---|
| Plausible but generic; could apply to any company or system | Precision too low | Add the specific facts, names, and quantities |
| Includes things nobody asked for, or misses critical cases | Precision too low | Name the required parts and list the edge cases |
| Discussion or "it depends" when a decision was needed | Directness too low | Lead with an imperative ("Decide", "Choose one") and name the answer's parts |
| A forced verdict when the user wanted options explored | Directness too high for the goal | Switch to question form; ask for options and trade-offs |
| A long prompt that does no better than a short one | Brevity: filler instead of specification | Cut the filler; turn adjectives into concrete requirements |
| Output longer or broader than needed | Brevity: scope-widening words ("comprehensive", "thorough") | Remove them and state the exact sections needed |
| Filler or contradictory output | Precision: conflicting requirements | Resolve the conflict; drop the lower-priority requirement |
| Correct content, wrong tone or framing | Secondary refinements | Specify audience and register; reframe negatives; check salience words |

## 10. Failure modes to avoid

- **Adjectives standing in for requirements.** "Clean", "robust", "engaging" with nothing
  concrete behind them.
- **Hedged execution prompts.** "Could you maybe share thoughts on..." when the user needs
  a decision.
- **Self-cancelling instructions.** "Be thorough but concise" or "detailed but short".
  Give a range instead.
- **Over-specifying exploration.** Locking down every detail of a brainstorm until no
  room is left for range.
- **Precision without consistency.** Many specific requirements that contradict each
  other.
- **Politeness padding.** Tokens spent on courtesy, repeated in every run of a template.
- **Negative-heavy prompts.** Long "don't" lists without the desired alternative.
- **Accidental salience.** "Critical" or "urgent" in a prompt that needs neutral analysis.

## 11. Pre-delivery checklist

Before returning a prompt, confirm:
- [ ] The exploration-or-execution choice is made, and the Precision and Directness
      levels match it.
- [ ] No vague quality word stands in for a requirement.
- [ ] The facts, quantities, named items, and edge cases the task depends on are stated.
- [ ] Requirements are consistent with each other.
- [ ] Execution prompts start with an imperative verb that names the deliverable, with
      no hedges.
- [ ] There is no politeness filler, restated instruction, or self-cancelling pair.
- [ ] Every sentence sets the task, adds a fact or constraint, or defines the output.
- [ ] Negative instructions are specific, and paired with an alternative where possible.
- [ ] Tone is specified when it isn't obvious; salience words appear only where emphasis
      is intended.
