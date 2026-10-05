# OSTEP Ch.4 Homework Answers

状态表每行表示一个 tick；CPU 或 IOs 的 `-` 表示该列为空，`*` 表示有 I/O 在该 tick 完成。一次 I/O 需要发起 1 tick、阻塞 5 ticks、处理完成 1 tick。

## Q1

**题目：** 两个进程各运行 5 条纯 CPU 指令时，CPU 是否会空闲？预测总运行时间和 CPU 利用率。

**Command（预测阶段，不含 `-c`）：**

```bash
python3 process-run.py -l 5:100,5:100
```

**指令列表：**

```text
PID 0: cpu, cpu, cpu, cpu, cpu
PID 1: cpu, cpu, cpu, cpu, cpu
```

- Prediction / 预测:

```text
Time   PID0         PID1         CPU   IOs
1      RUN:cpu      READY        1     -
2      RUN:cpu      READY        1     -
3      RUN:cpu      READY        1     -
4      RUN:cpu      READY        1     -
5      RUN:cpu      READY        1     -
6      DONE         RUN:cpu      1     -
7      DONE         RUN:cpu      1     -
8      DONE         RUN:cpu      1     -
9      DONE         RUN:cpu      1     -
10     DONE         RUN:cpu      1     -
```

- Total time: **10 ticks**。
- CPU Busy: **10 ticks**。
- CPU utilization: **10 / 10 × 100% = 100%**。

- Reasoning / 理由: PID 0 先连续执行 5 条 CPU 指令，随后 PID 1 连续执行自己的 5 条 CPU 指令。两个进程都不会因 I/O 阻塞，因此每个 tick 都有进程使用 CPU。模拟器的进程切换不额外占用 tick，所以总时间就是 5 + 5 = 10 ticks，CPU 不会空闲。


- Verified result / 验证结果: [q1.txt](q1.txt) 的实际输出为总时间 10 ticks、CPU Busy 10 ticks、CPU 利用率 100.00%，I/O Busy 0 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: 两个进程各执行五条 CPU 指令，没有 I/O 阻塞；PID0 完成后，PID1 立即接着运行。因此十个 tick 都有 CPU 工作，模拟器也没有额外的切换时间。预测中的 5 + 5 = 10 ticks 得到验证，CPU 没有空闲。

## Q2

**题目：** 先运行含 4 条纯 CPU 指令的进程，再运行发起一次 I/O 的进程，会如何使用 CPU？预测总时间，并说明进程顺序的影响。

**Command（预测阶段，不含 `-c`）：**

```bash
python3 process-run.py -l 4:100,1:0
```

**指令列表：**

```text
PID 0: cpu, cpu, cpu, cpu
PID 1: io, io_done
```

- Prediction / 预测:

```text
Time   PID0         PID1         CPU   IOs
1      RUN:cpu      READY        1     -
2      RUN:cpu      READY        1     -
3      RUN:cpu      READY        1     -
4      RUN:cpu      READY        1     -
5      DONE         RUN:io       1     -
6      DONE         BLOCKED      -     1
7      DONE         BLOCKED      -     1
8      DONE         BLOCKED      -     1
9      DONE         BLOCKED      -     1
10     DONE         BLOCKED      -     1
11*    DONE         RUN:io_done  1     -
```

- Total time: **11 ticks**。
- CPU Busy: **6 ticks**。
- CPU utilization: **6 / 11 × 100% ≈ 54.55%**。

- Reasoning / 理由: PID 0 在前 4 个 tick 用完全部 CPU 指令，PID 1 到第 5 个 tick 才发起 I/O。接下来 5 个 tick 中 PID 1 阻塞，而 PID 0 已结束，没有其他就绪进程可以使用 CPU。第 11 个 tick I/O 完成，PID 1 执行 `io_done`。发起 I/O 和处理 I/O 完成都各占用一个 CPU tick，因此 CPU 忙碌时间为 4 + 1 + 1 = 6 ticks。这个顺序使 CPU 工作与 I/O 等待无法重叠。


- Verified result / 验证结果: [q2.txt](q2.txt) 的实际输出为总时间 11 ticks、CPU Busy 6 ticks、CPU 利用率 54.55%，I/O Busy 5 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: PID0 在第 1–4 tick 完成全部 CPU 工作，PID1 才在第 5 tick 发起 I/O；第 6–10 tick 中没有可运行的其他进程，CPU 因而空闲五个 tick。第 11 tick 执行 io_done。CPU Busy 包括四条 cpu，以及 io 和 io_done 各一条，共六个 tick。这个顺序没有把 CPU 工作与 I/O 等待重叠，验证了原预测的 11 ticks 和 54.55%。

## Q3

**题目：** 将 Q2 的进程顺序交换，让发起 I/O 的进程先运行，会发生什么变化？顺序为什么影响效率？

**Command（预测阶段，不含 `-c`）：**

```bash
python3 process-run.py -l 1:0,4:100
```

**指令列表：**

```text
PID 0: io, io_done
PID 1: cpu, cpu, cpu, cpu
```

- Prediction / 预测:

```text
Time   PID0         PID1         CPU   IOs
1      RUN:io       READY        1     -
2      BLOCKED      RUN:cpu      1     1
3      BLOCKED      RUN:cpu      1     1
4      BLOCKED      RUN:cpu      1     1
5      BLOCKED      RUN:cpu      1     1
6      BLOCKED      DONE         -     1
7*     RUN:io_done  DONE         1     -
```

