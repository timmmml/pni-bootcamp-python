# Afternoon Session — Slide Stack Specification

**Author:** Tim Lin · PNI Python/Computation Bootcamp, Day 1 (afternoon)
**Purpose:** This markdown is a *complete content spec* for a Beamer deck. Claude Code should render it to Beamer LaTeX. Everything that goes **on a slide** is in the bullet/equation/code body; everything meant for the **speaker** is in a `> Notes:` blockquote and should map to `\note{...}`.

---

## Conventions for the implementer (Claude Code)

- `#` at top level = document metadata (this section) — do **not** render as a frame.
- `#` used as `# SECTION:` = a Beamer `\section{}` divider (render a section title slide).
- `##` = one Beamer **frame**. The text after `##` is the frame title.
- Fenced code blocks: render with `listings` (or `minted` if available). Use a light background, small font (`\small` or `\footnotesize`), keep Python keywords highlighted. Watch underscores/asterisks — use verbatim/listings, never inline markdown emphasis inside code.
- `$...$` and `$$...$$` are LaTeX math — pass through verbatim.
- `> Notes:` blockquote → `\note{...}`. Enable notes with a `\setbeameroption{show notes}` toggle the user can comment out for the projected version.
- Tags in slide bodies:
  - `[ADVANCED]` → material for the quantitatively strong students; render as a dimmed/`\alert`-boxed aside or a `\only<2->` overlay so it can be skipped live.
  - `[SAFETY NET]` → a note for the lighter-computational students; usually belongs in `> Notes:`, not on the slide.
  - `[NOTEBOOK]` → a checkpoint where students switch to the Jupyter notebook. Render as a distinctly colored callout box (e.g. a `beamercolorbox` titled "Notebook checkpoint").
- Suggested theme: `metropolis` (clean, good code rendering) with the PNI orange (`RGB 237,125,49`) as the accent color. Fallback: `default` + `\usecolortheme{owl}`.
- Aspect ratio 16:9 (`\documentclass[aspectratio=169]{beamer}`).
- Global timing target: **Python 101 ≈ 40 min + 15 min exercise; DP ≈ 25 min + notebook interleaved; HMM ≈ 25 min lecture + 35 min coding.** Frame counts below are sized to that.

---

## Title slide

**Title:** Python as a Tool, and One Technique to Reuse: Dynamic Programming
**Subtitle:** From primitives to a working hidden Markov model
**Author / date:** Tim Lin · yl0124@princeton.edu · Day 1, Afternoon

> Notes: One-line framing to say aloud: "This morning we built a mental model of Python as a machine. This afternoon we make that model concrete in code, then learn one algorithmic idea — reusing overlapping work — and use it to write real inference code by the end of the day." Set expectations: the afternoon ends with everyone's code recovering hidden states from data they simulated themselves.

---

# SECTION: Part 1 — Python 101, a Guided Tour

> Notes: Section divider. Say: "This is not 'what is a loop.' Most of you have programmed. This is *Python-specific* — the object model from this morning, made real in code, and the two or three idioms that make research Python concise." Pitch the core at the mid level. [SAFETY NET] Pair Lola and Stephen each with a strong partner before the exercise; hand out the one-page primitives cheat sheet now.

## Roadmap for the tour

- Object model in code: identity, type, value
- Mutable vs immutable — the aliasing trap
- Scalar types and strings
- Containers: `list`, `tuple`, `dict`, `set`
- Control flow: `for`, `while`, `enumerate`, `zip`
- Functions: arguments, defaults, the mutable-default gotcha
- Two Pythonic idioms: comprehensions, duck typing
- `import` and the `__main__` pattern
- Exercise (15 min)

> Notes: Keep this slide up for ~30 s. Tell them the through-line: "every item here cashes in a claim from the morning deck." Roughly 40 min for the tour; do not over-run — the DP and HMM blocks are where the payoff is.

## The object triple: identity, type, value

Every Python object has three things (from the morning deck, now in code):

```python
x = [1, 2, 3]
id(x)      # identity: which object (its address)
type(x)    # type: what operations it supports  -> <class 'list'>
x          # value: the data/state it holds       -> [1, 2, 3]
```

- `a is b`  → do the **names** point to the *same object*? (compares identity)
- `a == b`  → do the objects have the *same value*? (compares value)

> Notes: This is the single highest-value five minutes of the tour — it is the #1 bug source for experimentalists. Demo live: `a = [1,2]; b = [1,2]; a == b` is `True` but `a is b` is `False`. Then `c = a; c is a` is `True`. Ask the room: "predict `is` vs `==` before I hit enter." Pointer back to the morning deck's identity/type/value slide.

## Mutable vs immutable: the aliasing trap

