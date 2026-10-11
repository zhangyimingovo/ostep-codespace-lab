# Chapter 7: CPU Scheduling

This assignment covers the seven homework questions in [OSTEP Chapter 7](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf). The guide does not include the question bodies, so the official chapter supplies Q1-Q7.

## Results

[analysis.md](analysis.md) contains the manual predictions followed by the computed results and discussion. [data/](data/) holds 28 unedited simulator outputs. [run-manifest.json](data/run-manifest.json) records every workload, command, exit code, output hash, and checked metric.

All 28 cases agree with the predicted per-job response, turnaround, and wait times, and their averages, within the simulator's two-decimal display precision.

- FIFO and SJF agree when FIFO already follows shortest-job order. The unsorted 100,10,10 example reduces average turnaround from 110 to 50 under SJF.
- RR with quantum 1 reduces average response to 1 for both original workloads, but increases average turnaround to 599 and 465.67.
- SJF response depends on earlier jobs. Scaling every length scales response; extending only the final job leaves response unchanged.
- RR's worst initial response is at most (N-1)q under the stated model. Larger quanta increase initial response until earlier jobs fit in one slice.

## Method and environment

The runs used the existing GitHub Codespaces environment in /workspaces/ostep-codespace-lab, Python 3.12.3, and ext/ostep-homework/cpu-sched/scheduler.py. The devcontainer config specifies bash .devcontainer/setup.sh as its startup command. Python and simulator help were checked directly.

All predictions were saved before any computed run. Their snapshot was recorded at 2026-10-11T02:44:45.941954+00:00, with SHA256 bdabfba044f732406d8f91e3f1af3c8fa45cdb4920226c2b72590ae613ced199. The manifest records that receipt and the later verification times. Each of the 28 cases was previewed without -c before verification with -c. Seed 1234 was used throughout; explicit lengths fix these workloads independently of the seed.

All jobs arrive at time zero, have known CPU lengths, and do no I/O. There is no context-switch cost. Times are simulator units. These assumptions limit the conclusions to this simple model.

## Reproduce

From the repository root, first predict without the computed flag:

~~~bash
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p FIFO -s 1234
~~~

Then check the trace and metrics:

~~~bash
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p FIFO -s 1234 -c
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p RR -q 1 -s 1234 -c
~~~

To print all 28 verification runs from the saved command list:

~~~bash
python3 - <<'PY'
import json
import subprocess
with open('hw/chapter7/data/run-manifest.json') as f:
    manifest = json.load(f)
for case in manifest['cases']:
    subprocess.run(case['command'], check=True)
PY
~~~

The saved output files remain unchanged by these reproduction commands.