- Total time: **7 ticks**。
- CPU Busy: **6 ticks**。
- CPU utilization: **6 / 7 × 100% ≈ 85.71%**。

- Reasoning / 理由: PID 0 在第 1 个 tick 发起 I/O 后阻塞。默认的 `SWITCH_ON_IO` 允许立即切换到 PID 1，因此第 2–5 个 tick 可以在 I/O 进行期间执行四条 CPU 指令。第 6 个 tick 时 PID 1 已结束，PID 0 的 I/O 尚未完成，CPU 只需空闲这一个 tick。第 7 个 tick 执行 `io_done` 后结束。与 Q2 相比，两题 CPU 工作量相同，但 Q3 将 4 个 CPU tick 与 I/O 等待重叠，使总时间由 11 ticks 减少到 7 ticks。


- Verified result / 验证结果: [q3.txt](q3.txt) 的实际输出为总时间 7 ticks、CPU Busy 6 ticks、CPU 利用率 85.71%，I/O Busy 5 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: PID0 第 1 tick 发起 I/O 后，默认的 SWITCH_ON_IO 让 PID1 在第 2–5 tick 执行四条 CPU 指令。这四个 tick 覆盖了 I/O 等待中的四个 tick；只有第 6 tick 没有就绪进程，随后第 7 tick 执行 io_done。与 Q2 相比，指令和 I/O 等待长度没有变化，但重叠减少了四个空闲 tick，使总时间从 11 降到 7 ticks。

## Q4

**题目：** 对 Q3 的进程顺序使用 `SWITCH_ON_END`，只在进程结束时切换。这种策略是否有效利用 CPU？

**Command（预测阶段，不含 `-c`）：**

```bash
python3 process-run.py -l 1:0,4:100 -S SWITCH_ON_END
```

**指令列表：**

```text
PID 0: io, io_done
PID 1: cpu, cpu, cpu, cpu
```

- Prediction / 预测:

```text
Time   PID0         PID1         CPU   IOs
1      RUN:io       READY        1     -
2      BLOCKED      READY        -     1
3      BLOCKED      READY        -     1
4      BLOCKED      READY        -     1
5      BLOCKED      READY        -     1
6      BLOCKED      READY        -     1
7*     RUN:io_done  READY        1     -
8      DONE         RUN:cpu      1     -
9      DONE         RUN:cpu      1     -
10     DONE         RUN:cpu      1     -
11     DONE         RUN:cpu      1     -
```

- Total time: **11 ticks**。
- CPU Busy: **6 ticks**。
- CPU utilization: **6 / 11 × 100% ≈ 54.55%**。

- Reasoning / 理由: `SWITCH_ON_END` 不会在 PID 0 发起 I/O 时切换到 PID 1。虽然 PID 1 一直处于 READY 状态，第 2–6 个 tick 的 CPU 仍然空闲。PID 0 在第 7 个 tick 执行 `io_done`，结束之后才轮到 PID 1 在第 8–11 个 tick 执行四条 CPU 指令。该策略在这个工作负载下效率较低，因为它让可运行的进程等待，失去了 CPU 工作与 I/O 等待重叠的机会。


- Verified result / 验证结果: [q4.txt](q4.txt) 的实际输出为总时间 11 ticks、CPU Busy 6 ticks、CPU 利用率 54.55%，I/O Busy 5 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: SWITCH_ON_END 要等当前进程结束才切换。第 2–6 tick 虽然 PID1 为 READY，PID0 的阻塞仍使 CPU 空闲；PID0 第 7 tick 执行 io_done 后，PID1 才在第 8–11 tick 运行。实际轨迹确认，这个策略没有利用已有的就绪进程来覆盖 I/O 等待，因此比 Q3 多用四个 tick，CPU 利用率降到 54.55%。

## Q5

**题目：** 对 Q4 的相同进程使用 `SWITCH_ON_IO`，遇到 I/O 就切换。总时间与 CPU 利用率会怎样变化？

**Command（预测阶段，不含 `-c`）：**

```bash
python3 process-run.py -l 1:0,4:100 -S SWITCH_ON_IO
```

**指令列表：**

```text
PID 0: io, io_done
PID 1: cpu, cpu, cpu, cpu
```

- Prediction / 预测:

```text
Time   PID0         PID1         CPU   IOs
1      RUN:io       READY        1     -
2      BLOCKED      RUN:cpu      1     1
3      BLOCKED      RUN:cpu      1     1
4      BLOCKED      RUN:cpu      1     1
5      BLOCKED      RUN:cpu      1     1
6      BLOCKED      DONE         -     1
7*     RUN:io_done  DONE         1     -
```

- Total time: **7 ticks**。
- CPU Busy: **6 ticks**。
- CPU utilization: **6 / 7 × 100% ≈ 85.71%**。

- Reasoning / 理由: 这次 PID 0 在第 1 个 tick 发起 I/O 后，调度器会切换到 PID 1。PID 1 在第 2–5 个 tick 执行 CPU 指令，覆盖了 5 个 I/O 等待 tick 中的 4 个。CPU 仅在第 6 个 tick 空闲，第 7 个 tick 执行 `io_done` 后结束。这个结果与 Q3 相同，因为 `SWITCH_ON_IO` 就是默认切换策略。相较于 Q4，总时间减少 4 ticks，CPU 利用率从约 54.55% 上升至约 85.71%。


