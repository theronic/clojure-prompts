# AI Prompts for Clojure &  Electric Clojure v3 / Cursor Rules

AI coding guidelines for Clojure & Electric Clojure v3 (ala `.cursor/rules`) to help LLMs write idiomatic Clojure code when working on large projects.

These guidelines were derived by Claude Sonnet 3.7 by reading various books & Clojure repositories with some manual intervention to clarify contradictions.

## Goal

As we enter the 4th Industrial Revolution, I hope that this repo can be a starting point to bring idiomatic Clojure to AI models.

Currently the files are quite long and could be more concise, but I would rather err on the side of verbosity.

## How to Use These Clojure Prompts

If you're using Cursor, you'll want to add these `*.mdc` files under `.cursor/rules` in your workspace. Here's how:

1. `cd ~/my-project`
2. `cd .cursor/rules` or make `.cursor/rules` dir
3. `git clone git@github.com:theronic/clojure-prompts.git clojure` (now you'll have .cursor/rules/clojure/*.mdc)
4. Update your `.cursor/rules/README.md` and mention something like alongside your other architecture rules:

```markdown
## Language-Specific Rules

- **[Clojure Rules](./clojure/clojure-rules.mdc)**: Clojure development guidelines
- **[Electric Clojure v3 Rules](./clojure/electric-clojure-v3.mdc)**: Electric Clojure v3 development guidelines. Electric clojure namespaces are usually `*.cljc` files and `(:require [hyperfiddle.electric3 :as e])` in top-level namespace.
```

If you're using Claude Code, you'll want to mention the same (with corrected paths) in `CLAUDE.md`.

A git submodule may be more convenient to keep up-to-date on the latest prompts. I'll add instructions once I figure out a convenient workflow.

## Personal Conventions

- The `!` prefix notation for references like atoms, e.g. `(def !state (atom {}))` is my preference because then `(swap! !state conj ...)` looks right, as well as `@!state` to distinguish from values.
- The `!` suffix notation for effectful functions like `(defn save-user! [conn user-data] ...)` warns the user that this function is not pure.

## Knowledge Was Derived from these Sources:

- [Elements of Clojure](http://elementsofclojure.com/) by [Zachary Tellman](https://github.com/ztellman).
- [The Joy of Clojure (2nd Edition)](https://www.manning.com/books/the-joy-of-clojure-second-edition) by Michael Fogus and Chris Houser
- [Electric Clojure v3 Tutorials (as of 2025-03-23)](https://gitlab.com/hyperfiddle/electric-fiddle/-/tree/de2bad00eb29312bae7a0641385f42cd07a218fd) by [Hyperfiddle](https://www.hyperfiddle.net/).

Elements of Clojure is the book I would have written about Clojure, if I could. All hail, Tellman.

## WARNING: Electric Clojure v3 is in active development & guidelines will have mistakes

I added guidelines for Electric v3 because models with a knowledge cut-off in 2024  tend to output Electric v2 code even when I ask for v3 code.

[Electric Clojure v3](https://electric.hyperfiddle.net/) is being actively developed and some of these guidelines are almost certainly wrong – sorry about that! Happy to fix as discovered :).

PRs welcome if they improve outcomes.
