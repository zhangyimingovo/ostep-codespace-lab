# Chapter 7: Scheduling — Introduction

## Setup

All jobs arrive at time 0 and have known CPU lengths. There is no I/O or switching cost. Job IDs start at 0 and follow the input order. Jobs of equal length keep that order. Times are in simulator units.

- Response = first start − arrival.
- Turnaround = completion − arrival.
- Wait = turnaround − CPU length.

The calculations are below. The simulator results and explanations follow in the next section.

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

**Prediction.** The averages match when FIFO already lists the jobs from shortest to longest. Equal-length jobs also work. If a longer job of length a comes just before a shorter job of length b, swapping them reduces total turnaround by a − b. For positive lengths, any such pair makes FIFO worse than SJF.

The tables use the original job IDs. Equal-length jobs keep their order here. Swapping them can change individual finish times but leaves the average unchanged.

| Lengths | Policy | Responses by job | Turnarounds by job | Avg response | Avg turnaround |
| --- | --- | --- | --- | ---: | ---: |
| 20,20,20 | FIFO / SJF | 0,20,40 | 20,40,60 | 20.00 | 40.00 |
| 10,10,100 | FIFO / SJF | 0,10,20 | 10,20,120 | 10.00 | 50.00 |
| 100,10,10 | FIFO | 0,100,110 | 100,110,120 | 70.00 | 110.00 |
| 100,10,10 | SJF | 20,0,10 | 120,10,20 | 10.00 | 50.00 |

The final pair shows the convoy effect: short jobs wait behind the long job under FIFO.

## Q5 — Equal response under SJF and RR

**Prediction.** One way to get the same response for every job is to list the RR queue in shortest-job order and make q long enough for every job before the last one to finish in one turn. The final job can be longer than q because the other jobs have already finished. For equal-length jobs, q at least that length works. A single job always has response 0.

For 10,20,40, q=20 is sufficient, even though 40 exceeds q. With q=5, jobs 1 and 2 respond sooner.

| Lengths | Policy | Responses by job | Turnarounds by job | Avg response | Avg turnaround |
| --- | --- | --- | --- | ---: | ---: |
| 10,20,40 | SJF | 0,10,30 | 10,30,70 | 13.33 | 36.67 |
| 10,20,40 | RR q=20 | 0,10,30 | 10,30,70 | 13.33 | 36.67 |
| 10,20,40 | RR q=5 | 0,5,10 | 20,45,70 | 5.00 | 45.00 |
| 30,10,20 | SJF | 30,0,10 | 60,10,30 | 13.33 | 33.33 |
| 30,10,20 | RR q=15 | 0,15,25 | 55,25,60 | 13.33 | 46.67 |

The last two rows have the same average response, but the individual responses differ. The averages can match even when the schedules do not.

## Q6 — SJF as job lengths increase

**Prediction.** For three equal jobs of length L, responses are 0,L,2L, so the mean is L. Completions are L,2L,3L, so mean turnaround is 2L. Doubling all lengths doubles the response times.

| Lengths | SJF responses | SJF turnarounds | Avg response | Avg turnaround |
| --- | --- | --- | ---: | ---: |
| 10,10,10 | 0,10,20 | 10,20,30 | 10.00 | 20.00 |
| 20,20,20 | 0,20,40 | 20,40,60 | 20.00 | 40.00 |
| 40,40,40 | 0,40,80 | 40,80,120 | 40.00 | 80.00 |
| 10,10,40 | 0,10,20 | 10,20,60 | 10.00 | 30.00 |

Response depends on earlier jobs. Extending only the last job need not change any first-start time, as the first and last rows show.

## Q7 — RR as the quantum increases

**Prediction.** Job IDs start at 0. Job j first runs after each earlier job takes one turn:

R_j = sum(i < j) min(L_i, q).

Each earlier job runs for at most q, so R_j <= j*q. With N jobs, worst response is at most (N−1)*q. This value is reached when all N−1 earlier jobs are at least q long. Average response is at most q*(N−1)/2.

This assumes all jobs are ready at time 0, with no I/O, new arrivals, or switching cost. It describes the first run of each job.

For three length-200 jobs, mean response is min(q,200). It rises with q, then stays at 200. When q divides 200 and q <= 200, final completions are 600−2q,600−q,600, so mean turnaround is 600−q. For q >= 200, RR runs each job to completion and behaves like FIFO.

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

The results match the calculations above. All jobs arrive at time 0, and the simulator has no I/O or switching cost. Job IDs start at 0. Response time is how long a job waits before its first run. Turnaround time is how long it takes to finish. Total wait time is turnaround minus the job's length.

The runs use seed 1234. The full output is in [data/](data/), and the commands are in [run-manifest.json](data/run-manifest.json). The simulator rounds to two decimal places, so 400/3 is shown as 133.33.

## Q1: Three jobs of length 200

FIFO and SJF give the same result. All three jobs have the same length, so both run job 0, then job 1, then job 2. Their response times are 0, 200, and 400. They finish at 200, 400, and 600.

The averages are 200.00 for response, 400.00 for turnaround, and 200.00 for wait, as calculated above.

## Q2: Lengths 100, 200, and 300

The jobs are already listed from shortest to longest. SJF therefore runs them in the same order as FIFO. Their response times are 0, 100, and 300, and they finish at 100, 300, and 600.

Both policies give average response 133.33, average turnaround 333.33, and average wait 133.33. SJF does not improve this example because FIFO already puts the short jobs first.

## Q3: RR with quantum 1

With a quantum of 1, the jobs first run at 0, 1, and 2. Both workloads have average response 1.00.

