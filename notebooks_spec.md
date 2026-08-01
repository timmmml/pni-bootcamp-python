# Afternoon Notebooks — Build Specification

**Author:** Tim Lin · PNI Python/Computation Bootcamp, Day 1 (afternoon)
**For:** Claude Code, to implement three Jupyter notebooks that accompany `afternoon_slides_spec.md`.

This spec gives **exact cell contents**: markdown cells, code cells with blanked scaffolds, reference solutions, and the `assert`-based self-checks that ship *inside* the student notebook. Implement faithfully; the pedagogy depends on *which* line is blanked.

---

## 0. Conventions for the implementer (read first)

**Deliverables — six `.ipynb` files** (student + solutions for each of three notebooks):

| Student file | Solutions file | Session |
|---|---|---|
| `01_python101.ipynb` | `01_python101_solutions.ipynb` | tour + 15-min exercise |
| `02_dynamic_programming.ipynb` | `02_dynamic_programming_solutions.ipynb` | 3 DP checkpoints |
| `03_hmm_project.ipynb` | `03_hmm_project_solutions.ipynb` | scaffolded mini-project |

**How to generate the two versions.** Author each notebook once with the reference solution in place, wrapped in solution markers, then produce the student version by replacing the marked region with the blank stub. Use this exact convention so a strip script (or manual pass) is unambiguous:

```python
### BEGIN SOLUTION
<reference solution lines>
### END SOLUTION
```

becomes, in the student file:

```python
# YOUR CODE HERE
raise NotImplementedError("Checkpoint N: implement <function>")
```

Everything **outside** `BEGIN/END SOLUTION` markers (imports, given helpers, the `assert` self-checks, plotting) is **identical** in both versions. The `assert` cells stay in the student notebook — that is how they self-check.

**Cell mechanics.**
- Markdown cells: keep short, point back to the deck ("see slide: *Forward algorithm*"). Render equations with `$...$` (Jupyter MathJax).
- Code cells: one logical unit each. Put every blanked function in its **own** cell, immediately followed by its **own** self-check cell, so a student can fill → run → see green before moving on.
- Each blanked line carries a trailing comment naming the idea, e.g. `# forward recursion: mass in from every predecessor, then emission`.
- No cell should depend on a later cell. Top-to-bottom execution must work in the solutions notebook with zero edits (this is the acceptance test — see §7).
- Add `# %% [markdown]` / `# %%` structure only if delivering as jupytext; otherwise native `.ipynb`.

**Environment.** Assume the pinned env from setup (`numpy`, `scipy`, `matplotlib`, `jupyter`; `hmmlearn` optional). Every notebook's first code cell is the shared setup cell in §1. Guard the optional `hmmlearn` import.

**Tone of markdown cells.** Terse and instructional. These are checkpoints to confirm understanding, not a textbook — the lecture carries the exposition.

---

## 1. Shared setup cell (first code cell of every notebook)

```python
import numpy as np
import matplotlib.pyplot as plt

RNG = np.random.default_rng(0)   # fixed seed -> reproducible for everyone

np.set_printoptions(precision=3, suppress=True)
print("numpy", np.__version__)
```

> Implementer note: identical in all three notebooks. The fixed seed matters — the accuracy `assert`s in Notebook 3 assume a deterministic stream.

---

# Notebook 01 — `01_python101.ipynb`

Purpose: a light follow-along during the tour, then the 15-minute exercise. Most cells are "predict, then run." Only the final exercise is a blanked scaffold with tests.

## Cell 1 (markdown)

```
# Python 101 — follow along
Run each cell *after predicting its output out loud*. The last section is your exercise.
See slides: Part 1.
```

## Cell 2 (code, given) — the aliasing trap

```python
x = [1, 2, 3]
y = x
y.append(4)
print("x is now:", x)     # predict before running

a = 3
b = a
b = b + 1
print("a, b =", a, b)     # predict before running
```

## Cell 3 (code, given) — identity vs value

```python
p = [1, 2]; q = [1, 2]
print("p == q:", p == q)   # same value?
print("p is q:", p is q)   # same object?
r = p
print("r is p:", r is p)
```

## Cell 4 (code, given) — containers & slicing

```python
a = [10, 20, 30, 40, 50]
print(a[0], a[-1], a[1:4], a[::2], a[::-1])

counts = {"fair": 0, "loaded": 0}
counts["fair"] += 1
print(counts)
```

