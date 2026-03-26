---
title: "Git and the Price of Fidelity"
description: "Why your history is never as clean as you'd like, what causes it, and how it could be otherwise."
date: 2026-03-25
tags: ["git", "development", "open source"]
readingTime: 12
---

There’s a conversation every developer ends up having sooner or later after working with Git long enough—and that’s the one I want to talk about. No, not the one about the merge conflict that ruined your Friday—that one too, sure, but not today. I’m talking about the conversation about history.

More specifically, about why `git log` on `main` always turns into a graveyard of `fix typo`, `wip`, `this fix is the real one, seriously this time`, and commits that nobody cares about except the person who wrote them at 11 PM with way too much caffeine in their system.

So let’s talk about that tension: where it comes from, why it still exists, and how it might not have to. And yes, there’s a proposal at the end. Spoiler: I’m not going to implement it—I’m not *that* crazy.

## A Weekend in 2005

Git wasn’t designed. It was distilled.

In April 2005, Linus Torvalds—yes, the same guy who wrote the Linux kernel—suddenly found himself without a version control system after a conflict with the one they were using. And instead of looking for alternatives—like a reasonable person—he did what any developer would do: doubled down and wrote Git in about ten days.

His goals were very specific: speed, data integrity, support for distributed workflows, and the ability to handle the brutal scale of Linux kernel development.

There was no design document. No product team. No whiteboard sessions about “vision.” Just a concrete problem and an engineer with a lot of coffee and zero patience for existing tools.

What’s fascinating is that the internal structure he came up with that weekend is still, fundamentally, the same today—twenty years later (with some polish here and there). That says a lot about how solid it was.

The problem—and this explains a lot—is that it was designed for the Linux kernel workflow, and the rest of the world adopted it for everything else without questioning it much.

## How Git Works Internally (For Humans)

Before we talk about the problem, it helps to understand—at least a little bit—how Git stores history. I promise it won’t hurt… much.

### Commits, Branches, and the Graph

Every time you run `git commit`, Git creates what it internally calls an object—a snapshot of your project at that moment.

That snapshot contains three things: your files, your message, and a reference to the previous snapshot.

That reference is what chains everything together. Imagine a series of photos where each one points to the one taken before it. That’s Git history.

A branch—`main`, `dev`, whatever—is just a sticky note that says “the latest commit is here.” That’s it. When you make a new commit, the sticky note moves.

### What Happens When You Merge

When you merge two branches, Git creates a special commit with two references instead of one: one to `main`, and one to the branch being merged.

That creates the visible fork you see in graphs—the classic “knot.” And here’s where the problem begins: those two references are completely equal to Git. There’s no concept of “primary” vs “secondary” parent. They’re just parents, and Git follows both equally when traversing history.

## The Three Strategies and Their Trade-offs

Today there are three main ways to integrate a branch into `main`, and all of them force you to give something up. It’s basically choosing which finger to cut off.

### Merge `--no-ff`

The classic merge. It creates the visible knot. The branch remains connected. Git knows it was merged.

The cost: `git log main` shows every single commit from that branch. Every `fix typo`, every `wip`, every “okay *this* one is really the final fix I swear.” The history is complete and faithful—but also a mess to read.

```
main:  A───B───────────M
                      /
feat:      C───D───E─/
```

### Squash Merge

It squashes all commits from the branch into a single one and places it on `main`. The history becomes clean—one commit per feature, readable like a changelog.

The cost: the structural connection disappears. The branch is left “dangling” in the graph, as if it was never merged. Git doesn’t recognize it as integrated. You have to force-delete it because Git insists “this hasn’t been merged”—even though it clearly has.

You also lose traceability unless the commit message is well-written—and short enough that you’re willing to read it.

```
main:  A───B───S          ← S contains everything, no visible connection
feat:      C───D───E      ← dangling, no knot
```

### Rebase + Merge `--no-ff`

The idea is to clean the branch before merging. Rebase moves your commits on top of the latest `main`, as if they had always started there. Then you merge and get the knot.

The cost: commits are rewritten. Technically they’re new commits with identical content but different identity—like photocopies. If the branch is yours alone, fine. If others are working on it, you’ve just created a mess.

Three strategies, three trade-offs. None give you clean history, visible structure, and no rewriting at the same time. Pick two out of three—like “good, fast, cheap.”

## The Existing Workaround: `--first-parent`

Git *does* have an answer: `--first-parent`.

