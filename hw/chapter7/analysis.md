# Chapter 7: Scheduling — Introduction

## Model and method

All jobs arrive at time 0, use only the CPU, and have known lengths. There is no context-switch cost. Job IDs follow the input list; ties keep their original order. Values use simulator time units.

- Response = first start − arrival.
- Turnaround = completion − arrival.
- Wait = turnaround − CPU length.

The following predictions were calculated before any run with -c. The simulator was first used without that flag. Verification and comparisons will be appended after the predictions are saved.

## Q1 — Three jobs of length 200

**Prediction.** FIFO and SJF both use order 0, 1, 2. Their intervals are 0–200, 200–400, 400–600.

| Policy | Job | Response | Turnaround | Wait |
| --- | --- | ---: | ---: | ---: |
| FIFO / SJF | 0 | 0 | 200 | 0 |
| FIFO / SJF | 1 | 200 | 400 | 200 |
| FIFO / SJF | 2 | 400 | 600 | 400 |
| FIFO / SJF | Average | 200.00 | 400.00 | 200.00 |

Average response is (0 + 200 + 400)/3 = 200. Average turnaround is (200 + 400 + 600)/3 = 400.

## Q2 — Lengths 100, 200, 300

**Prediction.** The input is already sorted, so FIFO and SJF both use order 0, 1, 2. Their intervals are 0–100, 100–300, 300–600.

| Policy | Job | Response | Turnaround | Wait |
| --- | --- | ---: | ---: | ---: |
| FIFO / SJF | 0 | 0 | 100 | 0 |
| FIFO / SJF | 1 | 100 | 300 | 100 |
| FIFO / SJF | 2 | 300 | 600 | 300 |
| FIFO / SJF | Average | 133.33 | 333.33 | 133.33 |

Exact average response and wait are 400/3. Exact average turnaround is 1000/3.

## Q3 — Round Robin, quantum 1

Both Q1 and Q2 workloads are checked with RR.

**Prediction: 200,200,200.** First starts are 0, 1, 2. After 199 full rounds, time is 597; each job has one unit left. The final slices finish at 598, 599, 600.

| Job | Response | Turnaround | Wait |
| --- | ---: | ---: | ---: |
| 0 | 0 | 598 | 398 |
| 1 | 1 | 599 | 399 |
| 2 | 2 | 600 | 400 |
| Average | 1.00 | 599.00 | 399.00 |

**Prediction: 100,200,300.** After 99 rounds, time is 297. Job 0 finishes at 298; the other two reach 100 units of service by time 300. After another 99 two-job rounds, time is 498. Job 1 finishes at 499, job 2 takes a slice through 500, then runs alone to 600.

| Job | Response | Turnaround | Wait |
| --- | ---: | ---: | ---: |
| 0 | 0 | 298 | 198 |
| 1 | 1 | 499 | 299 |
| 2 | 2 | 600 | 300 |
| Average | 1.00 | 465.67 | 265.67 |

The unequal-job means are exactly 1397/3 for turnaround and 797/3 for wait. RR improves first response but increases average turnaround compared with Q1/Q2 FIFO and SJF.

## Q4 — Equal turnaround under SJF and FIFO

**Prediction.** The averages match when the FIFO list is already in nondecreasing length order. Equal-length jobs are one example. A longer job before a shorter one raises the total completion time: exchanging adjacent lengths a > b reduces it by a − b. For positive lengths, an input containing such an inversion therefore has a larger average turnaround under FIFO.

The lists below show metrics in original job-ID order. Tied jobs retain that order in these checks; changing tie order can exchange individual completion times without changing the average.

| Lengths | Policy | Responses by job | Turnarounds by job | Avg response | Avg turnaround |
| --- | --- | --- | --- | ---: | ---: |
| 20,20,20 | FIFO / SJF | 0,20,40 | 20,40,60 | 20.00 | 40.00 |
| 10,10,100 | FIFO / SJF | 0,10,20 | 10,20,120 | 10.00 | 50.00 |
| 100,10,10 | FIFO | 0,100,110 | 100,110,120 | 70.00 | 110.00 |
| 100,10,10 | SJF | 20,0,10 | 120,10,20 | 10.00 | 50.00 |