## Cell 5 (code, given) — comprehension & duck typing

```python
print([n*n for n in range(6)])
print([n for n in range(10) if n % 2 == 0])

def add_all(items):
    total = items[0]
    for z in items[1:]:
        total = total + z
    return total

print(add_all([1, 2, 3]))
print(add_all(["a", "b", "c"]))
```

## Cell 6 (markdown) — the exercise

```
## Exercise (15 min): tally observations by state
Given a list of `(state, observation)` pairs, return a dict mapping each state
to a dict of observation counts. This is a warm-up for the emission counts
you'll meet in the HMM project (fair vs loaded die).
```

## Cell 7 (code, blanked) — `count_by_state`

Student version:
```python
def count_by_state(pairs):
    counts = {}
    # YOUR CODE HERE
    raise NotImplementedError("Exercise: fill counts[state][obs]")
    return counts
```

Reference solution (between markers):
```python
def count_by_state(pairs):
    counts = {}
    ### BEGIN SOLUTION
    for state, obs in pairs:
        counts.setdefault(state, {})
        counts[state][obs] = counts[state].get(obs, 0) + 1
    ### END SOLUTION
    return counts
```

## Cell 8 (code, given) — self-check

```python
pairs = [("fair","3"), ("loaded","6"), ("fair","1"), ("loaded","6")]
out = count_by_state(pairs)
assert out == {"fair": {"3": 1, "1": 1}, "loaded": {"6": 2}}, out
assert count_by_state([]) == {}
print("passed ✓")
```

## Cell 9 (markdown, `[ADVANCED]`)

```
### Stretch (optional)
Rewrite the inner tally with `collections.Counter`. Then, given `states` and
`obs` as two equal-length lists, build the same structure using `zip`.
```

## Cell 10 (code, blanked, `[ADVANCED]`) — Counter version

Reference solution:
```python
from collections import Counter, defaultdict
def count_by_state_v2(pairs):
    ### BEGIN SOLUTION
    d = defaultdict(Counter)
    for state, obs in pairs:
        d[state][obs] += 1
    return {k: dict(v) for k, v in d.items()}
    ### END SOLUTION
```
Self-check: assert `count_by_state_v2(pairs) == out` from Cell 8.

> Implementer note: `[SAFETY NET]` — in a markdown cell above Cell 7, add one line: "A plain nested `for` loop with regular dicts is a complete, correct answer. The advanced cells are optional concision."

---

# Notebook 02 — `02_dynamic_programming.ipynb`

Three checkpoints, escalating: 1D memo → 2D table + traceback → temporal trellis. Each = a blanked function + a self-check. The trellis is deliberately the Viterbi shell.

## Cell 1 (markdown)

```
# Dynamic programming — three checkpoints
DP = recursion + memory. We reuse overlapping subproblems instead of recomputing.
Checkpoints: (1) Fibonacci, (2) edit distance, (3) trellis.
See slides: Part 2.
```

(Then the shared setup cell from §1.)

## Checkpoint 1 — Fibonacci

### Cell (markdown)
```
## Checkpoint 1 — Fibonacci: caching kills the exponential
Naive recursion recomputes the same subproblems exponentially often; a cache
makes it linear. See slide: *Example 1 — Fibonacci*.
```

### Cell (code, given) — naive, to feel the cost
```python
def fib_naive(n):
    if n < 2:
        return n
    return fib_naive(n-1) + fib_naive(n-2)
```

### Cell (code, blanked) — memoized
Student:
```python
def fib_memo(n, cache=None):
    if cache is None:
        cache = {}
    if n < 2:
        return n
    # YOUR CODE HERE: return cached value, computing & storing it once
    raise NotImplementedError("Checkpoint 1: memoize fib")
```
Reference:
```python
def fib_memo(n, cache=None):
    if cache is None:
        cache = {}
    if n < 2:
        return n
    ### BEGIN SOLUTION
    if n not in cache:
        cache[n] = fib_memo(n-1, cache) + fib_memo(n-2, cache)
    return cache[n]
    ### END SOLUTION
```

### Cell (code, blanked) — bottom-up, O(1) space
Reference:
```python
def fib_bu(n):
    ### BEGIN SOLUTION
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a
    ### END SOLUTION
```

