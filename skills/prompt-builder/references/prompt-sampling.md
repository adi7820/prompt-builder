# Prompt Sampling

Read this file when one generation isn't reliable enough: a score, label, or response
that drives a decision, and that can come out differently from run to run. Prompt
sampling sends the same prompt several times as independent generations, then
aggregates the results or selects among them. This file covers when sampling is worth
its cost, how to choose a technique by output type, how to design the prompt so its
outputs can be aggregated, and how to deliver the full sampling workflow.

## Contents
1. Why sample
2. Sampling fundamentals
3. Choosing a technique
4. Mean (numeric outputs)
5. Mode (categorical outputs)
6. Self-Consistency (analytical responses)
7. Best-of-N (selecting one complete response)
8. When sampling is worth it
9. Designing prompts for sampling
10. Decisions to raise with the user
11. Delivering a sampling workflow
12. Failure modes to avoid
13. Pre-delivery checklist

---

## 1. Why sample

Even a strong prompt can give different answers on different runs when temperature is
above zero, or when other sampling settings such as top_p or top_k add variation. Each
generation is one draw from a probability distribution. For low-stakes tasks, one draw is
usually enough. When the answer decides something, one draw can mislead.

Example: a scholarship committee advances any applicant essay that scores 6.5 or higher.
One call on an essay returns 6, and the applicant is rejected. Another call on the same
essay returns 7, and the applicant advances. A single generation can't tell you which
answer to trust. Several independent generations, aggregated in a structured way, can.

## 2. Sampling fundamentals

**Workflow:** one fixed prompt → N independent generations → an aggregation or
selection method → one stable result.

**Temperature.** Variation is useful for sampling only when it stays bounded.
- At **temperature 0**, outputs are close to deterministic, so sampling adds almost
  nothing. Some hosted models still vary slightly because of hardware and batching
  differences.
- At **moderate temperature**, roughly 0.5–0.9 depending on the model, outputs stay
  coherent but vary enough for aggregation to mean something. Treat this as a starting
  point and adjust per model.
- At **very high temperature**, noise grows and aggregation gets worse.

If the user's platform doesn't expose temperature, the default sampling usually varies
enough. If the platform forces deterministic output, sampling isn't useful.

**Sample size.** 2–8 generations is enough for most tasks.
- 2–4 for categorical labels and convergence checks.
- 4–8 for averaging numeric scores.
- Start at the low end when the categories are well defined and the domain is stable.
  Move toward the high end for ambiguous or open-ended inputs.
- For label voting, prefer an **odd number** (3 or 5) to reduce ties.

**Independence.** Each generation must not see the others:
- **In code:** use separate API calls with no shared history.
- **Without code:** use the same prompt in separate, new chats. Never ask for "another
  answer" in the same conversation, because the model would see its earlier answers.

**Who aggregates.** Do the averaging, counting, and spread calculation in code or by
hand, not by asking the model. A model is unreliable at arithmetic over its own outputs.
Model passes are reserved for the comparison step in Self-Consistency and for scoring in
Best-of-N.

## 3. Choosing a technique

| Output type | Technique | Typical sample size | Aggregation |
|---|---|---|---|
| A number (score, estimate, confidence) | **Mean** | 4–8 | Average, with the spread as a confidence signal |
| A label from a fixed set (severity, route, decision) | **Mode** | 3–5 (odd) | Majority vote, with the vote split as a confidence signal |
| An analytical response where the conclusion must be stable | **Self-Consistency** | 2–4 | Normalize the responses into shared fields, then compare for convergence |
| A complete response where you need the strongest one | **Best-of-N** | 3 | Score the candidates against criteria defined beforehand, then select one |

## 4. Mean (numeric outputs)

Use Mean when the prompt returns a number, such as a quality score, risk score, effort
estimate, or confidence value, and when variation between runs could change a decision.
Averaging reduces the effect of outlier runs. Don't use it for labels (use Mode) or for
complete responses (use Best-of-N).

**Report the spread too.** Keep the mean as the decision number and the spread (the
lowest and highest score, or the standard deviation) as an ambiguity signal. Agree on a
spread rule up front, such as "if the scores differ by more than 3 points, send the case
to a human reviewer".

