# CacheGen-Optimization
CacheGen Optimization 
Overview
The goal of this experiment is to optimize the transmission strategy of CacheGen, a framework designed for efficient KV-cache streaming in Large Language Models (LLMs).I tackle the problem of Deadline-Constrained Transmission, where a batch of data blocks with varying importance levels must be delivered before a strict deadline.

The Problem
Existing transmission strategies, such as Pure Adaptive (averaging), often struggle with importance-weighted data. They tend to be Importance Blind, treating all blocks as equal. This often leads to wasting precious bandwidth on low-priority blocks early in the sequence, leaving high-priority blocks with suboptimal quality.

Our Solution: Dynamic Iterative Optimization
We introduced a Dynamic Iterative Optimization algorithm. Instead of following a fixed path or a simple average, our algorithm treats transmission as a continuous optimization problem that is recalculated at every step.

Algorithm Logic (Pseudo-code):
Python
For each block in the batch:
    1. Check remaining time and remaining blocks.
    2. Start with a "Safe Plan": Assume all remaining blocks will use Level 1 (lowest quality).
    3. Optimization Loop (Prioritize Value):
       For each importance level (from 5 down to 1):
           For each block in the remaining batch with that importance:
               Try upgrading this block to Level 3.
               If it fits the deadline (considering the rest stay at Level 1), keep Level 3.
               Else, try upgrading to Level 2.
    4. Commit: Execute only the current block at the level determined by this plan.
    5. Repeat: Go to next block and recalculate everything based on new elapsed time.
Key Mechanisms:
Global Look-Ahead Planning: Before transmitting each block, the algorithm performs a "mental simulation" of the entire remaining sequence to find the best possible quality levels.

Immediate Commitment: After finding the optimal plan for the entire remaining batch, the algorithm only commits to the encoding level for the current block.

Continuous Correction: By recalculating the plan before every single block, the algorithm constantly adjusts to the actual time elapsed, ensuring that the most valuable blocks always get the best possible treatment.

## Methodology and Experimental Setup

This simulation evaluates two real-time transmission strategies: Pure Adaptive (Uniform Resource Allocation) and Dynamic Iterative (Priority-based Optimization).

### 1. System Constants
- Data Volume: 40 blocks per session.
- Network Speed: Fixed at 1 Mbps (Base Speed).
- Quality Tiers:
    - L1: 2,400 bits (0.3 accuracy)
    - L2: 4,000 bits (0.5 accuracy)
    - L3: 8,000 bits (2.0 accuracy)
- Scoring: Total Score = Sum of (Importance x Accuracy)

### 2. Experimental Design
To ensure statistical robustness, the simulation utilizes a multi-layered testing approach:
- 10 Independent Iterations: The entire test suite runs 10 times to ensure results are not skewed.
- Randomized Block Generation: In each iteration, 40 blocks are generated with randomized Importance levels (1-5).
- Deadline Sweeping: Each set of blocks is tested against 120 unique deadlines (ranging from 0.05s to 0.35s).
- Result Averaging: Scores are averaged across all 10 iterations to provide a stable performance curve.

### 3. Performance Metrics
- Success: If Total Time <= Deadline, the total accumulated score is awarded.
- Failure: If Total Time > Deadline, a Zero score is awarded (Hard deadline penalty).
- Reliability Tracking: Failure counts are recorded for every deadline to measure the robustness of each algorithm under stress.

