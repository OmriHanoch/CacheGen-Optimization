# CacheGen-Optimization
CacheGen Optimization Project - Phase 1
Overview
The goal of this experiment is to optimize the transmission strategy of CacheGen, a framework designed for efficient KV-cache streaming in Large Language Models (LLMs). In Phase 1, we tackle the problem of Deadline-Constrained Transmission, where a batch of data blocks with varying importance levels must be delivered before a strict deadline.

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

Results & Analysis
The experiment was conducted using 40 blocks with importance levels ranging from 1 to 5.

Enhanced Accuracy Curves: The results show that our Dynamic Iterative strategy consistently maintains a higher total accuracy score across the entire range of deadlines.
![image_alt](https://github.com/OmriHanoch/CacheGen-Optimization/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202026-02-04%20132335.png?raw=true)
Strategic Sacrifice (Pruning): The transmission trace reveals that our algorithm intelligently "sacrifices" low-importance blocks (sending them at Level 1) to "save" time for high-importance blocks, ensuring they are sent at Level 3.

APPROACH: Pure Adaptive
Status: SUCCESS
Sequence: (Imp:2, L:2) -> (Imp:4, L:2) -> (Imp:5, L:2) -> (Imp:2, L:2) -> (Imp:5, L:2) -> (Imp:5, L:2) -> (Imp:5, L:2) -> (Imp:4, L:2) -> (Imp:4, L:2) -> (Imp:4, L:2) -> (Imp:2, L:2) -> (Imp:5, L:2) -> (Imp:4, L:2) -> (Imp:4, L:2) -> (Imp:1, L:2) -> (Imp:2, L:2) -> (Imp:5, L:2) -> (Imp:5, L:2) -> (Imp:2, L:2) -> (Imp:1, L:2) -> (Imp:4, L:2) -> (Imp:5, L:2) -> (Imp:1, L:2) -> (Imp:2, L:2) -> (Imp:5, L:2) -> (Imp:5, L:2) -> (Imp:5, L:2) -> (Imp:3, L:2) -> (Imp:4, L:2) -> (Imp:3, L:2) -> (Imp:1, L:2) -> (Imp:3, L:3) -> (Imp:5, L:3) -> (Imp:3, L:3) -> (Imp:2, L:3) -> (Imp:2, L:3) -> (Imp:4, L:3) -> (Imp:5, L:3) -> (Imp:5, L:3) -> (Imp:4, L:3)
Final Score: 120.5
--------------------------------------------------
APPROACH: Dynamic Iterative
Status: SUCCESS
Sequence: (Imp:2, L:1) -> (Imp:4, L:3) -> (Imp:5, L:3) -> (Imp:2, L:1) -> (Imp:5, L:3) -> (Imp:5, L:3) -> (Imp:5, L:3) -> (Imp:4, L:3) -> (Imp:4, L:3) -> (Imp:4, L:3) -> (Imp:2, L:1) -> (Imp:5, L:3) -> (Imp:4, L:2) -> (Imp:4, L:2) -> (Imp:1, L:1) -> (Imp:2, L:1) -> (Imp:5, L:3) -> (Imp:5, L:3) -> (Imp:2, L:1) -> (Imp:1, L:1) -> (Imp:4, L:1) -> (Imp:5, L:3) -> (Imp:1, L:1) -> (Imp:2, L:1) -> (Imp:5, L:3) -> (Imp:5, L:3) -> (Imp:5, L:3) -> (Imp:3, L:1) -> (Imp:4, L:2) -> (Imp:3, L:1) -> (Imp:1, L:1) -> (Imp:3, L:1) -> (Imp:5, L:3) -> (Imp:3, L:1) -> (Imp:2, L:1) -> (Imp:2, L:1) -> (Imp:4, L:2) -> (Imp:5, L:3) -> (Imp:5, L:2) -> (Imp:4, L:1)
Final Score: 184.5