### 4.1 Example: scholarship essay scoring

Generation prompt:
~~~
# Role
You are a scholarship committee reviewer who scores applicant essays consistently
against the committee's rubric.

# Instructions
Score the applicant essay below on a 1–10 scale using the rubric. Base the score only
on the essay.

# Rubric
- 1–3: Off-prompt, or no clear goal or evidence.
- 4–6: A clear goal with some supporting evidence; generic or loosely connected to the
  program.
- 7–8: A clear, specific goal backed by concrete experiences tied to the program.
- 9–10: All of the above, plus distinctive insight and a convincing plan.

# Applicant Essay
```text
{{essay_text}}
```

# Output Format
Return only the score as a single integer.
~~~

Five independent generations: **6, 7, 7, 6, 7**. The mean is 33 ÷ 5 = **6.6**, computed
outside the model, and the spread is 6–7.

A single run of 6 would have rejected the applicant. The mean of 6.6 clears the 6.5
cutoff, and the narrow spread shows that the runs broadly agree. The anchored rubric
matters as much as the sampling: it reduces variation at the source, and sampling
smooths what remains.

## 5. Mode (categorical outputs)

Use Mode when a prompt returns one label from a fixed set and single runs can disagree.
Generate several classifications and select the label that appears most often. This is
majority voting. The vote split shows confidence: 5/0 is clear, 3/2 is contested. Don't
use Mode for numbers (use Mean) or for complete responses (use Best-of-N).

**Ties and narrow splits.** If no label wins a clear majority, run more generations or
send the case to a human. A tie often means the input is genuinely ambiguous rather than
that the model made a mistake.

### 5.1 Example: bank complaint routing

A bank routes customer complaints as Standard, Escalate, or Regulatory. Regulatory
complaints go to compliance under a legal response deadline.

Generation prompt:
~~~
# Role
You are a complaints-handling specialist at a retail bank who routes complaints
according to the bank's handling policy.

# Instructions
Classify the complaint below with exactly one routing label, using these definitions:
- Standard: a service issue the front-line team can resolve.
- Escalate: needs a senior handler (repeat contact, high value, or strong
  dissatisfaction).
- Regulatory: alleges unfair treatment, mis-selling, discrimination, or a breach of
  consumer protection rules.

# Complaint
```text
{{complaint_text}}
```

# Output Format
Return only the label: Standard, Escalate, or Regulatory.
~~~

Five independent generations for a complaint that says the customer was pushed into an
unsuitable loan after raising concerns twice: **Regulatory, Escalate, Regulatory,
Regulatory, Regulatory.** The mode is **Regulatory, 4 of 5**, counted outside the model.

A single run returning Escalate would have sent the complaint to a senior handler instead
of compliance, which could miss the regulatory deadline. The 4/1 split is a clear
majority. A 3/2 split would justify a human check.

## 6. Self-Consistency (analytical responses)

Use Self-Consistency when the output is an analytical response that could be worded
many ways but should reach one shared conclusion, and you need confidence that the
conclusion is stable. Mode counts labels. Self-Consistency checks whether full responses
converge on the same conclusion. Wording doesn't need to match.

Workflow:
1. Run one fixed prompt as **2–4 independent generations**.
2. **Normalize** each response into the same few comparison fields, for example where
   the problem occurs, the cause, the evidence, and the recommended action.
3. Run a **comparison prompt** (a separate pass) to identify the shared conclusion, the
   divergences, and a confidence level.
4. If the responses diverge on the conclusion itself, **increase the sample size or
   send the case to an analyst** rather than forcing a consensus.

### 6.1 Example: why did sales conversion drop?

Generation prompt:
~~~
# Role
You are a sales operations analyst who explains funnel changes using only the data
provided.

# Instructions
Explain why the Q3 lead-to-customer conversion rate dropped, using the funnel data
below.

# Constraints
- Use only the provided data. Do not infer causes the data does not support.

# Funnel Data
```text
Leads: Q2 2,100 → Q3 2,950 (new paid social campaign launched July 1)
Lead-to-demo rate: Q2 18% → Q3 12%
Demo-to-close rate: Q2 25% → Q3 25%
Sales team headcount: unchanged
Overall lead-to-customer conversion: Q2 4.5% → Q3 3.0%
```