### Cell (code, given) — self-check + timing
```python
for k in range(15):
    assert fib_memo(k) == fib_naive(k) == fib_bu(k)
assert fib_memo(50) == 12586269025
print("correctness ✓")

import time
t0 = time.perf_counter(); fib_naive(32); t_naive = time.perf_counter() - t0
t0 = time.perf_counter(); fib_memo(32);  t_memo  = time.perf_counter() - t0
print(f"naive fib(32): {t_naive:.4f}s   memo fib(32): {t_memo:.6f}s")
assert t_memo < t_naive
print("speedup ✓  (this is overlapping subproblems, paid for once)")
```

## Checkpoint 2 — Edit distance

### Cell (markdown)
```
## Checkpoint 2 — Edit distance: 2D table, min-over-choices, traceback
D[i][j] = edit distance between prefixes a[:i] and b[:j]. Three new ideas vs
Fibonacci: 2D state, a genuine choice (insert/delete/substitute), and recovering
the solution via backpointers. This DP *is* sequence alignment. See slide:
*Example 2 — Edit distance*.
```

### Cell (code, blanked) — `edit_distance`
Student:
```python
def edit_distance(a, b):
    m, n = len(a), len(b)
    D = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1): D[i][0] = i     # delete i chars
    for j in range(n+1): D[0][j] = j     # insert j chars
    for i in range(1, m+1):
        for j in range(1, n+1):
            # YOUR CODE HERE: free match, else 1 + min(delete, insert, substitute)
            raise NotImplementedError("Checkpoint 2: edit-distance recurrence")
    return D[m][n]
```
Reference (replace the inner body):
```python
            ### BEGIN SOLUTION
            if a[i-1] == b[j-1]:
                D[i][j] = D[i-1][j-1]
            else:
                D[i][j] = 1 + min(D[i-1][j],      # delete a[i-1]
                                  D[i][j-1],      # insert b[j-1]
                                  D[i-1][j-1])    # substitute
            ### END SOLUTION
```

### Cell (code, given) — self-check
```python
assert edit_distance("kitten", "sitting") == 3
assert edit_distance("", "abc") == 3
assert edit_distance("abc", "abc") == 0
assert edit_distance("flaw", "lawn") == 2
print("passed ✓")
```

### Cell (code, given) — visualize the table
```python
def edit_table(a, b):
    m, n = len(a), len(b)
    D = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1): D[i][0] = i
    for j in range(n+1): D[0][j] = j
    for i in range(1, m+1):
        for j in range(1, n+1):
            cost = 0 if a[i-1] == b[j-1] else 1
            D[i][j] = min(D[i-1][j]+1, D[i][j-1]+1, D[i-1][j-1]+cost)
    return np.array(D)

print("   ", "  ".join(" " + c for c in b_ := "sitting"))
print(edit_table("kitten", "sitting"))
```

### Cell (markdown, `[ADVANCED]`)
```
### Stretch — recover the edits
Store a backpointer per cell (which of match/delete/insert/substitute you took),
then walk from D[m][n] back to D[0][0] to print the operations. The backpointer
idea returns verbatim in Viterbi.
```

### Cell (code, blanked, `[ADVANCED]`) — `edit_ops`
Reference:
```python
def edit_ops(a, b):
    ### BEGIN SOLUTION
    m, n = len(a), len(b)
    D = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1): D[i][0] = i
    for j in range(n+1): D[0][j] = j
    for i in range(1, m+1):
        for j in range(1, n+1):
            cost = 0 if a[i-1] == b[j-1] else 1
            D[i][j] = min(D[i-1][j]+1, D[i][j-1]+1, D[i-1][j-1]+cost)
    ops = []
    i, j = m, n
    while i > 0 or j > 0:
        if i > 0 and j > 0 and D[i][j] == D[i-1][j-1] + (0 if a[i-1]==b[j-1] else 1):
            ops.append("match" if a[i-1]==b[j-1] else f"sub {a[i-1]}->{b[j-1]}")
            i, j = i-1, j-1
        elif i > 0 and D[i][j] == D[i-1][j] + 1:
            ops.append(f"del {a[i-1]}"); i -= 1
        else:
            ops.append(f"ins {b[j-1]}"); j -= 1
    return list(reversed(ops))
    ### END SOLUTION
```
Self-check: `assert edit_ops("kitten","sitting").count("match") + sum('sub' in o or 'del' in o or 'ins' in o for o in edit_ops("kitten","sitting"))` — simpler: assert the number of non-match ops equals `edit_distance("kitten","sitting")`:
```python
ops = edit_ops("kitten", "sitting")
assert sum(not o.startswith("match") for o in ops) == 3
print(ops); print("passed ✓")
```