The final pair shows the convoy effect: short jobs wait behind the long job under FIFO.

## Q5 — Equal response under SJF and RR

**Prediction.** A sufficient condition is that the initial RR queue follows the SJF order, and q covers every job except the final one. Every earlier job then finishes before the next job first starts. The final job can be longer than q: its later slices cannot delay another first response. For equal-length jobs, q at least the common length is sufficient. A single job always has response 0.

For 10,20,40, q=20 is sufficient, even though 40 exceeds q. With q=5, all jobs respond sooner.

| Lengths | Policy | Responses by job | Turnarounds by job | Avg response | Avg turnaround |
| --- | --- | --- | --- | ---: | ---: |
| 10,20,40 | SJF | 0,10,30 | 10,30,70 | 13.33 | 36.67 |
| 10,20,40 | RR q=20 | 0,10,30 | 10,30,70 | 13.33 | 36.67 |
| 10,20,40 | RR q=5 | 0,5,10 | 20,45,70 | 5.00 | 45.00 |
| 30,10,20 | SJF | 30,0,10 | 60,10,30 | 13.33 | 33.33 |
| 30,10,20 | RR q=15 | 0,15,25 | 55,25,60 | 13.33 | 46.67 |

The last two rows have equal average response but different individual responses. Matching averages alone does not establish identical scheduling or per-job response times.

## Q6 — SJF as job lengths increase

**Prediction.** For three equal jobs of length L, responses are 0,L,2L, so the mean is L. Completions are L,2L,3L, so mean turnaround is 2L. Scaling all lengths increases response linearly.

| Lengths | SJF responses | SJF turnarounds | Avg response | Avg turnaround |
| --- | --- | --- | ---: | ---: |
| 10,10,10 | 0,10,20 | 10,20,30 | 10.00 | 20.00 |
| 20,20,20 | 0,20,40 | 20,40,60 | 20.00 | 40.00 |
| 40,40,40 | 0,40,80 | 40,80,120 | 40.00 | 80.00 |
| 10,10,40 | 0,10,20 | 10,20,60 | 10.00 | 30.00 |

Response depends on earlier jobs. Extending only the last job need not change any first-start time, as the first and last rows show.

## Q7 — RR as the quantum increases

**Prediction.** In the initial queue order, zero-based job j first runs after the first slices of all earlier jobs:

R_j = sum(i < j) min(L_i, q).

Thus R_j <= j*q, and the worst first response among N jobs is at most (N−1)*q. The bound is attained when all N−1 earlier jobs have lengths at least q. Averaging the bounds gives mean response <= q*(N−1)/2.

These bounds assume a fixed set of ready jobs, arrival at time 0, no I/O or later arrivals, and zero switch cost. They concern first response, not total waiting or completion.

For three length-200 jobs, mean response is min(q,200). It rises with q, then saturates at 200. When q divides 200 and q <= 200, final completions are 600−2q,600−q,600, so mean turnaround is 600−q. For q >= 200, RR runs each job to completion and behaves like FIFO.

| RR q | Responses | Turnarounds | Avg response | Avg turnaround |
| ---: | --- | --- | ---: | ---: |
| 1 | 0,1,2 | 598,599,600 | 1.00 | 599.00 |
| 5 | 0,5,10 | 590,595,600 | 5.00 | 595.00 |
| 10 | 0,10,20 | 580,590,600 | 10.00 | 590.00 |
| 50 | 0,50,100 | 500,550,600 | 50.00 | 550.00 |
| 100 | 0,100,200 | 400,500,600 | 100.00 | 500.00 |
| 300 | 0,200,400 | 200,400,600 | 200.00 | 400.00 |

For N=5, lengths 1000,1000,1000,1000,1000 and q=7, responses are 0,7,14,21,28. Worst response is (5−1)*7=28 and the mean is 14. After 142 full rounds, time is 4970; each job has 6 units left. Completions are 4976,4982,4988,4994,5000 and the mean is 4988.


# Verification and discussion

The assignment guide ends without the question text. The scope is the seven homework questions at the end of the official OSTEP Chapter 7, as confirmed for this assignment.

