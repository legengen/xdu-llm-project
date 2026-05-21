# Data Race Detection Summary (Step 2)

## Results Overview

| File | Data Race? | Reason |
|------|-----------|---------|
| DRB001.c | **YES** | Loop-carried dependency: iteration i-1 reads a[i] while iteration i writes to a[i] |
| DRB005.c | **YES** | Indirect indexing with potential duplicates causes concurrent R/W on same locations |
| DRB009.c | **YES** | Multiple threads write to shared variable x without synchronization |
| DRB013.c | **YES** | nowait clause allows single section to read a[9] while for loop may still write to it |
| DRB022.c | **YES** | Multiple threads update shared variable sum without reduction clause |
| DRB045.c | **NO** | Each thread accesses distinct array elements |
| DRB048.c | **NO** | Proper use of firstprivate, distinct array element access |
| DRB051.c | **NO** | Implicit barrier at end of parallel region provides synchronization |
| DRB059.c | **NO** | Correct use of lastprivate clause |
| DRB065.c | **NO** | Correct use of reduction clause |
| DRB073.c | **NO** | Each thread accesses distinct rows |
| DRB108.c | **NO** | Correct use of atomic directive |

## Statistics

- **Total programs analyzed:** 12
- **Programs with data races:** 5 (41.7%)
- **Programs without data races:** 7 (58.3%)

## Data Race Categories

### Programs with Data Races:

1. **Loop-carried dependencies** (DRB001.c)
   - Cross-iteration read/write conflicts

2. **Unprotected shared variable updates** (DRB009.c, DRB022.c)
   - Multiple writes or read-modify-write without synchronization

3. **Indirect indexing conflicts** (DRB005.c)
   - Array access through index array with potential duplicates

4. **Missing synchronization barriers** (DRB013.c)
   - nowait clause removes necessary synchronization

### Programs without Data Races:

1. **Independent data access** (DRB045.c, DRB073.c)
   - Each thread accesses distinct memory locations

2. **Proper OpenMP clauses** (DRB048.c, DRB059.c, DRB065.c, DRB108.c)
   - Correct use of firstprivate, lastprivate, reduction, atomic

3. **Implicit synchronization** (DRB051.c)
   - Implicit barriers provide necessary ordering

## Individual Verdict Files

Each program has a detailed verdict file:
- DRB001_race_verdict.txt
- DRB005_race_verdict.txt
- DRB009_race_verdict.txt
- DRB013_race_verdict.txt
- DRB022_race_verdict.txt
- DRB045_race_verdict.txt
- DRB048_race_verdict.txt
- DRB051_race_verdict.txt
- DRB059_race_verdict.txt
- DRB065_race_verdict.txt
- DRB073_race_verdict.txt
- DRB108_race_verdict.txt