- Verified result / 验证结果: [q5.txt](q5.txt) 的实际输出为总时间 7 ticks、CPU Busy 6 ticks、CPU 利用率 85.71%，I/O Busy 5 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: SWITCH_ON_IO 在 PID0 发起 I/O 时就切到 PID1，四条 CPU 指令覆盖第 2–5 tick 的 I/O 等待。第 6 tick 仍需等待，所以没有达到 100% 的 CPU 利用率。输出与 Q3 相同，因为 Q3 使用的默认策略也是 SWITCH_ON_IO；相较于 Q4，CPU Busy 仍为六个 tick，但空闲时间由五个减到一个，总时间减少四个 tick。

## Q6

**题目：** 使用 `-l 3:0,5:100,5:100,5:100`，在遇到 I/O 时切换进程，但 I/O 完成后让该进程稍后运行。CPU 和 I/O 是否得到有效利用？为什么？

**Command（预测阶段）：**

```bash
python3 process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER
```

**指令序列：**

- PID0：`io, io_done, io, io_done, io, io_done`
- PID1：`cpu, cpu, cpu, cpu, cpu`
- PID2：`cpu, cpu, cpu, cpu, cpu`
- PID3：`cpu, cpu, cpu, cpu, cpu`

- Prediction / 预测: 总时间 31 ticks，CPU Busy 为 21 ticks，CPU 利用率为 `21 / 31 × 100% = 67.74%`。

以下各表中 `*` 表示该 tick 有 I/O 完成；`CPU` 为 1 表示 CPU 正在执行指令，`IOs` 为正在进行的 I/O 数量，`-` 表示空闲。最后一条指令所在 tick 仍显示 `RUN`，后续 tick 才显示 `DONE`。

```text
Time  PID0         PID1     PID2     PID3     CPU  IOs
1     RUN:io       READY    READY    READY    1    -
2     BLOCKED      RUN:cpu  READY    READY    1    1
3     BLOCKED      RUN:cpu  READY    READY    1    1
4     BLOCKED      RUN:cpu  READY    READY    1    1
5     BLOCKED      RUN:cpu  READY    READY    1    1
6     BLOCKED      RUN:cpu  READY    READY    1    1
7*    READY        DONE     RUN:cpu  READY    1    -
8     READY        DONE     RUN:cpu  READY    1    -
9     READY        DONE     RUN:cpu  READY    1    -
10    READY        DONE     RUN:cpu  READY    1    -
11    READY        DONE     RUN:cpu  READY    1    -
12    READY        DONE     DONE     RUN:cpu  1    -
13    READY        DONE     DONE     RUN:cpu  1    -
14    READY        DONE     DONE     RUN:cpu  1    -
15    READY        DONE     DONE     RUN:cpu  1    -
16    READY        DONE     DONE     RUN:cpu  1    -
17    RUN:io_done  DONE     DONE     DONE     1    -
18    RUN:io       DONE     DONE     DONE     1    -
19    BLOCKED      DONE     DONE     DONE     -    1
20    BLOCKED      DONE     DONE     DONE     -    1
21    BLOCKED      DONE     DONE     DONE     -    1
22    BLOCKED      DONE     DONE     DONE     -    1
23    BLOCKED      DONE     DONE     DONE     -    1
24*   RUN:io_done  DONE     DONE     DONE     1    -
25    RUN:io       DONE     DONE     DONE     1    -
26    BLOCKED      DONE     DONE     DONE     -    1
27    BLOCKED      DONE     DONE     DONE     -    1
28    BLOCKED      DONE     DONE     DONE     -    1
29    BLOCKED      DONE     DONE     DONE     -    1
30    BLOCKED      DONE     DONE     DONE     -    1
31*   RUN:io_done  DONE     DONE     DONE     1    -
```

- Reasoning / 理由: PID0 先发起 I/O，PID1 的五条 CPU 指令覆盖了第一次等待。第 7 tick 的 I/O 完成时，PID2 已被选中；`IO_RUN_LATER` 让 PID0 进入 READY，继续等待 PID2 和 PID3 完成。因此 PID0 到第 17 tick 才能运行 `io_done`，第 18 tick 才发起第二次 I/O。此时其他进程都已结束，剩下两次等待各造成五个 CPU 空闲 ticks。CPU 共执行 `6 + 5 + 5 + 5 = 21` 条指令，加上 10 个空闲 ticks，总计 31 ticks。I/O 的三个等待段共占 15 ticks，利用率为 `15 / 31 = 48.39%`；第 7 至 18 tick 间没有在进行的 I/O，这段时间没有充分重叠 CPU 与 I/O。


