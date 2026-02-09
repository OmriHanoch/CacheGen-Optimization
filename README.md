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
## Analysis of Results: Baseline Scenario (Noise Type 1)

The results from the zero-noise simulation confirm our initial hypotheses regarding the performance and behavior of both algorithms under stable network conditions.

### Key Observations
- Superior Utility: As expected, the Dynamic Iterative approach significantly outperforms the Pure Adaptive strategy across almost all tested deadlines.
- Intelligent Prioritization: The iterative algorithm successfully maximizes the total score by prioritizing high-importance blocks for L3 quality while downscaling less critical data, whereas the adaptive approach remains constrained by its uniform resource allocation.
- Deterministic Reliability: In this stable 1 Mbps environment, both algorithms show a clear "success threshold." Once the deadline is long enough to support at least L1 quality for all blocks, the failure rate drops to zero, as the predictable speed allows for perfect time-budgeting.

### Conclusion
These findings validate our baseline expectations: when network conditions are predictable, a dynamic, priority-aware optimization strategy is far more efficient than a simple uniform adaptation.

## Noise Scenario: Gaussian Variability (Noise Type 2)

This scenario introduces stochastic variability to the network, moving beyond the idealized 1 Mbps constant speed to simulate common network jitter.

### 1. Simulation of Network Jitter
- Nature of Noise: We apply a Gaussian (Normal) distribution to the network speed.
- Parameters: The speed fluctuates around the 1 Mbps base, with a standard deviation of 10% (100 Kbps).
- Practical Meaning: This simulates a typical real-world connection where the bandwidth is relatively stable but suffers from constant, minor fluctuations due to interference or cross-traffic.

### 2. Experimental Design
The testing methodology remains consistent to ensure a fair comparison:
- Dynamic Speed Updates: Unlike the baseline, the actual speed is recalculated after every block transmission.
- Feedback Loop: The algorithms receive the "measured speed" from the previous block and must use it to estimate the time required for the next transmission.
- Iterative Testing: 10 independent iterations with randomized importance levels to average out the impact of specific noise patterns.

### 3. Expectations
- Performance Margin: The Dynamic Iterative approach is still expected to lead in score due to its prioritization logic, but the gap may narrow.
- Risk of Failure: We expect to see the first signs of "Hard Failures" (Score 0) at tight deadlines. Since the speed is no longer deterministic, the Dynamic Iterative algorithm might "over-promise" quality (L3) based on a high speed measurement, only to be penalized if the speed drops during the actual transmission.
- Adaptive Resilience: The Pure Adaptive approach is expected to be more robust. By not aiming for maximum quality, it naturally maintains a larger time buffer, making it less sensitive to minor Gaussian fluctuations.

### 4. Real-World Representation
This test represents a "Good Quality" link (like a stable Wi-Fi or Fiber connection) where the main challenge is not a total outage, but the accumulation of small delays that can lead to a deadline violation in high-load scenarios.


