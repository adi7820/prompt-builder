---
name: prompt-builder
description: Build well-designed prompts and reusable prompt templates for any task the user describes. Use this skill whenever the user asks to write, create, design, draft, improve, restructure, review, or debug a prompt, system prompt, prompt template, or LLM instructions — including requests like "make a prompt for X", "turn this into a prompt", "write instructions for a model to do Y", "parameterize this prompt", or "why is my prompt giving bad output". Also use it when the user pastes a messy prompt and wants it cleaned up, or is building an LLM feature, agent, or pipeline step and needs the prompt for it, even if they never say the word "prompt engineering".
---

# Prompt Builder

Turn a described task into a prompt that is clear, controllable, and easy to maintain.
The design guidance lives in the reference files below. Mandatory references are read
for every request, in order. Conditional references are read only when the request
shows one of their signals, so requests don't pay for guidance they don't need.

## Reference index

### Mandatory references (read for every request, in this order)

| Order | Reference | What it decides |
|---|---|---|
| 1 | `references/prompt-design-structural-elements.md` | Which sections the prompt has (Instructions, Constraints, Context, Input Parameters, Output Format, Delimiters) and how they are laid out. |
| 2 | `references/prompt-design-linguistic-elements.md` | How the wording inside those sections is written (Precision, Directness, Brevity), and how to diagnose generic, hedged, or bloated output. |
| 3 | `references/prompt-patterns.md` | The strategy: role, multi-source inputs, fixed output schema, repetition, or a critique-and-improve workflow, plus how to deliver multi-stage prompt sequences. |
| 4 | `references/prompt-templates.md` | How to compose everything into a reusable template: parameter design, multi-step template chains, storage, testing across varied inputs, and template delivery. |

Read them in this order. Each layer works on the output of the one before it: structure
creates the sections, wording tunes them, patterns add the strategy, and templates
package the result for reuse.

### Conditional references (read only when a signal matches)

Read a conditional reference when **any** of its signals appears in the request. If
you're unsure whether a signal applies, read it. If no signal matches, the mandatory
references are enough. Apply conditional references after the mandatory ones.

| Reference | Read it when the request shows any of these signals |
|---|---|
| `references/prompt-types.md` | The user mentions a system prompt, user prompt, chat model, message roles, or API calls; the prompt is for a chatbot, assistant, agent, or multi-turn conversation; it will run repeatedly inside an application, product, or pipeline with the same instructions; the user asks to split, decompose, or convert a prompt into system and user parts, or asks where something belongs; or the user pastes an existing system prompt or user prompt to review. |
| `references/contextual-prompting.md` | The user mentions examples, few-shot, zero-shot, one-shot, demonstrations, or in-context learning; supplies sample inputs and outputs or says "like this one"; the task is classification, labeling, tagging, routing into fixed categories, structured extraction, or format conversion; the task uses organization-specific categories, codes, or conventions the model can't know; or the user reports outputs that are inconsistent across runs or don't follow the required labels or format. |
| `references/prompt-sampling.md` | The user mentions sampling, multiple runs or generations, majority voting, self-consistency, best-of-N, temperature, or top_p; the output is a score, rating, or label that drives a decision such as a gate, cutoff, approval, or routing; the user reports that the same prompt gives different answers across runs on a consequential task; or the user wants a high-stakes output made more reliable, or wants the best of several drafts selected. |
| `references/advanced-prompt-patterns.md` | The user mentions chain of thought, step-by-step reasoning, showing reasoning, ReAct, agents, tool use, or a prompt that generates prompts (meta prompting); the task needs auditable reasoning over rules, thresholds, or multiple signals; the user wants prompt templates generated for a category of recurring tasks; or the task requires gathering information from logs, APIs, databases, or other systems step by step before concluding (investigation, triage, diagnosis). |
| `references/prompt-security.md` | The user mentions security, prompt injection, jailbreaks, hardening, guardrails, red-teaming, adversarial users, or system prompt leakage; the prompt will process input the user doesn't control (messages from customers or the public, emails, web pages, retrieved documents, uploaded files, API or tool responses); it is for a public-facing chatbot or an agent with tool or data access; the context will hold sensitive data (personal data, credentials, internal documents); or the user asks for a prompt to be reviewed for vulnerabilities. |

<!-- Maintaining this index:
- New reference: create the file in references/ and, unless told it is mandatory, add a
  row to the conditional table listing concrete signals visible in a user's request,
  not vague labels like "advanced prompts".
- Making a reference mandatory: add its row to the mandatory table with the next order
  number.
Existing references do not need to change. -->

## Workflow

