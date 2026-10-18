# Contextual Prompting: Zero-Shot and Few-Shot

Read this file when a prompt might need examples: classification, labeling, extraction,
format conversion, organization-specific conventions, or output that drifts between
runs. It covers the mechanism behind example-driven prompts, when to start without
examples, when and how to add them, how to choose and format them, and the advanced
variations built on the same idea.

## Contents
1. In-context learning: the mechanism
2. Zero-shot prompting: the baseline
3. The escalation path: zero-shot to few-shot
4. Few-shot prompting
5. Example: shift log to maintenance ticket
6. Format consistency
7. Example selection
8. Where examples go in the prompt
9. Advanced variations
10. Decisions to raise with the user
11. Failure modes to avoid
12. Pre-delivery checklist

---

## 1. In-context learning: the mechanism

Zero-shot and few-shot prompting both rely on **in-context learning**: the model shapes
its response from the text in the current request. Nothing is retrained. The model
predicts the next token based on its training and on the patterns in the prompt. When the
prompt contains input-output examples, the model treats them as the local pattern to
continue for the new input.

| | Zero-shot | Few-shot |
|---|---|---|
| The prompt contains | Instructions, context, constraints | The same, plus a small set of input-output examples |
| The model relies on | Its training plus the instructions | Its training, the instructions, and the pattern the examples show (wording, format, label use) |
| Token cost | Lowest | Higher: examples consume input tokens every request |

Three properties follow from the mechanism:
- **It is request-scoped.** The effect lasts only for this request. Future requests start
  fresh unless the examples are sent again.
- **It doesn't change the model.** Unlike fine-tuning, there are no weight updates, no
  curated training data, no compute, and no deployment overhead. That makes it fast to
  test and revise.
- **It trades against the context budget.** Too little guidance gives weak conditioning.
  Too many examples spend tokens and can crowd out important content. Balance example
  quality against the tokens available.

## 2. Zero-shot prompting: the baseline

A zero-shot prompt gives instructions and no examples, and relies on the model's training
for the rest. It is the **default starting point** for every task:
- **Maximum flexibility**: no examples narrow the model's interpretation.
- **Lowest token cost**: nothing is spent on examples.
- **Enough accuracy for well-understood tasks**, where the model's training covers the
  domain thoroughly.

Zero-shot results become inconsistent when the task depends on things the model can't
know: **proprietary categories, organization-specific conventions, house formats, or
specialized knowledge** absent from training data. Few-shot examples earn their cost by
closing that gap.

## 3. The escalation path: zero-shot to few-shot

1. **Start zero-shot.** Ask for the task without examples.
2. **Evaluate the outputs** on a varied test set for accuracy, consistency, format
   adherence, and edge cases.
3. **Are the outputs reliable and aligned?**
   - Yes → stay zero-shot.
   - No → **add a small number of high-quality examples** that cover the key cases and
     edge cases and show the exact desired format.
4. **Re-evaluate** with the same criteria and the same test inputs, and adjust the
   examples if needed.

Escalate only when examples meaningfully improve the results. Few-shot prompting
improves reliability but adds token cost and complexity to every request.

**Applying this when building a prompt.** You usually can't run the baseline yourself, so
choose the starting point from the signals:
- If the task is well understood and uses common categories, deliver a zero-shot prompt,
  and offer a few-shot version for the user to switch to if testing shows drift.
- Deliver few-shot from the start when the task depends on proprietary labels, house
  conventions, or a strict format that downstream systems parse, or when the user already
  reports drift. Explain in the design notes why examples are needed.

## 4. Few-shot prompting

Few-shot prompting adds input-output examples when the baseline drifts, misses domain
conventions, or needs more consistent output. It is most effective for tasks that reward
consistency: **classification, structured extraction, format conversion, and
domain-specific labeling**. Open-ended generation usually benefits less, because the
examples narrow the range of output you actually want.

Use it when:
- Zero-shot output is inconsistent.
- The task uses categories or formats specific to the user's domain.
- The output must follow a precise structure for downstream processing.
- The task depends on conventions the model wouldn't infer from instructions alone.

**Use one or two examples.** One example establishes the format. A second adds a
contrasting case or an important boundary. If evaluation shows gaps, test a different
pair of examples rather than lengthening the list. Long static example lists overfit the
prompt to narrow patterns and add tokens to every request.

**For classification, pair definitions with examples.** The instructions or system prompt
define what each label means. The examples show how to apply the labels. Definitions
without examples leave the boundaries fuzzy, and examples without definitions leave the
labels unexplained.

## 5. Example: shift log to maintenance ticket

A hotel's front desk passes guest-reported problems to the maintenance team. The team's
convention is a four-field ticket with house priority codes: P1 for safety risks or
active damage, P2 for issues affecting a guest's stay, P3 for comfort issues. The model
can't know those codes.

Zero-shot prompt:
~~~
# Role
You are a hotel front-desk operations assistant.