# Output Format
4–6 sentences covering where in the funnel the drop occurs, the most likely cause, the
supporting evidence, and one recommended action.
~~~

Three independent responses, normalized:

| Field | Response 1 | Response 2 | Response 3 |
|---|---|---|---|
| Where the drop occurs | Lead-to-demo stage | Lead-to-demo stage | Lead-to-demo stage |
| Cause | Lower-quality campaign leads | Campaign leads less qualified | Lead quality, *or* sales capacity limiting demo bookings |
| Evidence | Demo-to-close unchanged at 25% | Lead volume up 40%, lead-to-demo down | Volume up, headcount flat |
| Recommended action | Tighten campaign targeting | Add lead qualification before demo | Check lead sources and demo capacity |

All three agree on where the drop occurs. Two attribute it to lead quality. Response 3
adds a capacity explanation. That explanation is plausible, but the data doesn't directly
support it, since demo-to-close stayed flat. This is exactly the kind of divergence the
comparison pass should surface.

Comparison prompt (template):
~~~
# Role
You are a sales operations reviewer comparing independent analyses of the same data.

# Instructions
Identify what the responses below agree on and where they diverge. Flag any claim that
appears in only one response.

# Response 1
```text
{{response_1}}
```
# Response 2
```text
{{response_2}}
```
# Response 3
```text
{{response_3}}
```

# Output Format
1) Shared conclusion (1–2 sentences)
2) Divergences (one bullet each)
3) Confidence: high, medium, or low, with a one-sentence reason
~~~

Typical result: the shared conclusion is that the drop sits at the lead-to-demo stage
and is driven by the quality of the new campaign's leads. The capacity explanation is
listed as a divergence. Confidence is medium-to-high, pending a check of demo capacity.

## 7. Best-of-N (selecting one complete response)

Use Best-of-N when you need **one final complete response**, not an average or a vote,
and when candidates differ enough in quality to justify selecting one explicitly. Skip
it for labels (use Mode), for numbers (use Mean), and for low-stakes drafts that will be
heavily edited anyway.

Rules:
- **Define the criteria before reading any candidate.** Scoring after reading tends to
  anchor on whichever candidate came first.
- **Put factual grounding first among the criteria.** If the prompt supplies the facts, a
  candidate that adds unsupported claims loses, however polished it is. Rewarding
  invented specifics is the most common Best-of-N mistake.
- Generate every candidate from **the identical prompt**.
- If a model scores the candidates, **shuffle their order**, use the same rubric for
  each, and ask for a score per criterion before the final choice.
- Keep the rubric and scores with the chosen response, so the selection can be audited.

### 7.1 Example: price-change announcement email

Generation prompt:
~~~
# Role
You are a customer communications lead at a project-management software company who
writes clear, honest account notices.

# Instructions
Write an email to Pro plan customers announcing the price change described in the facts
below.

# Constraints
- Use only the facts provided; do not invent features, discounts, or dates.
- Keep the email between 120 and 160 words.
- Use a calm, direct tone with no promotional language.

# Facts
```text
- Pro plan price rises from $12 to $15 per user per month.
- Effective for renewals on or after June 1.
- Reason: added unlimited file storage and priority support this year.
- Customers can lock in the current price by switching to annual billing before May 15.
- Questions: billing@example.com
```

# Output Format
Subject line, then the email body ending with the contact address.
~~~

Criteria, defined before reading: (1) factual grounding, (2) the change and date are
stated in the first two sentences, (3) the customer's option and deadline are clear,
(4) tone. Each criterion is scored 1–3.

Three candidates, scored:

| Candidate | Grounding | Change and date upfront | Option clear | Tone | Total |
|---|---|---|---|---|---|
| 1 | 3 | 1 (price only in paragraph 3) | 3 | 3 | 10 |
| 2 | 1 (adds "a 20% loyalty discount") | 3 | 3 | 3 | 10 |
| 3 | 3 | 3 | 3 | 2 (one apologetic sentence too many) | **11** |

Candidate 3 is selected. Candidate 2 reads best but invents a discount, so it fails the
criterion that matters most. Sending it would create a commitment the company never
made.

## 8. When sampling is worth it