```bash
git log main --first-parent --oneline
```

This tells Git: “when walking history, only follow the first parent.” For merge commits, that’s always `main`. The result is exactly the clean view we want.

So… problem solved?

Not quite.

This clean view is opt-in. The default is still noisy. You have to remember the flag—and so does every CI tool, every changelog script, every GUI client. Miss it once, and you’re back in the commit graveyard.

And here’s the uncomfortable part: the intent existed at merge time. You *knew* you wanted a clean integration point. But that intent was never recorded anywhere tools can use automatically. It just… vanished.

## The Proposal: `symbolic-parent`

This leads to a different question: what if the problem isn’t how we *read* history, but how we *write* it?

The idea: introduce a new type of parent reference in commits—`symbolic-parent`.

Instead of two equal parents:

```
parent <hash>           ← main (always followed)
symbolic-parent <hash>  ← branch (graph only)
```

A `symbolic-parent` would tell Git:

> “This branch was merged here. This is where it came from. But don’t follow it by default.”

The result:

| Command                  | Behavior                          |
| ------------------------ | --------------------------------- |
| `git log`                | Clean — ignores `symbolic-parent` |
| `git log --graph`        | Shows the knot                    |
| `git log --full-history` | Shows everything                  |
| `git branch --merged`    | Recognizes the branch as merged   |
| `git bisect`             | Clean by default                  |

And usage:

```bash
git merge --symbolic feat
```

Clean logs by default. Visible structure. No history rewriting. No trade-offs.

Fidelity *and* readability—turns out they’re not inherently opposed.

## Why This Doesn’t Exist (And Why That Makes Sense)

Git’s object model has had a strong invariant since 2005: all parents are equal.

That simplicity is powerful. Every tool assumes that if something is a parent, it’s a real parent—no special cases.

Introducing `symbolic-parent` breaks that rule and opens a can of worms:

* How does `git revert` behave?
* How do you compute merge bases?
* What happens with older Git clients?

That last one is critical. The same repo behaving differently depending on client version is exactly what Git has avoided from day one.

There’s also a philosophical stance: Git doesn’t want to privilege one interpretation of history over another. Different teams need different views. The neutral model lets consumers decide.

It’s coherent. It even makes sense.

But it pushes the burden to the reader, not the writer.

The developer who did the merge *knew* the intent. That information got lost. Now every tool has to reconstruct—or guess—it.

## Practical Alternatives Today

If `symbolic-parent` isn’t coming to core Git, what can we actually do?

### Structured Commit Trailers

Git supports metadata in commit messages:

```
Merge-Type: symbolic
Source-Branch: feat/credit-validation
Source-Tip: abc1234def5678
```

No compatibility issues. Tools can parse it.

The downside: it’s just text unless something reads it.

### A Wrapper Tool

A CLI on top of Git that enforces conventions, reads trailers, and provides clean logs.

Sounds great—until someone runs `git merge` directly and breaks the convention.

### GUI Configuration

Tools like Git Graph in VS Code support:

```json
"git-graph.onlyFollowFirstParent": true
```

Quick fix—but only in that tool. The underlying problem remains.

## The Ideal Version Control System: A Thought Experiment

If I could design a system today:

* **Semantic parents.** Different types with different traversal behavior.
* **Sane defaults.** Clean history by default. Full history via explicit flags.
* **First-class merge intent.** Structured, not buried in text.
* **Clear separation of layers.** Low-level vs user-facing commands.
* **Versioned object model.** Compatibility handled explicitly.

Git’s core idea—the immutable commit graph—is brilliant. That part stays.

## The Elephant in the Room

None of this is going to happen.

Not because it’s impossible—but because Git has twenty years of universal adoption, a massive ecosystem, and a very conservative approach to changing its core.

And that’s reasonable.

Git didn’t win because it’s the perfect design. It won because GitHub made it social infrastructure in 2008 before the debate could even happen.

“Good enough with universal adoption” beats “better but niche” every time.

Still, it’s worth stating:

There’s a difference between *“this is how it was designed”* and *“this is how it should be.”*

Historical fidelity and readability are not mutually exclusive. Git treats them as if they were because of a very specific context—Linux kernel development in 2005.

Maybe someone builds something better someday.

The pieces are there. The need is real.

It just won’t be me—but when it happens, let me know.

---

*By the way—are you using a merge strategy I didn’t mention? Found the one true workflow I missed? I’d genuinely like to hear it. It might just be my OCD talking.*
