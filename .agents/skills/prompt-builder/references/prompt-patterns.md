# Prompt Patterns

Read this file after the prompt's structure and wording are settled. Structure decides the
sections, wording decides how they are written, and patterns decide the **strategy**: how
the model should approach the task. A pattern can make the model adopt an expert
perspective, follow a fixed output schema, read the input twice, critique a draft, or
question it from outside. Patterns transfer across domains. Once you know when
Self-Reflection helps, you can apply it to a launch plan, a contract, or a lesson plan
without changing the method.

## Contents
1. The eight patterns at a glance
2. Choosing patterns
3. Role Assignment
4. Delimiter Pattern
5. Template Pattern
6. Repetition Pattern
7. Self-Reflection Pattern
8. Inversion Pattern
9. Refinement Pattern
10. Comment-Driven Generation
11. Delivering multi-stage workflows
12. Failure modes to avoid
13. Pre-delivery checklist

---

## 1. The eight patterns at a glance

| Category | Pattern | What it does | Turns |
|---|---|---|---|
| Structural: shape how the model approaches the task and keep results consistent | Role Assignment | Has the model adopt a specific, realistic expert role | Single |
| | Delimiter | Organizes several input sources into labelled, fenced sections | Single |
| | Template | Fixes the output to a repeatable schema of named fields | Single |
| | Repetition | Sends the identical prompt twice so the model attends to details again | Single |
| Contextual: surface insights and outside perspectives that reveal blind spots | Self-Reflection | Has the model critique its own output | Multi (follows a draft) |
| | Inversion | Has the model generate questions that challenge the content | Multi (questions, then the user answers) |
| Transformational: improve existing content based on analysis | Refinement | Revises content using the preceding critique or answers | Multi (always follows analysis) |
| | Comment-Driven Generation | Resolves several reviewer comments anchored to specific passages in one pass | Multi |

## 2. Choosing patterns

Match the pattern to the problem. Add only what the task needs; stacking patterns on a
simple task adds cost without adding quality.

| The user's problem | Pattern(s) |
|---|---|
| Output is competent but generic, and lacks domain judgment | Role Assignment |
| The prompt combines several input sources, or others will maintain the template | Delimiter |
| The same kind of output is produced repeatedly, compared, or fed into a system | Template |
| The model must pick details out of a list or long context, reasoning is off, and one answer matters | Repetition |
| A draft exists and the user wants the model to find its gaps | Self-Reflection → Refinement |
| A draft exists and the user wants to anticipate objections from other people | Inversion → user answers → Refinement |
| Reviewers left specific comments on specific passages | Comment-Driven Generation → (optional Inversion) → Refinement |

Common combinations:
- **Role Assignment fits almost every prompt** and combines with any other pattern.
- **Delimiter plus Template** is the standard pair for reusable templates: organized
  inputs, predictable outputs.
- **Inversion plus Role Assignment** work independently. The role sets each questioner's
  domain perspective; Inversion makes the output questions rather than analysis.
- **Repetition** is independent of everything else, because it changes only how the input
  is sent.

## 3. Role Assignment

Role Assignment tells the model to adopt a specific persona, expertise, or voice before
starting the task. It changes more than style. The role shifts the model's priorities, so
it decides what information matters to the person who would actually do this work. Use it
in almost every prompt: it costs one sentence and noticeably improves domain judgment. It
matters most when the task needs expertise the user lacks, a specific audience, or
professional credibility.

**Writing the role.** Choose a real-world role that would actually do this task, and make
it specific:
- **Profession or expertise**: "in-house recruiter", "pediatric nurse educator".
- **Domain background**: industry, organization type, or setting.
- **A credibility marker tied to outcomes**: experience or a track record that implies
  good judgment ("has filled more than 300 frontline supervisor roles").

Avoid superlatives that carry no information, such as "world's best expert", and avoid
roles that would never do the task. The closer the role matches reality, the more useful
the output.

**Placement.** Put the role at the start of the prompt, as the opening sentence of
Additional Context ("You are..."). If the prompt has no other context, give the role its
own `# Role` section at the top.

### 3.1 Example: job posting

Without a role:
~~~
# Instructions
Write a job posting for a night-shift warehouse supervisor at a regional distribution
center.
~~~
Typical result: a generic posting ("seeking a motivated leader... competitive pay...").
It leaves out the details applicants actually filter on.

With a role:
~~~
# Additional Context
You are an in-house recruiter at a regional logistics company who has filled more than
300 frontline supervisor roles and is known for postings that attract qualified
applicants within the first week.