## Checkpoint 3 — Trellis (the Viterbi shell)

### Cell (markdown)
```
## Checkpoint 3 — Min-cost path through a trellis
T time steps, K states each. node_costs[t, j] = cost of state j at time t;
trans_costs[i, j] = cost of moving i -> j. Find the cheapest path picking one
state per step, and recover it with backpointers.

Keep this in mind: with cost = -log(probability), THIS IS VITERBI.
See slide: *Example 3 — Trellis*.
```

### Cell (code, blanked) — `min_cost_path`
Student:
```python
def min_cost_path(node_costs, trans_costs):
    """node_costs: (T, K); trans_costs: (K, K) with trans_costs[i, j] = cost i->j.
    Returns (best_cost, best_path) where best_path is a list of T state indices."""
    T, K = node_costs.shape
    V  = np.full((T, K), np.inf)   # V[t, j] = best cost of a path ending at (t, j)
    bp = np.zeros((T, K), dtype=int)
    V[0] = node_costs[0]           # base case
    for t in range(1, T):
        for j in range(K):
            # YOUR CODE HERE: V[t,j] = node_costs[t,j] + min_i (V[t-1,i] + trans[i,j])
            #                 bp[t,j] = argmin_i (...)
            raise NotImplementedError("Checkpoint 3: trellis recurrence + backpointer")
    # backtrack from the cheapest final state
    best_last = int(np.argmin(V[-1]))
    path = [best_last]
    for t in range(T-1, 0, -1):
        path.append(int(bp[t, path[-1]]))
    return float(V[-1, best_last]), list(reversed(path))
```
Reference (inner loop body):
```python
            ### BEGIN SOLUTION
            candidates = V[t-1] + trans_costs[:, j]     # cost to arrive at j via each i
            bp[t, j] = int(np.argmin(candidates))
            V[t, j]  = node_costs[t, j] + candidates[bp[t, j]]
            ### END SOLUTION
```

### Cell (code, given) — self-check
```python
# Test 1: transitions free -> pick the cheapest node each step
nc = np.array([[0., 5.], [1., 1.], [0., 4.]])
tc = np.zeros((2, 2))
cost, path = min_cost_path(nc, tc)
assert np.isclose(cost, 1.0), cost
assert path == [0, 0, 0], path

# Test 2: nodes free, switching expensive -> path must stay put
nc = np.zeros((3, 2))
tc = np.array([[0., 10.], [10., 0.]])
cost, path = min_cost_path(nc, tc)
assert np.isclose(cost, 0.0) and len(set(path)) == 1, (cost, path)
print("passed ✓  — you just wrote Viterbi (with cost = -log prob)")
```

### Cell (markdown)
```
You now have every ingredient for HMM inference:
a table keyed by (time, state), a recurrence that reuses the previous column,
and backpointers. Next notebook: give these a probabilistic meaning.
```

---

# Notebook 03 — `03_hmm_project.ipynb`

Scaffolded mini-project. **Given:** casino parameters, `sample_hmm`, a brute-force validator, and all plotting. **Students write:** `forward` (scaled), `backward` (scaled), `posterior`. Extensions: `viterbi`, one Baum–Welch step. The forward/backward recursions are the Checkpoint-3 trellis with sums instead of mins.

## Cell 1 (markdown)
```
# Mini-project: a hidden Markov model (the occasionally-dishonest casino)
Hidden state z_t in {fair, loaded}; observation x_t in {1..6}. The casino
sometimes swaps dice. Goal: from the rolls alone, infer WHEN the loaded die
was in play. You'll write forward, backward, and the smoothing posterior.
See slides: Part 3.
```

(Then the shared setup cell from §1.)

## Cell 2 (code, given) — casino parameters

```python
# States: 0 = fair, 1 = loaded.  Observations: 0..5  (die faces 1..6).
K, M = 2, 6
pi = np.array([0.5, 0.5])                       # initial state distribution
A  = np.array([[0.95, 0.05],                    # transition: fair sticky, loaded sticky
               [0.10, 0.90]])
B  = np.array([[1/6, 1/6, 1/6, 1/6, 1/6, 1/6],  # fair die: uniform
               [0.1, 0.1, 0.1, 0.1, 0.1, 0.5]]) # loaded die: sixes favored

assert np.allclose(A.sum(1), 1) and np.allclose(B.sum(1), 1)   # rows are distributions
STATE_NAMES = ["fair", "loaded"]
```

