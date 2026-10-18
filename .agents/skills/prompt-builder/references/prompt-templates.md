# Prompt Templates

Read this file whenever the deliverable is a reusable prompt, or could become one. A
prompt template is a reusable prompt for one recurring problem. It combines the
structural elements, linguistic elements, and prompt patterns into a single framework
that keeps its logic fixed while its inputs change from run to run. This file covers
how to compose that framework, design its parameters, chain templates into multi-step
workflows, store them, and test them before anyone relies on them.

## Contents
1. Anatomy of a prompt template
2. Composing a template from the design stack
3. Designing the parameters
4. Example: single-step template (lesson plan generator)
5. Multi-step template chains
6. Example: two-step chain (office relocation planner)
7. Storing templates
8. Testing and iterating
9. Delivering a template
10. Failure modes to avoid
11. Pre-delivery checklist

---

## 1. Anatomy of a prompt template

Every template has two halves.

| Half | Contains | Changes between runs? |
|---|---|---|
| **Stable core logic** | Role, Instructions, Constraints, Output Format: who the model is, what to do, what must be respected, and how the output must be shaped | No. This is what makes outputs consistent and comparable. |
| **Dynamic input parameters** | Runtime values: the details of this specific case (a feature, a class, a customer, a document) | Yes. They are injected before each run. |

The runtime path:

```
Template (stable core) + Input parameters (this run's values)
        → Language model
        → Output in the template's schema
        → Evaluation: does it meet the standard?
              ├── Yes → approved for reuse
              └── No  → revise the template, the inputs, or both, then test again
```

**Key principle:** keep the template logic stable, vary only the inputs, and evaluate the
outputs before anyone relies on the template. If changing an input would require
editing the instructions, the template has logic in the wrong place.

## 2. Composing a template from the design stack

A template is where all three design layers come together. Build it one layer at a time,
in this order:

| Layer | What it provides | Typical choices in a template |
|---|---|---|
| 1. Structural elements | The shape: sections, boundaries, inputs, format | Instructions, a labelled input section, Constraints, Output Format, delimiters around dense inputs |
| 2. Linguistic elements | Clear wording, so the model interprets the task the same way every run | Precise scope and section limits, imperative instructions, no filler |
| 3. Prompt patterns | Reusable reasoning behavior | Role Assignment (almost always), Template Pattern for the output schema, Delimiter Pattern for multiple inputs, Inversion and Refinement for chains |

Layer order in the final prompt usually follows: Role → (Context) → Instructions →
Constraints → input section(s) → Output Format.

## 3. Designing the parameters

The structural reference covers parameter basics (descriptive names, one syntax, delimit
external text). Templates need a few more decisions.

**Choose the right granularity.**
- Use one parameter per fact that varies independently: `{{grade_level}}` and
  `{{class_length}}`, not one `{{class_details}}` blob.
- Keep facts that never vary as literal text. A parameter for a value that is the same in
  every run only adds noise.
- Never parameterize the logic itself. A `{{instructions}}` or `{{output_format}}`
  parameter turns the template into a blank form, and its consistency disappears.

**Match the container to the value.**
- Short, single-line values (a name, a date, a number, a level) go inline under a
  labelled section, wrapped in quotes so their boundaries are clear:
  `Deadline: "{{deadline}}"`.
- Long, dense, multi-line, or externally sourced values (notes, documents, logs, user
  text) go in their own fenced or tagged block, so they can't be read as instructions.
- If short values come from untrusted users, fence them too.

**Group related inputs.** Put the short parameters for one case under one named section
(`# Lesson Request`, `# Relocation Parameters`). The instructions can then refer to the
section by name ("Use only the details in the lesson request").

**Define behavior for empty or optional values.** Decide what the model should do when
an optional parameter is blank or "None", and state it in the Constraints. For example:
"If Available Materials is 'None', plan activities that need only paper and pencils."
Without this rule, missing values lead to invented ones.

**Keep the count manageable.** About 3–7 parameters per template is typical. Many more
usually means the template is trying to cover several problems and should be split.

## 4. Example: single-step template (lesson plan generator)