All jobs in these cases arrive at time 0. There is no I/O, idle time, or context-switch cost. Job IDs follow the input order. Response is the first start time, turnaround is the completion time, and wait is turnaround minus job length. FIFO and SJF run each job to completion, so their response and wait are equal. RR can make a job wait again after its first run.

The predictions above were saved before the computed runs. Each case was previewed without `-c`, then checked with `-c` and seed 1234. The 28 output files and `data/run-manifest.json` record the cases and commands. The checked per-job metrics and averages agree with the predictions. Values such as `400/3` appear as `133.33` because the simulator prints two decimal places; this is rounding, not a scheduling discrepancy.

## Q1: Three jobs of length 200

FIFO and SJF both run jobs 0, 1, and 2. Their responses are 0, 200, and 400, and their turnarounds are 200, 400, and 600. Both computed averages match the prediction: response 200.00, turnaround 400.00, and wait 200.00. Equal lengths give SJF no shorter job to move ahead.

## Q2: Lengths 100, 200, and 300

The input already follows SJF order. Both policies give responses 0, 100, and 300, and turnarounds 100, 300, and 600. The computed averages are response 133.33, turnaround 333.33, and wait 133.33, matching the exact predicted values `400/3`, `1000/3`, and `400/3`. Different lengths alone do not make SJF better than FIFO when FIFO is already sorted.

## Q3: RR with quantum 1

Both workloads have first starts at 0, 1, and 2, so their average response is 1.00.

| Lengths | Completion times by job ID | Avg response | Avg turnaround | Avg wait |
|---|---|---:|---:|---:|
| 200,200,200 | 598,599,600 | 1.00 | 599.00 | 399.00 |
| 100,200,300 | 298,499,600 | 1.00 | 465.67 | 265.67 |

The output agrees with both predictions. RR starts every job quickly, but delays completion by sharing the CPU. For equal lengths, average turnaround rises from 400 to 599. For unequal lengths, it rises from `1000/3` to `1397/3`. Total work remains 600 in each case.

## Q4: Equal turnaround under SJF and FIFO

| Lengths | Policy | Avg response | Avg turnaround | Avg wait |
|---|---|---:|---:|---:|
| 20,20,20 | FIFO / SJF | 20.00 | 40.00 | 20.00 |
| 10,10,100 | FIFO / SJF | 10.00 | 50.00 | 10.00 |
| 100,10,10 | FIFO | 70.00 | 110.00 | 70.00 |
| 100,10,10 | SJF | 10.00 | 50.00 | 10.00 |

These results match the predictions. Under the stated assumptions, positive job lengths give the same average turnaround exactly when FIFO is in nondecreasing length order, including ties. Swapping adjacent lengths `a > b` reduces total turnaround by `a - b`; any inversion therefore makes FIFO worse. Equal-length ties can exchange individual completion times without changing the average. The last case shows the convoy effect: FIFO makes both short jobs wait behind the length-100 job.

## Q5: Equal response under SJF and RR

A sufficient condition for the same response for every job is that the RR queue follows SJF order and its quantum covers every job before the final job. Earlier jobs then finish before the next job first runs. The final job can exceed the quantum because all other jobs have finished before its first start.

| Lengths | Policy / quantum | Responses by job ID | Avg response | Avg turnaround | Avg wait |
|---|---|---|---:|---:|---:|
| 10,20,40 | SJF | 0,10,30 | 13.33 | 36.67 | 13.33 |
| 10,20,40 | RR / 20 | 0,10,30 | 13.33 | 36.67 | 13.33 |
| 10,20,40 | RR / 5 | 0,5,10 | 5.00 | 45.00 | 21.67 |
| 30,10,20 | SJF | 30,0,10 | 13.33 | 33.33 | 13.33 |
| 30,10,20 | RR / 15 | 0,15,25 | 13.33 | 46.67 | 26.67 |

All predicted per-job values match the output. Quantum 20 is sufficient for 10,20,40 even though the longest job is 40. Quantum 5 gives faster first responses but longer average turnaround. In the unsorted case, the two averages happen to be `40/3`, while every job has a different response under the two policies. Equality of averages is a weaker condition than equality for each job.

## Q6: Increasing job lengths under SJF

