# CPU Scheduler & Deadlock Detection and Recovery System
Simulation of a single-core CPU scheduling system implementing preemptive priority scheduling with round robin, aging, and deadlock detection and recovery. The system manages multiple processes and resource types, tracks execution using a Gantt chart, and reports performance metrics.

---

## Program Features

* **CPU Scheduling:** Preemptive Priority combined with Round Robin (RR) for equal priorities.
* **Resource Management:** Multiple resource types with multiple instances.
* **Deadlock Detection:** Implementation of a detection algorithm to identify circular waits.
* **Deadlock Recovery:** Process Termination or Resource Preemption
* **Aging Mechanism:** Automatically increments process priority (decreases priority value) after 10 time units in the Ready Queue.
* **I/O Handling:** Non-blocking I/O operations with a dedicated waiting queue.

---

## Input Format

The simulator reads from a text file using the following structure:

1.  **System Resources:** `[ResourceID, Instances], ...`
2.  **Process Definition:** `[PID] [Arrival Time] [Priority] [Sequence of Bursts]`

### Example Input
```text
[1,5], [2,3], [5,1]
0 0 1 CPU {R[1,2], 50, F[1,1], 20, F[1,1]}
1 5 1 CPU {20} IO{30} CPU{20, R[2,3], 30, F[2,3], 10}

```
---
### Test Cases:

```
1- [1,1]
0 0 2 CPU{20}
1 5 1 CPU{10}
2 5 1 CPU{10}
```


| PID | Arrival Time | Priority | CPU Burst | Notes                                |
| --: | -----------: | :------: | --------: | ------------------------------------ |
|   0 |            0 |  2 (Low) |        20 | Starts first, but has lower priority |
|   1 |            5 | 1 (High) |        10 | Arrives later with higher priority   |
|   2 |            5 | 1 (High) |        10 | Arrives later with higher priority   |


<img width="665" height="202" alt="Screenshot (270)" src="https://github.com/user-attachments/assets/37b4acb0-fac7-4cb1-9ccd-621267b8a1d2" />

```
Time=0: P0 arrives at t=0, and it starts running for 5 units.
Time=5: P1 and P2 arrive. Both have priority 1. Since P1 has a higher priority than P0, P0 is preempted and moves to the READY queue.
Time=10: P1 finishes its 5-unit time slice, and P2 starts running.
Time=15: P2 finishes its 5-unit time slice. P0 has been sitting in the READY queue for 10 time units, so its priority improves from 2 to 1. The scheduler chooses P1.
T=20: P1 finishes execution, and P2 is dispatched.
T=25: P2 completes its burst, and P0 is dispatched.
T=40: P0 terminates.
```


| PID | Arrival | Finish | Turnaround | Waiting |
| --: | ------: | -----: | ---------: | ------: |
|   0 |       0 |     40 |         40 |      20 |
|   1 |       5 |     25 |         20 |      10 |
|   2 |       5 |     30 |         25 |      15 |

Average Turnaround Time: 28.33
Average Waiting Time: 15.0


----------------------------------------------------------------------------------

```
2-[1,1]
0 0 1 CPU{4} IO{6} CPU{3}
1 2 1 CPU{5}
2 4 2 CPU{2} IO{4} CPU{2}
```


| PID | Arrival Time | Priority | Execution Sequence        |
| --: | -----------: | :------: | ------------------------- |
|   0 |            0 | 1 (High) | `CPU{4} → IO{6} → CPU{3}` |
|   1 |            2 | 1 (High) | `CPU{5}`                  |
|   2 |            4 |  2 (Low) | `CPU{2} → IO{4} → CPU{2}` |

<img width="665" height="202" alt="Screenshot (270)" src="https://github.com/user-attachments/assets/9f77c7a9-8a4d-4c9c-9b03-da342091ca97" />

```
T=0: P0 arrives and starts execution.
T=2: P1 arrives, but it waits in the READY queue.
T=4: P0 completes its first CPU burst and moves to the I/O queue for 6 units. P2 arrives. The scheduler chooses P1 because it has a higher priority.
T=9: P1 completes its burst and terminates. P2 starts its first CPU burst.
T=10: P0 finishes I/O and returns to the READY queue. P0 has a higher priority than P2 (currently running), so P2 is preempted and moved to the READY queue.
T=13: P0 finishes its second CPU burst and terminates. P2 is dispatched to the CPU.
T=14: P2 completes its first CPU burst, a nd moves to the I/O queue for 4 units.
T=14 - 18: CPU is idle.
T=18: P2 finishes I/O and returns to the READY queue. It starts its second CPU burst.
T=20: P2 terminates.
```

