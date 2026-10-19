# Assignment 2: Capacitated Facility Location Problem (TS+GA) — Student Guide

**Course:** CS616 - Optimization Algorithms  
**Semester:** 1, AY 2026-2027  
**Due Date:** December 05, 2026, 11:59 PM  
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
8. [Bonus Challenges](#8-bonus-challenges-optional--up-to-10-extra-credit)
9. [Grading Rubric](#9-grading-rubric)
10. [Submission Guidelines](#10-submission-guidelines)
11. [Academic Integrity](#11-academic-integrity)
12. [Getting Help](#12-getting-help)
13. [Your Analysis and Answers](#13-your-analysis-and-answers)

---

## 1. Assignment Overview

In this assignment, you will:

1. **Study** two complementary metaheuristics for the Capacitated Facility Location Problem (CFLP): **Tabu Search (TS)** and a **Genetic Algorithm (GA)**.
2. **Compare** both metaheuristics against the exact MIP solver **Gurobi** on 144 test instances.
3. **Analyze** the performance of each approach in terms of solution quality, computational time, and scalability.
4. **Document** your findings in `My_Answers.md` with detailed, data-driven analysis.

### What You Need to Do

✅ **Set your student ID and Gurobi version** in the Student Configuration cell (Section 10 of the notebook)  
✅ **Run ALL cells** in the notebook from top to bottom  
✅ **Analyze the generated results** (CSV, Excel, convergence plots, visualization plots)  
✅ **Answer all questions** in `My_Answers.md` with specific references to your results  
✅ **Submit** the link of your GitHub fork (renamed with your ID and name) on Blackboard - see [Section 10](#10-submission-guidelines)

**IMPORTANT:** You do NOT need to implement anything! All the code is provided. You only need to:
1. Set `STUDENT_ID` and `GUROBI_VERSION` in the Student Configuration cell (Section 10 of the notebook)
2. Run all cells in order
3. Analyze the results and answer the questions in `My_Answers.md`

> **Total experiment runtime: about 10-20 minutes** (depends on your hardware and Gurobi license).

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
| 3 | CFLP instance generator (144 instances: 3 sizes × 3 difficulties × 16), seeded with **your student ID** |
| 4 | Core utilities (`compute_total_cost`, `is_feasible`, greedy initial solution) |
| **5** | **Tabu Search (TS) implementation** |
| **6** | **Genetic Algorithm (GA) implementation** |
| 7 | Gurobi exact solver (`gurobi_solve_cflp`) |
| 8 | Experiment runner (`run_experiments`) — runs all three methods |
| 9 | Visualization (`visualize_results`) |
| **10** | **Student Configuration** ← edit this section (`STUDENT_ID`, `GUROBI_VERSION`) |
| 11 | Main execution pipeline (`main()`) |
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

1. **Python Environment:** Python 3.8+
2. **Required libraries:** numpy, pandas, matplotlib, seaborn, scipy, openpyxl, gurobipy

   ```bash
   pip install numpy pandas matplotlib seaborn scipy openpyxl gurobipy
   ```

3. **Gurobi license:**
   - **Academic license** (recommended): free for students, no size limit. Get it here: https://www.gurobi.com/academia/
   - **Free license:** limited to 2000 variables and 2000 constraints. Large instances will be skipped for Gurobi (this is acceptable) but are still solved by TS and GA.

### Step-by-Step

#### Step 1: Open the notebook
Open `Assignment_2.ipynb` in Jupyter Lab or VS Code.

#### Step 2: Update your student information
Go to **Section 10 (Student Configuration)** and set the two variables:

```python
# TODO: replace 12345 with YOUR actual student ID (it seeds YOUR unique instances)
STUDENT_ID = 12345

GUROBI_VERSION = "academic"   # "academic" or "free"
```

- **`STUDENT_ID`**: your own student ID (e.g., `20231234`). The cell stops with an error if you leave `12345`.
- **`GUROBI_VERSION`**: `"academic"` if you have an academic license, `"free"` otherwise.

**Example:**
```python
STUDENT_ID = 20231234
GUROBI_VERSION = "academic"
```

> Your student ID is the random seed of the instance generator: **your instances and your results are unique**, and the ID appears in the name of every output file. Keep the default TS/GA hyperparameters for the main experiment (they can be changed for the bonus challenges only).

#### Step 3: Run all cells
Select **Kernel → Restart & Run All** (or run the cells one by one from top to bottom with Shift+Enter). The last code cell executes `main()` with your configuration.

#### Step 4: Wait for completion
- **Estimated time: 10-20 minutes** (depends on your machine and Gurobi license).
- Progress is displayed instance by instance.

#### Step 5: Check the generated files
Verify that the output files described in [Section 6](#6-output-files-structure) exist.

#### Step 6: Analyze the results
Open the CSV/Excel files and the generated plots.

#### Step 7: Answer the questions
Complete `My_Answers.md` (do **not** edit this guide).

---

## 6. Output Files Structure

After running the notebook, your working directory contains (for the student ID `20231234`):

```
📁 Your_Working_Directory/
│
├── 📄 Assignment_2.ipynb                              # Your notebook
├── 📄 Assignment_2_Student_Guide.md                   # Guide (do not edit)
├── 📄 My_Answers.md                                   # Your answers (to complete)
│
├── 📁 cflp_instances_student_20231234/                # 144 JSON instances (+ preview PNGs)
├── 📁 convergence_plots_student_20231234/             # One convergence PNG per instance
├── 📁 visualization_plots_student_20231234/           # 6 summary plots
│   ├── gap_by_size.png
│   ├── gap_by_difficulty.png
│   ├── computation_time.png
│   ├── speedup_factor.png
│   ├── ts_vs_ga_scatter.png
│   └── gap_heatmap.png
│
├── 📊 results_student_20231234.csv                    # Raw results (CSV)
└── 📊 results_student_20231234.xlsx                   # Detailed results (multi-sheet)
```

**All generated files and folders include your student ID in their name**, which makes your submission easy to identify and verify. The instance files inside `cflp_instances_student_<ID>/` use standard names, but their content depends on your ID.

### `results_student_<ID>.csv`

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

### `results_student_<ID>.xlsx`

Multi-sheet Excel file with:
- `All Results`: full results table
- `TS by Size` / `TS by Difficulty`: TS summary statistics grouped by size or difficulty
- `GA by Size` / `GA by Difficulty`: GA summary statistics
- `TS Gap Matrix` / `GA Gap Matrix`: mean gap (%) in a size × difficulty pivot table

### `convergence_plots_student_<ID>/`

One PNG per instance showing:
- **Blue line**: Tabu Search best cost trajectory over iterations
- **Orange line**: GA best cost trajectory over generations
- **Red dashed line**: Gurobi optimal cost (reference)

### `visualization_plots_student_<ID>/`

Summary plots across all instances:
- `gap_by_size.png` — Boxplots of TS and GA gap by instance size
- `gap_by_difficulty.png` — Boxplots of TS and GA gap by difficulty
- `computation_time.png` — Mean time comparison (Gurobi vs TS vs GA)
- `speedup_factor.png` — TS and GA speedup over Gurobi
- `ts_vs_ga_scatter.png` — Head-to-head scatter plot of TS gap vs GA gap
- `gap_heatmap.png` — Size × difficulty heatmap of mean gap for TS and GA

---

## 7. Analysis Questions

Answer all questions in **`My_Answers.md`**. Use **specific numbers from YOUR results** (mean gaps, speedup factors, etc.) and reference the plots by name (e.g., "As shown in `gap_by_size.png`..."). Vague answers without data references will not receive full marks. Explain **why** things happen, not only **what** you observe.

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

**D3.** The convergence plots show how best cost improves over time. For a typical large instance: at what fraction of the total iterations/generations has TS found its best solution? How about GA? What does this tell you about early stopping? *(Use 3-5 convergence plots from `convergence_plots_student_<ID>/` and push them in `sample_convergence_plots/`, see Section 10.)*

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

### 📤 Submission for the Bonus

Push the **additional improved notebook(s)** to your forked repository, in addition to your original notebook:

```
StudentID_FirstName_LastName_Assignment2_IMPROVED.ipynb
```

**Example:** `20231234_Ali_Ben_Ahmed_Assignment2_IMPROVED.ipynb`

You may use one improved notebook for both bonuses, or two notebooks (`..._Assignment2_IMPROVED_TS.ipynb` and `..._Assignment2_IMPROVED_GA.ipynb`).

- In the improved notebook, **comment your changes** clearly.
- **Change the output names** in the Student Configuration cell (e.g., add `_improved` to `RESULTS_FILE`, `CONVERGENCE_PLOTS_FOLDER` and `VISUALIZATION_FOLDER`) so that you do **not overwrite** the results of your original run. Push the improved results files (CSV/XLSX and the visualization folder) too.
- Report the screening table, the selected enhancement and the before/after results in the **Bonus A / Bonus B** sections of `My_Answers.md`.

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

- **Full marks**: Data-driven answers that cite specific numbers from YOUR results; clear conceptual understanding; insightful analysis.
- **Partial marks**: Correct but vague answers without data references; minor conceptual errors.
- **No marks**: Answers that do not reference results, copy text from the notebook, or are left blank.

---

## 10. Submission Guidelines

### Submission = Your Own GitHub Fork

You do **not** submit a ZIP file. Instead, you work in **your own fork** of the course repository and submit **the link to your fork** on Blackboard.

### Step 1: Fork and rename the repository

1. Sign in to GitHub (create a free account if needed).
2. Open the course repository: https://github.com/makopt/OA_Assignment_2_481
3. Click **Fork** (top-right) → **Create fork**.
4. In your fork, go to **Settings → General → Repository name** and rename it to:

   ```
   OA_Assignment_2_481_StudentID_FirstName_LastName
   ```

   **Example:** `OA_Assignment_2_481_20231234_Ali_Ben_Ahmed`

5. Clone **your fork** (not the original) to your computer:

   ```bash
   git clone https://github.com/<your-username>/OA_Assignment_2_481_20231234_Ali_Ben_Ahmed.git
   ```

> Keep the fork **public** (GitHub does not allow a fork of a public repository to be made private), so that the instructors can open it without extra access.

### Step 2: Do the work inside your cloned fork

Complete the assignment as described in this guide. Save everything inside the cloned folder.

### Step 3: Commit and push your work

```bash
git add Assignment_2.ipynb My_Answers.md results_student_*.csv results_student_*.xlsx visualization_plots_student_* sample_convergence_plots
git commit -m "Assignment 2 - <StudentID> <FirstName LastName>"
git push origin main
```

Then refresh your repository page on GitHub and check that all files are visible and that the notebook shows its outputs.

### What Your Repository Must Contain

1. ✅ **Completed Jupyter notebook:** `Assignment_2.ipynb`
   - All cells executed successfully, with all outputs visible (do not clear the outputs)
   - Your own student ID set in the Student Configuration cell

2. ✅ **Completed answers file:** `My_Answers.md`
   - All questions (Sections A-F) answered completely
   - Your student ID and name at the top
   - Do not modify `Assignment_2_Student_Guide.md`

3. ✅ **Results files:**
   - `results_student_{YOUR_ID}.csv`
   - `results_student_{YOUR_ID}.xlsx`

4. ✅ **Key visualizations:**
   - All 6 files from `visualization_plots_student_{YOUR_ID}/`

5. ✅ **Sample convergence plots** (needed for question D3): 3-5 plots in `sample_convergence_plots/`

6. ❌ **DO NOT push:**
   - Instance files (`cflp_instances_student_*` folder - too large)
   - The full convergence-plots folder (`convergence_plots_student_*` - too large)

7. ✅ **BONUS (Optional - up to +10%):**
   - `StudentID_FirstName_LastName_Assignment2_IMPROVED.ipynb` (your improved TS and/or GA version)
   - Results of the improved version (with different file names, see [Section 8](#8-bonus-challenges-optional--up-to-10-extra-credit))

**Expected repository content:**
```
OA_Assignment_2_481_20231234_Ali_Ben_Ahmed/
├── Assignment_2.ipynb
├── Assignment_2_Student_Guide.md (unchanged)
├── My_Answers.md (your answers)
├── results_student_20231234.csv
├── results_student_20231234.xlsx
├── visualization_plots_student_20231234/
│   ├── gap_by_size.png
│   ├── gap_by_difficulty.png
│   ├── computation_time.png
│   ├── speedup_factor.png
│   ├── ts_vs_ga_scatter.png
│   └── gap_heatmap.png
└── sample_convergence_plots/ (3-5 plots for D3)
```

> Tip: the repository already contains a `.gitignore` that excludes the generated instance and convergence-plot folders. Check `git status` before committing to be sure nothing large is added.

### Submission Checklist

Before submitting the link, verify:

- [ ] You forked the repository and renamed it `OA_Assignment_2_481_StudentID_FirstName_LastName`
- [ ] Your own student ID is used in the notebook (NOT someone else's!)
- [ ] All cells in the notebook executed successfully and the outputs are visible
- [ ] All questions of this guide are answered completely in `My_Answers.md`
- [ ] Specific numerical values from YOUR results are included
- [ ] Plots are referenced in your answers
- [ ] All required files are committed and **pushed** (visible on GitHub in your browser)
- [ ] The repository is public and the link opens without logging in (test it in a private/incognito window)

### Submission Platform

Submit via: **Blackboard, on the dedicated rubric**

**What to submit on Blackboard:** the **link (URL) of your forked repository**, e.g.
`https://github.com/<your-username>/OA_Assignment_2_481_20231234_Ali_Ben_Ahmed`

**Deadline:** December 05, 2026, 11:59 PM

> The deadline applies to your **last commit**. Commits pushed after the deadline are considered late.

**Late Policy:** Late submissions will be graded 0, no exceptions (plan ahead!)

---

## 11. Academic Integrity

⚠️ **Your student ID ensures unique results!**
- The instances are generated with your student ID as the random seed, so each student has DIFFERENT instances and numerical results.
- Copying results from classmates is EASILY detected.
- Focus on YOUR analysis and interpretation.

✅ **Acceptable collaboration:**
- Discussing general concepts and algorithms
- Helping each other with technical issues (installation, errors)

❌ **NOT acceptable:**
- Copying numerical results or written answers from classmates
- Sharing code modifications
- Using another student's generated files

AI tools (ChatGPT, GitHub Copilot, etc.) may be used to **understand** the algorithms, but the written analysis must reflect **your own results** from running the notebook.

**Consequences:** Violations of academic integrity result in zero marks for the assignment, a report to academic affairs, and possible disciplinary action.

---

## 12. Getting Help

**Installation problems:**
- Check your Python version (3.8+)
- Try: `pip install --upgrade gurobipy numpy pandas matplotlib seaborn scipy openpyxl`

**Gurobi license issues:**
- Academic license: https://www.gurobi.com/academia/
- Free version: set `GUROBI_VERSION = "free"` in the Student Configuration cell

**Memory errors:**
- Close other applications; use a machine with 8GB+ RAM if possible

**Long runtime:**
- 10-20 minutes is normal. If it takes much longer, check your hardware or Gurobi license.

**GitHub problems (fork, rename, push):**
- Check that you cloned **your fork**, not the original repository
- Check `git status` and that `git push origin main` finished without errors
- If you still have a problem, contact me by email.

---

# 13. YOUR ANALYSIS AND ANSWERS

Your answers go in a **separate file**, `My_Answers.md`, which is provided in the repository as a template (student information, Sections A-F, bonus sections, additional comments and declaration).

- Fill in `My_Answers.md` — do **not** write your answers in this guide.
- Keep this guide (`Assignment_2_Student_Guide.md`) unchanged.
- Fill in your student ID and name at the top of `My_Answers.md`.

---

**END OF ASSIGNMENT DOCUMENT**

*Please ensure `My_Answers.md` is completed before submission. Good luck!*
