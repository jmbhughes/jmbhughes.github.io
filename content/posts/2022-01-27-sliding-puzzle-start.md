---
title: Sliding puzzles - 1. my beginning
date: 2022-01-27
draft: false
---

I recently have become interested in sliding puzzles. You can read all about them
on [Wikipedia](https://en.wikipedia.org/wiki/Sliding_puzzle). In undergraduate AI,
I was ask to implement a solver for the rush-hour game, a version of a sliding puzzle.
I forgot all about it until recently when I was preparing an AI lesson to teach for
Code Connects, a coding education non-profit. I stumbled upon the 8-puzzle.

At first, it seemed pretty simple, and I thought an AI would do easy on it for
bigger boards, but I was sorely mistaken. When you think about it, the N-puzzle, or sometimes
called the N<sup>2</sup>-1 puzzle, has (N<sup>2</sup> -1)!/2 possible states. That grows
incredibly rapidly with N meaning even N=5 is very hard to solve.

There's been a plethora of research on the puzzles, and I hope to explore it, implement it, and
maybe even contribute to it.

As a start, I've begun implementing [my own version of the sliding puzzle](https://github.com/jmbhughes/slidingpuzzle).
So far, I just have implemented the core sliding puzzle and N-puzzle representations with a basic (and slow)
breadth-first search and A* search with the Manhattan heuristic. I'm also using this
as an opportunity to learn more about [Read the Docs](https://readthedocs.org/).
So, you can see [my beginning docs](https://slidingpuzzle.readthedocs.io/en/latest/index.html).
They're not automatically generating everything yet, but I also haven't written docstrings, so I wouldn't expect much.

Expect more regular updates on the developments.
