# Chapter 7: CPU Scheduling

This exercise compares FIFO, SJF, and Round Robin using the homework questions in [OSTEP Chapter 7](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf).

The calculations and explanations are in [analysis.md](analysis.md). The 28 simulator outputs are in [data/](data/), and [run-manifest.json](data/run-manifest.json) lists the commands and results.

## Main results

- FIFO and SJF give the same result for 200,200,200 and 100,200,300. For 100,10,10, SJF reduces average turnaround from 110 to 50 by running the short jobs first.
- RR with quantum 1 gives average response 1 for both original workloads. Average turnaround is higher: 599 for equal lengths and 465.67 for unequal lengths.
- Increasing all job lengths makes response under SJF increase. Increasing only the final job does not change the first starts.
- A larger RR quantum can increase the wait for the first turn. With N jobs ready at time 0, worst response is at most (N-1)q.

The simulator results agree with the calculations. All jobs arrive at time 0, and there is no I/O or switching cost. Times are in simulator units.

## Running the examples

The examples use Python 3.12.3 in GitHub Codespaces. Run these commands from the repository root. Seed 1234 is used throughout.

First run without -c and calculate the response and turnaround times:

```bash
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p FIFO -s 1234
```

Then add -c to check the result:

```bash
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p FIFO -s 1234 -c
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p RR -q 1 -s 1234 -c
```

The saved command list can also be used to repeat all 28 examples:

```bash
python3 - <<'PY'
import json
import subprocess
with open('hw/chapter7/data/run-manifest.json') as f:
    cases = json.load(f)['cases']
for case in cases:
    subprocess.run(case['command'], check=True)
PY
```