### 1. Understand the task

From the user's message, identify:
- **The goal**: what the model receiving the prompt must produce.
- **The audience of the output**: who reads it, and how technical they are.
- **The inputs**: what material the model will act on (documents, code, data, user text).
- **The mode**: a **one-off prompt** with fixed values, or a **reusable template** with
  values that change between runs.
- **Hard requirements**: length, tone, required or forbidden content, required sections.
- **The purpose**: **execution** (a decision, an artifact, something to ship) or
  **exploration** (ideas, options, open reasoning). This choice sets how precise and how
  direct the wording should be.

Don't assume any of these. Anything the user hasn't stated becomes a question in
step 2.

If the user supplies an existing prompt to improve, start by sorting its material into
the structural elements (see the placement guide in the structural reference). That step
usually shows what is missing, duplicated, or in the wrong place.

### 2. Ask before building

Never assume an important decision. Before writing the prompt, ask the user about every
open decision and every point of doubt, and give a recommendation for each one. The user
either accepts the recommendation or gives their own answer.

**What counts as important (always ask unless the user already stated it):**
- Anything from step 1 the user didn't state: goal, audience, inputs, mode, hard
  requirements, purpose.
- Facts the prompt must contain that only the user knows: names, dates, figures,
  policies, thresholds.
- Design choices that change what the prompt does: which sections or constraints to
  include, which pattern or multi-stage workflow to use, the number of steps in a chain,
  which parameters to create and how to handle empty values, and the placeholder syntax
  when the template will run in the user's own tooling.
- Conflicts or ambiguities in the request, such as two requirements that pull against
  each other.

**What not to ask about:** mechanics the references already settle, such as section
order, the delimiter type for a given kind of content, or the outer fence around the
delivered prompt. Decide these and explain them in the design notes if they're not
obvious.

**How to ask:**
- Ask everything in one turn, so the user isn't pulled through many rounds. If the
  `ask_user_input_v0` tool is available, use it: up to 3 questions per call, 2–4 short
  options each, with the recommended option first and labelled "(Recommended)". Put the
  most important decisions first. Ask any remaining questions in prose in the same
  message, or in a second round if they depend on the first answers.
- Without the tool, use a short numbered list. Give each question one line saying why
  it matters, the options, and the recommendation with a one-line reason.
- Base each recommendation on the references and on what the user has said. Never offer
  a bare question without a recommendation.
- If the user replies "go with your recommendations" or similar, use them all.

**Doubts that appear mid-build.** If a new decision or doubt comes up while designing
(conflicting constraints, a fact you'd otherwise invent, a pattern that might or might
not apply), stop and ask the same way before continuing. Don't fill the gap and mention
it afterward.

**When there is nothing to ask.** If the user's request already settles every important
decision, skip the questions and build the prompt.

### 3. Route and design

Read the mandatory references in order. Then check the request against the conditional
signals, if any exist, and read any reference that matches. Build the prompt following everything you
read, using only the elements and patterns the task needs.

### 4. Check before delivering

Run the pre-delivery checklist of every reference you read. Fix anything that fails
before showing the prompt to the user.

### 5. Deliver

Give the user:
1. **The prompt**, in a single fenced block they can copy. Wrap it in `~~~` (or a
   four-backtick fence), because the prompt often contains its own ```` ``` ````
   delimiters.
2. **A parameter table** (templates only): parameter, description, example value. For
   templates, also include the extras described in section 9 of the prompt-templates
   reference (data flow for chains, a suggested test set, and a component map when
   useful).
3. **Decisions**, in a short list: what the user chose and which recommendations they
   accepted, so the record of why the prompt is built this way travels with it.
4. **Design notes**, only if useful: a sentence or two on any non-obvious choice, such as
   why XML tags instead of backticks, or why a constraint is conditional.

For a multi-stage workflow, deliver one fenced prompt per stage, number and label each
stage, and state what the user does between stages. Section 11 of the prompt-patterns
reference covers this.

Don't run the prompt or show a sample model response unless the user asks. If the user
asks for a filled-in example of a template, add one after the parameter table.

### 6. Iterate

When the user reports bad output from the prompt, find the cause instead of rewriting
from scratch. Wrong task or scope points to Instructions. Ignored limits point to
Constraints (vague, contradictory, or too many). Misread situations point to Context.
Wrong shape points to Output Format. Content treated as instructions points to
Delimiters. If the structure is sound but the output is generic, hedged, discursive when
a decision was needed, or bloated, the cause is the wording: use the diagnosis table in
the linguistic reference. Change only the responsible section or element and explain the
fix.