| PID | Arrival | Finish | Turnaround | Waiting |
| --: | ------: | -----: | ---------: | ------: |
|   0 |       0 |     13 |         13 |       0 |
|   1 |       2 |      9 |          7 |       2 |
|   2 |       4 |     20 |         16 |       8 |
```
Average Turnaround Time: 12.0
Average Waiting Time: 3.33
```
----------------------------------------------------------------------------------
```
3-[1,1]
0 0 2 CPU{7} IO{5} CPU{6} IO{4} CPU{3}
1 1 1 CPU{5} IO{6} CPU{5}
2 2 1 CPU{4} IO{3} CPU{7}
3 4 0 CPU{6} IO{2} CPU{4}
```

| PID | Arrival Time |  Priority  | Execution Sequence                         |
| --: | -----------: | :--------: | ------------------------------------------ |
|   0 |            0 |   2 (Low)  | `CPU{7} → IO{5} → CPU{6} → IO{4} → CPU{3}` |
|   1 |            1 | 1 (Medium) | `CPU{5} → IO{6} → CPU{5}`                  |
|   2 |            2 | 1 (Medium) | `CPU{4} → IO{3} → CPU{7}`                  |
|   3 |            4 |  0 (High)  | `CPU{6} → IO{2} → CPU{4}`                  |

<img width="665" height="202" alt="Screenshot (270)" src="https://github.com/user-attachments/assets/f11c61bf-94a8-4c18-8be9-c9477cbdfeea" />

```
T=0: P0 arrives and starts its first CPU burst.
T=1: P1 arrives. P0 is preempted because P1 has a higher priority. P1 starts its first burst.
T=2: P2 arrives and waits in the READY queue.
T=4: P3 arrives. P1 is preempted because P3 has a higher priority. P3 starts its first burst.
T=9: P3 finishes its 5-unit time slice. No other process has a higher priority, so it continues.
T=10: P3 moves to I/O. The scheduler selects P1 from the READY queue (which has the highest priority and arrived before P2).
T=12: P1 finishes its first CPU burst and moves to I/O. P3 returns from I/O and starts its second burst.
T=16: P3 finishes its second burst, and the scheduler selects P2.
T=18: P1 returns from I/O, but it doesn’t preempt P2 because both have the same priority.
T=20: P2 finishes its first CPU burst and moves to I/O. P1 starts its second burst.
T=23: P2 returns from I/O, but it doesn’t preempt P1 because they have the same priority.
T=25: P1 finishes its second burst, and P2 is dispatched to the CPU.
T=30: P2 5-unit quantum expires, but there are no higher priority processes in the READY queue, so it continues.
T=32: P2 finishes its second burst. P0 starts.
T=38: P0 finishes its first burst. Goes to I/O for 5 units (CPU is idle).
T=43: P0 returns and starts its second burst (6 units).
T=49: P0 finishes its second burst. Goes to I/O for 4 units (CPU is idle).
T=53: P0 returns. Starts its third burst (3 units).
T=56: P0 finishes.
```
----------------------------------------------------------------------------------
```
4- [1,1], [2,1] 
0 0 2 CPU{R[1,1], 5, R[2,1], 10, F[1,1], 5, F[2,1]} 
1 2 1 CPU{R[2,1], 5, R[1,1], 10, F[2,1], 5, F[1,1]}
```
| PID | Arrival Time | Priority | Sequence of Actions                                                    |
| --: | -----------: | :------: | ---------------------------------------------------------------------- |
|   0 |            0 |  2 (Low) | `Request R1 → Run 5 → Request R2 → Run 10 → Free R1 → Run 5 → Free R2` |
|   1 |            2 | 1 (High) | `Request R2 → Run 5 → Request R1 → Run 10 → Free R2 → Run 5 → Free R1` |

<img width="665" height="202" alt="Screenshot (270)" src="https://github.com/user-attachments/assets/93635852-fa73-468e-8a68-d3fea329edef" />

