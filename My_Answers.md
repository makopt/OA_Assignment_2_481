# My Answers - Assignment 2 (CFLP: Tabu Search vs Genetic Algorithm vs Gurobi)

**Student Information:**
- **Student ID:** ___________________________
- **Student Name:** ___________________________
- **Gurobi Version Used:** [ ] Academic [ ] Free
- **Date Completed:** ___________________________

> Use specific numbers from **YOUR** results (`results_student_<ID>.csv/.xlsx`) and reference the plots by name (e.g., "As shown in `gap_by_size.png`..."). Explain **why**, not only **what**.

---

## Section A: Solution Quality Analysis (4 marks)

### A1 (1 mark) - Overall Mean Optimality Gap

| | Tabu Search | Genetic Algorithm |
|---|---|---|
| Number of instances used (Gurobi optimal) | | |
| Mean gap (%) | | |

**Method with the lower overall gap:** [ ] TS [ ] GA

**Comment:**

[Report and interpret the overall gaps]

---

### A2 (1 mark) - Gap by Instance Size

| | Small | Medium | Large |
|---|---|---|---|
| TS mean gap (%) | | | |
| GA mean gap (%) | | | |

**Does the gap increase with size?** TS: [ ] Yes [ ] No  GA: [ ] Yes [ ] No

**Comparison of the trends (TS vs GA):**

[Describe the trend and compare the two methods]

---

### A3 (1 mark) - Gap by Difficulty Level

| | Easy | Moderate | Expensive |
|---|---|---|---|
| TS mean gap (%) | | | |
| GA mean gap (%) | | | |

- Hardest difficulty for TS: ___________
- Hardest difficulty for GA: ___________

**Explanation:**

[Explain WHY these difficulty levels are harder]

---

### A4 (1 mark) - TS vs GA Head-to-Head (`ts_vs_ga_scatter.png`)

- Fraction of instances where TS has a lower gap than GA: ___________ ( _____% )
- Fraction of instances where GA has a lower gap than TS: ___________ ( _____% )

**Instance types where one method consistently dominates:**

[Describe, with evidence from the plot / results]

---

## Section B: Computational Efficiency Analysis (4 marks)

### B1 (1 mark) - Computation Time and TS Speedup

| Mean time (s) | Small | Medium | Large |
|---|---|---|---|
| Gurobi | | | |
| Tabu Search | | | |
| Genetic Algorithm | | | |

**TS speedup over Gurobi on large instances:** ______x

**Comment:**

[Interpret the times]

---

### B2 (1 mark) - GA Speedup vs TS Speedup

- Mean GA speedup over Gurobi (all sizes): ______x
- Mean TS speedup over Gurobi (all sizes): ______x
- Faster metaheuristic and by how much: ___________

**Comment:**

[Compare the two metaheuristics]

---

### B3 (1 mark) - Growth of Gurobi Time vs Metaheuristic Time

| | Small -> Medium (x) | Medium -> Large (x) |
|---|---|---|
| Gurobi time growth | | |
| TS time growth | | |
| GA time growth | | |

**Comparison:**

[Compare the growth of the exact solver with the growth of the metaheuristics]

---

### B4 (1 mark) - Quality-Speed Trade-off

**Method with the best balance of low gap and low time:** [ ] TS [ ] GA [ ] Gurobi

**Justification (with numbers):**

[Reference specific gap and time values]

---

## Section C: Scalability Analysis (3 marks)

### C1 (1 mark) - Gap Heatmap (`gap_heatmap.png`)

- Highest TS gap: (size, difficulty) = ___________ with ______%
- Highest GA gap: (size, difficulty) = ___________ with ______%

**Interpretation:**

[What does this mean?]

---

### C2 (1 mark) - Expected Scalability to Very Large Instances

**Method expected to scale better:** [ ] TS [ ] GA

**Justification (algorithmic complexity and your observed results):**

[Your answer]

---

### C3 (1 mark) - Validity of Gap Measurements When Gurobi Hits the Time Limit

- Instances where Gurobi did not prove optimality (from `gurobi_status`): ______ out of ______

**Effect on the validity of the gaps:**

[Your answer]

**Better benchmark:**

[Your suggestion]

---

## Section D: Quality-Time Tradeoff Analysis (3 marks)

### D1 (1 mark) - 10-Second Time Budget (Medium Instances)

**Recommended method:** [ ] TS [ ] GA

- Mean TS time / gap on medium instances: ________ s / ________%
- Mean GA time / gap on medium instances: ________ s / ________%

**Justification:**

