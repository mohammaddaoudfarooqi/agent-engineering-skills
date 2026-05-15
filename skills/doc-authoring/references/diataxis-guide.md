# Diataxis Documentation Framework

The Diataxis framework classifies documentation into four types along two axes:
**Practical vs. Theoretical** and **Learning vs. Working**. Each type has
distinct rules for content, voice, and structure. The central discipline is
keeping these types separate.

## The Four Types

```
                    LEARNING              WORKING
                 (acquisition)          (application)
              ┌──────────────────┬──────────────────┐
  PRACTICAL   │                  │                  │
  (doing)     │    TUTORIALS     │   HOW-TO GUIDES  │
              │                  │                  │
              ├──────────────────┼──────────────────┤
  THEORETICAL │                  │                  │
  (thinking)  │   EXPLANATION    │    REFERENCE     │
              │                  │                  │
              └──────────────────┴──────────────────┘
```

## Quick Classification

| If the user needs to... | Write a... |
|------------------------|-----------|
| Learn by doing for the first time | Tutorial |
| Accomplish a specific task they already understand | How-to Guide |
| Look up technical details during work | Reference |
| Understand why something works the way it does | Explanation |

---

## Type 1: Tutorials (Learning-Oriented)

**Purpose:** Guide a beginner through a hands-on learning experience.
The user is studying, not yet working.

### Do

- Show the end goal upfront so the reader knows what they are building
- Deliver visible, concrete results after every step
- Use first-person plural: "First, we create..." "Now we configure..."
- Provide narrative feedback: "You should see...", "Notice that..."
- Flag likely failure points before they happen
- Acknowledge accomplishment at the end: "You have built..."
- Keep steps concrete and specific — this action, this result
- Minimize content to only what the learner needs right now
- Test exhaustively — a broken tutorial destroys learner trust

### Do Not

- Do not explain concepts in detail (link to Explanation docs)
- Do not offer choices or alternatives — give one path
- Do not overload with optional information
- Do not use presumptuous framing ("In this tutorial you will learn...")
- Do not generalize or abstract prematurely
- Do not skip steps that produce no visible output — reorder so every step shows something

### Voice

Encouraging, sequential, concrete.

```
"First, create a new directory:"
"Run the following command:"
"You should see output like:"
"Notice that the file now contains..."
"You have successfully configured the application."
```

### Structure

A strict linear sequence:
1. Prerequisites
2. Step 1 (action + visible result)
3. Step 2 (action + visible result)
4. ...
5. Summary of what was accomplished + next steps

### Boundary

A tutorial TEACHES. It does not explain (link to Explanation), does not
provide reference detail (link to Reference), and does not solve an
arbitrary problem (that is a How-to Guide).

---

## Type 2: How-to Guides (Task-Oriented)

**Purpose:** Help a competent user accomplish a specific goal. The user
already knows what they want; they need directions.

### Do

- Focus entirely on the goal — action only
- Assume the reader knows what they want and can follow instructions
- Use conditional imperatives: "If you want x, do y. To achieve w, do z."
- Title with what the guide achieves: "How to deploy to production"
- Provide a logical sequence of actions grounded in the user's task
- Stay adaptable — the user's situation may differ from the example
- Cross-reference Reference docs for parameter details

### Do Not

- Do not teach or explain concepts (link to Explanation or Tutorial)
- Do not include every possible variation — focus on the common case
- Do not confuse with a tutorial — a how-to assumes prior knowledge
- Do not write from the machinery's perspective; write from the user's
- Do not pad with background context or theory

### Voice

Imperative, direct, practical.

```
"Configure the database connection in config/database.yml."
"Set the API key environment variable."
"If the connection fails, check that the port is not already in use."
"To use a custom domain, add a CNAME record pointing to..."
```

### Structure

A series of action steps, ordered logically. May include:
- Conditional branches ("If you are using Docker, do X instead")
- Prerequisites stated briefly at the top
- Verification step at the end

No preamble, no teaching, no extended explanation.

### Boundary

A how-to guide DIRECTS. It does not teach (that is a Tutorial), does not
explain (link to Explanation), and does not catalog all options (that is
Reference).

---

## Type 3: Reference (Information-Oriented)

**Purpose:** Describe the machinery — its interfaces, parameters, behaviors —
as succinctly and completely as possible. The user consults reference during
work; they do not read it start-to-finish.

### Do

- Describe with neutral, objective, factual language
- Mirror the structure of the product in the doc structure
  (organize by modules, classes, endpoints — not by user tasks)
- Be complete — document every public interface
- Be consistent — same format for every entry
- Include brief usage examples that illustrate (not instruct)
- State facts as declarative propositions: "Returns a string." "Raises ValueError."
- Provide warnings where appropriate: "You must not call X before Y."
- Use tables for parameter listings — always include name, type, default, description