```python
x = [1, 2, 3]
y = x            # y is a NEW NAME for the SAME object
y.append(4)
print(x)         # -> [1, 2, 3, 4]   (!)  x changed too
```

```python
x = 3
y = x
y = y + 1        # rebinds y to a NEW int object
print(x, y)      # -> 3 4            x is untouched
```

**Rule:** *assignment binds names; mutation changes objects.*

- Mutable: `list`, `dict`, `set` — can be changed in place.
- Immutable: `int`, `float`, `str`, `tuple`, `bool` — never change; you rebind.

> Notes: This is literally the morning deck's two examples — run them live so the abstract claim becomes a "gotcha they watched happen." Emphasize: a function that mutates a list argument changes the caller's list. This bites everyone doing data analysis. If you want a mnemonic: "= is a label-maker, not a copier." [NOTEBOOK] optional 30-second predict-the-output cell here if pacing allows.

## Scalar types and strings

```python
n = 42            # int   (arbitrary precision in Python 3)
r = 3.14          # float (IEEE-754 double)
flag = True       # bool  (a subclass of int: True == 1)
s = "spike"       # str   (immutable sequence of characters)

f"n={n}, r={r:.2f}"        # f-string -> 'n=42, r=3.14'
```

- Division: `7 / 2 == 3.5` (true division), `7 // 2 == 3` (floor), `7 % 2 == 1` (mod).
- Beware float equality: `0.1 + 0.2 == 0.3` is `False`.