# Instructions
Write a job posting for a night-shift warehouse supervisor at a regional distribution
center.
~~~
Typical result: the role moves the priorities toward what applicants decide on. The shift
hours and rotation come first, then the pay range, team size, required certifications,
physical demands, what a typical night involves, and a simple way to apply.

## 4. Delimiter Pattern

As a structural element, delimiters separate content from instructions. As a pattern, they
become a repeatable layout for **prompts with several input sources**:
- Each source gets a heading that names it.
- A `---` separator marks the move to the next source.
- The content goes in a fenced block, with a language tag where it helps (` ```csv `,
  ` ```json `, ` ```sql `).
- The instruction comes after the inputs and refers to each one by name.

| Delimiter | Best for |
|---|---|
| Triple-backtick fences, with optional language tag | Content with special characters: data, code, copied text, logs |
| `#` headings with `---` separators | Human-readable sections that make each input source easy to identify |
| XML tags | Semantic or nested organization, or content that contains backticks |

Use this pattern when building reusable templates, combining several data sources, or
writing prompts that other people will maintain. The benefits are mainly for the people
who maintain the template. The structure is visible at a glance, each source can be
updated on its own, and it is obvious which variable controls which part of the prompt.
Modern models handle mixed content well, but clear boundaries still matter for
user-generated content and long prompts.

**Troubleshooting:** if a template gives inconsistent results, check whether the delimiter
appears inside the variable content. If it does, switch delimiter types.

### 4.1 Example: churn analysis template

Without the pattern:
~~~
Compare {{exit_survey}} with support history {{ticket_summary}} and usage data
{{usage_data}} for account {{account_name}}. Identify the most likely reason the account
cancelled in one sentence.
~~~

With the pattern:
~~~
# Exit Survey Response
```text
{{exit_survey}}
```
---
# Support Ticket Summary (last 90 days)
```text
{{ticket_summary}}
```
---
# Product Usage Data
Account: {{account_name}}
```csv
{{usage_data}}
```
---
# Instructions
Review the exit survey, support ticket summary, and usage data above. Identify the most
likely reason {{account_name}} cancelled, in one sentence.
~~~
In the first version, a survey answer containing a comma, a multi-line ticket, or a CSV
blends into the instruction sentence. In the second, each source sits in its own labelled
block. A broken run can then be traced to the input that caused it.

## 5. Template Pattern

The Template Pattern fixes the output to a named schema. Output Format, as a structural
element, describes the shape of the response. The pattern turns that shape into a
reusable contract: **the same fields, in the same order, every run.** The Delimiter
Pattern organizes inputs; the Template Pattern controls outputs.

Use it when outputs must look the same across many runs, feed into a system, or be
written by several people. Skip it for one-off creative work where flexibility matters
more than standardization. Strict JSON or XML contracts are a separate topic; this
pattern covers plain-text schemas.

Template design rules:
- Use **concrete placeholders** that say what goes in each field (`[Main Objection]`, not
  `[Thing 2]`).
- Keep templates to **4–6 fields**. Fewer is too loose; more becomes a form the model fills
  badly.
- Give a length rule per field ("one sentence each").
- Give a **fallback for missing information** ("Write 'Not discussed' if the notes don't
  cover it"). Missing data then shows up visibly instead of being invented.

### 5.1 Example: sales call CRM summary

Without the pattern:
~~~
# Instructions
Summarize the sales call notes below.
# Call Notes
```text
{{call_notes}}
```
~~~
Typical result: a readable paragraph whose structure changes from call to call, so
summaries can't be compared or pasted into the CRM.

With the pattern:
~~~
# Instructions
Summarize the sales call notes below for the CRM record.
# Call Notes
```text
{{call_notes}}
```
# Output Format
Fill every field with one sentence. Write "Not discussed" if the notes do not cover it.
Account: [Company name and segment]
Deal Stage: [Discovery, Evaluation, Negotiation, or Closed]
Decision Maker: [Name and role of the person who signs]
Main Objection: [The strongest concern the buyer raised]
Next Step: [Agreed action, owner, and date]
Expected Close: [Month and year]
~~~
Typical result: every summary has the same six fields. A rep who forgot to ask about the
decision maker sees "Not discussed" straight away.

## 6. Repetition Pattern

The Repetition Pattern sends the **identical prompt twice, back to back**, so the model
reads the whole thing again before it responds. Details that appeared early and were
missed on the first read get attention on the second. Research by Leviathan, Kalman, and
Matias at Google Research found that repetition improved results in 47 of 70
benchmark-model combinations, with no losses, when reasoning was disabled. The pattern
changes only the input, so it is easy to add to an existing workflow.

| Use when | Skip when |
|---|---|
| First-answer accuracy matters and reasoning or chain-of-thought is off | Reasoning is already enabled |
| Important context appears before the question | Doubling the prompt would strain the context window |
| The model must retrieve details from the middle of a list or long prompt | Token cost matters in a high-volume workflow |
| One high-stakes answer matters more than token efficiency | |

Rules:
- **Repeat the prompt exactly.** Don't paraphrase or summarize. The gain comes from
  attending to the same tokens again; adding length with filler produces no gain.
- **Size guidance:** under about 1,000 tokens, doubling is usually safe and cheap. From
  1,000 to 3,000, it is still reasonable. Above 3,000, check the model's context limits
  and weigh whether a single pass with explicit emphasis would be more efficient.
- Repetition doubles input tokens and can add latency. Input tokens usually cost much less
  than output tokens, but very long prompts may slow down noticeably.
- For templates in code, repeat programmatically (`prompt + "\n\n" + prompt`) rather than
  duplicating the template text.

| Variant | Structure | Best for |
|---|---|---|
| Standard | The prompt twice, back to back | Most tasks |
| Verbose | The prompt, then "Let me repeat that:", then the prompt again | When explicit signposting helps the model register the repeat |
| ×3 | Three full copies with transition phrases | Long-list retrieval, or tasks very sensitive to mid-prompt details |

### 6.1 Example: laptop selection

~~~
Our field-sales team needs new laptops. The options are:
1. Model A: 14-inch, 1.45 kg, 10-hour battery, LTE modem, $1,150
2. Model B: 13-inch, 1.19 kg, 13-hour battery, no LTE, $1,280
3. Model C: 13-inch, 1.24 kg, 12-hour battery, LTE modem, $1,390
4. Model D: 14-inch, 1.29 kg, 15-hour battery, LTE modem, $1,620
Which model meets all three requirements (under 1.3 kg, at least 12 hours of battery,
and built-in LTE) at the lowest price?
Our field-sales team needs new laptops. The options are:
1. Model A: 14-inch, 1.45 kg, 10-hour battery, LTE modem, $1,150
2. Model B: 13-inch, 1.19 kg, 13-hour battery, no LTE, $1,280
3. Model C: 13-inch, 1.24 kg, 12-hour battery, LTE modem, $1,390
4. Model D: 14-inch, 1.29 kg, 15-hour battery, LTE modem, $1,620
Which model meets all three requirements (under 1.3 kg, at least 12 hours of battery,
and built-in LTE) at the lowest price?
~~~
The correct answer depends on details spread across the list. Model B is the lightest and
cheaper, but has no LTE. Model D qualifies but costs more. Model C is the cheapest model
that meets all three requirements. A single pass is more likely to settle on the most
attractive-looking option. The repeated prompt makes the model more likely to check every
option against every requirement.

## 7. Self-Reflection Pattern

After the model produces a draft, the Self-Reflection Pattern asks it to critique that
draft. The critique names what works and exposes gaps and assumptions that writers and
models both tend to miss. It is the first half of an evaluate-then-improve loop and is
usually followed by Refinement.

Use it for complex analysis, plans, recommendations, and high-stakes outputs where the
extra step is worth the cost. Skip it for simple factual questions or urgent work.

Writing the reflection prompt:
- Ask for **both** strengths and gaps, so the later revision keeps what works.
- Frame the critique around the task's real goal ("for this business", "for a developer
  integrating this").
- Limit the length. A short critique stays focused on the most important gaps.

### 7.1 Example: bakery launch plan

Stage 1, the draft (template):
~~~
# Business Details
```text
{{business_context}}
```
---
# Instructions
Create a 4-week launch plan for this business's new online ordering. Keep the plan to
4–5 sentences.
~~~
Example value for `{{business_context}}`: *Family-run bakery, two locations, 6 staff.
Launching next-day pickup ordering through its point-of-sale system in 4 weeks. Goal: 40
online orders a week by week 8. The owner handles marketing; no developer on staff.*

Typical draft: a sensible sequence of setup, menu photos, social media posts, and a launch
promotion.

Stage 2, self-reflection:
~~~
Critique the launch plan above. Name what it does well and the most important things it
overlooks for this business. Keep the critique to 3–4 sentences.
~~~
Typical critique: the plan is strong on sequence and promotion. It overlooks the staff
workflow for packing and pickup, a daily order cutoff linked to baking capacity,
processing fees in online pricing, and weekly tracking toward the 40-order goal.

Stage 3 is Refinement (section 9).

## 8. Inversion Pattern

The Inversion Pattern reverses the usual direction: instead of task → execution, it runs
**content → interrogation**. You give the model your content and ask it for questions
that expose blind spots and weak points. The output is questions, not completed analysis.

Inversion is valuable because the model can take perspectives the author rarely takes
themselves: a skeptical budget holder, an implementer, someone affected by the change.
Use it to stress-test content before a high-stakes review, to surface knowledge gaps, or
to produce related material such as FAQs or interview questions. Skip it for simple
factual content, urgent work, or cases where outside perspectives won't change the
result.

Writing the inversion prompt:
- Name each perspective and **what it cares about** ("a finance director focused on cost
  and billable revenue").
- Limit the number of questions and their length ("one question per perspective, one
  sentence each").
- Expect a multi-step workflow: the model asks, the user answers, then Refinement
  incorporates the answers.

The two critique paths join at the same final step:

```
Draft ──┬── Path A: Self-Reflection → model critiques → Refinement on the critique ──┐
        └── Path B: Inversion → model asks questions → user answers → Refinement ────┴→ Improved output
```

### 8.1 Example: four-day work week proposal

Stage 1, inversion (template):
~~~
# Proposal
```text
{{proposal}}
```
---
# Instructions
Generate challenging questions about this proposal from three perspectives: a finance
director focused on cost and billable revenue, a client services lead focused on client
coverage and response times, and a junior designer focused on workload and wellbeing.
Write one question per perspective, each one sentence.
~~~
Example value for `{{proposal}}`: *Pilot a four-day work week (Monday–Thursday, 32 hours,
no pay cut) for all 120 staff at a marketing agency for 6 months starting January.
Expected outcomes: lower attrition, no drop in billable hours, higher employee
satisfaction scores.*

Typical questions:
- **Finance director:** How will billable hours stay flat when available hours drop by 20%,
  and what happens if they don't?
- **Client services lead:** Who handles client requests on Fridays, and how will urgent
  issues be covered without eroding the day off?
- **Junior designer:** Will the same workload just be compressed into four longer days, and
  how will the pilot detect burnout rather than only measuring satisfaction?

Each question turns a stated goal into a concrete risk the proposal doesn't address yet.
Stage 2 is Refinement (section 9).

## 9. Refinement Pattern

Refinement revises content using the analysis that came before it. It is the
implementation step of multi-step workflows and **is never used on its own**. After
Self-Reflection, it applies the critique. After Inversion, it incorporates the user's
answers. After Comment-Driven Generation, it applies the recommendations.

Use it when the preceding analysis produced specific, actionable points. Skip it when the
draft already meets the requirements, or when the analysis was too vague to act on. In
that case, redo the analysis with a sharper prompt first.

Writing the refinement prompt:
- **Name the points to address** instead of relying on "fix what you mentioned". This
  keeps the revision on target and makes the prompt work even when it is sent as a
  separate call.
- **Say what to keep**: what worked, the tone, the original goals.
- **Limit the length**, so the revision doesn't grow just because it is a revision.

### 9.1 Example: Path A, after self-reflection (bakery plan)

~~~
Revise the launch plan to address the gaps in your critique: the staff packing and
pickup workflow, a daily order cutoff tied to baking capacity, processing fees in online
pricing, and weekly tracking toward 40 orders. Keep the sequence and promotion steps
that worked. Keep the plan to 5–6 sentences.
~~~

### 9.2 Example: Path B, after inversion (four-day week proposal)

~~~
Here are my answers to your questions:
1. Finance: teams will cut internal meetings by a third; if billable hours fall more than
   5% for two consecutive months, the pilot pauses for review.
2. Client coverage: a rotating Friday on-call pair per team handles urgent requests and
   takes a day off the following week.
3. Workload: weekly hours are capped at 32, and a monthly burnout survey runs alongside
   the satisfaction survey.
Revise the proposal to include these answers. Keep the original goals and keep it under
150 words.
~~~

## 10. Comment-Driven Generation

Comment-Driven Generation, developed by Richard Davies, handles **several targeted
changes in one pass**. Each reviewer comment is anchored to a selected passage, so it's
clear which text needs work and why. Batching the comments is more efficient than
writing a separate prompt per change, and it keeps related changes in context together.

Workflow:
1. Comment-Driven Generation: recommend how to resolve each comment.
2. Optionally, Inversion: generate questions about the recommendations.
3. Refinement: apply the changes to the draft.

Use it when content needs targeted improvements rather than a rewrite, or when a team
needs a structured review workflow.

Comment format: one block per comment, containing the selected text and the comment, with
blocks separated by `---`. Put the comments in their own fenced section, apart from the
draft.

**Comment quality decides output quality.** "No measurable outcome; state the target and
how it is assessed" produces a useful recommendation. "This needs work" produces generic
advice. If the user's comments are vague, suggest sharpening them before running the
prompt.

### 10.1 Example: grant application review

Stage 1 (template):
~~~
# Draft
```markdown
{{draft_content}}
```
---
# Reviewer Comments
```text
{{reviewer_comments}}
```
---
# Instructions
For each reviewer comment, recommend how to resolve it in the draft. Quote the selected
text, then give the recommendation in 2–3 sentences.
~~~
Example values:
- `{{draft_content}}`: *Our program helps local kids read better. Many families in our area
  need support. We will run sessions after school throughout the year. The money will
  cover program costs.*
- `{{reviewer_comments}}`:
  ```
  Selected Text: "helps local kids read better"
  Comment: "No measurable outcome. State the target improvement and how it is assessed."
  ---
  Selected Text: "Many families in our area need support"
  Comment: "Needs evidence. Cite local literacy rates or school data."
  ---
  Selected Text: "The money will cover program costs"
  Comment: "Budget not justified. Break down the main cost lines."
  ```

Typical result: one anchored recommendation per comment. These include a target such as a
reading-level gain measured by a standard assessment, a citation of district literacy
data, and a short budget breakdown covering tutors, materials, and transport. The
recommendations are specific additions, not general advice to improve the draft.

Stage 2, refinement:
~~~
Revise the draft by applying each recommendation above. Change only the selected
passages and their immediate sentences; keep the rest of the draft as written.
~~~

## 11. Delivering multi-stage workflows

Self-Reflection, Inversion, Refinement, and Comment-Driven Generation produce **a sequence
of prompts**, not a single one. Deliver them this way:
- Number each stage and label it: "Stage 1: Draft", "Stage 2: Self-Reflection",
  "Stage 3: Refinement".
- Between stages, state what the user does: review the critique, answer the questions, or
  collect comments.
- **In a chat**, later stages can refer to "the plan above". **In an automated pipeline**,
  each stage is a separate call that doesn't share history. Include the previous output in
  a delimited block (`<draft>...</draft>`, `<critique>...</critique>`) and refer to it by
  name.
- Keep every stage's prompt explicit and imperative ("Critique...", "Revise..."), not
  hedged ("Could you please reflect...").

## 12. Failure modes to avoid

- **Refinement with nothing before it.** "Improve this" without a critique, answers, or
  comments produces arbitrary changes.
- **Vague analysis feeding Refinement.** If the critique says "could be more detailed",
  sharpen the critique before revising.
- **Unrealistic or empty roles.** "World's greatest writer", or a role that would never do
  the task.
- **Paraphrased repetition.** Repetition only works when the text is identical. Also skip
  repetition when reasoning is enabled.
- **Overgrown templates.** More than 6 fields, vague placeholders, or no fallback for
  missing information.
- **Delimiter collisions.** Fences or tags that also appear inside the variable content.
- **Pattern stacking.** Adding every pattern to a simple task.
- **Vague review comments.** Unanchored or non-specific comments lead to generic
  recommendations.
- **Pipelines that rely on "above".** Separate calls can't see earlier output unless it is
  passed in.

## 13. Pre-delivery checklist

Before returning a prompt, confirm:
- [ ] Each pattern used solves a problem the task actually has (section 2).
- [ ] The role, if used, is realistic and specific: expertise, domain, and a credibility
      marker.
- [ ] Multi-source inputs each have a heading, a separator, and a fence, and the
      instruction refers to each by name.
- [ ] Templates have 4–6 concrete fields, a length rule per field, and a
      missing-information fallback.
- [ ] Repetition, if used, is verbatim, with reasoning off and the prompt size checked.
- [ ] Multi-stage workflows are numbered, the user's actions between stages are stated,
      and Refinement names the specific points to address.
- [ ] Pipeline stages pass earlier outputs in delimited blocks.
- [ ] Every stage's prompt is explicit, with no hedging.