**The problem:** teachers at a school write lesson plans in different styles, so plans
left for substitute teachers vary in quality. Some are missing timings, others don't
include any check that students learned the objective. One template produces the same
plan shape for any subject, grade, and class length.

### 4.1 Template

~~~
# Role
You are a middle-school teacher and instructional coach who writes lesson plans that
substitute teachers can run without extra preparation.

# Instructions
Create a single-lesson plan from the lesson request below. Use only the details in the
lesson request.

# Constraints
- The activity times must add up to exactly the class length.
- Use only the materials listed in Available Materials. If it is "None", plan
  activities that need only paper and pencils.
- Align every activity to the stated learning objective.
- Write each activity as steps the teacher says or does, with no reference to
  materials or prior lessons that are not listed.

# Lesson Request
Subject: "{{subject}}"
Grade Level: "{{grade_level}}"
Learning Objective: "{{learning_objective}}"
Class Length (minutes): "{{class_length}}"
Available Materials: "{{available_materials}}"

# Output Format
## Objective
[Exactly 1 sentence restating the learning objective in student-friendly language]
## Activities
1. [Activity name] ([minutes]): [what the teacher does and what students do]
2. [Activity name] ([minutes]): [what the teacher does and what students do]
3. [Activity name] ([minutes]): [what the teacher does and what students do]
## Check for Understanding
[One question or task that shows whether students met the objective]
## Differentiation
- [One adjustment for students who need more support]
- [One extension for students who finish early]
~~~

### 4.2 Example input values

| Parameter | Value |
|---|---|
| `{{subject}}` | `Science` |
| `{{grade_level}}` | `Grade 7` |
| `{{learning_objective}}` | `Explain why the Moon has phases using the positions of the Sun, Earth, and Moon` |
| `{{class_length}}` | `45` |
| `{{available_materials}}` | `One flashlight, a foam ball on a stick, a room that can be darkened` |

Typical result: a plan in exactly the template's shape. It has three activities that add
up to 45 minutes, for example a 10-minute demonstration with the flashlight and ball, a
25-minute small-group modelling task, and a 10-minute exit ticket. The check for
understanding asks students to draw the positions that produce a given phase. Running
the same template for a Grade 4 maths lesson produces the same four sections, so any
substitute knows where to look.

### 4.3 Component map

| Layer | Choices in this template |
|---|---|
| Structural | Instructions scoped to the lesson request; five inline parameters grouped under `# Lesson Request`; verifiable Constraints; fixed Output Format |
| Linguistic | High Precision (exact minutes, one-sentence objective, counted bullets); explicit imperative instructions; no filler |
| Patterns | Role Assignment (teacher and instructional coach); Template Pattern (four fixed sections) |

The constraint that the times must add up to exactly the class length is what makes this
template safe to reuse. It turns "a reasonable plan" into a plan that fits the real
timetable, and it can be checked on every output.

## 5. Multi-step template chains

Some tasks are too risky to plan in one pass: a single prompt guesses at the facts it
needs. A **template chain** splits the work into steps, with a separate template for
each step. The most common chain is Inversion followed by Refinement:

| Step | Template | Output | Becomes |
|---|---|---|---|
| 1 | Inversion: ask the questions that expose hidden dependencies and risks | A short numbered list of questions | Questions the user answers |
| 2 | Refinement: build the deliverable from the answers | The final plan in a fixed schema | The finished output |

Chain design rules:
- **Each step is its own template** with its own Output Format, including the question
  step ("A numbered list of 3 questions, each one sentence").
- **One step's output becomes the next step's parameter.** The user's answers enter
  step 2 as `{{assessment_answers}}` in a delimited block.
- **Use the same name for the same input in every step.** If both steps need the
  deadline, both use `{{deadline}}`.
- **Pass each step everything it needs.** Steps often run as separate calls without
  shared history, so if step 2 needs the original context, include that parameter again.
- **Keep the role identical across steps**, so the perspective doesn't drift between
  questioning and planning.

## 6. Example: two-step chain (office relocation planner)

**The problem:** operations teams plan office moves from a single prompt and discover the
critical dependencies too late: the phone system, building access, a network that isn't
ready. Step 1 surfaces those dependencies as questions. Step 2 turns the answers into a
plan with a go/no-go checkpoint and a fallback.