![image_alt](https://github.com/OmriHanoch/CacheGen-Optimization/blob/18780dce3dd133d4e2ee295215b4540595a26b42/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202026-02-09%20161226.png)


## Analysis of Results: Gaussian Variability (Noise Type 2)

The results from the Gaussian noise simulation confirm that minor fluctuations in network speed introduce a "risk factor" that specifically impacts optimization-heavy strategies.

### Key Observations
- Consistent Performance Lead: Despite the noise, the Dynamic Iterative approach maintains a higher average score for most deadlines. Its ability to prioritize high-importance blocks remains its core advantage.
- The "Stability Gap" at Tight Deadlines: A critical shift occurs at deadlines between 0.10s and 0.15s. In this range, the Pure Adaptive approach actually achieves higher scores. This happens because the Adaptive approach is more conservative; it doesn't "over-commit" to high quality, allowing it to survive minor speed drops that cause the aggressive Iterative approach to fail and receive a zero score.
- Convergence at High Deadlines: As the deadline increases beyond 0.35s, both algorithms converge toward the maximum possible score. With enough time buffer, even the most aggressive optimization can survive Gaussian jitter.

### Conclusion
These findings confirm our hypothesis: Gaussian noise penalizes "greedy" optimization. While the Dynamic Iterative algorithm is more efficient on average, the Pure Adaptive algorithm proves to be more reliable in high-pressure (tight deadline) scenarios where there is no room for error.



## Noise Scenario: Bursty Traffic (Noise Type 3)

This scenario simulates extreme network instability, where the bandwidth is high for most of the time but suffers from sudden, severe drops.

### 1. Simulation of Bursty Conditions
- Nature of Noise: The network speed follows a "Bursty" pattern rather than a smooth distribution.
- Drop Severity: During a burst event, the capacity drops to 10% of the base speed (from 1 Mbps to 100 Kbps).
- Duration and Frequency: The drops occur every 8-10 blocks and last for 1-2 blocks at a time.
- Practical Meaning: This simulates real-world conditions like moving between cellular cells or sudden interference in a crowded wireless environment.

### 2. Experimental Design
- Instantaneous Feedback: The algorithms measure the speed after each block.
- High-Risk Decision Making: The Dynamic Iterative approach must decide whether to "gamble" on high quality during stable periods, knowing a massive drop could be imminent.
- Cumulative Delay: Because the drops are so severe (90% loss), any delay caused by a burst is difficult to recover from within a tight deadline.

### 3. Expectations
- The Optimization Paradox: We expect the Dynamic Iterative approach to suffer significantly. Its strategy of using up the "time budget" for high-importance blocks leaves it with no margin to handle a 90% speed drop.
- Survival of the Simplest: The Pure Adaptive approach is expected to be the clear winner in terms of reliability. By maintaining a low-quality profile (L1/L2), it naturally builds a large "Time Buffer" that allows it to absorb the impact of a burst without failing the deadline.
- Bimodal Results: At very short deadlines, we expect both algorithms to fail (score 0) as the average speed under bursty conditions becomes mathematically insufficient to send 40 blocks.

### 4. Real-World Representation
This test represents a "Stress Test" for real-time systems. It proves that in unstable environments, the ability to survive a worst-case scenario (Resilience) is often more valuable than the ability to maximize quality during best-case scenarios (Optimization).

![image_alt](https://github.com/OmriHanoch/CacheGen-Optimization/blob/18780dce3dd133d4e2ee295215b4540595a26b42/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202026-02-09%20162044.png)


## Analysis of Results: Bursty Traffic (Noise Type 3)

The Bursty Traffic simulation provides the most significant results of the study, demonstrating a complete reversal of the performance trends observed in stable environments.

### Key Observations
- Failure of Optimization: In this scenario, the Dynamic Iterative approach (Green line) consistently underperforms the Pure Adaptive approach (Blue line) for deadlines between 0.20s and 0.40s. Its "greedy" strategy of maximizing quality during high-speed periods leaves it with zero time-margin, causing it to fail entirely when a 90% speed drop occurs.
- The Resilience Advantage: The Pure Adaptive approach proves far more reliable. Because it does not attempt to maximize quality, it inherently maintains a "Time Buffer." This buffer allows it to absorb the impact of sudden network drops, leading to successful transmissions (and thus higher average scores) where the Dynamic approach results in a zero score.
- Late-Stage Convergence: Only after the deadline exceeds 0.45s—providing enough time to survive multiple bursts regardless of strategy—does the Dynamic Iterative approach regain its lead by optimizing the remaining "safe" time.
Convergence Analysis: The Recovery Point
- Late-Stage Convergence: A key feature of the Bursty Traffic graph is the point where the lines finally meet (around 0.45s - 0.50s). 
- Resource Saturation: This represents the moment where the Deadline becomes large enough to absorb even multiple worst-case bursts. At this point, the "Time Buffer" advantage of the Adaptive approach is no longer necessary, as there is enough time for both algorithms to send all blocks at maximum quality (L3).

### Final Conclusion: Optimization vs. Robustness
The three stages of this experiment confirm a fundamental engineering trade-off:
1. Stable Environments: Dynamic optimization is superior, providing maximum utility with zero risk.
2. Moderate Jitter: Optimization remains effective but begins to show vulnerability at the edges.
3. Volatile (Bursty) Environments: Simplistic, conservative strategies (Adaptive) are superior to complex optimization. Resilience and "margin for error" become more valuable than raw efficiency.

The results confirm our final hypothesis: An algorithm that is "perfectly" tuned for the present is often the most vulnerable to a sudden change in the future.

