- Verified result / 验证结果: [q6.txt](q6.txt) 的实际输出为总时间 31 ticks、CPU Busy 21 ticks、CPU 利用率 67.74%，I/O Busy 15 ticks、I/O 利用率 48.39%。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: 第一次 I/O 等待由 PID1 的五条 CPU 指令完全覆盖，但第 7 tick I/O 完成后，IO_RUN_LATER 让 PID0 保持 READY，PID2 和 PID3 继续运行。PID0 到第 17 tick 才执行 io_done，第 18 tick 才启动第二次 I/O；此时其他进程已结束，后两段等待各造成五个 CPU 空闲 tick。因此 21 个 CPU 忙 tick 加 10 个空闲 tick 得到 31 ticks。三个 I/O 等待段合计 15 ticks，延迟恢复 PID0 拉长了没有 I/O 在进行的间隔，CPU 与 I/O 都未充分利用。

## Q7

**题目：** 对 Q6 的进程设置使用 `IO_RUN_IMMEDIATE`，I/O 完成后立即运行该进程。运行结果如何变化，为什么立即运行 I/O 进程可能更好？

**Command（预测阶段）：**

```bash
python3 process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE
```

**指令序列：**

- PID0：`io, io_done, io, io_done, io, io_done`
- PID1：`cpu, cpu, cpu, cpu, cpu`
- PID2：`cpu, cpu, cpu, cpu, cpu`
- PID3：`cpu, cpu, cpu, cpu, cpu`

- Prediction / 预测: 总时间 21 ticks，CPU Busy 为 21 ticks，CPU 利用率为 `21 / 21 × 100% = 100%`。

```text
Time  PID0         PID1     PID2     PID3     CPU  IOs
1     RUN:io       READY    READY    READY    1    -
2     BLOCKED      RUN:cpu  READY    READY    1    1
3     BLOCKED      RUN:cpu  READY    READY    1    1
4     BLOCKED      RUN:cpu  READY    READY    1    1
5     BLOCKED      RUN:cpu  READY    READY    1    1
6     BLOCKED      RUN:cpu  READY    READY    1    1
7*    RUN:io_done  DONE     READY    READY    1    -
8     RUN:io       DONE     READY    READY    1    -
9     BLOCKED      DONE     RUN:cpu  READY    1    1
10    BLOCKED      DONE     RUN:cpu  READY    1    1
11    BLOCKED      DONE     RUN:cpu  READY    1    1
12    BLOCKED      DONE     RUN:cpu  READY    1    1
13    BLOCKED      DONE     RUN:cpu  READY    1    1
14*   RUN:io_done  DONE     DONE     READY    1    -
15    RUN:io       DONE     DONE     READY    1    -
16    BLOCKED      DONE     DONE     RUN:cpu  1    1
17    BLOCKED      DONE     DONE     RUN:cpu  1    1
18    BLOCKED      DONE     DONE     RUN:cpu  1    1
19    BLOCKED      DONE     DONE     RUN:cpu  1    1
20    BLOCKED      DONE     DONE     RUN:cpu  1    1
21*   RUN:io_done  DONE     DONE     DONE     1    -
```

- Reasoning / 理由: 每次 I/O 完成后，PID0 立即运行 `io_done`，随后尽早发起下一次 I/O。它的三段五 tick 等待分别由 PID1、PID2、PID3 的五条 CPU 指令覆盖。因此全部 21 个 ticks 都有 CPU 工作，总时间比 Q6 少 10 ticks。I/O 等待仍共占 15 ticks，利用率变为 `15 / 21 = 71.43%`。优先让 I/O 进程恢复运行有利于尽早发起下一次 I/O，从而让等待与其他进程的 CPU 工作重叠。


- Verified result / 验证结果: [q7.txt](q7.txt) 的实际输出为总时间 21 ticks、CPU Busy 21 ticks、CPU 利用率 100.00%，I/O Busy 15 ticks、I/O 利用率 71.43%。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: IO_RUN_IMMEDIATE 在第 7、14、21 tick 的 I/O 完成时立即恢复 PID0。前两次恢复后，PID0 很快发起下一次 I/O，使三段五 tick 的等待分别与 PID1、PID2、PID3 的五条 CPU 指令重叠。CPU Busy 和 I/O Busy 分别仍为 21 和 15 ticks，但 Q6 的十个 CPU 空闲 tick 消失，总时间由 31 降为 21 ticks。立即恢复 I/O 进程的收益来自更早启动下一次 I/O，从而保留其他进程覆盖等待的机会。

## Q8

**题目：** 用随机种子 1、2、3，分别预测 `-l 3:50,3:50` 生成的两进程运行轨迹；比较默认设置、`IO_RUN_IMMEDIATE` 和 `SWITCH_ON_END` 对总时间及 CPU 利用率的影响。

默认设置为 `SWITCH_ON_IO` 与 `IO_RUN_LATER`。下面按已经显示的指令序列逐 tick 手工预测，不额外改变 I/O 等待长度。

### Q8 — seed 1

**指令序列：**

- PID0：`cpu, io, io_done, io, io_done`
- PID1：`cpu, cpu, cpu`

#### 默认设置

**Command（预测阶段）：**

```bash
python3 process-run.py -s 1 -l 3:50,3:50
```

- Prediction / 预测: 总时间 15 ticks，CPU Busy 为 8 ticks，CPU 利用率为 `8 / 15 × 100% = 53.33%`。