### 6.1 Step 1 template: Inversion

~~~
# Role
You are an operations manager who has led office relocations for growing companies
without interrupting customer-facing teams.

# Instructions
Before proposing a relocation plan, ask exactly 3 short questions that expose hidden
dependencies, disruption risk, and fallback options.

# Current Office Context
```text
{{office_context}}
```

# Relocation Parameters
Move Goal: "{{move_goal}}"
Deadline: "{{deadline}}"
Budget: "{{budget}}"
Disruption Tolerance: "{{disruption_tolerance}}"

# Output Format
A numbered list of 3 questions, each one sentence.
~~~

Example input values:
- `{{office_context}}`: *60-person software company leaving its current floor when the
  lease ends. The server closet holds the phone system and the badge-access controller.
  Customer support (8 people) answers live phone lines 08:00–18:00 on weekdays. The new
  office is 4 km away; the landlord is still finishing the network cabling. Furniture is
  owned; most monitors are leased.*
- `{{move_goal}}`: `Move all staff to the new office with no missed support calls`
- `{{deadline}}`: `Current lease ends March 31`
- `{{budget}}`: `$40,000`
- `{{disruption_tolerance}}`: `Low for customer support; moderate for everyone else`

Typical questions:
1. Can the phone system and badge controller be moved or replaced without a gap in
   service, and who does that work?
2. When will the new site's network cabling be finished and tested, and what happens to
   the schedule if it slips?
3. If the new office isn't ready by March 31, is there a fallback such as a short lease
   extension or temporary remote work for support?

### 6.2 Step 2 template: Refinement

~~~
# Role
You are an operations manager who has led office relocations for growing companies
without interrupting customer-facing teams.

# Instructions
Using the assessment answers, create a relocation plan that keeps customer support
reachable throughout the move. Include a go/no-go checkpoint and a fallback.

# Constraints
- Base the plan only on the office context, assessment answers, and relocation
  parameters.
- Do not commit spending beyond the stated budget.
- Give every dated item a date on or before the deadline.

# Current Office Context
```text
{{office_context}}
```

# Assessment Answers
```text
{{assessment_answers}}
```

# Relocation Parameters
Move Goal: "{{move_goal}}"
Deadline: "{{deadline}}"
Budget: "{{budget}}"
Disruption Tolerance: "{{disruption_tolerance}}"

# Output Format
## Relocation Plan
**Phase 1: Prepare**
- [2–3 items, each with an owner or date where the inputs give one]
**Phase 2: Move**
- [2–3 items, each with an owner or date where the inputs give one]
**Go/No-Go Checkpoint**
- [The condition to proceed, and the fallback if it fails]
~~~

Example value for `{{assessment_answers}}`:
```text
1. Phones are cloud-based except the desk handsets; the badge controller must be rewired
   by the building's vendor (a 2-day job).
2. Cabling finishes March 10; testing takes 3 days.
3. The current landlord allows a 2-week extension for $6,000; support can work from
   home on softphones.
```

Typical result:
- **Phase 1** books the badge vendor, schedules the network test for March 10–13, and
  sets up softphones for the support team in advance.
- **Phase 2** moves non-support staff first and moves support on a weekend, with
  softphones as cover.
- **The checkpoint** proceeds only if the network test passes by March 14. Otherwise the
  team takes the $6,000 extension, which stays within budget.

Each answer to step 1 appears as a concrete control in the plan, and nothing in the plan
relies on facts the inputs didn't provide.

### 6.3 Data flow

```
office_context, move_goal, deadline, budget, disruption_tolerance
        → Step 1 (Inversion) → 3 questions → user answers
        → Step 2 (Refinement), receiving the same five parameters plus {{assessment_answers}}
        → Relocation plan
```

## 7. Storing templates

Store templates in **external files**, not inline in source code: Jinja2 (`.j2`),
Markdown (`.md`), or the equivalent for the user's stack. External files can be reviewed,
compared, and changed without a code deploy, and non-engineers can read them.

