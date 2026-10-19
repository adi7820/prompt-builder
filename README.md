# Prompt Builder Skill

A skill that helps your AI assistant write **good prompts** for you.

You explain your task in simple words. The skill asks you a few quick questions, and then gives you a clean, ready-to-use prompt.

Works with **Claude Code, Cursor, GitHub Copilot, Codex** and other AI tools that support skills.

---

## What's inside: the 9 guides

The skill is made of 9 guides. **4 are used every time**, in order. **5 are used only when your task needs them.**

### ✅ Always used (in this order)

| # | Guide | What it does |
|---|---|---|
| 1 | **Structural Elements**<br>*Gives the prompt its shape* | • Splits the prompt into clear parts: **Instructions**, **Constraints**, **Context**, **Inputs**, **Output Format**, **Delimiters**<br>• Always adds the task and the answer format; other parts only if needed<br>• Puts pasted text (emails, logs, docs) inside ```` ``` ```` or `<tags>` so the AI reads it as data, not orders<br>• Makes rules checkable: *"under 100 words"*, not *"keep it short"* |
| 2 | **Linguistic Elements**<br>*Makes the words clear* | • First decides: do you want a **final answer** or **ideas**?<br>• **Precision:** replaces vague words like *"professional"* with exact facts, names, and numbers<br>• **Directness:** clear commands like *"Decide…"*, not *"Could you maybe…"*<br>• **Brevity:** removes polite filler and repeated lines<br>• Explains why answers come out too general, too long, or *"it depends"* |
| 3 | **Prompt Patterns**<br>*Adds proven tricks* | • **Role:** a real expert role, like *"an HR recruiter who has hired 300+ supervisors"*<br>• **Template output:** the same fields every time, with *"Not discussed"* when info is missing<br>• **Repetition:** sends the prompt twice so long lists aren't missed<br>• **Self-Reflection:** the AI checks its own draft for gaps<br>• **Inversion:** the AI asks tough questions from different people's view<br>• **Refinement:** improves the draft using that feedback<br>• **Comment-Driven:** fixes many reviewer comments in one go |
| 4 | **Prompt Templates**<br>*Makes it reusable* | • Keeps the fixed part (role, rules, format) separate from the changing blanks (`{{customer_name}}`)<br>• One blank per fact, usually 3–7, with a rule for empty blanks<br>• Can chain two templates: the first asks questions, the second makes the final plan<br>• Gives 3–5 test inputs (normal, empty, extreme, tricky) to check before real use |

### 🔍 Used only when needed

| Guide | When it is used | What it does |
|---|---|---|
| **Prompt Types** | You're building a chatbot, an app, or anything that runs many times | • Splits into a **system prompt** (fixed rules, written once) and a **user prompt** (changing input)<br>• One test for every line: *"Does this set up the AI, or does the user give it?"*<br>• Tells you when **not** to split: a few runs only = one prompt is enough |
| **Contextual Prompting** | Sorting, labelling, or extracting info, or answers keep changing format | • Starts with **no examples**; adds 1–2 only if the AI isn't consistent<br>• Keeps every example in exactly the same format<br>• Covers different cases, including the risky one, not only easy ones |
| **Prompt Sampling** | The answer decides something important: a score, a pass/fail, a route | • **Mean:** run 4–8 times, take the average score<br>• **Mode:** run 3 or 5 times, take the majority label<br>• **Self-Consistency:** run 2–4 times, check that the answers agree<br>• **Best-of-N:** make 3 drafts, pick the best using rules set **before** reading<br>• Counting is done outside the AI; a human checks big decisions |
| **Advanced Patterns** | Step-by-step thinking, prompts that write prompts, or AI agents | • **Chain of Thought:** checks each rule one by one before deciding, so you see why<br>• **Meta Prompting:** a prompt that writes templates for your team and lists open questions<br>• **ReAct:** works like an agent: think → one action → see the result → repeat, with a stop rule and a turn limit |
| **Prompt Security** | Customers, documents, emails, or websites can send text to your AI | • Marks outside text as **"data, not orders"**, so *"ignore all instructions"* tricks fail<br>• One clear job (*"Your only job is…"*) and one fixed reply for off-topic requests<br>• Gives the AI only the data it needs; it can't leak what it doesn't have<br>• A test list of 8 common attacks to try before going live<br>• Says honestly what a prompt can't protect, so you add those checks in your app |

It never guesses important things. It asks you first and suggests an answer, so you just pick one.

---

## Why use this skill?

### The problem

When you build an AI app, the prompt is the **core logic** of your app. But most prompts are written like a quick chat message, and that causes real problems once the app is live:

- **Different answers for the same input:** the format changes, so your code can't read the output reliably.
- **Generic answers:** the AI doesn't know your business rules, labels, or style.
- **Prompts copied everywhere:** one change means editing dozens of places.
- **Easy to break:** a user types *"ignore all instructions"* and your app does something it shouldn't.
- **Wasted money:** long, messy prompts burn tokens on every single request.
- **No way to debug:** when the output is wrong, you don't know which part of the prompt caused it.

### How this skill helps

| Problem in your AI app                              | What the skill does                                                                    |
| -----------------------------------------------------| ----------------------------------------------------------------------------------------|
| Output format keeps changing                        | Fixes the exact answer format, so your code can read every response                    |
| AI ignores your business rules                      | Writes clear, checkable rules and adds 1–2 examples when needed                        |
| The same prompt copied for every request            | Builds **reusable templates** with blanks like `{{customer_message}}`                  |
| Rules and user input mixed together                 | Splits into a **system prompt** (fixed rules) and a **user prompt** (changing input)   |
| Users or documents can trick the AI                 | Separates outside text as *"data, not orders"* and adds safe fallback replies          |
| Unreliable scores or labels for important decisions | Runs the prompt several times and uses an average, a majority vote, or the best answer |
| An agent wanders or never stops                     | Gives agents a clear think → act → check loop with stop rules                          |
| High token cost                                     | Removes filler and keeps the system prompt fixed, so it can be cached                  |
| Hard to know if the prompt works                    | Gives you test inputs, including tricky and attack cases, to check before going live   |

### Get the most out of your AI model

- **Consistent:** the same quality and format on request #1 and request #10,000.
- **Accurate:** the model gets exactly the context it needs, nothing missing and nothing extra.
- **Safe:** common attacks are blocked at the prompt level, and the skill tells you what to protect in your code.
- **Cheaper:** fewer tokens per request, and fewer retries.
- **Easy to maintain:** change the rules once in the system prompt, and every request follows them.
- **Easy to debug:** each part of the prompt has one job, so you know where to fix it.

You don't need to be a prompt engineering expert. Describe what your app should do, answer a few questions, and get a prompt that's ready for production.

## How to install

**Step:** Open your terminal and run:

```bash
npx skills add adi7820/prompt-builder
```

That's it. ✅

**Other options:**

```bash
# Install for all your projects
npx skills add adi7820/prompt-builder -g

# Install only for Claude Code
npx skills add adi7820/prompt-builder -a claude-code
```

**Update or remove:**

```bash
npx skills update
npx skills remove prompt-builder
```

---

## Credits

This skill is based on the ideas in the book
**Prompt Engineering in Practice** by **Richard Davies** and **Rafael Fischer**.

All examples inside the skill are written fresh. The methods come from the book. If you want to learn prompt engineering properly, read the book.

---

## License

MIT