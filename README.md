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

Most people write prompts like a quick message: *"Write a reply to this customer."*

The AI then has to guess everything: who the customer is, what tone to use, how long the reply should be, what to include. So you get:

- Answers that are too general or off-topic
- A different style every time you run the same prompt
- Many rounds of "no, not like this, try again"
- Prompts that break when someone types something tricky

Writing a good prompt takes skill and time, and most people don't have either.

### How this skill helps

| Without the skill | With the skill |
|---|---|
| You guess what to write | It builds the prompt for you, step by step |
| The AI guesses what you want | It asks you first, with a suggested answer |
| Messy, one-line prompts | Clean prompts with clear sections |
| A new prompt every time | Reusable templates you fill in again and again |
| Different answers each run | The same format, every time |
| Easy to trick | Protected against common tricks |

### In short

- **Saves time:** fewer retries and less back-and-forth with the AI.
- **Better answers:** clear prompts give clear results.
- **Reusable:** make a prompt once, use it many times.
- **Beginner-friendly:** you don't need to know prompt engineering. Just describe your task.
- **Saves money:** short, clean prompts use fewer tokens.

## How to install

**Step 1:** Install [Node.js](https://nodejs.org) if you don't have it.

**Step 2:** Open your terminal and run:

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

## How to use

Just ask your AI assistant in normal words, for example:

- *"Write a prompt to reply to customer WhatsApp messages for my shop."*
- *"Make a template that summarizes meeting notes."*
- *"Why is my prompt giving different answers every time?"*

The skill starts working on its own.

---

## Credits

This skill is based on the ideas in the book
**Prompt Engineering in Practice** by **Richard Davies** and **Rafael Fischer**.

All examples inside the skill are written fresh. The methods come from the book. If you want to learn prompt engineering properly, read the book.

---

## License

MIT