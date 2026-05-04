# Assignment 2: Capacitated Facility Location Problem (TS+GA) — Student Guide

**Course:** CS616 - Optimization Algorithms  
**Semester:** 2, AY 2025-2026  
**Due Date:** May 16, 2026, 11:59 PM  
**Instructors:** Dr. Mahdi Khemakhem & Dr. Essra Aldessouki  
**Total Marks:** 20 + 10% Bonus (5% + 5%)

---

## 📋 Table of Contents

1. [Assignment Overview](#1-assignment-overview)
2. [Learning Objectives](#2-learning-objectives)
3. [Problem Description](#3-problem-description)
4. [Implementation Guide](#4-implementation-guide)
5. [How to Run the Notebook](#5-how-to-run-the-notebook)
6. [Output Files Structure](#6-output-files-structure)
7. [Analysis Questions](#7-analysis-questions)
8. [Bonus Challenges](#8-bonus-challenges-optional--up-to-4-extra-marks)
9. [Grading Rubric](#9-grading-rubric)
10. [Submission Guidelines](#10-submission-guidelines)
11. [Academic Integrity](#11-academic-integrity)
12. [Your Analysis and Answers](#12-your-analysis-and-answers)
    - [Section A: Solution Quality Analysis](#section-a-solution-quality-analysis-4-marks)
    - [Section B: Computational Efficiency Analysis](#section-b-computational-efficiency-analysis-4-marks)
    - [Section C: Scalability Analysis](#section-c-scalability-analysis-3-marks)
    - [Section D: Quality-Time Tradeoff Analysis](#section-d-quality-time-tradeoff-analysis-3-marks)
    - [Section E: Algorithm Understanding](#section-e-algorithm-understanding-3-marks)
    - [Section F: Critical Reflection](#section-f-critical-reflection-3-marks)

---

## 1. Assignment Overview

In this assignment, you will:

1. **Study** two complementary metaheuristics for the Capacitated Facility Location Problem (CFLP): **Tabu Search (TS)** and a **Genetic Algorithm (GA)**.
2. **Compare** both metaheuristics against the exact MIP solver **Gurobi** on 144 test instances.
3. **Analyze** the performance of each approach in terms of solution quality, computational time, and scalability.
4. **Document** your findings in this file with detailed, data-driven analysis.

### What You Need to Do

✅ **Set your Gurobi version** in the Student Configuration cell (Section 11 of the notebook)  
✅ **Run ALL cells** in the notebook from top to bottom  
✅ **Analyze the generated results** (CSV, Excel, convergence plots, visualization plots)  
✅ **Answer all questions** in Section 13 of this document with specific references to your results  
✅ **Submit** the notebook + this completed document + all result files  

---

## 2. Learning Objectives

Upon completing this assignment, you will be able to:

### Tabu Search (TS)
- Explain the role of the **tabu list** and **tabu tenure** in preventing cycles.
- Describe the four move types used (open, close, swap, reassign) and how candidate selection works.
- Apply the **aspiration criterion** to override tabu status when beneficial.
- Tune TS hyperparameters (tenure, candidate size, stopping criterion).

### Genetic Algorithm (GA)
- Represent CFLP solutions as **binary chromosomes** and explain the repair operator.
- Apply **tournament selection**, **uniform crossover**, and **bit-flip mutation**.
- Use **elitism** to preserve the best solutions across generations.
- Understand how GA explores the solution space differently from TS.

### Comparative Analysis
- Measure and interpret **optimality gap (%)** relative to Gurobi for both TS and GA.
- Assess **computational speedup** of each metaheuristic over the exact solver.
- Evaluate **scalability** across problem sizes and difficulty levels.
- Choose the right metaheuristic for a given problem context.

---

## 3. Problem Description

The **Capacitated Facility Location Problem (CFLP)** is a classical NP-hard combinatorial optimization problem.

### Decision Variables
- $y_j \in \{0, 1\}$: whether facility $j$ is opened.
- $x_{ij} \in \{0, 1\}$: whether client $i$ is served by facility $j$.

### Objective
$$\min \sum_{j} f_j \cdot y_j + \sum_{i} \sum_{j} c_{ij} \cdot x_{ij}$$

where $f_j$ is the fixed opening cost of facility $j$ and $c_{ij}$ is the service cost of assigning client $i$ to facility $j$.

### Constraints
- Each client is assigned to exactly one facility: $\sum_j x_{ij} = 1 \quad \forall i$
- A facility can only serve clients if it is open: $x_{ij} \leq y_j \quad \forall i, j$
- Capacity constraint: $\sum_i d_i \cdot x_{ij} \leq Q_j \cdot y_j \quad \forall j$

---

## 4. Implementation Guide

The notebook `Assignment_2.ipynb` is organized into **12 sections**:

| Section | Content |
|---------|---------|
| 1 | Problem introduction and algorithm overview |
| 2 | Library imports |
| 3 | CFLP instance generator (144 instances: 3 sizes × 3 difficulties × 16) |
| 4 | Core utilities (`compute_total_cost`, `is_feasible`, greedy initial solution) |
| **5** | **Tabu Search (TS) implementation** |
| **6** | **Genetic Algorithm (GA) implementation** |
| 7 | Gurobi exact solver (`gurobi_solve_cflp`) |
| 8 | Experiment runner (`run_experiments`) — runs all three methods |
| 9 | Visualization (`visualize_results`) |
| 10 | Main execution pipeline |
| **11** | **Student Configuration** ← edit this section |
| 12 | Bonus challenges (optional) |

### Section 5 — Tabu Search

The TS implementation uses:
- **Four move types**: `open` (open a closed facility), `close` (close an open facility), `swap` (exchange two facilities), `reassign` (move a client to a different facility).
- **Tabu list**: stores inverse moves (as tuples) for `tabu_tenure` iterations to prevent cycling.
- **Aspiration criterion**: a tabu move is accepted if its cost is better than the **global best** cost found so far.
- **Candidate list strategy**: evaluates `max_candidates` neighbors per iteration and selects the best non-tabu (or aspiration) move.
- **Early stopping**: halts after `max_no_improve` consecutive non-improving iterations.

Key functions:
- `generate_open_moves`, `generate_close_moves`, `generate_swap_moves`, `generate_reassign_moves`
- `get_inverse_move`, `generate_all_candidates`, `select_best_candidate`
- `tabu_search_solve_cflp(instance, max_iterations, tabu_tenure, max_candidates, max_no_improve)`

### Section 6 — Genetic Algorithm

The GA implementation uses:
- **Chromosome**: binary vector $y \in \{0,1\}^m$ indicating which facilities are open.
- **Repair operator** (`repair_chromosome`): if the chromosome is infeasible (insufficient capacity), randomly opens more facilities until the solution becomes feasible.
- **Fitness**: total cost = fixed costs + service costs (after computing greedy client assignments).
- **Tournament selection**: randomly picks `tournament_size` chromosomes and returns the best.
- **Uniform crossover**: each gene is independently taken from parent 1 or parent 2 with probability 0.5.
- **Bit-flip mutation**: each gene is flipped independently with probability `mutation_rate`.
- **Elitism**: the top `elitism` individuals survive unchanged into the next generation.

Key functions:
- `repair_chromosome`, `initialize_ga_population`, `evaluate_chromosome`
- `tournament_selection`, `uniform_crossover`, `mutate_chromosome`
- `genetic_algorithm_solve_cflp(instance, population_size, max_generations, crossover_rate, mutation_rate, tournament_size, elitism, max_no_improve)`

---

## 5. How to Run the Notebook

### Prerequisites

```bash
pip install numpy pandas matplotlib seaborn scipy openpyxl gurobipy
```

You also need a valid Gurobi license. Visit [gurobi.com/academia](https://www.gurobi.com/academia/) for a free academic license.

### Step-by-Step

1. **Open** `Assignment_2.ipynb` in Jupyter Lab or VS Code.
2. **Navigate to Section 11** (Student Configuration) and set:
   - `GUROBI_VERSION = "academic"` or `"free"` depending on your license.
   - Adjust TS and GA hyperparameters if needed (defaults are reasonable).
3. **Run all cells** (Kernel → Restart & Run All).
4. Wait for all 144 instances to be processed (~10–60 minutes depending on your machine and Gurobi version).
5. Review the output files described in Section 6.

> **Tip**: If you have only the free Gurobi license, set `GUROBI_VERSION = "free"`. Large instances (50 facilities, 100 clients) will be skipped for Gurobi but will still be solved by TS and GA.

---

## 6. Output Files Structure

After running the notebook, the following files will be generated:

### `results_comparison.csv`

One row per instance with the following columns:

| Column | Description |
|--------|-------------|
| `instance_id` | Unique instance identifier |
| `instance_name` | Filename of the instance |
| `size` | `small` / `medium` / `large` |
| `difficulty` | `easy` / `moderate` / `expensive` |
| `num_facilities` | Number of facilities |
| `num_clients` | Number of clients |
| `gurobi_cost` | Optimal cost found by Gurobi |
| `gurobi_time` | Gurobi solve time (seconds) |
| `gurobi_status` | `Optimal` / `Failed` / `Skipped (Size Limit)` |
| `ts_cost` | Best cost found by Tabu Search |
| `ts_time` | TS solve time (seconds) |
| `ts_iterations` | Total TS iterations executed |
| `ts_status` | `Completed` / `Failed` |
| `ts_gap_percent` | TS optimality gap: (ts_cost − gurobi_cost) / gurobi_cost × 100 |
| `ts_speedup_factor` | gurobi_time / ts_time |
| `ga_cost` | Best cost found by Genetic Algorithm |
| `ga_time` | GA solve time (seconds) |
| `ga_generations` | Total GA generations executed |
| `ga_status` | `Completed` / `Failed` |
| `ga_gap_percent` | GA optimality gap: (ga_cost − gurobi_cost) / gurobi_cost × 100 |
| `ga_speedup_factor` | gurobi_time / ga_time |

### `results_comparison.xlsx`

Multi-sheet Excel file with:
- `All Results`: full results table
- `TS by Size` / `TS by Difficulty`: TS summary statistics grouped by size or difficulty
- `GA by Size` / `GA by Difficulty`: GA summary statistics
- `TS Gap Matrix` / `GA Gap Matrix`: mean gap (%) in a size × difficulty pivot table

### `convergence_plots/`

One PNG per instance showing:
- **Blue line**: Tabu Search best cost trajectory over iterations
- **Orange line**: GA best cost trajectory over generations
- **Red dashed line**: Gurobi optimal cost (reference)

### `visualization_plots/`

Summary plots across all instances:
- `gap_by_size.png` — Boxplots of TS and GA gap by instance size
- `gap_by_difficulty.png` — Boxplots of TS and GA gap by difficulty
- `computation_time.png` — Mean time comparison (Gurobi vs TS vs GA)
- `speedup_factor.png` — TS and GA speedup over Gurobi
- `ts_vs_ga_scatter.png` — Head-to-head scatter plot of TS gap vs GA gap
- `gap_heatmap.png` — Size × difficulty heatmap of mean gap for TS and GA

---

## 7. Analysis Questions

Answer all questions in **Section 13** of this document. Use **specific numbers from your results** (mean gaps, speedup factors, etc.). Vague answers without data references will not receive full marks.

---

### Section A: Solution Quality Analysis (4 marks)

**A1.** Report the **mean optimality gap (%)** for Tabu Search and Genetic Algorithm across all instances. Which method achieves lower gap overall?

**A2.** Compare the gaps of TS and GA broken down by **instance size** (small / medium / large). Does gap increase with problem size for both methods? Is the trend the same for TS and GA?

**A3.** Compare the gaps broken down by **difficulty** (easy / moderate / expensive). Which difficulty level is hardest for TS? For GA? Explain why this may be the case.

**A4.** Looking at the `ts_vs_ga_scatter.png` plot: on what fraction of instances does TS outperform GA (lower gap)? Are there instance types where one method consistently dominates?

---

### Section B: Computational Efficiency Analysis (4 marks)

**B1.** Report the **mean computation time** for Gurobi, TS, and GA for each instance size. By how many times (speedup factor) is TS faster than Gurobi on large instances?

**B2.** Report the **mean speedup factor** for GA over Gurobi across all sizes. Compare it to TS speedup. Which metaheuristic is faster and by how much?

**B3.** How does the Gurobi solving time grow as instance size increases (small → medium → large)? Compare this growth rate to the growth in TS and GA times.

**B4.** Is there a trade-off between quality and speed? Identify which method offers the best balance of low gap and low computation time, and justify your answer.

---

### Section C: Scalability Analysis (3 marks)

**C1.** Analyze the `gap_heatmap.png` plot. Which combination of (size, difficulty) results in the highest gap for TS? For GA?

**C2.** Based on your results, do you expect TS or GA to scale better to very large CFLP instances (e.g., 200 facilities, 500 clients)? Justify your answer by referring to algorithmic complexity.

**C3.** For large instances, Gurobi may hit its time limit without proving optimality. How does this affect the validity of the gap measurements? What would be a better benchmark in this case?

---

### Section D: Quality-Time Tradeoff Analysis (3 marks)

**D1.** If a practitioner needs a **solution within 10 seconds** for a medium instance, which method would you recommend: TS or GA? Support your recommendation with numbers from your results.

**D2.** If solution quality (minimizing gap) is the **top priority** and time is not a constraint, which approach — TS, GA, or Gurobi — would you recommend for each instance size?

**D3.** The convergence plots show how best cost improves over time. For a typical large instance: at what fraction of the total iterations/generations has TS found its best solution? How about GA? What does this tell you about early stopping?

---

### Section E: Algorithm Understanding (3 marks)

**E1 (Tabu Search).** Explain in your own words how the **tabu list** prevents cycling in Tabu Search. What is the role of the **aspiration criterion**? Give a concrete example from the CFLP context (e.g., describe what a specific tabu move looks like and when aspiration would override it).

**E2 (Genetic Algorithm).** Explain how the **repair operator** in the GA ensures solution feasibility. What would happen without it? Describe how **uniform crossover** combines two parent solutions in the CFLP context.

**E3 (Comparison).** Tabu Search is a *trajectory-based* method, while GA is a *population-based* method. Explain what this means in terms of how each method explores the solution space. What are the key advantages and disadvantages of each paradigm for the CFLP?

---

### Section F: Critical Reflection (3 marks)

**F1.** What are the **three most important hyperparameters** for Tabu Search in this problem? How did the default values affect the results you observed? What would you change and why?

**F2.** What are the **three most important hyperparameters** for the Genetic Algorithm in this problem? How does increasing `population_size` or `max_generations` affect solution quality and runtime?

**F3.** Propose **one concrete algorithmic improvement** for each method (TS and GA) that you believe would most significantly reduce the optimality gap on large instances. Justify your choices based on the theoretical properties of each algorithm.

---

## 8. Bonus Challenges (Optional — Up to +10% Extra Credit)

These challenges require you to **systematically test all proposed enhancements**, then **select and fully implement the single best one** for each algorithm. Simply trying one without comparing the others will not receive full marks.

### Bonus A — Improve Tabu Search (+5%)

**Step 1 — Explore all options.** Run a quick experiment (e.g., on the small/moderate subset) for each of the following enhancements and record the mean gap improvement:

| Enhancement | Description |
|-------------|-------------|
| **Adaptive tenure** | Dynamically increase tenure when the search stagnates; decrease it when improvements are frequent. |
| **Aspiration+ criterion** | Allow a tabu move if it improves the global best by a threshold δ (stronger than the standard criterion). |
| **Frequency-based memory** | Track long-term move frequencies and add a penalty to discourage frequently revisited configurations. |
| **Diversification restart** | After a stagnation threshold, apply a strong multi-swap perturbation to escape local optima. |

**Step 2 — Choose ONE.** Select the enhancement that gave the best gap reduction in Step 1. Justify your choice.

**Step 3 — Full implementation.** Apply your chosen enhancement to the full 144-instance experiment. Compare mean gap and runtime before vs after.

Report in Section 12: (a) screening results for all four options, (b) your selected enhancement and justification, (c) full results (gap and runtime) before and after.

### Bonus B — Improve Genetic Algorithm (+5%)

**Step 1 — Explore all options.** Run a quick experiment on the small/moderate subset for each of the following enhancements and record the mean gap improvement:

| Enhancement | Description |
|-------------|-------------|
| **Adaptive mutation** | Decrease mutation rate as population converges; increase it if diversity drops below a threshold. |
| **Two-point crossover** | Replace uniform crossover with two-point crossover; compare convergence speed and final gap. |
| **Memetic GA (TS + GA)** | Apply a short Tabu Search local refinement to the best chromosome each generation. |
| **Diversity preservation** | Track Hamming distance between chromosomes; actively reintroduce diversity when the population converges. |

**Step 2 — Choose ONE.** Select the enhancement that gave the best gap reduction in Step 1. Justify your choice.

**Step 3 — Full implementation.** Apply your chosen enhancement to the full 144-instance experiment. Compare mean gap and runtime before vs after.

Report in Section 12: (a) screening results for all four options, (b) your selected enhancement and justification, (c) full results (gap and runtime) before and after.

---

## 9. Grading Rubric

| Section | Topic | Marks |
|---------|-------|-------|
| **A** | Solution Quality Analysis | 4 |
| **B** | Computational Efficiency Analysis | 4 |
| **C** | Scalability Analysis | 3 |
| **D** | Quality-Time Tradeoff Analysis | 3 |
| **E** | Algorithm Understanding (TS tabu list, GA operators, TS vs GA) | 3 |
| **F** | Critical Reflection | 3 |
| **Bonus A** | TS Improvement (best of 4 tested) | +5% |
| **Bonus B** | GA Improvement (best of 4 tested) | +5% |
| **Total** | | **20 (+10%)** |

### Marking Criteria

- **Full marks**: Data-driven answers that cite specific numbers from results; clear conceptual understanding; insightful analysis.
- **Partial marks**: Correct but vague answers without data references; minor conceptual errors.
- **No marks**: Answers that do not reference results, copy text from the notebook, or are left blank.

---

## 10. Submission Guidelines

Submit a **single ZIP archive** named `StudentID_Assignment2.zip` containing:

```
StudentID_Assignment2/
├── Assignment_2.ipynb          (fully executed notebook with all outputs)
├── Assignment_2_Student_Guide.md  (this file, with Section 13 fully completed)
├── results_comparison.csv
├── results_comparison.xlsx
├── convergence_plots/          (all convergence PNGs)
└── visualization_plots/        (all visualization PNGs)
```

> **Important**: The notebook must show all cell outputs (do not clear outputs before submitting). Notebooks that have not been run will not be graded.

---

## 11. Academic Integrity

- All submitted work must be your own.
- You may discuss concepts with classmates, but **do not share code or written answers**.
- Suspected plagiarism will be handled according to university policy and may result in a mark of 0.
- AI tools (ChatGPT, GitHub Copilot, etc.) may be used to **understand** algorithms but the written analysis in Section 13 must reflect **your own results** from running the notebook.


---

## 12. Your Analysis and Answers

> **Instructions**: Replace each *[Your answer here]* placeholder with your own analysis. Use specific numbers from `results_comparison.csv` and the generated plots. Answers should be written in complete sentences. Bullet points are acceptable for structured comparisons.

---

### Section A: Solution Quality Analysis (4 marks)

#### A1. Overall Mean Optimality Gap

*[Report the mean gap (%) for TS and GA across all instances where Gurobi found an optimal solution. State which method achieves lower gap.]*

#### A2. Gap by Instance Size

*[Report mean TS gap and GA gap for small, medium, and large instances. Describe the trend and compare between TS and GA.]*

#### A3. Gap by Difficulty Level

*[Report mean TS gap and GA gap for easy, moderate, and expensive instances. Identify which difficulty is hardest for each method and explain why.]*

#### A4. TS vs GA Head-to-Head (ts_vs_ga_scatter.png)

*[State what fraction of instances TS outperforms GA (point is below the diagonal). Identify instance types (if any) where one method consistently dominates.]*

---

### Section B: Computational Efficiency Analysis (4 marks)

#### B1. Computation Time and TS Speedup

*[Report mean Gurobi time, TS time, and GA time for each size. Report the TS speedup factor on large instances.]*

#### B2. GA Speedup vs TS Speedup

*[Report mean GA speedup factor across all sizes. Compare to TS speedup.]*

#### B3. Gurobi Time Growth vs Metaheuristic Time Growth

*[Describe how much Gurobi time increases from small → medium → large. Compare to TS and GA time growth.]*

#### B4. Best Balance of Quality and Speed

*[State which method offers the best quality-speed balance. Reference specific gap and time values.]*

---

### Section C: Scalability Analysis (3 marks)

#### C1. Gap Heatmap Analysis (gap_heatmap.png)

*[Identify the (size, difficulty) cell with the highest TS gap and the cell with the highest GA gap. Interpret what this means.]*

#### C2. Expected Scalability to Very Large Instances

*[State whether TS or GA is expected to scale better to very large instances. Justify using algorithmic complexity and your observed results.]*

#### C3. Validity of Gap Measurements When Gurobi Hits Time Limit

*[Explain how gap validity is affected when Gurobi does not prove optimality. Suggest a better benchmark.]*

---

### Section D: Quality-Time Tradeoff Analysis (3 marks)

#### D1. Best Method for 10-Second Time Budget (Medium Instances)

*[State your recommendation (TS or GA) for a 10-second time budget on medium instances. Cite mean times and gaps.]*

#### D2. Best Method When Quality Is Top Priority (per size)

*[State your recommendation for each size (small / medium / large) when solution quality is paramount and time is not a constraint.]*

#### D3. Convergence Timing in Convergence Plots

*[For a typical large instance, at what fraction of total iterations (TS) and generations (GA) is the best solution found? What does this imply about early stopping?]*

---

### Section E: Algorithm Understanding (3 marks)

#### E1. Tabu Search: Tabu List and Aspiration Criterion

*[Explain in your own words how the tabu list prevents cycling. Explain the aspiration criterion. Give a concrete CFLP example of a tabu move and when aspiration would override it.]*

#### E2. Genetic Algorithm: Repair Operator and Crossover

*[Explain how the repair operator ensures feasibility and what would happen without it. Describe how uniform crossover combines two parent CFLP solutions.]*

#### E3. Trajectory-based vs Population-based Exploration

*[Explain what trajectory-based (TS) and population-based (GA) mean for exploration. State key advantages and disadvantages of each paradigm for CFLP.]*

---

### Section F: Critical Reflection (3 marks)

#### F1. Most Important TS Hyperparameters

*[Name the three most important TS hyperparameters. Discuss how the defaults affected your results and what you would change.]*

#### F2. Most Important GA Hyperparameters

*[Name the three most important GA hyperparameters. Discuss how increasing population_size and max_generations would affect quality and runtime.]*

#### F3. Proposed Algorithmic Improvements

*[Propose one concrete improvement for TS and one for GA. Justify each based on theoretical properties and your observed results.]*

---

### Bonus A: TS Improvement (if attempted)

**(a) Screening results — one row per enhancement tested on the small/moderate subset:**

| Enhancement tested | Mean gap before (%) | Mean gap after (%) | Improvement |
|-------------------|--------------------|--------------------|-------------|
| Adaptive tenure | | | |
| Aspiration+ criterion | | | |
| Frequency-based memory | | | |
| Diversification restart | | | |

**(b) Selected enhancement:** *[Name the enhancement you chose and explain why it gave the best improvement.]*

**(c) Full 144-instance results:**
- Mean gap before enhancement: *[value]* %
- Mean gap after enhancement: *[value]* %
- Mean runtime before: *[value]* s — after: *[value]* s

---

### Bonus B: GA Improvement (if attempted)

**(a) Screening results — one row per enhancement tested on the small/moderate subset:**

| Enhancement tested | Mean gap before (%) | Mean gap after (%) | Improvement |
|-------------------|--------------------|--------------------|-------------|
| Adaptive mutation | | | |
| Two-point crossover | | | |
| Memetic GA (TS + GA) | | | |
| Diversity preservation | | | |

**(b) Selected enhancement:** *[Name the enhancement you chose and explain why it gave the best improvement.]*

**(c) Full 144-instance results:**
- Mean gap before enhancement: *[value]* %
- Mean gap after enhancement: *[value]* %
- Mean runtime before: *[value]* s — after: *[value]* s