# Instructions
Write a maintenance handoff note for the issue in the target input.

# Target Input
```text
Guest in 412 reports water dripping from the bathroom ceiling since 6am, towels on the
floor. Room 312 below is occupied. Guest is asking to change rooms.
```

# Output Format
3–5 sentences covering the issue, urgency, responsible team, and next step.
~~~
Typical result: an accurate paragraph that calls the leak "urgent" and suggests
"maintenance staff". The engineering team still has to pull out the location, assign
their own priority code, and work out who owns the job.

Few-shot prompt:
~~~
# Role
You are a hotel front-desk operations assistant.

# Instructions
Write a maintenance ticket for the target input. Follow the reference examples exactly
for structure, level of detail, and priority codes: P1 = safety risk or active damage,
P2 = affects the guest's stay, P3 = comfort issue the guest can tolerate.

# Reference Examples
## Example 1
Report: AC in 207 blowing warm air; guest says it's fine for tonight.
Location: Room 207
Issue: AC not cooling
Priority: P3
Team: Engineering
Next step: Check the unit before 10:00 tomorrow
## Example 2
Report: Smoke smell in the third-floor corridor near the ice machine.
Location: Floor 3 corridor, ice machine
Issue: Smoke odor near electrical equipment
Priority: P1
Team: Engineering + Duty Manager
Next step: Inspect immediately and cut power to the ice machine
---
# Target Input
```text
Guest in 412 reports water dripping from the bathroom ceiling since 6am, towels on the
floor. Room 312 below is occupied. Guest is asking to change rooms.
```

# Output Format
Five lines: Location, Issue, Priority, Team, Next step, in that order.
~~~
Typical result: *Location: Room 412 (check 312 below) · Issue: Active ceiling leak in
the bathroom · Priority: P1 · Team: Engineering + Housekeeping · Next step: Isolate the
water source now, inspect 312, and move the guest.* The examples add no facts about the
target. They teach the team's schema and priority rules, at a small token cost.

## 6. Format consistency

Every example must use **the same structure and the same value shape**. If the field
names match but one example's value is a bare label and another adds an explanation, the
model gets two conflicting patterns. Across runs, it will sometimes follow one and
sometimes the other.

Inconsistent examples:
~~~
# Instructions
Categorize each expense using the pattern shown in the reference examples.

# Reference Examples
## Example 1
Expense: "Uber to airport, $42"
Category: Travel
## Example 2
Expense: "Team lunch with client, $86"
Category: Meals. This counts as a client meal because a customer attended, so it may
need a note for the finance team.
---
# Target Input
Expense: "Monthly Figma subscription, $45"
~~~
Typical result: *Category: Software*, sometimes followed by a sentence of explanation and
sometimes not. An expense system that expects one line breaks on the runs that include
the explanation.

Consistent examples (same prompt; only Example 2 changes):
~~~
## Example 2
Expense: "Team lunch with client, $86"
Category: Meals
~~~
Typical result: *Category: Software* on a single line, every run.

Rules:
- Keep the same field names, field order, and value shape in every example.
- The examples' outputs must match the Output Format exactly. If they differ, the model
  has to choose between them.
- Label examples consistently (`## Example 1`, `## Example 2`) and separate them from the
  target with `---` and a `# Target Input` heading.

## 7. Example selection

Format controls how examples look. Selection controls what they represent. With only one
or two examples, every one has to earn its place:
- **Content diversity.** Don't spend both examples on the same easy case. Use examples
  that show different input patterns.
- **Edge-case coverage.** When the task has high-risk ambiguity, use one example for the
  boundary case.
- **Label coverage trade-offs.** Two examples can't cover every label in a three-label
  task. Choose the common case and the label most likely to be missed, then test the
  omitted labels separately.
- **Realistic input quality.** If production inputs are messy (typos, fragments, mixed
  formats), the examples should be messy too.
- **Never reuse the target as an example.** An example identical or nearly identical to
  the target lets the model copy the answer, so the test proves nothing.

**Example: document sensitivity labels** (Public, Internal, Confidential).

Narrow examples, both from the easy Public case:
~~~
# Reference Examples
Document: "Office holiday schedule 2026"
Classification: Public
Document: "Careers page copy – engineering roles"
Classification: Public
---
# Target Input
Document: "Salary bands by role and level – Q3 review"
Classification:
~~~
Typical risk: both examples show only Public, so the model is pulled toward a lower
sensitivity label for a document that should clearly be Confidential.

Diverse, boundary-focused examples:
~~~
# Reference Examples
Document: "Office holiday schedule 2026"
Classification: Public
Document: "Disciplinary notes – J. Ortiz, March"
Classification: Confidential
---
# Target Input
Document: "Salary bands by role and level – Q3 review"
Classification:
~~~
Typical result: Confidential. One example anchors the common case and the other anchors
the high-risk label, while the target is different from both. Internal isn't
represented, so test it separately with a document such as "All-hands slides – Q3
priorities".