[Cite the numbers above]

---

### D2 (1 mark) - Quality Is the Top Priority

| Size | Recommended approach (TS / GA / Gurobi) | Justification (numbers) |
|---|---|---|
| Small | | |
| Medium | | |
| Large | | |

---

### D3 (1 mark) - Convergence Timing (3-5 convergence plots, pushed in `sample_convergence_plots/`)

**Selected plots:** ___________, ___________, ___________

| Instance | TS: iteration of best solution / total iterations | GA: generation of best solution / total generations |
|---|---|---|
| | | |
| | | |
| | | |

**What this implies for early stopping:**

[Your answer]

---

## Section E: Algorithm Understanding (3 marks)

### E1 (1 mark) - Tabu Search: Tabu List and Aspiration Criterion

**How the tabu list prevents cycling:**

[Explain in your own words]

**Role of the aspiration criterion:**

[Explain]

**Concrete CFLP example** (a specific tabu move, and when aspiration would override it):

[Example]

---

### E2 (1 mark) - Genetic Algorithm: Repair Operator and Crossover

**How the repair operator ensures feasibility, and what would happen without it:**

[Explain]

**How uniform crossover combines two parent CFLP solutions:**

[Explain, preferably with a small example]

---

### E3 (1 mark) - Trajectory-Based vs Population-Based Exploration

**Meaning for the exploration of the solution space:**

[Explain]

| | Advantages for the CFLP | Disadvantages for the CFLP |
|---|---|---|
| Tabu Search (trajectory) | | |
| Genetic Algorithm (population) | | |

---

## Section F: Critical Reflection (3 marks)

### F1 (1 mark) - Most Important TS Hyperparameters

1. [Hyperparameter 1 - role - effect of its default value]

2. [Hyperparameter 2 - role - effect of its default value]

3. [Hyperparameter 3 - role - effect of its default value]

**What I would change and why:**

[Your answer]

---

### F2 (1 mark) - Most Important GA Hyperparameters

1. [Hyperparameter 1 - role]

2. [Hyperparameter 2 - role]

3. [Hyperparameter 3 - role]

**Effect of increasing `population_size` / `max_generations` on quality and runtime:**

[Your answer]

---

### F3 (1 mark) - Proposed Algorithmic Improvements

**Improvement for Tabu Search:**

[Description + justification (theoretical properties and observed results)]

**Improvement for the Genetic Algorithm:**

[Description + justification (theoretical properties and observed results)]

---

## BONUS A: Tabu Search Improvement (+5%, optional)

*Leave this section empty if you do not attempt the bonus. See Section 8 of the guide. Your improved notebook (`StudentID_FirstName_LastName_Assignment2_IMPROVED.ipynb`) must be pushed to your repository.*

**(a) Screening results (small/moderate subset):**

| Enhancement tested | Mean gap before (%) | Mean gap after (%) | Improvement |
|---|---|---|---|
| Adaptive tenure | | | |
| Aspiration+ criterion | | | |
| Frequency-based memory | | | |
| Diversification restart | | | |

**(b) Selected enhancement and justification:**

[Name the enhancement you chose and explain why it gave the best improvement]

**(c) Full 144-instance results:**

| Metric | Original TS | Improved TS |
|---|---|---|
| Mean gap (%) | | |
| Mean runtime (s) | | |

**Improved results files (names in your repository):** ___________

---

## BONUS B: Genetic Algorithm Improvement (+5%, optional)

*Leave this section empty if you do not attempt the bonus.*

**(a) Screening results (small/moderate subset):**

| Enhancement tested | Mean gap before (%) | Mean gap after (%) | Improvement |
|---|---|---|---|
| Adaptive mutation | | | |
| Two-point crossover | | | |
| Memetic GA (TS + GA) | | | |
| Diversity preservation | | | |

**(b) Selected enhancement and justification:**

[Name the enhancement you chose and explain why it gave the best improvement]

**(c) Full 144-instance results:**

| Metric | Original GA | Improved GA |
|---|---|---|
| Mean gap (%) | | |
| Mean runtime (s) | | |

**Improved results files (names in your repository):** ___________

---

## Additional Comments (Optional)

[Any additional observations, challenges faced, or insights gained]

---

## Declaration

I hereby declare that:
- I have completed this assignment independently
- All numerical results are from MY experiments using MY student ID
- All analysis and interpretations are MY own work
- I have not copied results or answers from other students
- I understand the consequences of academic dishonesty

**Signature:** _______________________

**Date:** ___________________________

---

**END OF MY ANSWERS**