### 4. Baseline Expectations (Noise Type 1)
Under zero-noise conditions, the Dynamic Iterative approach is expected to:
1. Maintain zero failures due to precise, deterministic time-budgeting.
2. Outperform the Pure Adaptive approach by intelligently prioritizing L3 quality for high-importance blocks while downscaling low-importance ones to L1.
![image_alt](https://github.com/OmriHanoch/CacheGen-Optimization/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202026-02-04%20132335.png?raw=true)




================================================================================
REAL DATA TRACE (Deadline: 0.3282s)
================================================================================

APPROACH: Pure Adaptive
Status: SUCCESS
Sequence: Block 0: (Imp:3, L:3, Speed:1000Kbps, Time:0.0080s) -> Block 1: (Imp:1, L:3, Speed:1000Kbps, Time:0.0160s) -> Block 2: (Imp:3, L:3, Speed:1000Kbps, Time:0.0240s) -> Block 3: (Imp:1, L:3, Speed:1000Kbps, Time:0.0320s) -> Block 4: (Imp:3, L:3, Speed:1000Kbps, Time:0.0400s) -> Block 5: (Imp:2, L:3, Speed:1000Kbps, Time:0.0480s) -> Block 6: (Imp:4, L:3, Speed:1000Kbps, Time:0.0560s) -> Block 7: (Imp:2, L:3, Speed:1000Kbps, Time:0.0640s) -> Block 8: (Imp:5, L:3, Speed:1000Kbps, Time:0.0720s) -> Block 9: (Imp:3, L:3, Speed:1000Kbps, Time:0.0800s) -> Block 10: (Imp:5, L:1, Speed:100Kbps, Time:0.1040s) -> Block 11: (Imp:5, L:1, Speed:100Kbps, Time:0.1280s) -> Block 12: (Imp:1, L:2, Speed:1000Kbps, Time:0.1320s) -> Block 13: (Imp:2, L:2, Speed:1000Kbps, Time:0.1360s) -> Block 14: (Imp:1, L:2, Speed:1000Kbps, Time:0.1400s) -> Block 15: (Imp:1, L:2, Speed:1000Kbps, Time:0.1440s) -> Block 16: (Imp:4, L:2, Speed:1000Kbps, Time:0.1480s) -> Block 17: (Imp:5, L:2, Speed:1000Kbps, Time:0.1520s) -> Block 18: (Imp:3, L:3, Speed:1000Kbps, Time:0.1600s) -> Block 19: (Imp:5, L:3, Speed:1000Kbps, Time:0.1680s) -> Block 20: (Imp:3, L:3, Speed:1000Kbps, Time:0.1760s) -> Block 21: (Imp:4, L:1, Speed:100Kbps, Time:0.2000s) -> Block 22: (Imp:3, L:2, Speed:1000Kbps, Time:0.2040s) -> Block 23: (Imp:4, L:2, Speed:1000Kbps, Time:0.2080s) -> Block 24: (Imp:1, L:2, Speed:1000Kbps, Time:0.2120s) -> Block 25: (Imp:1, L:2, Speed:1000Kbps, Time:0.2160s) -> Block 26: (Imp:3, L:3, Speed:1000Kbps, Time:0.2240s) -> Block 27: (Imp:4, L:3, Speed:1000Kbps, Time:0.2320s) -> Block 28: (Imp:5, L:3, Speed:1000Kbps, Time:0.2400s) -> Block 29: (Imp:1, L:3, Speed:1000Kbps, Time:0.2480s) -> Block 30: (Imp:5, L:3, Speed:1000Kbps, Time:0.2560s) -> Block 31: (Imp:5, L:1, Speed:100Kbps, Time:0.2800s) -> Block 32: (Imp:5, L:2, Speed:1000Kbps, Time:0.2840s) -> Block 33: (Imp:5, L:2, Speed:1000Kbps, Time:0.2880s) -> Block 34: (Imp:1, L:2, Speed:1000Kbps, Time:0.2920s) -> Block 35: (Imp:4, L:2, Speed:1000Kbps, Time:0.2960s) -> Block 36: (Imp:4, L:3, Speed:1000Kbps, Time:0.3040s) -> Block 37: (Imp:4, L:3, Speed:1000Kbps, Time:0.3120s) -> Block 38: (Imp:2, L:3, Speed:1000Kbps, Time:0.3200s) -> Block 39: (Imp:3, L:3, Speed:1000Kbps, Time:0.3280s)
Final Score: 162.7

APPROACH: Dynamic Iterative
Status: FAILED
Sequence: Block 0: (Imp:3, L:3, Speed:1000Kbps, Time:0.0080s) -> Block 1: (Imp:1, L:3, Speed:1000Kbps, Time:0.0160s) -> Block 2: (Imp:3, L:3, Speed:1000Kbps, Time:0.0240s) -> Block 3: (Imp:1, L:3, Speed:1000Kbps, Time:0.0320s) -> Block 4: (Imp:3, L:3, Speed:1000Kbps, Time:0.0400s) -> Block 5: (Imp:2, L:3, Speed:1000Kbps, Time:0.0480s) -> Block 6: (Imp:4, L:3, Speed:1000Kbps, Time:0.0560s) -> Block 7: (Imp:2, L:3, Speed:1000Kbps, Time:0.0640s) -> Block 8: (Imp:5, L:1, Speed:100Kbps, Time:0.0880s) -> Block 9: (Imp:3, L:1, Speed:100Kbps, Time:0.1120s) -> Block 10: (Imp:5, L:3, Speed:1000Kbps, Time:0.1200s) -> Block 11: (Imp:5, L:3, Speed:1000Kbps, Time:0.1280s) -> Block 12: (Imp:1, L:3, Speed:1000Kbps, Time:0.1360s) -> Block 13: (Imp:2, L:3, Speed:1000Kbps, Time:0.1440s) -> Block 14: (Imp:1, L:3, Speed:1000Kbps, Time:0.1520s) -> Block 15: (Imp:1, L:2, Speed:1000Kbps, Time:0.1560s) -> Block 16: (Imp:4, L:3, Speed:1000Kbps, Time:0.1640s) -> Block 17: (Imp:5, L:3, Speed:1000Kbps, Time:0.1720s) -> Block 18: (Imp:3, L:1, Speed:100Kbps, Time:0.1960s) -> Block 19: (Imp:5, L:3, Speed:1000Kbps, Time:0.2040s) -> Block 20: (Imp:3, L:3, Speed:1000Kbps, Time:0.2120s) -> Block 21: (Imp:4, L:3, Speed:1000Kbps, Time:0.2200s) -> Block 22: (Imp:3, L:3, Speed:1000Kbps, Time:0.2280s) -> Block 23: (Imp:4, L:3, Speed:1000Kbps, Time:0.2360s) -> Block 24: (Imp:1, L:1, Speed:1000Kbps, Time:0.2384s) -> Block 25: (Imp:1, L:1, Speed:1000Kbps, Time:0.2408s) -> Block 26: (Imp:3, L:1, Speed:100Kbps, Time:0.2648s) -> Block 27: (Imp:4, L:2, Speed:1000Kbps, Time:0.2688s) -> Block 28: (Imp:5, L:3, Speed:1000Kbps, Time:0.2768s) -> Block 29: (Imp:1, L:1, Speed:1000Kbps, Time:0.2792s) -> Block 30: (Imp:5, L:3, Speed:1000Kbps, Time:0.2872s) -> Block 31: (Imp:5, L:3, Speed:1000Kbps, Time:0.2952s) -> Block 32: (Imp:5, L:3, Speed:1000Kbps, Time:0.3032s) -> Block 33: (Imp:5, L:3, Speed:1000Kbps, Time:0.3112s) -> Block 34: (Imp:1, L:1, Speed:100Kbps, Time:0.3352s) -> Block 35: (Imp:4, L:1, Speed:1000Kbps, Time:0.3376s) -> Block 36: (Imp:4, L:1, Speed:1000Kbps, Time:0.3400s) -> Block 37: (Imp:4, L:1, Speed:1000Kbps, Time:0.3424s) -> Block 38: (Imp:2, L:1, Speed:1000Kbps, Time:0.3448s) -> Block 39: (Imp:3, L:1, Speed:1000Kbps, Time:0.3472s)
Final Score: 0
================================================================================


============================================================
FAILURE COUNTS BY DEADLINE
============================================================
Deadline     Adaptive Failures    Iterative Failures
------------------------------------------------------------
0.1500       200                  200
0.1534       200                  200
0.1567       200                  200
0.1601       200                  200
0.1634       200                  200
0.1668       200                  200
0.1702       200                  200
0.1735       200                  200
0.1769       200                  200
0.1803       200                  200
0.1836       200                  200
0.1870       200                  200
0.1903       200                  200
0.1937       200                  200
0.1971       200                  200
0.2004       200                  200
0.2038       200                  200
0.2071       200                  200
0.2105       200                  200
0.2139       200                  200
0.2172       199                  200
0.2206       199                  200
0.2239       200                  200
0.2273       194                  200
0.2307       194                  200
0.2340       188                  200
0.2374       186                  200
0.2408       185                  200
0.2441       190                  200
0.2475       178                  200
0.2508       167                  200
0.2542       167                  200
0.2576       163                  200
0.2609       161                  200
0.2643       157                  200
0.2676       146                  200
0.2710       156                  198
0.2744       157                  200
0.2777       142                  199
0.2811       148                  200
0.2845       149                  200
0.2878       142                  197
0.2912       146                  199
0.2945       143                  193
0.2979       151                  199
0.3013       150                  194
0.3046       144                  196
0.3080       140                  196
0.3113       144                  194
0.3147       146                  191
0.3181       132                  191
0.3214       147                  187
0.3248       138                  192
0.3282       141                  193
0.3315       137                  187
0.3349       140                  186
0.3382       151                  186
0.3416       150                  184
0.3450       151                  179
0.3483       152                  176
0.3517       137                  171
0.3550       133                  177
0.3584       154                  168
0.3618       140                  170
0.3651       134                  157
0.3685       147                  165
0.3718       144                  162
0.3752       135                  160
0.3786       134                  136
0.3819       140                  142
0.3853       123                  133
0.3887       119                  138
0.3920       121                  118
0.3954       106                  96
0.3987       107                  120
0.4021       92                   86
0.4055       79                   73
0.4088       90                   72
0.4122       53                   47
0.4155       59                   58
0.4189       30                   33
0.4223       30                   32
0.4256       29                   36
0.4290       21                   15
0.4324       7                    4
0.4357       5                    9
0.4391       4                    1
0.4424       1                    3
0.4458       5                    2
0.4492       0                    0
0.4525       0                    0
0.4559       0                    0
0.4592       0                    0
0.4626       0                    0
0.4660       0                    0
0.4693       0                    0
0.4727       0                    0
0.4761       0                    0
0.4794       0                    0
0.4828       0                    0
0.4861       0                    0
0.4895       0                    0
0.4929       0                    0
0.4962       0                    0
0.4996       0                    0
0.5029       0                    0
0.5063       0                    0
0.5097       0                    0
0.5130       0                    0
0.5164       0                    0
0.5197       0                    0
0.5231       0                    0
0.5265       0                    0
0.5298       0                    0
0.5332       0                    0
0.5366       0                    0
0.5399       0                    0
0.5433       0                    0
0.5466       0                    0
0.5500       0                    0
============================================================