**Ordering.** Models can be biased toward the pattern of the last example. Swap the order
of the examples across evaluation runs to check that the results hold either way.

## 8. Where examples go in the prompt

**Combined prompt.** Give the examples their own `# Reference Examples` section. Place it
after the instructions and before the target input, followed by a `---` separator and a
`# Target Input` section. Tell the model in the instructions to follow the examples'
pattern, and say what to follow: structure, level of detail, labels.

**Split system and user prompts.**
- Put **label definitions, instructions, and the Output Format in the system prompt**.
- Put **examples in the user prompt by default**, in a `# Reference Examples` section
  before the delimited target input.
- If the examples never change between requests and prompt caching matters, they can
  instead sit at the end of the system prompt. Examples that change per request, such
  as retrieved examples (section 9), always go in the user prompt.

**Templates.** Static examples are part of the template's stable core. Dynamic examples
are a parameter (`{{retrieved_examples}}`) in a delimited block.

## 9. Advanced variations

All of these refine the same in-context learning idea. Use them only when the basic one-
or two-example approach falls short.

**Contrastive few-shot.** Pair a correct example with an incorrect one, so the model
learns both what to do and what to avoid. This is useful when a specific mistake keeps
recurring.
~~~
Correct:
Report: Smoke smell in the third-floor corridor.
Priority: P1
Incorrect:
Report: Smoke smell in the third-floor corridor.
Priority: P3
Why incorrect: possible fire risk is always P1, even if no guest has complained.
~~~
Label the incorrect example clearly and explain why it is wrong. Otherwise the model may
copy it.

**Few-shot chain of thought.** The examples include brief reasoning steps before the
answer, so the model shows the same reasoning for multi-step decisions.
~~~
Request: Return of unopened headphones bought 20 days ago.
Reasoning: Within the 30-day window; item unopened; no exclusions apply.
Decision: Approve
~~~
This increases output tokens. Use it when the decision depends on several rules applied
in order, and keep the final answer in a fixed last field so code can still parse it. If
the model already has reasoning enabled, test whether the reasoning traces still add
value.

**Few-shot with retrieved examples.** At runtime, retrieve the most relevant labeled
examples from a corpus (for example, by similarity to the target input), and inject them
through a delimited parameter. This scales few-shot prompting to large, varied input
sets. Keep the retrieved examples in one consistent format, and cap them at a small
number.

## 10. Decisions to raise with the user

When this file applies, these decisions usually can't be assumed. Ask each with a
recommendation, following SKILL.md's ask-before-building step:
- **Zero-shot or few-shot?** Recommend zero-shot for well-understood tasks with common
  categories. Recommend few-shot when the task uses proprietary labels, house conventions,
  or strict parsed formats, or the user reports drift (section 3).
- **Where do the examples come from?** Recommend real examples from the user's own data.
  If none are available, offer drafted examples, but ask the user to confirm the labels
  and conventions, because only they know the correct answers for their domain.
- **Which cases should the examples cover?** For classification, propose the common label
  plus the label most likely to be missed, and name the labels that will need separate
  testing.
- **Static or retrieved examples?** Recommend static examples unless the input variety
  is large and a labeled corpus exists.
- **Where should the examples live?** In split prompts, recommend the user prompt by
  default, or the system prompt if the examples are static and caching matters
  (section 8).

## 11. Failure modes to avoid

- **Few-shot by reflex.** Adding examples to a task zero-shot already handles well.
- **Too many examples.** Long lists that overfit and add tokens to every request. Test a
  different pair instead.
- **Inconsistent example shapes.** Mixed value formats, extra prose in one example, or
  fields in a different order.
- **Examples that contradict the Output Format.**
- **Two examples of the same easy case**, leaving the risky label unanchored.
- **The target reused as an example**, which makes the evaluation meaningless.
- **Clean examples for messy production data.**
- **Example facts leaking into output.** The model copies a detail from an example
  ("Room 207") into the target's answer. Keep example details clearly distinct from
  likely targets, and check for this in testing.
- **Invented labels for a proprietary taxonomy** without the user verifying them.
- **Unlabeled incorrect examples** in contrastive prompts, which the model may imitate.

## 12. Pre-delivery checklist

Before returning a prompt, confirm:
- [ ] The choice between zero-shot and few-shot matches the signals in section 3, and
      the user confirmed it.
- [ ] There are one or two examples, unless the user asked for more after evaluation.
- [ ] Every example uses identical fields, order, and value shape, and matches the
      Output Format.
- [ ] The examples cover distinct cases, including the riskiest boundary, and the
      uncovered labels are named for separate testing.
- [ ] No example duplicates or nearly duplicates the target input.
- [ ] The example input quality matches real production input.
- [ ] Examples sit in a labelled `# Reference Examples` section, separated from the
      `# Target Input`.
- [ ] For classification, the labels are defined in the instructions or system prompt.
- [ ] The design notes say to swap example order during testing, and to try a different
      pair before adding more.
