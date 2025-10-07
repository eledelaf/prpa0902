# PRPA0902 — Parallel Programming: Software Mutual Exclusion with `multiprocessing`

This repository contains a set of Python programs from the **Parallel Programming** module. The goal is to study **race conditions** and progressively achieve **mutual exclusion** in a shared counter using Python’s `multiprocessing` primitives.

## 📂 Files

### `decker01.py` — Baseline (race conditions)
- Spawns `N = 8` processes, each attempting to increment a shared `Value('i')` 100 times.
- Minimal/incorrect coordination (a `turn` variable exists but isn’t robustly enforced).
- **Outcome:** Non-deterministic final counter (often **< 800**) due to lost updates.

### `decker02.py` — Intent flags (no tie-breaker)
- Introduces `critical: Array('i', [0]*N)` as per-process **intent flags** and checks via `is_anybody_inside(…)`.
- No fairness/tie-break mechanism; competing processes can block each other.
- **Outcome:** Fewer races than v01, but can **deadlock/livelock** or still miscount.

### `decker3.py` — Stricter busy-wait (closer, not complete)
- Retains **intent flags** with a tighter waiting policy before entering the critical section.
- Still lacks a proper **tie-breaker** (e.g., shared `turn`) to resolve symmetric contention.
- **Outcome:** More stable than v02 but still **not correct for all interleavings**.

### `decker4.py` — Dekker-style software mutual exclusion
- Combines **intent flags** (`critical`) with a shared **`turn: Value('i')`** as a **tie-breaker**.
- Entry protocol:
  1) Set `critical[tid] = 1` (express intent).  
  2) If others also want the CS and it’s **not your turn**, back off (`critical[tid] = 0`), wait until `turn == tid`, then try again.  
  3) Enter CS, increment shared counter, set `turn` to the next process, clear intent.  
- **Outcome:** **Mutual exclusion holds**; final counter should be **exactly 800** (`8 * 100`).

> ℹ️ These are **software-only locks** implemented with shared memory and busy-waiting to illustrate classic algorithms (Dekker-style), without OS mutexes.

## 🔧 Requirements
- Python **3.8+**
- Standard library only: `multiprocessing`

## ▶️ How to run
```bash
python decker01.py
python decker02.py
python decker3.py
python decker4.py
