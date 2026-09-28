# Newsfeed A/B Test

Did a public Newsfeed make students start their labs earlier? An observational A/B test analysis on real activity logs from an educational platform, using SQL, pandas and matplotlib.

## Question

The platform launched a "Newsfeed" page where lab submission logs are visible to all students. The hypothesis: seeing peers' activity creates peer pressure and pushes students to start labs earlier, which leaves more time for iterations and experiments.

**Metric:** the gap (in hours) between a student's first commit on a lab and that lab's deadline. More negative means an earlier start.

## Data

SQLite database with real logs from the platform:

| Table | Content |
|---|---|
| `checker` | Lab submission logs: user, lab, attempt number, timestamp, status |
| `pageviews` | Newsfeed visits: user, timestamp |
| `deadlines` | Deadline of each lab |

## Approach

1. **Datamart.** One row per user and lab: first commit time and first Newsfeed view time. Filters: student accounts only (no admins), finished checks only (`status = 'ready'`), first attempts only (`numTrials = 1`), labs active during the experiment.
2. **Test / control split.** Users who visited the Newsfeed form the test group, users who never did form the control group. Control users have no real view time, so the test group's mean view time is used as their reference point.
3. **A/B comparison.** Average gap before vs. after the (real or reference) first view, per group. Only users with observations both before and after are included; `project1` is excluded as an outlier (much longer deadline).
4. **Visual checks.** Boxplots to check what averages hide (variance, outliers), and a scatter matrix to check for non-linear patterns behind a weak correlation coefficient.

## Key Results

| Group | Before first view (h) | After first view (h) | Shift |
|---|---|---|---|
| Test | -66.7 | -100.2 | ~33.5 h earlier |
| Control | -98.5 | -99.8 | ~1.3 h earlier |

- Test users shifted much more than control users, which is the direction the hypothesis predicts.
- The boxplots show that control's variance shrank substantially even though its mean barely moved, so "control didn't change" is not fully true.
- Pageviews and the commit-deadline gap have a weak correlation (-0.28), and the scatter matrix shows no hidden non-linear pattern. If the Newsfeed were driving the effect, heavier users would be expected to show a stronger shift, and they don't.

Pictures TBA 

**Conclusion:** the evidence is mixed. There is a directional signal consistent with the hypothesis, but not strong or clean evidence for it.

## Limitations

- **No random assignment.** Users self-selected into test/control by visiting the Newsfeed, so the groups may differ in ways unrelated to the Newsfeed.
- **Different baselines.** Test and control already differed noticeably before any Newsfeed exposure.
- **Artificial split point for control.** The reference timestamp is an approximation, not a real behavioral event.
- **No formal significance testing.** Comparisons are based on averages and distributions only.
- **Small sample.** The scatter matrix is built on a small number of test users.

## Repository Structure

```
newsfeed-ab-test/
├── README.md
├── requirements.txt
├── data/
│   └── checking-logs.sqlite
├── notebooks/
│   └── newsfeed_behavior_analysis.ipynb
└── figures/
    ├── boxplot.png
    └── scatter_matrix.png
```

## How to Run

```bash
git clone https://github.com/<username>/newsfeed-ab-test.git
cd newsfeed-ab-test
pip install -r requirements.txt
jupyter notebook notebooks/newsfeed_behavior_analysis.ipynb
```

The notebook expects the database at `data/checking-logs.sqlite`.

## Tech Stack

Python, SQL (SQLite), pandas, NumPy, matplotlib