### Do Not

- Do not include instruction or explanation where description suffices
- Do not mix opinions, speculation, or marketing language
- Do not depart from consistent formatting patterns
- Do not force docs into a structure that doesn't match the product
- Do not teach (link to Tutorials) or provide step-by-step guidance (link to How-to)

### Voice

Austere, authoritative, factual.

```
"Accepts a string parameter. Returns an integer."
"Raises ConnectionError if the server is unreachable."
"Default: 30 seconds. Maximum: 300 seconds."
"You must authenticate before calling this endpoint."
```

### Structure

Organized by the product's own logical structure:
- Modules → classes → methods
- API groups → endpoints → parameters
- CLI → commands → flags

Each entry in the same format. Tables for parameters. Code blocks for examples.

### Boundary

Reference DESCRIBES. It does not instruct (that is a How-to), does not
teach (that is a Tutorial), and does not explain reasoning (that is
Explanation). If a reader needs to understand why, link to Explanation.

---

## Type 4: Explanation (Understanding-Oriented)

**Purpose:** Provide context, background, and reasoning. Answers the
implicit question "Can you tell me about...?" The user is studying,
seeking deeper understanding.

### Do

- Make connections to other things — even outside the immediate topic
- Provide historical context and design rationale
- Discuss the bigger picture: choices, alternatives, trade-offs
- Weigh alternatives and consider multiple perspectives
- Admit opinion where appropriate and offer reasoned judgments
- Use "why" questions as writing prompts
- Title with an implicit "About": "About authentication", "Security model"
- Draw clear boundaries around topics

### Do Not

- Do not include step-by-step instructions (that is a How-to)
- Do not include detailed specifications (that is Reference)
- Do not let instructional content creep in
- Do not treat explanation as optional or a luxury — it is critical
- Do not scatter explanation fragments across other doc types

### Voice

Discursive, reflective, contextual.

```
"The reason we chose PostgreSQL over MongoDB is..."
"This approach works well for read-heavy workloads, but..."
"Historically, the system used polling. We switched to WebSockets because..."
"Some teams prefer X. This can work, but Y is generally more maintainable because..."
```

### Structure

Organized by topic or concept — not by product structure or user task.
Suitable for reading away from the keyboard. Paragraphs, not tables.

Common patterns:
- Architecture documents
- Design decision records (ADRs)
- "How X works" deep dives
- Background on conventions or patterns used

### Boundary

Explanation EXPLAINS. It does not instruct (that is a How-to), does not
teach step-by-step (that is a Tutorial), and does not catalog interfaces
(that is Reference).

---

## Mixing Types Is the Primary Failure Mode

The most common documentation problem is mixing types within a single document.

| Symptom | What Went Wrong |
|---------|----------------|
| Tutorial has long paragraphs explaining concepts | Tutorial absorbed Explanation |
| How-to guide teaches background before each step | How-to absorbed Explanation |
| Reference includes step-by-step setup instructions | Reference absorbed How-to |
| Architecture doc includes "run this command" steps | Explanation absorbed How-to |
| README tries to be tutorial, reference, and guide | Everything mixed |

### The fix

When you feel the urge to include content from another type, **link** to it
instead. For example:

- In a Tutorial: "For more on how authentication works, see [About Authentication](docs/architecture.md#authentication)."
- In Reference: "For a step-by-step guide to setting up the database, see [How to Configure the Database](docs/developer-guide.md#database-setup)."
- In a How-to: "For background on why we use this approach, see [ADR 003](docs/adr/003-authentication-strategy.md)."

---

## Classification Decision Tree

```
Is the reader learning or working?
  |
  LEARNING --> Is the content practical or theoretical?
  |             |
  |             PRACTICAL --> TUTORIAL
  |             THEORETICAL --> EXPLANATION
  |
  WORKING --> Is the content practical or theoretical?
                |
                PRACTICAL --> HOW-TO GUIDE
                THEORETICAL --> REFERENCE
```

### Edge cases

| Document | Why It's This Type |
|----------|--------------------|
| README | Hybrid — not strictly one type. Keep it brief, link to typed docs. |
| CHANGELOG | Reference — factual record of changes. |
| CONTRIBUTING | How-to — directions for accomplishing a task (contributing). |
| ADR | Explanation — reasoning behind a decision. |
| API docs | Reference — factual description of interfaces. |
| Getting Started | Tutorial — guided first-time experience. |
| Architecture doc | Explanation — context and reasoning about design. |
| Deployment guide | How-to — steps to accomplish deployment. |
| Config reference | Reference — factual listing of options. |
| FAQ | Avoid. Content belongs in one of the four types instead. |