| Lengths | Avg response | Avg turnaround | Avg wait |
|---|---:|---:|---:|
| 10,10,10 | 10.00 | 20.00 | 10.00 |
| 20,20,20 | 20.00 | 40.00 | 20.00 |
| 40,40,40 | 40.00 | 80.00 | 40.00 |
| 10,10,40 | 10.00 | 30.00 | 10.00 |

The results match the predictions. For three equal jobs of length `L`, responses are `0,L,2L`, so average response is `L`. Scaling all lengths scales all response times by the same factor. More generally, for sorted lengths `L_0,...,L_(N-1)`, average response is `sum((N-1-i)*L_i)/N`. It depends on earlier jobs. Extending only the final job in 10,10,10 to 40 leaves all first starts unchanged, although average turnaround rises to 30.

## Q7: Increasing the RR quantum and the worst response

| Lengths | Quantum | Avg response | Avg turnaround | Avg wait |
|---|---:|---:|---:|---:|
| 200,200,200 | 1 | 1.00 | 599.00 | 399.00 |
| 200,200,200 | 5 | 5.00 | 595.00 | 395.00 |
| 200,200,200 | 10 | 10.00 | 590.00 | 390.00 |
| 200,200,200 | 50 | 50.00 | 550.00 | 350.00 |
| 200,200,200 | 100 | 100.00 | 500.00 | 300.00 |
| 200,200,200 | 300 | 200.00 | 400.00 | 200.00 |
| 1000,1000,1000,1000,1000 | 7 | 14.00 | 4988.00 | 3988.00 |

The results match all predicted first starts and completions. For a fixed initial queue, job `j` first runs after earlier jobs each take one slice:

`R_j = sum(min(L_i,q), for i < j)`.

Thus `R_j <= j*q`, and the worst response among `N` jobs is at most `(N-1)*q`. The bound is attained for the last job when every earlier job has length at least `q`. The corresponding average bound is `q*(N-1)/2`. These bounds concern first response for jobs all ready at time 0, with no extra arrivals or switching cost.

For three length-200 jobs, average response is `min(q,200)`: it increases with the quantum until 200, then stays at 200. Once each job fits in one slice, RR has FIFO behavior. For `q <= 200` that divides 200, average turnaround is `600-q`. This completion formula must not be applied to a quantum that does not divide the job length.

For `N` equal jobs of length `L` and any positive quantum, let `k = ceil(L/q)` and `r = L-(k-1)*q`, the positive length of the final slice. After `k-1` full rounds, completion of job `j` is `N*(k-1)*q+(j+1)*r`. Average turnaround is therefore `N*L-(N-1)*r/2`. When `q` divides `L`, `r=q`; when `q >= L`, `r=L`, giving the FIFO average `L*(N+1)/2`.

The five-job check uses `L=1000`, `q=7`, `k=143`, and `r=6`. Responses are 0,7,14,21,28, so the maximum is `(5-1)*7=28` and the average is 14. Completions are 4976,4982,4988,4994,5000. Their average, 4988, confirms the final partial-slice calculation.

## Output index

