# wakaran

[日本語](README.md) | English

When you let an AI agent handle implementation and design, you increasingly get asked to make calls in territory you don't know well.
Asked "should the lock-wait timeout be 2 seconds or 3?", you have nothing to answer with.
Ask for an explanation and you get a fluent one back, but reading it doesn't leave you able to decide.
What you don't understand isn't the content of the explanation. It's what you're being asked to decide in the first place.

wakaran is a Claude Code plugin for handling this state.
It doesn't take the decision off your hands; it reframes the decision as "a factual question about your own situation."
A situational question is one you can answer even without domain familiarity.

It has three parts, one for each kind of "I don't know."

- "I don't know what I'm being asked to decide" → **`/wakaran`**: writes the four-line chain (goal, current work, link, done condition), picks a mode from where the chain stops writing, and narrows the decision
- "I don't know if this approach is really right" → **`/critic`**: hands a single decision memo to an agent with no conversation history, and reports back contradictions and gaps
- "I keep getting stuck in the same place" → **`/catchup`**: aggregates the Learnings left behind by `/wakaran` and turns recurring gaps into a reading list

## When to use it, and when to stop

It's a narrow tool. Used off-label, it stops narrowing the decision and instead becomes a way to swallow a plausible-sounding default whole.

- Use it once per confusion. Read the output, answer the situational question, act.
- Wanting to run it a second time on the same spot isn't confusion, it's a knowledge gap. Route it to `/catchup` instead. A topic `/catchup` marks with ⚠ is a signal that it's time to stop repeating this tool there and read instead.
- Mode C, "Lean on the default," only works for technology choices. Product and organizational trade-offs have no de facto standard, and asking for a default there fabricates a baseline with no basis.
- The confidence tags on the output (Measured, Read, Assumed) are a priority order for what to verify first, not a guarantee of correctness. Even "Read" is the agent's own self-report.
- Reading only the default and skipping the "conditions not to choose it" is the most common misuse. It erases the point of narrowing the decision.
- When the chain check finds you can't even write the goal (Mode B), don't force your way into Mode A. What you narrowed down to has drifted from the goal.
- This is a tool for sorting out your own confusion, not one for settling an argument with someone else.
- If you can't answer the two self-check questions at the end of the output, don't adopt the default.
- `/critic` reviews only the target file and code or documents explicitly referenced from it. Facts discussed only in conversation are not reviewed; only missing information or rationale needed for the decision can come back as a gap.

## What `/wakaran` writes

### The four-line chain

First it writes these four lines.

1. The goal of the ticket or request
2. What you're currently working on
3. How 2 connects to 1
4. The done condition

It doesn't ask you "what don't you understand." Being able to name your own symptom already assumes a kind of understanding you don't have when you're out of your depth.
Instead, it looks at where in the four lines you stop being able to write.

| Where the chain breaks | Mode |
|---|---|
| Can't write the goal in line 1 at all | B. Report the broken link |
| Can write all four lines, but several points still need deciding | A. Narrow |
| One point needs deciding, with several candidates | C. Lean on the default |
| The break is in ordering or state transitions | D. Diagram |
| All four lines are written, and it's already decided | E. Critique |
| Can't write the chain because a term's meaning is unclear | F. Terms |

It states the chosen mode and the reason in one line, so if you disagree you can rerun it with an explicit `A` through `F` argument.

### The default and conditions not to choose it

It doesn't build a pros/cons table. A comparison table is only readable to someone who already has evaluation criteria.
Instead it names one **default** and lists the **conditions not to choose it**.
Your job becomes checking whether those conditions apply to your situation, which is no longer a technical question.

In Mode C, the default is limited to a sourced de facto standard: official documentation, a chapter in a well-known book, a framework's default behavior.
It avoids phrases like "generally speaking," and any claim without a nameable source gets tagged Assumed.
Where no de facto standard exists, it says so.

### Confidence tags

Every claim in the output carries one of the following tags.

- **Measured**: verified by running it
- **Read**: referenced the relevant part of code or documentation, with a path or source attached
- **Assumed**: not verified

You only need to read the Assumed lines.

### The situational question and self-check

It asks you at most one situational question per mode.
What it asks is a fact about your situation, such as "can you tolerate an irreversible change" or "is it worth half a day to verify," never a technical choice.