| Lengths | Finish times by job ID | Avg response | Avg turnaround | Avg wait |
|---|---|---:|---:|---:|
| 200,200,200 | 598,599,600 | 1.00 | 599.00 | 399.00 |
| 100,200,300 | 298,499,600 | 1.00 | 465.67 | 265.67 |

RR lets every job start sooner, but each job has to wait between turns. Average turnaround goes from 400 to 599 for the equal jobs, and from 333.33 to 465.67 for the unequal jobs. Both workloads still take 600 time units to finish all the work.

## Q4: When do FIFO and SJF have the same turnaround?

| Lengths | Policy | Avg response | Avg turnaround | Avg wait |
|---|---|---:|---:|---:|
| 20,20,20 | FIFO / SJF | 20.00 | 40.00 | 20.00 |
| 10,10,100 | FIFO / SJF | 10.00 | 50.00 | 10.00 |
| 100,10,10 | FIFO | 70.00 | 110.00 | 70.00 |
| 100,10,10 | SJF | 10.00 | 50.00 | 10.00 |

They have the same average turnaround when FIFO already lists the jobs from shortest to longest. Equal lengths also work. If a longer job of length a comes just before a shorter job of length b, swapping them reduces total turnaround by a-b. For positive lengths, any such pair makes FIFO's average worse than SJF's. Swapping equal-length jobs leaves the average unchanged, although individual finish times can differ.

The 100,10,10 example shows why the order matters. Under FIFO, both short jobs wait for the long job. SJF finishes the short jobs first, reducing average turnaround from 110 to 50.

## Q5: When do SJF and RR have the same response?

One way to get the same response for every job is to put the RR queue in shortest-job order and choose a quantum long enough for each job before the last one to finish in one turn. Then each next job starts at the same time as in SJF. The final job can be longer than the quantum because the other jobs have already finished.

| Lengths | Policy / quantum | Responses by job ID | Avg response | Avg turnaround | Avg wait |
|---|---|---|---:|---:|---:|
| 10,20,40 | SJF | 0,10,30 | 13.33 | 36.67 | 13.33 |
| 10,20,40 | RR / 20 | 0,10,30 | 13.33 | 36.67 | 13.33 |
| 10,20,40 | RR / 5 | 0,5,10 | 5.00 | 45.00 | 21.67 |
| 30,10,20 | SJF | 30,0,10 | 13.33 | 33.33 | 13.33 |
| 30,10,20 | RR / 15 | 0,15,25 | 13.33 | 46.67 | 26.67 |

For 10,20,40, quantum 20 is enough to match SJF. Quantum 5 starts the jobs sooner but makes them finish later on average.

The 30,10,20 example gives the same average response, 13.33, under SJF and RR with quantum 15. However, the response times of individual jobs are different. The averages can match even when the schedules do not.

## Q6: What happens when SJF job lengths increase?

| Lengths | Avg response | Avg turnaround | Avg wait |
|---|---:|---:|---:|
| 10,10,10 | 10.00 | 20.00 | 10.00 |
| 20,20,20 | 20.00 | 40.00 | 20.00 |
| 40,40,40 | 40.00 | 80.00 | 40.00 |
| 10,10,40 | 10.00 | 30.00 | 10.00 |

Longer jobs make later jobs wait longer. For three equal jobs of length L, response times are 0,L,2L, so average response is L. Doubling all job lengths doubles the response times.

For N jobs sorted by length, the average is sum((N-1-i)*L_i)/N. Each job's response depends on the jobs before it. Increasing only the final job from 10 to 40 leaves the first starts unchanged, so average response stays at 10. Its later finish raises average turnaround to 30.

## Q7: What happens when the RR quantum increases?

| Lengths | Quantum | Avg response | Avg turnaround | Avg wait |
|---|---:|---:|---:|---:|
| 200,200,200 | 1 | 1.00 | 599.00 | 399.00 |
| 200,200,200 | 5 | 5.00 | 595.00 | 395.00 |
| 200,200,200 | 10 | 10.00 | 590.00 | 390.00 |
| 200,200,200 | 50 | 50.00 | 550.00 | 350.00 |
| 200,200,200 | 100 | 100.00 | 500.00 | 300.00 |
| 200,200,200 | 300 | 200.00 | 400.00 | 200.00 |
| 1000,1000,1000,1000,1000 | 7 | 14.00 | 4988.00 | 3988.00 |

With a larger quantum, later jobs may wait longer for their first turn. For the three length-200 jobs, average response is min(q,200). It rises until q reaches 200, then stays at 200. When every job fits in one turn, RR runs like FIFO.

Job j first runs after each earlier job has taken one turn:

R_j = sum(min(L_i,q), for i < j).

With N jobs ready at time 0, the last job waits for at most N-1 slices. The worst response is therefore at most (N-1)q. This value is reached when every earlier job is at least q long. The average response is at most q(N-1)/2. These bounds assume no new arrivals or switching cost.

For the equal length-200 jobs, average turnaround is 600-q when q divides 200 and q <= 200. If q does not divide the length, the last turn is shorter. For N equal jobs of length L, let k=ceil(L/q) and r=L-(k-1)q. Here r is the length of the final turn. Job j finishes at N(k-1)q+(j+1)r, so average turnaround is NL-(N-1)r/2. When q divides L, r=q; when q >= L, r=L and the average is L(N+1)/2.

For five length-1000 jobs with q=7, the final turn is 6 time units long. The first starts are 0,7,14,21,28, giving worst response 28 and average response 14. The finish times are 4976,4982,4988,4994,5000, with average turnaround 4988. This agrees with the calculation using the shorter final turn.

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
