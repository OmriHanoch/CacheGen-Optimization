# Real-Time Adaptive Token Transmission Simulation

## Background & Goal
In Large Language Model (LLM) inference, data blocks (tokens) must often be transmitted over unpredictable networks under strict real-time deadlines. Bandwidth fluctuations can cause transmission delays that result in missed deadlines and failed requests.

The goal of this project is to simulate and evaluate various **Adaptive Transmission Strategies**. These algorithms decide the optimal quality/compression level for each block to maximize the total accuracy score while ensuring all data arrives before the deadline.

## Core Concept: Importance-Based Optimization
A key insight of this project is that **not all data blocks are equally important**. 
- Some tokens carry more semantic weight or are more critical for the model's accuracy.
- Our algorithms prioritize these "Heavy Hitters," ensuring they are sent at higher quality (requiring more bits) when bandwidth allows, while automatically compressing less critical blocks during network drops to save time.

## What We Are Trying to Achieve
We aim to find the optimal balance between **reliability** (meeting the deadline) and **accuracy** (maximizing the quality of transmitted blocks). The simulation tests various methods—from simple moving averages to dynamic safety buffers—to determine which approach handles network noise and "bursty" drops most effectively.

## Simulation Arguments

| Argument | Default | Description |
| :--- | :--- | :--- |
| `noise_type` | `1` | `1`: Constant speed, `2`: Markov Chain (dynamic states), `3`: Bursty (sharp drops). |
| `deadline_start` | `0.00` | Start of the time constraint range (seconds). |
| `deadline_end` | `0.15` | End of the time constraint range (seconds). |
| `num_blocks` | `10` | Total number of tokens/blocks per session. |
| `num_iterations` | `20` | Number of noisy simulations per deadline point for statistical averaging. |
| `ma_window_size` | `10` | Window size for the Moving Average speed predictor. |
| `safety_margin_percent`| `10` | Percentage of remaining time reserved as a safety buffer. |
| `success_only` | `false` | If `true`, accuracy is averaged only for runs that met the deadline. |
| `show_layered` | `false` | Enables **Layered Coding**, sending base layers before enhancements. |
| `save_pdf` | `false` | Exports the comparison graphs as PDF files. |

## Running the Simulation
To run the simulation, use the `python` command with the desired parameters. Ensure you have `numpy` and `matplotlib` installed.

### Example Execution Command:
```powershell
python client2(name_of_your_file).py deadline_start=0.05 deadline_end=0.45 num_blocks=40 num_iterations=100 ma_window_size=3 safety_margin_percent=50 noise_type=2 success_only=true show_layered=true save_pdf=1