```
T=0: P0 arrives. P0 requests R1 (granted) and starts its first CPU burst.
T=2: P1 arrives. P0 is preempted because P1 has a higher priority. P1 requests R2 (granted) and starts its first burst.
T=7: P1 finishes its first burst and requests R1. R1 is held by P0, so P1 blocks. P0 resumes its first burst (3 units remaining).
T=10: P0 finishes its first burst and requests R2. R2 is held by P1, so P0 blocks.
At T=10, Deadlock is detected. P0 is selected as the victim and is ABORTED. R1 is released. P1 unblocks (acquires R1) and is dispatched.
T=15: P1 5-unit quantum expires, but there are no other processes in the READY queue, so it continues.
T=20: P1 finishes its second burst (10 units). P1 releases R2 and starts its third burst (5 units).
T=25: P1 finishes its third burst. P1 releases R1 and terminates.
```
| PID | Finish Time |    State   | Notes                          |
| --: | ----------: | :--------: | ------------------------------ |
|   0 |          12 | **KILLED** | Sacrificed to resolve deadlock |
|   1 |          27 | Terminated | Survived and finished          |



```
----------------------------------------------------------------------------------
```
```
5- [1,1], [2,2], [3,1]
0 0 3 CPU{R[1,1], 5, R[2,2], 5, F[1,1], 5, F[2,2]}
1 2 2 CPU{R[2,2], 5, R[3,1], 5, F[2,2], 5, F[3,1]}
2 4 1 CPU{R[3,1], 5, R[1,1], 5, F[3,1], 5, F[1,1]}
```
<img width="665" height="202" alt="Screenshot (270)" src="https://github.com/user-attachments/assets/bc02a926-d518-4cc2-99a2-706915b4fdbe" />

```
T=0: P0 arrives. P0 requests R1 (granted) and starts its first CPU burst.
T=2: P1 arrives. P0 is preempted because P1 has a higher priority. P1 requests R2 (granted) and starts its first burst.
T=4: P2 arrives. P1 is preempted because P2 has a higher priority. P2 requests R3 (granted) and starts its first burst.
T=9: P2 finishes its first burst and requests R1. R1 is held by P0, so P2 blocks (WAITING). P1 resumes its first burst (3 units remaining).
T=12: P1 finishes its first burst and requests R3. R3 is held by P2, so P1 blocks (WAITING). P0 resumes its first burst (3 units remaining).
T=15: P0 finishes its first burst and requests R2. R2 is held by P1, so P0 blocks (WAITING).
At T=15, Deadlock is detected. P1 is selected as the victim (holds most resources: 2 instances of R2) and is ABORTED. R2 is released. P0 unblocks (acquires R2) and starts its second burst.
T=20: P0 finishes its second burst and releases R1. P2 unblocks (acquires R1). P2 preempts P0 because P2 has a higher priority. P2 starts its second burst.
T=25: P2 finishes its second burst and releases R3. P2 starts its third burst.
T=30: P2 finishes its third burst and releases R1. P2 terminates. P0 resumes its third burst.
T=35: P0 finishes its third burst and releases R2. P0 terminates.

```
| PID | Finish Time |    State   | Resource History                                        |
| --: | ----------: | :--------: | ------------------------------------------------------- |
|   1 |          15 | **KILLED** | Held R2 (2 instances). Sacrificed to break the deadlock |
|   2 |          30 | Terminated | Waited for R1. Survived                                 |
|   0 |          35 | Terminated | Waited for R2. Survived                                 |
```
----------------------------------------------------------------------------------
```
```
6-[1,4], [2,5], [3,4]
0 0 0 CPU{4} IO{2} CPU{2} IO{2} CPU{2} IO{2} CPU{2} IO{2} CPU{2}
1 5 1 CPU{R[2,5], 5, R[3,1], 10, F[2,5], 5, F[3,1]}
2 5 1 CPU{R[3,4], 5, R[1,2], 10, F[3,4], 5, F[1,2]}
3 5 1 CPU{R[1,3], 5, R[2,1], 10, F[1,3], 5, F[2,1]}
4 20 2 CPU{5} IO{5} CPU{5}
```
<img width="665" height="202" alt="Screenshot (270)" src="https://github.com/user-attachments/assets/44f4abea-ddaf-446f-8c6d-6273e29e0f98" />

