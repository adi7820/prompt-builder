# Prompt Builder Skill

A skill that helps your AI assistant write **good prompts** for you.

You explain your task in simple words. The skill asks you a few quick questions, and then gives you a clean, ready-to-use prompt.

Works with **Claude Code, Cursor, GitHub Copilot, Codex** and other AI tools that support skills.

---

## What it can do

## What it can do

This skill has 9 small guides inside it. Some are used **every time**. Others are used **only when needed**.

### ✅ Always used (for every prompt, in this order)

| # | Guide | What it does, in simple words |
|---|---|---|
| 1 | **Structural Elements** | Gives your prompt a clear shape: the task, rules, background, inputs, and how the answer should look. |
| 2 | **Linguistic Elements** | Makes the words clear and to the point. No confusing or extra words. |
| 3 | **Prompt Patterns** | Adds smart tricks, like giving the AI an expert role or a fixed answer format. |
| 4 | **Prompt Templates** | Turns your prompt into a reusable template with blanks like `{{customer_name}}`, so you can use it again and again. |

### Used only when needed

The skill reads your request and picks these guides **only when they fit**:

| Guide | When it is used | What it does |
|---|---|---|
| **Prompt Types** | You are making a chatbot or app | Splits the prompt into a **system prompt** (fixed rules) and a **user prompt** (changing input). |
| **Contextual Prompting** | Sorting or labelling tasks, or answers coming out in different formats | Adds 1–2 good examples so the AI follows your style every time. |
| **Prompt Sampling** | The answer decides something important, like a score or a yes/no | Runs the prompt a few times and takes the average, the majority vote, or the best answer. |
| **Advanced Patterns** | The task needs step-by-step thinking, or the AI must check other systems | Makes the AI show its thinking, write prompts for you, or work like an agent step by step. |
| **Prompt Security** | Outside people or documents can send text to your AI | Protects your prompt from tricks like *"ignore all previous instructions"*. |

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