```text
Time  PID0         PID1     CPU  IOs
1     RUN:cpu      READY    1    -
2     RUN:io       READY    1    -
3     BLOCKED      RUN:cpu  1    1
4     BLOCKED      RUN:cpu  1    1
5     BLOCKED      RUN:cpu  1    1
6     BLOCKED      DONE     -    1
7     BLOCKED      DONE     -    1
8*    RUN:io_done  DONE     1    -
9     RUN:io       DONE     1    -
10    BLOCKED      DONE     -    1
11    BLOCKED      DONE     -    1
12    BLOCKED      DONE     -    1
13    BLOCKED      DONE     -    1
14    BLOCKED      DONE     -    1
15*   RUN:io_done  DONE     1    -
```

- Reasoning / 理由: PID1 的三条 CPU 指令覆盖 PID0 第一次等待的前三个 ticks，随后 CPU 空闲两个 ticks。第二次等待开始时 PID1 已结束，再空闲五个 ticks。CPU 共忙 8 ticks，空闲 7 ticks，总时间为 15 ticks。


- Verified result / 验证结果: [q8-s1.txt](q8-s1.txt) 中 默认设置实际输出为总时间 15 ticks、CPU Busy 8 ticks、CPU 利用率 53.33%，I/O Busy 10 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: PID1 的三条 CPU 指令覆盖 PID0 第一次 I/O 等待的前三个 tick，随后第 6–7 tick CPU 空闲。第二次 I/O 开始时 PID1 已结束，第 10–14 tick 全部空闲。因此 CPU 忙八个 tick、空闲七个 tick，总时间为 15 ticks；默认切换策略可以覆盖部分等待，但另一个进程的 CPU 工作不足以覆盖全部等待。

#### `IO_RUN_IMMEDIATE`

**Command（预测阶段）：**

```bash
python3 process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_IMMEDIATE
```

- Prediction / 预测: 总时间 15 ticks，CPU Busy 为 8 ticks，CPU 利用率为 `8 / 15 × 100% = 53.33%`。

```text
Time  PID0         PID1     CPU  IOs
1     RUN:cpu      READY    1    -
2     RUN:io       READY    1    -
3     BLOCKED      RUN:cpu  1    1
4     BLOCKED      RUN:cpu  1    1
5     BLOCKED      RUN:cpu  1    1
6     BLOCKED      DONE     -    1
7     BLOCKED      DONE     -    1
8*    RUN:io_done  DONE     1    -
9     RUN:io       DONE     1    -
10    BLOCKED      DONE     -    1
11    BLOCKED      DONE     -    1
12    BLOCKED      DONE     -    1
13    BLOCKED      DONE     -    1
14    BLOCKED      DONE     -    1
15*   RUN:io_done  DONE     1    -
```

- Reasoning / 理由: 两次 I/O 完成时，PID0 都是唯一尚未结束的进程。默认设置已经会让它恢复运行，立即运行策略没有其他正在运行的进程可以抢占，因此轨迹与默认设置相同。


- Verified result / 验证结果: [q8-s1.txt](q8-s1.txt) 中 IO_RUN_IMMEDIATE实际输出为总时间 15 ticks、CPU Busy 8 ticks、CPU 利用率 53.33%，I/O Busy 10 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: 第 8 和第 15 tick 的 I/O 完成时，PID1 都已经结束，PID0 是唯一尚未结束的进程。默认设置此时也会直接恢复 PID0，因此立即运行策略没有改变恢复时间或下一次 I/O 的启动时间。七个 CPU 空闲 tick 仍然存在，总时间和利用率保持不变。

#### `SWITCH_ON_END`

**Command（预测阶段）：**

```bash
python3 process-run.py -s 1 -l 3:50,3:50 -S SWITCH_ON_END
```

- Prediction / 预测: 总时间 18 ticks，CPU Busy 为 8 ticks，CPU 利用率为 `8 / 18 × 100% = 44.44%`。

```text
Time  PID0         PID1     CPU  IOs
1     RUN:cpu      READY    1    -
2     RUN:io       READY    1    -
3     BLOCKED      READY    -    1
4     BLOCKED      READY    -    1
5     BLOCKED      READY    -    1
6     BLOCKED      READY    -    1
7     BLOCKED      READY    -    1
8*    RUN:io_done  READY    1    -
9     RUN:io       READY    1    -
10    BLOCKED      READY    -    1
11    BLOCKED      READY    -    1
12    BLOCKED      READY    -    1
13    BLOCKED      READY    -    1
14    BLOCKED      READY    -    1
15*   RUN:io_done  READY    1    -
16    DONE         RUN:cpu  1    -
17    DONE         RUN:cpu  1    -
18    DONE         RUN:cpu  1    -
```

- Reasoning / 理由: PID0 阻塞时不会切换到 READY 的 PID1，两次等待都使 CPU 空闲五个 ticks。PID0 结束后 PID1 才执行三条 CPU 指令，所以总时间为 `8 + 10 = 18` ticks。与默认设置相比，失去了三个 ticks 的 CPU 与 I/O 重叠。


- Verified result / 验证结果: [q8-s1.txt](q8-s1.txt) 中 SWITCH_ON_END实际输出为总时间 18 ticks、CPU Busy 8 ticks、CPU 利用率 44.44%，I/O Busy 10 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: PID0 的两次 I/O 等待期间，PID1 虽为 READY，却要等 PID0 结束后才运行。因此两段等待共造成十个 CPU 空闲 tick，PID1 的三条 CPU 指令最后才执行。八个忙 tick 加十个空闲 tick 等于 18 ticks；与默认设置相比，失去了三个 tick 的 CPU 与 I/O 重叠，利用率因此下降。