## Cell 3 (code, given) — the generative model

```python
def sample_hmm(pi, A, B, T, rng):
    """Sample (z, x): hidden states z[0..T-1] and observations x[0..T-1].
    This IS the joint factorization from the slides, run left to right."""
    K = len(pi)
    z = np.zeros(T, dtype=int)
    x = np.zeros(T, dtype=int)
    z[0] = rng.choice(K, p=pi)
    x[0] = rng.choice(B.shape[1], p=B[z[0]])
    for t in range(1, T):
        z[t] = rng.choice(K, p=A[z[t-1]])
        x[t] = rng.choice(B.shape[1], p=B[z[t]])
    return z, x
```

> Implementer note: leave `sample_hmm` fully given in both versions. Reading it is how they learn the model; writing it is not the point today.

## Cell 4 (code, given) — brute-force validator (for small T)

```python
from itertools import product

def loglik_bruteforce(pi, A, B, x):
    """Exact log P(x) by summing over all K**T state sequences. Only for small T."""
    K, T = len(pi), len(x)
    total = 0.0
    for z in product(range(K), repeat=T):
        p = pi[z[0]] * B[z[0], x[0]]
        for t in range(1, T):
            p *= A[z[t-1], z[t]] * B[z[t], x[t]]
        total += p
    return np.log(total)
```

> Implementer note: this is the exponential baseline the forward algorithm beats — it doubles as the correctness oracle in the forward self-check. Keep T <= 9 whenever it's called.

## Cell 5 (markdown) — forward
```
## Forward algorithm  (scaled, to avoid underflow)
alpha[t, i] = P(x[0:t+1], z_t = i), rescaled each step so it sums to 1.
The scaling constants c[t] give log P(x) = sum(log c) "for free."
Recurrence (before rescaling): alpha[t] = (alpha[t-1] @ A) * B[:, x[t]].
See slide: *Forward algorithm*.
```

## Cell 6 (code, blanked) — `forward`
Student:
```python
def forward(pi, A, B, x):
    """Returns (alpha, c, loglik): scaled forward vars (T,K), scaling (T,), log P(x)."""
    T, K = len(x), len(pi)
    alpha = np.zeros((T, K))
    c = np.zeros(T)
    # t = 0
    alpha[0] = pi * B[:, x[0]]              # YOUR CODE HERE: initial alpha (unnormalized)
    raise NotImplementedError("Forward: fill t=0 and the recursion, keep the scaling")
    c[0] = alpha[0].sum(); alpha[0] /= c[0]
    for t in range(1, T):
        alpha[t] = (alpha[t-1] @ A) * B[:, x[t]]   # YOUR CODE HERE: forward recursion
        c[t] = alpha[t].sum(); alpha[t] /= c[t]
    loglik = np.log(c).sum()
    return alpha, c, loglik
```
> Implementer note: in the STUDENT file, blank exactly the two lines tagged `# YOUR CODE HERE` (the `t=0` init and the recursion) and remove the `raise` — keep the scaling boilerplate (`c[t] = ...; alpha[t] /= c[t]`) visible so the bug surface is just the recurrence. In the SOLUTIONS file wrap those two lines with BEGIN/END SOLUTION and drop the `raise`.

Reference (the two lines are already correct above; solutions file = same minus the `raise`).

## Cell 7 (code, given) — forward self-check
```python
z_s, x_s = sample_hmm(pi, A, B, T=8, rng=np.random.default_rng(1))
_, _, ll = forward(pi, A, B, x_s)
assert np.isclose(ll, loglik_bruteforce(pi, A, B, x_s)), (ll, loglik_bruteforce(pi, A, B, x_s))
print(f"log P(x) = {ll:.4f}  matches brute force ✓  (O(T*K^2) vs O(K^T))")
```

## Cell 8 (markdown) — backward
```
## Backward algorithm  (same scaling constants c)
beta[t, i] = P(x[t+1:] | z_t = i)  — a CONDITIONAL, not a joint. So beta[T-1]=1
(no future left to explain). Reuse c from forward to keep it numerically safe.
See slide: *Backward algorithm*.
```

