# Clojure AI Prompts (`.cursor/rules`) + Electric Clojure v3

AI coding guidelines for Clojure & Electric Clojure v3 (ala `.cursor/rules`) to help LLMs write idiomatic Clojure code when working on large projects.

These guidelines were derived by Claude Sonnet 3.7 by reading various books & Clojure repositories with some manual intervention to clarify contradictions.

## Goal

As we enter the 4th Industrial Revolution, I hope thas this repo can be a starting point to bring idiomatic Clojure to AI models.

Currently the files are quite long and could be more concise, but I would rather err on the side of verbosity.

## How to Use These Prompts

TODO: but basically put these *.mdc under `.cursor/rules` in your project and mention them in your CLAUDE.md file.

A more convenient way to stay up-to-date with the latest versions will be to use a git submodule. I'm not sure how to do that without clobbering other files in your `.cursor/rules` path.

## Personal Conventions

- The `!` prefix notation for references like atoms, e.g. `(def !state (atom {}))` is my preference because then `(swap! !state conj ...)` looks right, as well as `@!state` to distinguish from values.
- The `!` suffix notation for effectful functions like `(defn save-user! [conn user-data] ...)` warns the user that this function is not pure.

## Knowledge Sources:

- [Elements of Clojure](http://elementsofclojure.com/) by [Zachary Tellman](https://github.com/ztellman).
- [The Joy of Clojure (2nd Edition)](https://www.manning.com/books/the-joy-of-clojure-second-edition) by Michael Fogus and Chris Houser
- [Electric Clojure v3 Tutorials (as of 2025-03-23)](https://gitlab.com/hyperfiddle/electric-fiddle/-/tree/de2bad00eb29312bae7a0641385f42cd07a218fd) by Hyperfiddle.

Elements of Clojure is the book I would have written about Clojure, if I could. All hail, Tellman.

## WARNING: Electric Clojure v3 is in Flux & Guidelines Will Be WRong

I added guidelines for Electric v3 because models with a knowledge cut-off in 2024  tend to output Electric v2 code even when I ask for v3 code.

[Electric Clojure v3](https://electric.hyperfiddle.net/) is being actively developed and some of these guidelines are almost certainly wrong – sorry about that! Hapay to fix.

PRs welcome if they improve outcomes.