| Case | Policy | Lengths | Quantum | Avg response | Avg turnaround | Avg wait | Raw output |
|---|---|---|---:|---:|---:|---:|---|
| q1_fifo | FIFO | 200,200,200 | - | 200.00 | 400.00 | 200.00 | [q1_fifo.txt](data/q1_fifo.txt) |
| q2_fifo | FIFO | 100,200,300 | - | 133.33 | 333.33 | 133.33 | [q2_fifo.txt](data/q2_fifo.txt) |
| q1_sjf | SJF | 200,200,200 | - | 200.00 | 400.00 | 200.00 | [q1_sjf.txt](data/q1_sjf.txt) |
| q2_sjf | SJF | 100,200,300 | - | 133.33 | 333.33 | 133.33 | [q2_sjf.txt](data/q2_sjf.txt) |
| q3_equal_rr | RR | 200,200,200 | 1 | 1.00 | 599.00 | 399.00 | [q3_equal_rr.txt](data/q3_equal_rr.txt) |
| q3_unequal_rr | RR | 100,200,300 | 1 | 1.00 | 465.67 | 265.67 | [q3_unequal_rr.txt](data/q3_unequal_rr.txt) |
| q4_equal_fifo | FIFO | 20,20,20 | - | 20.00 | 40.00 | 20.00 | [q4_equal_fifo.txt](data/q4_equal_fifo.txt) |
| q4_sorted_fifo | FIFO | 10,10,100 | - | 10.00 | 50.00 | 10.00 | [q4_sorted_fifo.txt](data/q4_sorted_fifo.txt) |
| q4_equal_sjf | SJF | 20,20,20 | - | 20.00 | 40.00 | 20.00 | [q4_equal_sjf.txt](data/q4_equal_sjf.txt) |
| q4_sorted_sjf | SJF | 10,10,100 | - | 10.00 | 50.00 | 10.00 | [q4_sorted_sjf.txt](data/q4_sorted_sjf.txt) |
| q4_convoy_fifo | FIFO | 100,10,10 | - | 70.00 | 110.00 | 70.00 | [q4_convoy_fifo.txt](data/q4_convoy_fifo.txt) |
| q4_convoy_sjf | SJF | 100,10,10 | - | 10.00 | 50.00 | 10.00 | [q4_convoy_sjf.txt](data/q4_convoy_sjf.txt) |
| q5_sorted_sjf | SJF | 10,20,40 | - | 13.33 | 36.67 | 13.33 | [q5_sorted_sjf.txt](data/q5_sorted_sjf.txt) |
| q5_sorted_rr_q20 | RR | 10,20,40 | 20 | 13.33 | 36.67 | 13.33 | [q5_sorted_rr_q20.txt](data/q5_sorted_rr_q20.txt) |
| q5_sorted_rr_q5 | RR | 10,20,40 | 5 | 5.00 | 45.00 | 21.67 | [q5_sorted_rr_q5.txt](data/q5_sorted_rr_q5.txt) |
| q5_equal_mean_sjf | SJF | 30,10,20 | - | 13.33 | 33.33 | 13.33 | [q5_equal_mean_sjf.txt](data/q5_equal_mean_sjf.txt) |
| q5_equal_mean_rr | RR | 30,10,20 | 15 | 13.33 | 46.67 | 26.67 | [q5_equal_mean_rr.txt](data/q5_equal_mean_rr.txt) |
| q6_length_10_sjf | SJF | 10,10,10 | - | 10.00 | 20.00 | 10.00 | [q6_length_10_sjf.txt](data/q6_length_10_sjf.txt) |
| q6_length_20_sjf | SJF | 20,20,20 | - | 20.00 | 40.00 | 20.00 | [q6_length_20_sjf.txt](data/q6_length_20_sjf.txt) |
| q6_length_40_sjf | SJF | 40,40,40 | - | 40.00 | 80.00 | 40.00 | [q6_length_40_sjf.txt](data/q6_length_40_sjf.txt) |
| q6_last_long_sjf | SJF | 10,10,40 | - | 10.00 | 30.00 | 10.00 | [q6_last_long_sjf.txt](data/q6_last_long_sjf.txt) |
| q7_rr_q1 | RR | 200,200,200 | 1 | 1.00 | 599.00 | 399.00 | [q7_rr_q1.txt](data/q7_rr_q1.txt) |
| q7_rr_q5 | RR | 200,200,200 | 5 | 5.00 | 595.00 | 395.00 | [q7_rr_q5.txt](data/q7_rr_q5.txt) |
| q7_rr_q10 | RR | 200,200,200 | 10 | 10.00 | 590.00 | 390.00 | [q7_rr_q10.txt](data/q7_rr_q10.txt) |
| q7_rr_q50 | RR | 200,200,200 | 50 | 50.00 | 550.00 | 350.00 | [q7_rr_q50.txt](data/q7_rr_q50.txt) |
| q7_rr_q100 | RR | 200,200,200 | 100 | 100.00 | 500.00 | 300.00 | [q7_rr_q100.txt](data/q7_rr_q100.txt) |
| q7_rr_q300 | RR | 200,200,200 | 300 | 200.00 | 400.00 | 200.00 | [q7_rr_q300.txt](data/q7_rr_q300.txt) |
| q7_rr_n5_q7 | RR | 1000,1000,1000,1000,1000 | 7 | 14.00 | 4988.00 | 3988.00 | [q7_rr_n5_q7.txt](data/q7_rr_n5_q7.txt) |