## Cell 9 (code, blanked) — `backward`
Student:
```python
def backward(A, B, x, c):
    """Returns scaled beta (T, K), using the forward scaling constants c."""
    T, K = len(x), A.shape[0]
    beta = np.zeros((T, K))
    beta[-1] = 1.0
    for t in range(T-2, -1, -1):
        # YOUR CODE HERE: beta[t] = (A @ (B[:, x[t+1]] * beta[t+1])) / c[t+1]
        raise NotImplementedError("Backward: fill the recursion (divide by c[t+1])")
    return beta
```
Reference (loop body):
```python
        ### BEGIN SOLUTION
        beta[t] = (A @ (B[:, x[t+1]] * beta[t+1])) / c[t+1]
        ### END SOLUTION
```

## Cell 10 (code, blanked) — `posterior`
Student:
```python
def posterior(alpha, beta):
    """Smoothing posterior gamma[t, i] = P(z_t = i | x_0:T-1)."""
    # YOUR CODE HERE: elementwise product, then normalize each row to sum to 1
    raise NotImplementedError("Posterior: gamma = normalize(alpha * beta)")
```
Reference:
```python
def posterior(alpha, beta):
    ### BEGIN SOLUTION
    gamma = alpha * beta
    gamma /= gamma.sum(axis=1, keepdims=True)
    return gamma
    ### END SOLUTION
```

## Cell 11 (code, given) — posterior self-check
```python
alpha, c, _ = forward(pi, A, B, x_s)
beta = backward(A, B, x_s, c)
gamma = posterior(alpha, beta)
assert np.allclose(gamma.sum(axis=1), 1.0)          # each timestep is a distribution
assert gamma.shape == (len(x_s), K)
print("gamma is a valid posterior ✓")
```

## Cell 12 (code, given) — the payoff: simulate → infer → plot
```python
# Long sequence with known ground truth
z_true, x_obs = sample_hmm(pi, A, B, T=300, rng=RNG)
alpha, c, ll = forward(pi, A, B, x_obs)
beta  = backward(A, B, x_obs, c)
gamma = posterior(alpha, beta)
z_hat = gamma.argmax(axis=1)                        # most likely state per timestep

acc = (z_hat == z_true).mean()
print(f"log P(x) = {ll:.2f}   state-recovery accuracy = {acc:.2%}")

fig, ax = plt.subplots(2, 1, figsize=(11, 4), sharex=True)
ax[0].step(range(300), z_true, where="mid", label="true state", lw=1.5)
ax[0].step(range(300), z_hat,  where="mid", label="inferred (argmax gamma)", lw=1, alpha=0.8)
ax[0].set_yticks([0, 1]); ax[0].set_yticklabels(STATE_NAMES); ax[0].legend(loc="upper right")
ax[1].plot(range(300), gamma[:, 1], lw=1)
ax[1].set_ylabel("P(loaded | x)"); ax[1].set_xlabel("time (roll #)")
plt.tight_layout(); plt.show()

assert acc > 0.75, "expected recovery > 75% with the seeded stream"
```

> Implementer note: the `acc > 0.75` threshold is safe for `RNG = default_rng(0)` and these parameters; verify empirically when you build it and loosen only if the seeded run comes in lower. This *simulate-then-recover* loop is the whole point — correctness is visible in the plot because ground truth is known.

## Cell 13 (code, given, optional) — cross-check against `hmmlearn`
```python
try:
    from hmmlearn.hmm import CategoricalHMM
    m = CategoricalHMM(n_components=K, init_params="", params="")
    m.startprob_, m.transmat_, m.emissionprob_ = pi, A, B
    ref_ll = m.score(x_obs.reshape(-1, 1))
    print(f"our log P(x) = {ll:.4f}   hmmlearn = {ref_ll:.4f}")
    assert np.isclose(ll, ref_ll, atol=1e-6)
    print("matches hmmlearn ✓")
except ImportError:
    print("hmmlearn not installed — skipping external check (optional).")
```

> Implementer note: `hmmlearn`'s categorical API has drifted across versions (`MultinomialHMM` → `CategoricalHMM`). Keep this in a `try/except`; it is a bonus check, never a required pass. If the installed version differs, leave a comment pointing to `CategoricalHMM`.