### Q8 — seed 2

**指令序列：**

- PID0：`io, io_done, io, io_done, cpu`
- PID1：`cpu, io, io_done, io, io_done`

#### 默认设置

**Command（预测阶段）：**

```bash
python3 process-run.py -s 2 -l 3:50,3:50
```

- Prediction / 预测: 总时间 16 ticks，CPU Busy 为 10 ticks，CPU 利用率为 `10 / 16 × 100% = 62.50%`。

```text
Time  PID0         PID1         CPU  IOs
1     RUN:io       READY        1    -
2     BLOCKED      RUN:cpu      1    1
3     BLOCKED      RUN:io       1    1
4     BLOCKED      BLOCKED      -    2
5     BLOCKED      BLOCKED      -    2
6     BLOCKED      BLOCKED      -    2
7*    RUN:io_done  BLOCKED      1    1
8     RUN:io       BLOCKED      1    1
9*    BLOCKED      RUN:io_done  1    1
10    BLOCKED      RUN:io       1    1
11    BLOCKED      BLOCKED      -    2
12    BLOCKED      BLOCKED      -    2
13    BLOCKED      BLOCKED      -    2
14*   RUN:io_done  BLOCKED      1    1
15    RUN:cpu      BLOCKED      1    1
16*   DONE         RUN:io_done  1    -
```

- Reasoning / 理由: 两个进程都含有两次 I/O。每个进程阻塞后，另一个进程可以执行就绪指令；部分 I/O 等待彼此重叠。不过第 4 至 6 tick、第 11 至 13 tick 两个进程都阻塞，CPU 共空闲 6 ticks。两进程合计执行 10 条指令，总时间为 `10 + 6 = 16` ticks。`IOs=2` 表示两次 I/O 同时在进行，不能把它理解为 CPU 也在执行两条指令。


- Verified result / 验证结果: [q8-s2.txt](q8-s2.txt) 中 默认设置实际输出为总时间 16 ticks、CPU Busy 10 ticks、CPU 利用率 62.50%，I/O Busy 14 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: 两个进程各有两次 I/O，遇到阻塞就切换使它们可以先后发起 I/O，部分等待同时进行。第 4–6 和第 11–13 tick 两个进程都为 BLOCKED，造成六个 CPU 空闲 tick；其余十个 tick 执行指令，所以总时间为 16 ticks。四次 I/O 各等待五个 tick，但存在重叠，至少一个 I/O 在进行的时间为 14 ticks；IOs 为 2 的行表示并发等待，不能将两次等待都算成额外的总运行时间。

#### `IO_RUN_IMMEDIATE`

**Command（预测阶段）：**

```bash
python3 process-run.py -s 2 -l 3:50,3:50 -I IO_RUN_IMMEDIATE
```

- Prediction / 预测: 总时间 16 ticks，CPU Busy 为 10 ticks，CPU 利用率为 `10 / 16 × 100% = 62.50%`。

```text
Time  PID0         PID1         CPU  IOs
1     RUN:io       READY        1    -
2     BLOCKED      RUN:cpu      1    1
3     BLOCKED      RUN:io       1    1
4     BLOCKED      BLOCKED      -    2
5     BLOCKED      BLOCKED      -    2
6     BLOCKED      BLOCKED      -    2
7*    RUN:io_done  BLOCKED      1    1
8     RUN:io       BLOCKED      1    1
9*    BLOCKED      RUN:io_done  1    1
10    BLOCKED      RUN:io       1    1
11    BLOCKED      BLOCKED      -    2
12    BLOCKED      BLOCKED      -    2
13    BLOCKED      BLOCKED      -    2
14*   RUN:io_done  BLOCKED      1    1
15    RUN:cpu      BLOCKED      1    1
16*   DONE         RUN:io_done  1    -
```

- Reasoning / 理由: 每次 I/O 完成时，另一个进程都在阻塞或已经结束。完成 I/O 的进程在默认设置下也会运行，因此立即运行策略不改变执行顺序，总时间和 CPU 利用率保持不变。


- Verified result / 验证结果: [q8-s2.txt](q8-s2.txt) 中 IO_RUN_IMMEDIATE实际输出为总时间 16 ticks、CPU Busy 10 ticks、CPU 利用率 62.50%，I/O Busy 14 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: 在第 7、9、14、16 tick 有 I/O 完成时，另一个进程正在阻塞或已经结束。默认策略也会运行刚完成 I/O 的进程，所以 IO_RUN_IMMEDIATE 没有可提前的恢复动作，也没有改变下一次 I/O 的启动时间。两个进程同时阻塞的六个 tick 仍然存在，总时间仍为十个忙 tick 加六个空闲 tick。

#### `SWITCH_ON_END`

**Command（预测阶段）：**

```bash
python3 process-run.py -s 2 -l 3:50,3:50 -S SWITCH_ON_END
```

- Prediction / 预测: 总时间 30 ticks，CPU Busy 为 10 ticks，CPU 利用率为 `10 / 30 × 100% = 33.33%`。