| Usually warranted | Usually not warranted |
|---|---|
| High-stakes classification or routing | Low-risk drafts that a human will edit anyway |
| Numeric scores that drive decisions (gates, cutoffs, prioritization) | Tasks that already give stable output in one run |
| Open-ended outputs where quality differences materially affect outcomes | Setups fixed at deterministic output |

**Consequence check:** if a wrong output costs more than the extra generations, sampling
is usually justified.

Sampling makes the signal more reliable, but it doesn't replace human review. Aggregated
scores, majority labels, and selected candidates should still be reviewed by a qualified
person before they drive consequential decisions. Recommend that review step whenever
the decision affects people, money, safety, or compliance.

**Cost.** Sampling multiplies token cost by N. Keep the generation prompt concise, so
each extra run adds signal rather than overhead.

## 9. Designing prompts for sampling

The generation prompt determines whether aggregation is even possible.
- **Mean:** "Return only the score as a single integer", plus an **anchored rubric**
  that describes each score band.
- **Mode:** "Return only one label: A, B, or C", with **definitions for each label**.
- **Self-Consistency:** a fixed Output Format that covers the same points every time, so
  the responses can be normalized into the same fields.
- **Best-of-N:** all the facts supplied in the prompt, plus Constraints that the
  selection criteria will check.
- Keep the prompt **identical across generations**: the same text, settings, and inputs.

## 10. Decisions to raise with the user

When this file applies, these decisions usually can't be assumed. Ask each with a
recommendation, following SKILL.md's ask-before-building step:
- **Is sampling warranted?** Apply the consequence check (section 8). Recommend against
  sampling for low-stakes or heavily edited output.
- **Which technique?** Recommend it from the output type (section 3).
- **How many generations?** Recommend from section 2: an odd number for voting, 4–8 for
  scores.
- **Thresholds and escalation rules:** the decision cutoff, the spread rule for Mean, the
  tie rule for Mode, and what counts as material divergence for Self-Consistency.
- **How will it run?** Code with separate API calls, or by hand in separate chats. Can
  the user set temperature?
- **Human review:** who reviews the aggregated result before it drives a decision.

## 11. Delivering a sampling workflow

Deliver the complete workflow, not just a prompt:
1. **The generation prompt**, designed for aggregation (section 9).
2. **The sampling plan:** technique, N, temperature guidance, and how to keep runs
   independent.
3. **The aggregation rule:** what to compute or count, the decision threshold, and the
   spread, tie, or divergence rule that triggers review.
4. **The comparison or scoring prompt**, for Self-Consistency or Best-of-N, as a template
   with one parameter per response.
5. **The review step:** who checks the result before it is acted on.

## 12. Failure modes to avoid

- **Sampling in one conversation.** Later answers see earlier ones, so the runs aren't
  independent.
- **Asking the model to average or count its own outputs.** Do the arithmetic in code or
  by hand.
- **Free-text scores or labels.** "About a 7" or "probably P1" can't be aggregated.
  Constrain the output.
- **No rubric or label definitions.** Sampling smooths variation but can't fix vague
  criteria.
- **Even N for voting**, which invites ties.
- **Forcing a consensus.** Treating a material divergence as agreement instead of
  escalating it.
- **Best-of-N criteria written after reading**, or a winner chosen for polish despite
  invented facts.
- **Sampling at temperature 0**, which pays N times the cost for near-identical
  outputs.
- **Sampling as a substitute for review** on consequential decisions.

## 13. Pre-delivery checklist

Before returning a sampling workflow, confirm:
- [ ] The consequence check justifies sampling, and the user confirmed the technique.
- [ ] The technique matches the output type, and N fits the guidance (odd for voting).
- [ ] The generation prompt constrains the output (a single integer, one label, or fixed
      fields) and includes a rubric or label definitions.
- [ ] The independence instructions are stated: separate calls or separate chats.
- [ ] Aggregation happens outside the model, with a stated threshold and a spread, tie,
      or divergence rule.
- [ ] Self-Consistency includes the normalization fields and a comparison prompt.
- [ ] Best-of-N criteria are defined first, put factual grounding first, and come with a
      scoring table or scoring prompt.
- [ ] A human review step is recommended for consequential decisions.