## Cell 14 (markdown, `[ADVANCED]`) — extensions
```
### Extensions (if you finish early)
1. Viterbi: the single most-likely PATH (not per-timestep). It's Checkpoint 3
   with cost = -log prob. Compare its path to argmax(gamma) — they can differ.
2. One Baum-Welch (EM) update: use alpha, beta, gamma as the E-step statistics
   to re-estimate (pi, A, B). This is the "model learns itself" step — the open
   hole today's project leaves for tomorrow.
3. Swap categorical B for a Gaussian emission and re-run on a continuous signal.
```

## Cell 15 (code, blanked, `[ADVANCED]`) — `viterbi`
Reference:
```python
def viterbi(pi, A, B, x):
    """Most likely state path (log space)."""
    ### BEGIN SOLUTION
    T, K = len(x), len(pi)
    logA, logB, logpi = np.log(A), np.log(B), np.log(pi)
    delta = np.full((T, K), -np.inf)
    psi = np.zeros((T, K), dtype=int)
    delta[0] = logpi + logB[:, x[0]]
    for t in range(1, T):
        for j in range(K):
            scores = delta[t-1] + logA[:, j]
            psi[t, j] = int(np.argmax(scores))
            delta[t, j] = logB[j, x[t]] + scores[psi[t, j]]
    path = [int(np.argmax(delta[-1]))]
    for t in range(T-1, 0, -1):
        path.append(int(psi[t, path[-1]]))
    return list(reversed(path))
    ### END SOLUTION
```
Self-check (small T, against brute force):
```python
def best_path_bruteforce(pi, A, B, x):
    K, T = len(pi), len(x)
    best, best_z = -np.inf, None
    for z in product(range(K), repeat=T):
        lp = np.log(pi[z[0]]) + np.log(B[z[0], x[0]])
        for t in range(1, T):
            lp += np.log(A[z[t-1], z[t]]) + np.log(B[z[t], x[t]])
        if lp > best: best, best_z = lp, list(z)
    return best_z

xs = sample_hmm(pi, A, B, 8, np.random.default_rng(3))[1]
assert viterbi(pi, A, B, xs) == best_path_bruteforce(pi, A, B, xs)
print("Viterbi matches brute-force best path ✓")
```

## Cell 16 (markdown) — where this lives in neuroscience
```
This toy is a real method. Event segmentation in fMRI (Baldassano et al., 2017,
Neuron) fits an HMM where the hidden state is "which event." GLM-HMMs (Ashwood
et al., 2022, Nat. Neuro.) model animals switching between engaged/disengaged
strategies. Single-trial neural decoding becomes an HMM once you add a
state-transition prior instead of treating timepoints independently.
```

---

## 7. Acceptance criteria (definition of done)

1. **Solutions notebooks run top-to-bottom with zero edits**, no errors, all `assert`s pass, all plots render. This is the primary gate — run `jupyter nbconvert --execute` on each `_solutions.ipynb`.
2. **Student notebooks** are identical except every `### BEGIN/END SOLUTION` region (and each `# YOUR CODE HERE` line) is replaced by the stub + `raise NotImplementedError(...)`. Running a student notebook unmodified must fail *only* at the first unfilled cell, with a clear message.
3. The **only** blanked lines in `forward`/`backward` are the recurrences (and the `t=0` init in forward). Scaling boilerplate stays visible.
4. `loglik_bruteforce`, `sample_hmm`, `best_path_bruteforce`, and all plotting cells are given in **both** versions.
5. Fixed seeds (`default_rng(0)` shared, plus the per-test seeds `1`/`3`) are used exactly as written, so the `acc > 0.75` and equality `assert`s are deterministic. If `acc > 0.75` fails on your build, re-verify the parameters before loosening the threshold.
6. `hmmlearn` cross-check is wrapped in `try/except ImportError` and never blocks a pass.
7. Markdown cells are terse and reference the corresponding slide by name.

## 8. Suggested repo layout
```
notebooks/
  01_python101.ipynb            02_dynamic_programming.ipynb            03_hmm_project.ipynb
  01_python101_solutions.ipynb  02_dynamic_programming_solutions.ipynb  03_hmm_project_solutions.ipynb
  data/                         # (none needed today; all data is simulated)
environment.yml                 # numpy, scipy, matplotlib, jupyter, hmmlearn (optional)
```
Also emit a Colab-ready copy of each student notebook (identical content; the shared setup cell already avoids local-only paths) as the zero-install fallback for anyone whose environment breaks.
