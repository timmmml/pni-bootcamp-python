# PNI Bootcamp Day 1

Notebooks for the afternoon session: Python fundamentals, dynamic programming, and a mini-project implementing a hidden Markov model (the occasionally-dishonest casino).

## Setup

```bash
conda env create -f environment.yml
conda activate pni-python-bootcamp
jupyter notebook notebooks/
```

No local install? Use the Colab copies in `notebooks/colab/` instead — open a notebook, upload it to [Colab](https://colab.research.google.com/), and run.

## Notebooks

Work through these in order. Each has a matching `_solutions.ipynb` — try the exercise yourself first, and check the solutions notebook if you get stuck or want to compare approaches.

| Notebook | What you'll do |
|---|---|
| `01_python101.ipynb` | Follow-along tour of core Python (containers, control flow, comprehensions), then a 15-minute exercise. |
| `02_dynamic_programming.ipynb` | Three checkpoints: memoized Fibonacci, edit distance with traceback, and a min-cost trellis. |
| `03_hmm_project.ipynb` | Implement `forward`, `backward`, and `posterior` to infer hidden states from simulated data. |

Sections marked `[ADVANCED]` are optional stretch goals — skip them if you're short on time.
