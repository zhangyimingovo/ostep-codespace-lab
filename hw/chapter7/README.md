# 第 7 章：CPU 调度

本次作业根据 [OSTEP 第 7 章](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf)的习题，比较 FIFO、SJF 和轮转调度（RR）。

计算过程和说明见 [analysis.md](analysis.md)。28 份模拟器输出保存在 [data/](data/)，运行命令和结果列在 [run-manifest.json](data/run-manifest.json) 中。

## 主要结果

- 对于 200,200,200 和 100,200,300，FIFO 和 SJF 的结果相同。对于 100,10,10，SJF 先运行短任务，使平均周转时间从 110 降到 50。
- 时间片为 1 时，两组任务在 RR 下的平均响应时间都是 1，但平均周转时间更长：相同长度为 599，不同长度为 465.67。
- 所有任务都变长时，SJF 的响应时间也会增加。只增加最后一个任务的长度，不会改变各任务第一次开始的时间。
- RR 的时间片越大，后面的任务可能需要等更久才能第一次运行。N 个任务都在时间 0 就绪时，最长响应时间不超过 (N-1)q。

模拟结果与计算一致。所有任务都在时间 0 到达，没有 I/O 和切换开销。时间均使用模拟器的单位。

## 运行示例

示例在 GitHub Codespaces 中使用 Python 3.12.3。以下命令从仓库根目录运行，随机种子统一为 1234。

先不加 -c，计算响应时间和周转时间：

```bash
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p FIFO -s 1234
```

再加上 -c 核对结果：

```bash
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p FIFO -s 1234 -c
python3 ext/ostep-homework/cpu-sched/scheduler.py -l 100,200,300 -p RR -q 1 -s 1234 -c
```

也可以根据保存的命令列表，重新运行全部 28 个例子：

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