```
T=0: P0 arrives and starts its first CPU burst.
T=4: P0 finishes its first burst and moves to I/O. 
T=5: P1, P2, P3 arrive. P1 starts its first burst and requests R2 (granted).
T=6: P0 returns from I/O. P1 is preempted because P0 has a higher priority. P0 starts its second burst.
T=8: P0 finishes its second burst and moves to I/O. P2 starts its first burst and requests R3 (granted).
T=10: P0 returns from I/O. P2 is preempted because P0 has a higher priority. P0 starts its third burst.
T=12: P0 finishes its third burst and moves to I/O. P3 starts its first burst and requests R1 (granted).
T=14: P0 returns from I/O. P3 is preempted because P0 has a higher priority. P0 starts its fourth burst.
T=16: P0 finishes its fourth burst and moves to I/O. P1 ages to Priority 0. P1 resumes.
T=18: P0 returns from I/O. P1 continues.
T=20: P1 requests R3 (held by P2), so P1 BLOCKS. P4 arrives. P2 ages to Priority 0. P0 starts the fifth burst.
T=22: P0 finishes fifth burst and TERMINATES. P2 resumes.
T=25: P2 requests R1 (held by P3), so P2 BLOCKS. P3 dispatched. 
T=28: P3 requests R2 (held by P1), so P3 BLOCKS. P4 dispatched.
T=33: P4 finishes first burst and moves to I/O. DEADLOCK DETECTED (P1, P2, P3). P1 (holding 5 units) is ABORTED. R2 released. P3 unblocks (acquires R2) and starts.
T=38: P4 returns from I/O. P3 continues.
T=43: P3 releases R1. P2 unblocks (acquires R1). P3 continues (Round Robin).
T=48: P2 dispatched.
T=53: P3 dispatched. P3 releases R2 and TERMINATES. P2 dispatched.
T=58: P2 releases R3. P2 continues. 
T=63: P2 releases R1 and TERMINATES. P4 dispatched. 
T=68: P4 finishes second burst and TERMINATES.
```
----------------------------------------------------------------------------------
```
7-[1,5][2,3][5,1]
0   0   1   CPU {R[1,2], 50, F[1,1], 20, F[1,1]}
1   5   1   CPU {20}  IO{30}  CPU{20, R[2,3], 30, F[2,3], 10}
2   10  0   CPU {R[2,2], 25, F[2,2]} IO{15} CPU{10, R[1,1], 15, F[1,1]}
3   12  2   CPU {30} IO{20} CPU{25}
4   15  1   CPU {10, R[1,2], 20, F[1,2]} IO{10} CPU{5}
```

| PID | Arrival Time |  Priority  | Execution Sequence                                            |
| --: | -----------: | :--------: | ------------------------------------------------------------- |
|   0 |            0 | 1 (Medium) | `CPU{R[1,2],50,F[1,1],20,F[1,1]}`                             |
|   1 |            5 | 1 (Medium) | `CPU{20} → IO{30} → CPU{20,R[2,3],30,F[2,3],10}`              |
|   2 |           10 |  0 (High)  | `CPU{R[2,2],25,F[2,2]} → IO{15} → CPU{10,R[1,1],15,F[1,1]}`   |
|   3 |           12 |   2 (Low)  | `CPU{30} → IO{20} → CPU{25}`                                  |
|   4 |           15 | 1 (Medium) | `CPU{10,R[1,2],20,F[1,2]} → IO{10} → CPU{5}`                  |



<img width="1191" height="357" alt="Screenshot (269)" src="https://github.com/user-attachments/assets/f5f9c258-8136-49dd-ae1d-5af01282c8a7" />


```
T=0: P0 arrives → requests R1:2 → granted → CPU50
T=5: P1 arrives → READY (RR)
T=10: P2 arrives → requests R2:2 → granted → CPU dispatched
T=12: P3 arrives → READY
T=15: P4 arrives → READY
T=20-70: RR preemptions → CPU switches among P0, P1, P2, P3, P4
T=70: P4 requests R1:2 → granted
T=75: P2 releases R2:2 → CPU/I/O switches
T=100-130: Further releases → unblocking
T=130-190: Final CPU bursts and resource releases
T=190-290: Terminations
```
----------------------------------------------------------------------------------
```