> Notes: Keep this fast — most know it. The two things worth stopping on: f-strings (they'll use them constantly for logging/plots) and float non-associativity (`0.1+0.2`), which foreshadows the numerical-underflow gotcha in the HMM. State the assumption: floats are finite-precision; equality tests on them are almost always a mistake — compare with a tolerance.

## Containers: the four workhorses

```python
xs = [1, 2, 3]              # list  — ordered, mutable
pt = (35.7, 139.8)          # tuple — ordered, immutable (good dict keys)
counts = {"a": 1, "b": 2}   # dict  — hash map, key -> value, O(1) lookup
seen = {1, 2, 3}            # set   — unordered, unique, O(1) membership
```

Indexing and slicing (lists, tuples, strings):

```python
a = [10, 20, 30, 40, 50]
a[0]        # 10        (0-indexed)
a[-1]       # 50        (negatives count from the end)
a[1:4]      # [20,30,40](start inclusive, stop exclusive)
a[::2]      # [10,30,50](step)
a[::-1]     # reversed
```

> Notes: The `dict` is *the* workhorse — it's what powers namespaces/modules (morning deck) and it's the memoization table in Part 2. Say that explicitly: "hold onto `dict` — in an hour it becomes the cache that turns exponential into linear." Slicing `[start:stop:step]` is worth one careful pass: stop is exclusive. [ADVANCED] mention `dict` is a hash table → average O(1), and that's the whole reason memoization is cheap.

## Control flow

```python
for i, val in enumerate(xs):     # index + value together
    print(i, val)

for a, b in zip(names, scores):  # iterate two sequences in lockstep
    ...

if score > 0.9:
    label = "high"
elif score > 0.5:
    label = "mid"
else:
    label = "low"

while not converged:             # loop until a condition
    step()
```

- `range(n)`, `range(a, b)`, `range(a, b, step)` — lazy integer sequences.
- Prefer iterating objects directly (`for x in xs`) over C-style index loops.

> Notes: `enumerate` and `zip` are the two that non-Python programmers miss — they'll write `for i in range(len(xs))` out of habit; show them the idiomatic form. Everything in Part 2's DP tables is built with these plus `range`.

## Functions

```python
def edit_cost(a, b, ins=1, dele=1, sub=1):   # positional + keyword defaults
    """Docstring: what the function computes."""
    return 0 if a == b else sub

edit_cost("A", "A")          # 0
edit_cost("A", "T", sub=2)   # 2   (call by keyword)
```

**Gotcha — mutable default arguments:**

```python
def bad(x, acc=[]):   # acc is created ONCE, shared across calls!
    acc.append(x); return acc

def good(x, acc=None):
    if acc is None: acc = []
    acc.append(x); return acc
```

> Notes: The mutable-default gotcha is the morning's names→objects model striking again: the default `[]` is one object bound at definition time, not per call. This genuinely surprises intermediate programmers. Say the fix pattern (`=None` then create inside) is a Python idiom worth memorizing.

## Idiom 1 — comprehensions

```python
squares  = [x*x for x in range(10)]                  # list comp
evens    = [x for x in xs if x % 2 == 0]             # with filter
lengths  = {w: len(w) for w in words}                # dict comp
pairs    = [(i, j) for i in range(3) for j in range(3)]
```

Reads as: *"collect `f(x)` for each `x` in `xs` where `p(x)` holds."*

> Notes: Comprehensions are the single most "Pythonic" construct and make research code dramatically shorter. Show the loop-equivalent side by side once so they see it's the same computation. [SAFETY NET] Tell the lighter-background students: a plain `for` loop is *always* an acceptable substitute — comprehensions are for concision, not correctness. Don't let anyone feel blocked by syntax.

## Idiom 2 — duck typing

```python
def add_all(items):
    total = items[0]
    for x in items[1:]:
        total = total + x     # asks: what does '+' mean for THESE objects?
    return total

add_all([1, 2, 3])            # 6           (ints)
add_all(["a", "b", "c"])     # 'abc'       (strings concatenate)
add_all([[1], [2], [3]])     # [1, 2, 3]   (lists concatenate)
```

"If it walks like a duck and quacks like a duck…" — operations are resolved **at runtime** by the object's type, not a declared type.

> Notes: This is the morning deck's dynamic-typing slide made concrete: one function, three behaviors, because `+` dispatches on the runtime type. State the trade-off honestly: **benefit** = generic code over many types; **cost** = type errors surface at runtime, not compile time. This is *why* modern code adds type hints (`def f(x: int) -> int`) — hints guide humans/tools but don't change execution. One sentence, then move on.

## `import` and the `__main__` pattern

```python
import numpy as np           # find package, init once, bind name 'np'
np.array([1, 2, 3])          # attribute lookup on the module object

def main():
    data = load_data()
    train(data)

if __name__ == "__main__":   # runs only when executed directly,
    main()                   # NOT when imported
```

- Run directly (`python train.py`) → `__name__ == "__main__"` → `main()` runs.
- Imported (`import train`) → `__name__ == "train"` → `main()` does **not** run.

> Notes: Tie `import` back to the morning deck: importing binds a *module object* to a name; `np.array` is just attribute lookup. The `__main__` guard matters for research code specifically: it stops expensive experiment code from firing accidentally on import. They'll graduate their HMM notebook into a script at the very end using exactly this pattern.

## Exercise (15 min)

**Task:** Given a list of `(state, observation)` pairs, return a `dict` mapping each state to a count of observations seen in that state.

```python
pairs = [("fair","3"), ("loaded","6"), ("fair","1"), ("loaded","6")]

def count_by_state(pairs):
    counts = {}
    for state, obs in pairs:
        counts.setdefault(state, {})
        counts[state][obs] = counts[state].get(obs, 0) + 1
    return counts
# expected: {'fair': {'3':1,'1':1}, 'loaded': {'6':2}}
```

Tests are `assert`-based in the notebook — green means correct.

[ADVANCED] Stretch: rewrite the inner tally as a single `collections.Counter`, and/or vectorize the counts with NumPy.

> Notes: Chosen deliberately: this is a warm-up for the emission counts they conceptually need in the HMM (fair vs loaded die). Exercises containers + control flow + dict-of-dicts. [NOTEBOOK] This is the first real notebook block — 15 min, walk the room. [SAFETY NET] Lighter students: a nested `for` loop with plain dicts is fine; ignore `setdefault`/`Counter`. Reconvene by showing one clean solution on screen.

---

# SECTION: Part 2 — Dynamic Programming

> Notes: Section divider. The framing you want, stated as a promise: "Every hard problem has a solution space that's exponential to enumerate. An algorithmic *paradigm* is a disciplined way to avoid enumerating it. Dynamic programming is one paradigm: exploit substructure so you solve each piece once and reuse it. We'll see it three times, each adding one new idea, and the third time *is* the algorithm you'll use this afternoon."

## The core idea: exploit substructure

A problem is a good fit for **dynamic programming** when it has:

1. **Optimal substructure** — the solution is built from solutions to *subproblems*.
2. **Overlapping subproblems** — the *same* subproblems recur, so caching pays off.

$$\textbf{Dynamic programming} \;=\; \text{recursion} \;+\; \text{memory.}$$

Contrast with **divide & conquer** (subproblems are *disjoint*, never repeat, so no cache needed).

> Notes: Put the two conditions up as a checklist they can apply to future problems. The one-line contrast with divide & conquer is important: DP is D&C *plus a memory*, used exactly when the subproblem dependency graph is a DAG (things get revisited) rather than a tree. Where explored: this is textbook (CLRS ch.15, Erickson ch.3). Where the open holes are: recognizing substructure in a *new* problem is the actual skill and it's not automatable — that's what the three examples train. Pointer: Bellman coined "dynamic programming" in the 1950s; the RL Bellman equation is the same idea (foreshadow for the decision-making folks).

## Two ways to fill the memory

- **Top-down (memoization):** write the natural recursion, cache each result the first time you compute it.
- **Bottom-up (tabulation):** identify the dependency order, fill a table from base cases upward.

Same answers, same complexity; different code shape.

```python
# top-down                          # bottom-up
cache = {}                          dp = [base...]
def f(state):                       for state in order:
    if state in cache: ...              dp[state] = combine(dp[...])
    cache[state] = combine(f(...))  return dp[final]
    return cache[state]
```

> Notes: Both appear in the three examples: Fibonacci shows the duality cleanly, edit distance is naturally bottom-up (a 2D table), the trellis is bottom-up over time. Tell them: bottom-up is usually faster (no recursion overhead) and gives control over memory; top-down is easier to write from the recurrence. Either is fine for this bootcamp.

## Example 1 — Fibonacci: seeing the overlap

$$F(n) = F(n-1) + F(n-2), \qquad F(0)=0,\; F(1)=1.$$

Naive recursion recomputes the same values exponentially often:

```
                 F(5)
             /          \
          F(4)          F(3)          <- F(3) computed here
        /     \        /    \
     F(3)     F(2)   F(2)   F(1)      <- ...and AGAIN here
```

- Naive cost: $O(\varphi^{\,n})$, $\varphi=\tfrac{1+\sqrt5}{2}\approx1.618$ (exponential).
- Memoized / tabulated: $O(n)$ time, $O(1)$ space bottom-up.

> Notes: Draw (or animate) the recursion tree and physically circle the repeated `F(3)`, `F(2)` nodes — *that picture is the definition of overlapping subproblems*. This is the "why memory helps" example; it has trivial substructure (just a sum) so attention stays on the reuse. Say the punchline: caching turns an exponential tree into a linear chain.

## Example 1 — Fibonacci: the code

```python
# top-down (memoization)
def fib(n, cache={}):
    if n < 2:
        return n
    if n not in cache:
        cache[n] = fib(n-1, cache) + fib(n-2, cache)
    return cache[n]

# bottom-up (tabulation), O(1) space
def fib_bu(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a
```

[NOTEBOOK] Checkpoint 1: run both, time naive vs memoized for `n = 35`.

> Notes: The timing demo is the "aha" — naive `fib(35)` visibly hangs, memoized returns instantly. Note the mutable-default `cache={}` here is the *intended* shared-state version (callback to the earlier gotcha slide — same mechanism, now used on purpose). [NOTEBOOK] 3–4 min. Ask: "what's the new structural idea?" Answer: a cache keyed by subproblem. Next example adds a *choice* inside the recurrence.

## Example 2 — Edit distance: the problem

**Edit distance** $D(a, b)$ = minimum number of single-character **insertions, deletions, or substitutions** to turn string $a$ into string $b$.

Example: `kitten` → `sitting` has edit distance 3
(k→s substitute, e→i substitute, insert g).

Why it's a step up from Fibonacci:

- state is **2-dimensional** (a prefix of $a$ *and* a prefix of $b$),
- the recurrence has a genuine **choice** (min over three operations),
- we'll want to recover *which* edits — the solution, not just its cost.

> Notes: This is the load-bearing middle example. It introduces the three ideas the trellis/Viterbi needs: 2D table, min-over-choices, and traceback. Motivate with the sequence-alignment connection now — this is *exactly* Needleman–Wunsch / Smith–Waterman from bioinformatics (Victor will recognize it). Say: "biologists have been doing this DP on DNA since the 1970s; you're about to write it."

## Example 2 — Edit distance: the recurrence

Let $D[i][j]$ = edit distance between prefixes $a[1{:}i]$ and $b[1{:}j]$.

Base cases (turn a prefix into the empty string):
$$D[i][0] = i, \qquad D[0][j] = j.$$

Recurrence:
$$
D[i][j] =
\begin{cases}
D[i-1][j-1], & a_i = b_j \quad(\text{free match})\\[4pt]
1 + \min\!\begin{cases}
D[i-1][j] & (\text{delete } a_i)\\
D[i][j-1] & (\text{insert } b_j)\\
D[i-1][j-1] & (\text{substitute})
\end{cases} & a_i \ne b_j
\end{cases}
$$

Answer: $D[m][n]$.  Complexity: $O(mn)$ time and space.

> Notes: Walk each branch's *meaning* — this is where the userPreference for motivating every term pays off. "Delete $a_i$" means we already know how to align $a[1{:}i-1]$ with $b[1{:}j]$, and we pay 1 to drop $a_i$. Etc. Emphasize the overlap: $D[i-1][j-1]$ is needed by three different cells, so the table reuse is the DP payoff. $O(mn)$ vs the exponential number of edit sequences.

## Example 2 — Edit distance: table + traceback

```python
def edit_distance(a, b):
    m, n = len(a), len(b)
    D = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1): D[i][0] = i
    for j in range(n+1): D[0][j] = j
    for i in range(1, m+1):
        for j in range(1, n+1):
            if a[i-1] == b[j-1]:
                D[i][j] = D[i-1][j-1]
            else:
                D[i][j] = 1 + min(D[i-1][j],     # delete
                                  D[i][j-1],     # insert
                                  D[i-1][j-1])   # substitute
    return D[m][n]
```

To recover the *edits*: store the argmin (a **backpointer**) per cell and walk from $D[m][n]$ back to $D[0][0]$.

[NOTEBOOK] Checkpoint 2: fill the table for `kitten`/`sitting`, print it, read off 3.

> Notes: The **backpointer** idea is the one to hammer — it reappears verbatim in Viterbi. "The table gives you the *cost*; the backpointers give you the *solution*." [NOTEBOOK] ~5 min: they fill the DP loop, print the table as a grid, and eyeball the traceback. [ADVANCED] mention rolling-array trick reduces space to $O(\min(m,n))$; and that with weighted operation costs this becomes global sequence alignment.

## Example 3 — Trellis: DP over time

Now the subproblems are indexed by **time** and **state**.

Setup: $T$ time steps ("layers"), $K$ states per step.
- $c_t(j)$ = cost of being in state $j$ at time $t$ (**node cost**).
- $w(i, j)$ = cost of moving from state $i$ to state $j$ (**transition cost**).

Goal: the minimum-cost path that picks one state per time step.

```
t=1        t=2        t=3
[s1]------>[s1]------>[s1]
   \  X   /   \  X   /
[s2]------>[s2]------>[s2]     each edge has cost w(i,j),
   /  X   \   /  X   \        each node has cost c_t(j)
[s3]------>[s3]------>[s3]
```

> Notes: This is the geometric heart of the whole afternoon. Draw the layered graph. Say the target sentence out loud: **"Keep this picture. In one hour it becomes a hidden Markov model, and the costs become negative log-probabilities."** New structural idea vs edit distance: state indexed by (time, discrete state) with a full $K\times K$ transition, and we sum over a *sequence* of decisions.

## Example 3 — Trellis: the recurrence

Let $V_t(j)$ = cost of the best path that **ends at state $j$ at time $t$**.

Base case:
$$V_1(j) = c_1(j).$$

Recurrence:
$$V_t(j) = c_t(j) + \min_{i}\big[\, V_{t-1}(i) + w(i, j)\,\big].$$

Answer: $\min_j V_T(j)$, with backpointers $\text{bp}_t(j) = \arg\min_i[\cdots]$ to recover the path.

Complexity: $O(T K^2)$ — versus $K^T$ paths if enumerated.

> Notes: Every term has a concrete reading: $V_t(j)$ is "best cost to arrive at $j$ at time $t$"; the inner $\min_i$ says "the best route into $j$ comes through whichever predecessor was cheapest," reusing the whole previous column $V_{t-1}$ — that reuse is the overlapping-subproblems payoff. $O(TK^2)$: $T$ columns × $K$ targets × $K$ sources. Backpointers again → Viterbi. [NOTEBOOK] Checkpoint 3: implement `min_cost_path` with backpointers on a small random trellis. This IS the Viterbi code with cost = −log prob.

## [ADVANCED] The same DP, two "modes"

Replace the operator inside the recurrence:

$$
\underbrace{\min_i\big[V_{t-1}(i) + w(i,j)\big]}_{\text{best single path (Viterbi)}}
\qquad\longleftrightarrow\qquad
\underbrace{\operatorname{logsumexp}_i\big[V_{t-1}(i) + w(i,j)\big]}_{\text{sum over all paths (forward)}}
$$

Same recursion, two **semirings**: $(\min, +)$ (tropical) gives the *best* path; $(\operatorname{logsumexp}, +)$ — equivalently $(+, \times)$ in probability space — gives the *total* path mass. Both are message passing on a chain (max-product vs sum-product).

> Notes: [ADVANCED] One slide, ~90 s, aimed at Maryam / the math-strong; skippable for the room. The point: you don't write two algorithms, you write one and swap the combining operation. This is the unifying view that makes Viterbi and the forward algorithm obviously "the same thing." Pointer: generalized distributive law (Aji & McEliece 2000); Bishop PRML ch.8/13.

## Part 2 wrap: what you now hold

- Recognize DP: **optimal substructure + overlapping subproblems**.
- Fill it two ways: **top-down memo** or **bottom-up table**.
- Recover solutions, not just costs, with **backpointers**.
- The **trellis** recurrence, $V_t(j) = c_t(j) + \min_i[V_{t-1}(i) + w(i,j)]$, is one operator-swap away from HMM inference.

Where DP shows up in your world: sequence alignment (genomics), Viterbi (this afternoon), the **Bellman equation** of reinforcement learning, CTC in speech models.

> Notes: 30-second recap slide before the break. Name the RL connection explicitly for the decision-making people (Kristen, Leo): value iteration $V(s)\leftarrow\max_a[r + \gamma\sum P V]$ is DP over states with a discount — same shape, and it's a natural next lecture if the bootcamp expands. Then break before the HMM block; make sure environments are healthy for the coding session.

---

# SECTION: Part 3 — Mini-Project: A Hidden Markov Model

> Notes: Section divider. Frame it: "You already wrote the hard part — the trellis. Now we give it a probabilistic interpretation and use it to *infer hidden states from data*. The model is the occasionally-dishonest casino; by the end your code will look at a stream of dice rolls and tell you when the casino was cheating." 25 min lecture, 35 min scaffolded coding.

## The setup: hidden states, visible observations

A system moves through **hidden states** $z_{1:T}$, $z_t \in \{1,\dots,K\}$, emitting **observations** $x_{1:T}$ we can see.

Running example — **the occasionally-dishonest casino:**
- hidden state $z_t \in \{\text{fair}, \text{loaded}\}$ (we can't see which die),
- observation $x_t \in \{1,\dots,6\}$ (the roll we *do* see),
- the casino sometimes swaps dice between rolls.

Inference question: given the rolls, **when was the loaded die in play?**

> Notes: The casino is the canonical teaching HMM (Durbin, Eddy, Krogh & Mitchison 1998) — Victor will recognize it from genomics. Keep emissions *discrete/categorical* precisely so the emission model is a trivial lookup and attention stays on the recursion. Map to the trellis: hidden state = which state per layer; observation = data attached to each layer.

## The model: three parameter groups $\theta = (\pi, A, B)$

$$
\pi_i = P(z_1 = i) \quad\text{— initial state distribution (where it starts)}
$$
$$
A_{ij} = P(z_t = j \mid z_{t-1} = i) \quad\text{— transition matrix (}K\times K\text{, how states switch)}
$$
$$
B_i(x) = P(x_t = x \mid z_t = i) \quad\text{— emission model (}K\times M\text{, how a state generates data)}
$$

Two defining assumptions:
- **Markov property:** $z_t \perp z_{1:t-2} \mid z_{t-1}$ (future depends on the past only through the present state).
- **Emission independence:** $x_t \perp \text{everything} \mid z_t$ (an observation depends only on its own hidden state).

> Notes: Motivate every symbol — this is the userPreference-critical slide. $\pi$ = a length-$K$ probability vector; $A$ = rows sum to 1, row $i$ is "where do I go from state $i$"; $B$ = row $i$ is state $i$'s emission distribution over the $M$ symbols. State the assumptions explicitly and flag that *everything downstream depends on them* — the whole factorization and the $O(TK^2)$ efficiency come from these two conditional independences.

## The model: joint factorization

Under the two assumptions, the joint probability of a full trajectory factorizes:

$$
P(z_{1:T}, x_{1:T}) = \pi_{z_1}\, B_{z_1}(x_1) \prod_{t=2}^{T} A_{z_{t-1} z_t}\, B_{z_t}(x_t).
$$

This is the generative story: pick $z_1$ from $\pi$, emit $x_1$ from $B_{z_1}$, transition via $A$, emit again, …

[NOTEBOOK] `sample_hmm(pi, A, B, T)` is provided — read it to see this story in code.

> Notes: Read the factorization as a left-to-right generative recipe; that recipe *is* the provided sampler. Point out this is a probabilistic graphical model (a chain). If you write `sample_hmm` live instead of pre-giving it, it doubles as teaching the model — but that costs ~5 min you may not have. Pre-give it and just read it aloud.

## Three questions you can ask an HMM (Rabiner's framing)

1. **Evaluation** — how likely is this data? $\;P(x_{1:T} \mid \theta)$  → **forward algorithm**
2. **Decoding** — what hidden path best explains it? $\;\arg\max_z P(z \mid x)$  → **Viterbi**
3. **Learning** — what parameters fit the data? $\;\arg\max_\theta P(x \mid \theta)$  → **Baum–Welch (EM)**

Today: we build **evaluation** and **smoothing** (forward–backward). Decoding is your Viterbi trellis. Learning is the [ADVANCED] extension.

> Notes: This taxonomy (Rabiner 1989, the classic tutorial — put it on the slide as a pointer) organizes the whole HMM world. Be explicit about scope: forward + backward + posterior today; Viterbi is a one-line swap from Part 2; Baum–Welch is the honest open hole we leave for the strong students. Naming all three now prevents "wait, which one are we doing?" confusion later.

## Why naive evaluation is hopeless

$$
P(x_{1:T}) = \sum_{z_{1:T}} P(z_{1:T}, x_{1:T})
$$

sums over $K^T$ possible state sequences — exponential. But it's the **same trellis**: replace "cost" with "probability" and $\min$ with $\sum$, and it's $O(TK^2)$.

> Notes: This is the payoff of Part 2. $K^T$ is astronomically large ($2^{300}$ for the casino with $T=300$). The forward algorithm computes the exact sum in linear time by the trellis DP — reusing $\alpha_{t-1}$ instead of re-walking every path. Same overlapping-subproblems argument as Fibonacci, now over probabilities.

## Forward algorithm

Define the **forward variable**
$$
\alpha_t(i) \equiv P(x_{1:t},\, z_t = i)
$$
— "joint probability of everything observed so far *and* being in state $i$ now."

$$
\alpha_1(i) = \pi_i\, B_i(x_1)
$$
$$
\alpha_t(j) = B_j(x_t) \sum_{i} \alpha_{t-1}(i)\, A_{ij}
$$
$$
P(x_{1:T}) = \sum_i \alpha_T(i)
$$

[ADVANCED] Vectorized: $\;\alpha_t = \operatorname{diag}\!\big(B(x_t)\big)\, A^\top \alpha_{t-1}$ — a repeatedly-applied linear operator on a belief vector.

> Notes: Read the recurrence: $\sum_i \alpha_{t-1}(i)A_{ij}$ is "collect probability mass flowing in from every predecessor $i$," then $B_j(x_t)$ weights by how well state $j$ explains the current observation. This is the Part-2 trellis with $\sum$ instead of $\min$. [ADVANCED] The linear-operator form is a direct gift to Maryam — "how a system evolves under a repeatedly-applied operator" is her stated interest; filtering is that iterated operator, and it's also the morning deck's "Python as control plane → NumPy kernel" (the whole forward pass is $T$ matrix–vector products).

## Backward algorithm

Define the **backward variable**
$$
\beta_t(i) \equiv P(x_{t+1:T} \mid z_t = i)
$$
— "probability of the *future* observations, **given** you're in state $i$ now." (A *conditional*, not a joint.)

$$
\beta_T(i) = 1
$$
$$
\beta_t(i) = \sum_j A_{ij}\, B_j(x_{t+1})\, \beta_{t+1}(j)
$$

> Notes: The single most common confusion in the whole topic: $\beta$ is a **conditional**, $\alpha$ is a **joint** — say it twice. That's why $\beta_T(i)=1$: there's no future left to explain, so the conditional probability of "nothing" is 1. The recursion runs *backward* in time. Students who conflate $\alpha$ and $\beta$ semantics will get $\gamma$ wrong on the next slide, so pause here.

## Smoothing posterior — the thing we actually want

$$
\gamma_t(i) \equiv P(z_t = i \mid x_{1:T}) = \frac{\alpha_t(i)\,\beta_t(i)}{\sum_j \alpha_t(j)\,\beta_t(j)}
$$

"Given the **whole** recording, what state was the system in at time $t$?"

- $\alpha_t(i)$ carries evidence from the **past** ($x_{1:t}$),
- $\beta_t(i)$ carries evidence from the **future** ($x_{t+1:T}$),
- their product, normalized, is the state posterior at each $t$.

[NOTEBOOK] This is what your plot will show: $\arg\max_i \gamma_t(i)$ overlaid on the true hidden states.

> Notes: The intuition is the sell: past-evidence × future-evidence = full-data belief. The denominator is just $P(x_{1:T})$ (any column of $\alpha\cdot\beta$ sums to it — a nice sanity check). "Smoothing" = using all the data to estimate each timepoint (vs "filtering," which uses only the past). This $\gamma$ is the payoff quantity of the coding session.

## Bridge back to Part 2: Viterbi is your trellis

Set the trellis costs to negative log-probabilities:
$$
c_1(j) = -\log \pi_j - \log B_j(x_1), \quad
c_t(j) = -\log B_j(x_t), \quad
w(i,j) = -\log A_{ij}.
$$
Then the min-cost path $V_t(j) = c_t(j) + \min_i[V_{t-1}(i) + w(i,j)]$ **is** the most-likely state sequence (Viterbi). Products of probabilities become sums of costs; "most likely" becomes "cheapest."

[ADVANCED] The forward algorithm is the *same* recursion with $\min \to \operatorname{logsumexp}$.

> Notes: Close the loop you opened in Part 2. This is the moment the whole afternoon clicks: the trellis they coded before lunch is Viterbi in disguise. Working in log space also pre-empts underflow (next slide). If time is short, this can be an [ADVANCED] extension rather than lecture — but at least *say* the sentence "your trellis is Viterbi."

## Two gotchas they will hit

**1. Numerical underflow.** Products of many probabilities → 0 fast (recall `0.1+0.2` from the morning).
- *Pragmatic fix (recommended today):* normalize $\alpha_t$ to sum to 1 each step (Rabiner scaling); the summed logs of the normalizers give $\log P(x_{1:T})$ for free.
- *Cleaner fix:* do everything in log space with `logsumexp` (why Part 2 used it).

**2. Smoothing $\ne$ decoding.** The pointwise-MAP states $\arg\max_i \gamma_t(i)$ can differ from the jointly-MAP Viterbi path — the sequence of individually-most-likely states need not be a coherent (or even legal) path.

> Notes: Gotcha 1 is a *when-not-if*. Recommend scaling for a 35-min build (fewer lines); mention log-space as the production choice. Gotcha 2 is subtle but genuine — one sentence for the strong students; a two-state example where $\gamma$-argmax picks an impossible transition drives it home if you have 60 seconds.

## The coding session (35 min, scaffolded)

**Provided in the notebook:** `sample_hmm(pi, A, B, T)` (generator) and a plotting cell (true vs inferred states).

**You write (≈14 lines total):**
```python
def forward(pi, A, B, x):      # ~6 lines: alpha recursion (+ scaling)
    ...
def backward(A, B, x):         # ~6 lines: beta recursion
    ...
def posterior(alpha, beta):    # ~2 lines: gamma = normalize(alpha*beta)
    ...
```

**Then:** simulate → infer → plot, and watch $\arg\max_i \gamma_t(i)$ track the true hidden states. Check $\log P(x_{1:T})$ against `hmmlearn`.

> Notes: Emphasize the *simulate-then-recover* loop — because the data is synthetic, ground truth is known and correctness is **visible in the plot**, not just asserted. That's the gold-standard way to trust inference code, and a habit worth instilling on day one. You wrote the recursions already in Part 2 (P2/P3); this is those with probabilities. [ADVANCED] extensions on the next slide for anyone who finishes early. [SAFETY NET] pair-program; the ≈14 lines are small enough that everyone can finish with a partner.

## [ADVANCED] Extensions & the open hole

- **Viterbi with backpointers** — decode the single best path; compare it to the $\gamma$-argmax path (illustrates gotcha 2).
- **One Baum–Welch (EM) update** — *learn* $\theta$ from data. The forward–backward quantities $\alpha,\beta,\gamma$ are exactly the E-step sufficient statistics. This is the honest open hole today's project leaves unfilled, and the natural next lecture.
- **Gaussian emissions** — swap the categorical $B$ for a Gaussian; re-run on a continuous toy signal.

> Notes: [ADVANCED] For the strong five (Narjes, Kristen, Rishab, Andrea, Maryam) so they don't idle. Baum–Welch is the intellectually satisfying "and now the model learns itself" — flag that today's forward–backward is 80% of an EM step. If the bootcamp gets a Day 2, this is the opener.

## Where this lives in neuroscience

- **Event segmentation:** Baldassano et al. (2017, *Neuron*) fit an HMM to fMRI to find event boundaries in continuous experience — hidden state = "which event." *(Narjes's prior area, w/ Baldassano.)*
- **Behavioral states:** the GLM-HMM (Ashwood et al., 2022, *Nat. Neuro.*; IBL) models animals switching between engaged/disengaged strategies. *(Adjacent to Kristen's choice modeling, Leo's decision/attention work.)*
- **Decoding with temporal structure:** an HMM is what single-trial neural decoding becomes when you add a state-transition prior instead of treating timepoints independently. *(Rishab's real-time fMRI decoding.)*

> Notes: One sentence each; they land differently for different students, so name the people lightly (or not, if that feels too pointed live). The message: this toy is a real method used across the systems/cognitive spectrum. Ends the afternoon on "you just built a research tool."

## References / pointers

- Cormen, Leiserson, Rivest, Stein, *Introduction to Algorithms* (CLRS), ch. 15 — dynamic programming.
- Erickson, *Algorithms* (free online), ch. 3 — DP, with the substructure framing.
- Rabiner (1989), "A Tutorial on Hidden Markov Models…", *Proc. IEEE* — the classic forward–backward/Viterbi reference.
- Bishop, *Pattern Recognition and Machine Learning*, ch. 13 — HMMs as graphical models; ch. 8 — sum-product/max-product.
- Durbin, Eddy, Krogh, Mitchison (1998), *Biological Sequence Analysis* — the casino, and edit distance ↔ alignment.
- Baldassano et al. (2017), *Neuron*; Ashwood et al. (2022), *Nature Neuroscience* — neuro applications.

> Notes: Leave this slide up during the coding session as a "where to read more" board. The Rabiner tutorial and Durbin book are the two most worth their time.