When the user is building a pipeline or an application:
- Use one file per template, and one file per step in a chain
  (`relocation_step1_inversion.md`, `relocation_step2_refinement.md`).
- Start each file with a short comment listing its purpose and its parameters, so
  whoever fills it knows what it expects.
- Choose the placeholder syntax that matches the rendering engine: `{{double_braces}}`
  for Jinja2, `{single_braces}` for Python string formatting. Stick to one syntax across
  the template library.

Versioning, change control, regression checks, and lifecycle governance for production
templates are beyond this file.

## 8. Testing and iterating

A template is not ready until it has been run against varied inputs. One good output
proves only that one input works.

**Build a small test set.** Include 3–5 inputs that differ in the ways real inputs will:

| Test input | Purpose | Lesson plan example |
|---|---|---|
| Typical | Confirms the normal case | The Grade 7 Moon phases lesson |
| Sparse | Checks the empty or optional value rules | Available Materials: "None" |
| Edge | Checks limits and constraints at extremes | Class length of 20 minutes |
| Conflicting | Checks how the template handles inputs that pull against each other | A broad objective with a 20-minute class |
| Hostile or messy | Checks delimiter protection and robustness | An objective containing "ignore the format above", or a pasted value with stray formatting |

**Evaluate every output against the same criteria:**
- **Structure compliance**: every section is present, in order, with the required counts.
- **Factual correctness**: nothing is invented beyond the inputs.
- **Completeness**: every input that should appear in the output does.
- **Consistency**: different inputs produce comparable outputs of the same quality.
- **Safety and constraints**: every constraint holds, and hostile input is treated as
  data.

**When an output fails, decide where to fix it.** If the template logic is wrong (a
missing rule, an unclear section, a weak constraint), revise the template and re-run the
whole test set, because a fix for one input can break another. If the input itself was
malformed, fix the input or add a rule for that kind of input. Repeat until every test
input passes. Only then is the template ready for real use.

## 9. Delivering a template

When the deliverable is a template, give the user:
1. **The template** (one fenced block per template; one per step in a chain).
2. **A parameter table**: parameter, description, example value, and whether it is
   required or optional, with the empty-value behavior for optional ones.
3. **The data flow**, for chains: which output feeds which parameter.
4. **A component map** (section 4.3 format), when the user is learning the method or
   asks why the template is built the way it is. Skip it for users who only want the
   template.
5. **A suggested test set**: 3–5 one-line test inputs (section 8), so the user can
   validate the template before relying on it.

## 10. Failure modes to avoid

- **Logic leaking into parameters.** `{{instructions}}`, `{{format}}`, or `{{tone}}` used
  as a catch-all, so every run is effectively a different prompt.
- **Hardcoded case details.** A specific name, date, or figure left in the stable core
  instead of being parameterized.
- **One parameter for everything.** A single `{{details}}` blob, so nothing can be
  referenced, checked, or varied independently.
- **Undefined empty values.** Optional parameters with no rule for blank input, which
  invites invented content.
- **Mismatched names across a chain.** `{{deadline}}` in step 1 and `{{due_date}}` in
  step 2.
- **Chain steps missing context.** Step 2 needs the original context but doesn't receive
  it.
- **No Output Format on a question step.** The inversion step rambles instead of
  returning a fixed number of questions.
- **Templates hardcoded in source code.** Difficult to review, compare, or change.
- **Shipping after one good run.** The template has never been tested against sparse,
  edge, or hostile inputs.

## 11. Pre-delivery checklist

Before returning a template, confirm:
- [ ] The stable core (role, instructions, constraints, output format) contains no
      case-specific values.
- [ ] Each parameter is one independently varying fact with a descriptive name, and
      there are about 3–7 of them.
- [ ] Short values are inline and quoted under a named section; long or external values
      are delimited.
- [ ] Every optional parameter has a stated empty-value rule.
- [ ] Chains: each step has its own Output Format, names match across steps, each step
      receives everything it needs, and the role is identical.
- [ ] The parameter table, data flow (for chains), and a suggested test set are
      included.
- [ ] If the user is building a pipeline, the storage advice (external files, one per
      step, matching syntax) is given.