```text
Time  PID0         PID1         CPU  IOs
1     RUN:io       READY        1    -
2     BLOCKED      READY        -    1
3     BLOCKED      READY        -    1
4     BLOCKED      READY        -    1
5     BLOCKED      READY        -    1
6     BLOCKED      READY        -    1
7*    RUN:io_done  READY        1    -
8     RUN:io       READY        1    -
9     BLOCKED      READY        -    1
10    BLOCKED      READY        -    1
11    BLOCKED      READY        -    1
12    BLOCKED      READY        -    1
13    BLOCKED      READY        -    1
14*   RUN:io_done  READY        1    -
15    RUN:cpu      READY        1    -
16    DONE         RUN:cpu      1    -
17    DONE         RUN:io       1    -
18    DONE         BLOCKED      -    1
19    DONE         BLOCKED      -    1
20    DONE         BLOCKED      -    1
21    DONE         BLOCKED      -    1
22    DONE         BLOCKED      -    1
23*   DONE         RUN:io_done  1    -
24    DONE         RUN:io       1    -
25    DONE         BLOCKED      -    1
26    DONE         BLOCKED      -    1
27    DONE         BLOCKED      -    1
28    DONE         BLOCKED      -    1
29    DONE         BLOCKED      -    1
30*   DONE         RUN:io_done  1    -
```

- Reasoning / 理由: 两个进程依次运行，I/O 等待不能与另一进程的指令或 I/O 重叠。四次 I/O 各造成五个 CPU 空闲 ticks，一共空闲 20 ticks。加上 10 个忙 ticks，总时间为 30 ticks。


- Verified result / 验证结果: [q8-s2.txt](q8-s2.txt) 中 SWITCH_ON_END实际输出为总时间 30 ticks、CPU Busy 10 ticks、CPU 利用率 33.33%，I/O Busy 20 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: PID0 结束后才运行 PID1，四次 I/O 等待都无法与另一进程的 CPU 指令或 I/O 等待重叠。每次等待占五个空闲 tick，合计二十个，加上十个 CPU 忙 tick 得到 30 ticks。默认设置的重叠在这里消失，I/O Busy 从 14 增为 20 ticks、总时间从 16 增为 30 ticks；工作量相同，时间差来自调度对重叠的限制。

### Q8 — seed 3

**指令序列：**

- PID0：`cpu, io, io_done, cpu`
- PID1：`io, io_done, io, io_done, cpu`

#### 默认设置

**Command（预测阶段）：**

```bash
python3 process-run.py -s 3 -l 3:50,3:50
```

- Prediction / 预测: 总时间 18 ticks，CPU Busy 为 9 ticks，CPU 利用率为 `9 / 18 × 100% = 50.00%`。

```text
Time  PID0         PID1         CPU  IOs
1     RUN:cpu      READY        1    -
2     RUN:io       READY        1    -
3     BLOCKED      RUN:io       1    1
4     BLOCKED      BLOCKED      -    2
5     BLOCKED      BLOCKED      -    2
6     BLOCKED      BLOCKED      -    2
7     BLOCKED      BLOCKED      -    2
8*    RUN:io_done  BLOCKED      1    1
9*    RUN:cpu      READY        1    -
10    DONE         RUN:io_done  1    -
11    DONE         RUN:io       1    -
12    DONE         BLOCKED      -    1
13    DONE         BLOCKED      -    1
14    DONE         BLOCKED      -    1
15    DONE         BLOCKED      -    1
16    DONE         BLOCKED      -    1
17*   DONE         RUN:io_done  1    -
18    DONE         RUN:cpu      1    -
```

- Reasoning / 理由: PID0 在第 8 tick 恢复，第 9 tick 执行最后的 CPU 指令。PID1 的第一次 I/O 虽然在第 9 tick 已完成，但延迟运行策略让它先保持 READY，到第 10 tick 才执行 `io_done`，第 11 tick 才发起第二次 I/O。CPU 在第 4 至 7 tick 空闲四个 ticks，之后第二次 I/O 又造成五个空闲 ticks。共忙 9 ticks、空闲 9 ticks，总计 18 ticks。


- Verified result / 验证结果: [q8-s3.txt](q8-s3.txt) 中 默认设置实际输出为总时间 18 ticks、CPU Busy 9 ticks、CPU 利用率 50.00%，I/O Busy 11 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: PID0 第 8 tick 恢复后，第 9 tick 继续执行最后一条 cpu；同一 tick PID1 的 I/O 已完成，但 IO_RUN_LATER 让 PID1 保持 READY。PID1 第 10 tick 执行 io_done、第 11 tick 才发起下一次 I/O，此时 PID0 已结束。第 4–7 tick 的四个空闲 tick 加上第 12–16 tick 的五个空闲 tick，共九个；再加九个忙 tick，验证了 18 ticks 的预测。

#### `IO_RUN_IMMEDIATE`

**Command（预测阶段）：**

```bash
python3 process-run.py -s 3 -l 3:50,3:50 -I IO_RUN_IMMEDIATE
```

- Prediction / 预测: 总时间 17 ticks，CPU Busy 为 9 ticks，CPU 利用率为 `9 / 17 × 100% = 52.94%`。