At the end sit two self-check questions.
State in your own words, in one line, what the condition not to choose the default is, and which of the Assumed lines you'll verify yourself.
It doesn't score the answers. The point is a self-judgment: if you can't answer, don't adopt the default.

### Learnings

At the end of the output file it leaves up to five lines of Learnings plus a one-line source.
`/catchup` harvests this section.

## `/critic`

Run it the moment you think "is this approach really right?"
Grading your own plan inside the same session doesn't help: you read the same premises in the same order and inherit the same blind spots.
So the critique happens in a separate agent with no conversation history, and you hand it a single file, nothing else.

1. Write the current state of the discussion to a file: what you're trying to decide, the current plan, its rationale, the plans you dropped and why, and your premises. Tag every claim Measured, Read, or Assumed. Anything you only discussed but didn't write down cannot be reviewed; if the document lacks information or rationale needed for the decision, the agent reports that absence as a gap.
2. Hand the `wakaran:critic` agent the file path only, no summary or extra context from the conversation.
3. Show the returned points to the user and ask only one question: which points to accept. Don't ask the user to judge whether a point is technically correct — leave those marked Unresolved.

The agent carries no conversation history, so it isn't dragged along by the flow of the main session.
It doesn't propose alternatives; each point it raises carries a confidence tag and a note on what happens if you ignore it, and it keeps the whole report within about 1,000 characters.
The agent `wakaran:critic` is `/critic`'s implementation.

## `/catchup`

Aggregates the Learnings sections across wherever decision memos live.
It ranks topics by recurrence, keeps the top three, and attaches one book (down to the chapter) and one article to each.
It only lists a book once its existence is confirmed on a publisher page or a major bookstore site.
Each topic gets a one-line note on what reading it will let you decide. The point of reading is not to accumulate knowledge but to be able to make the same call yourself next time.
Topics that have recurred three times or more get an ⚠.

## Install

To try it, point directly at this directory.

```
claude --plugin-dir /path/to/wakaran
```

To install it properly, go through a marketplace.

```
claude plugin marketplace add ug23/wakaran
claude plugin install wakaran@wakaran
```

## Usage

The formal names are `/wakaran:wakaran`, `/wakaran:critic`, and `/wakaran:catchup`; you can use the short form as long as no other skill shares the same name.

- `/wakaran`: write the chain and let it pick a mode
- `/wakaran 寄せて`: pin it to Mode C
- `/wakaran A`: pin the mode by letter (`A` through `F`)
- `/catchup 8w`: aggregate the last eight weeks of Learnings
- `/critic`: write the current discussion to a file first, then critique it
- `/critic path/to/memo.md`: critique that file directly, skipping the write-up step

The output goes wherever your project's CLAUDE.md says decision logs or discussion notes live; otherwise it goes to `docs/wakaran/`.

All three skills are user-invoked commands: Claude does not start them automatically. Mode E of `/wakaran` directly delegates to the bundled `wakaran:critic` agent.

## Design notes

Ask someone unfamiliar with a domain "which of these two options is better," and they end up choosing with nothing to base it on.
Adding more explanation doesn't fix this. The more fluent the explanation, the more it erases the "this feels thin" unease a human would notice when drafting it themselves, and the harder it becomes to notice that you still can't decide.
That's why wakaran never hands the evaluation of which technology is better back to the user.

The same reasoning is why it doesn't ask you to pick your own symptom.
The first version had you pick from one question among symptoms like "I don't know what I'm being asked to decide" or "I understand the options but can't choose." Testing it, people said they couldn't tell which one applied.
Picking the symptom was itself a judgment call.
Deciding by where the chain breaks replaced that.

A training wheel with no exit for harvesting what you've learned just repeats the same confusion indefinitely.
`/wakaran`'s Learnings section and `/catchup` exist so this tool doesn't stay a symptomatic treatment forever; they turn frequently recurring topics into reading.
Training wheels are meant to eventually come off.
Once `/catchup`'s ⚠ marks stop increasing for a given area, it's fine to take them off there.

## Related

- quiz-me ( https://github.com/juliaoesterle/quiz-me ): a skill that asks free-form comprehension questions about your recent diff before you push. wakaran narrows down a decision; quiz-me checks understanding after implementation. They combine well.
- evidence-labeling-protocol: a skill that tags claims with evidence labels, close in spirit to wakaran's confidence tags (URL unverified, name only).

## License

MIT. See `LICENSE`.