```text
Time  PID0         PID1         CPU  IOs
1     RUN:cpu      READY        1    -
2     RUN:io       READY        1    -
3     BLOCKED      RUN:io       1    1
4     BLOCKED      BLOCKED      -    2
5     BLOCKED      BLOCKED      -    2
6     BLOCKED      BLOCKED      -    2
7     BLOCKED      BLOCKED      -    2
8*    RUN:io_done  BLOCKED      1    1
9*    READY        RUN:io_done  1    -
10    READY        RUN:io       1    -
11    RUN:cpu      BLOCKED      1    1
12    DONE         BLOCKED      -    1
13    DONE         BLOCKED      -    1
14    DONE         BLOCKED      -    1
15    DONE         BLOCKED      -    1
16*   DONE         RUN:io_done  1    -
17    DONE         RUN:cpu      1    -
```

- Reasoning / 理由: 第 9 tick 的 I/O 完成后，PID1 立即抢占 PID0，执行 `io_done`。它在第 10 tick 发起下一次 I/O，比默认设置提前一个 tick；PID0 的最后一条 CPU 指令移到第 11 tick，覆盖这次等待的第一个 tick。空闲时间从 9 ticks 降为 8 ticks，总时间减少到 17 ticks，CPU 忙 ticks 仍为 9。


- Verified result / 验证结果: [q8-s3.txt](q8-s3.txt) 中 IO_RUN_IMMEDIATE实际输出为总时间 17 ticks、CPU Busy 9 ticks、CPU 利用率 52.94%，I/O Busy 11 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: 第 9 tick PID1 的 I/O 完成后立即运行 io_done，PID0 转为 READY；PID1 第 10 tick 发起第二次 I/O，比默认设置早一个 tick。PID0 的最后一条 cpu 移到第 11 tick，正好覆盖这次等待的第一个 tick，因此 CPU 空闲时间从九个降到八个 tick。CPU Busy 仍为九个 tick、I/O Busy 仍为十一个 tick，但更早启动 I/O 使总时间减少到 17 ticks。

#### `SWITCH_ON_END`

**Command（预测阶段）：**

```bash
python3 process-run.py -s 3 -l 3:50,3:50 -S SWITCH_ON_END
```

- Prediction / 预测: 总时间 24 ticks，CPU Busy 为 9 ticks，CPU 利用率为 `9 / 24 × 100% = 37.50%`。

```text
Time  PID0         PID1         CPU  IOs
1     RUN:cpu      READY        1    -
2     RUN:io       READY        1    -
3     BLOCKED      READY        -    1
4     BLOCKED      READY        -    1
5     BLOCKED      READY        -    1
6     BLOCKED      READY        -    1
7     BLOCKED      READY        -    1
8*    RUN:io_done  READY        1    -
9     RUN:cpu      READY        1    -
10    DONE         RUN:io       1    -
11    DONE         BLOCKED      -    1
12    DONE         BLOCKED      -    1
13    DONE         BLOCKED      -    1
14    DONE         BLOCKED      -    1
15    DONE         BLOCKED      -    1
16*   DONE         RUN:io_done  1    -
17    DONE         RUN:io       1    -
18    DONE         BLOCKED      -    1
19    DONE         BLOCKED      -    1
20    DONE         BLOCKED      -    1
21    DONE         BLOCKED      -    1
22    DONE         BLOCKED      -    1
23*   DONE         RUN:io_done  1    -
24    DONE         RUN:cpu      1    -
```

- Reasoning / 理由: PID0 完成后才开始 PID1。三次 I/O 都无法与其他进程工作重叠，共造成 15 个空闲 ticks。加上 9 个 CPU 忙 ticks，总时间为 24 ticks。


- Verified result / 验证结果: [q8-s3.txt](q8-s3.txt) 中 SWITCH_ON_END实际输出为总时间 24 ticks、CPU Busy 9 ticks、CPU 利用率 37.50%，I/O Busy 15 ticks。逐 tick 的进程状态与原预测一致。
- Analysis / 分析: PID0 先完整运行，PID1 到第 10 tick 才开始。三个 I/O 等待段都没有其他进程的工作可以覆盖，也不会互相重叠，每段五个 tick，共十五个 CPU 空闲 tick。九个忙 tick 加十五个空闲 tick 得到 24 ticks；与默认设置相比，阻止重叠增加了六个 tick，CPU 利用率降到 37.50%。

### Q8 预测比较

| Seed | 设置 | 总时间（ticks） | CPU Busy（ticks） | CPU 利用率 |
|---|---|---|---|---|
| 1 | 默认 | 15 | 8 | 53.33% |
| 1 | IO_RUN_IMMEDIATE | 15 | 8 | 53.33% |
| 1 | SWITCH_ON_END | 18 | 8 | 44.44% |
| 2 | 默认 | 16 | 10 | 62.50% |
| 2 | IO_RUN_IMMEDIATE | 16 | 10 | 62.50% |
| 2 | SWITCH_ON_END | 30 | 10 | 33.33% |
| 3 | 默认 | 18 | 9 | 50.00% |
| 3 | IO_RUN_IMMEDIATE | 17 | 9 | 52.94% |
| 3 | SWITCH_ON_END | 24 | 9 | 37.50% |

遇到 I/O 就切换能利用其他 READY 进程，减少 CPU 空闲。立即运行 I/O 完成的进程在 seed 1、2 中不会改变轨迹，在 seed 3 中则能提前发起下一次 I/O，缩短一个 tick。只在结束时切换让两个进程顺序执行，这三个种子下都会增加总时间并降低 CPU 利用率。
