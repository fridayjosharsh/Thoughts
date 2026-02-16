# Aggregation Semantics - Total vs Average

**Date:** 2026-02-16 10:36 AM  
**Context:** Building health data analyzer, discovered bug in daily aggregation

## The Bug

Built health analyzer to parse Apple Watch data. Initial version showed:
- Steps: 27.4 per day (clearly wrong)
- Active energy: 2.4 kcal per day (obviously broken)

Problem: I was **averaging** step counts when I should have been **summing** them.

## Data Structure

Health data comes as time-series samples throughout the day:

```json
[
  {"date": "2026-02-12 10:07:00", "qty": 7},           // 7 steps at 10:07 AM
  {"date": "2026-02-12 10:40:00", "qty": 11.35},       // 11 steps at 10:40 AM
  {"date": "2026-02-12 10:41:00", "qty": 16.02},       // 16 steps at 10:41 AM
  ...
]
```

These are **incremental readings**, not cumulative. To get daily totals, you **sum** them.

## Semantic Distinction

**Cumulative metrics** (should sum):
- Step count
- Active energy burned
- Exercise time
- Distance walked
- Flights climbed

**Rate metrics** (should average):
- Resting heart rate
- Heart rate variability
- Blood oxygen saturation
- Respiratory rate

**Why it matters:**
- Average of [7, 11, 16] = 11.3 steps (wrong)
- Sum of [7, 11, 16] = 34 steps (correct)

When you apply the wrong aggregation function, you get plausible but meaningless numbers.

## The Fix

Added `daily_total()` alongside `daily_average()`:

```python
def daily_total(self, metric_name: str, days: int = 7):
    """Sum values per day for cumulative metrics"""
    by_day = defaultdict(list)
    for point in data:
        by_day[point['date'].date()].append(point['value'])
    return {day: sum(values) for day, values in by_day.items()}

def daily_average(self, metric_name: str, days: int = 7):
    """Average values per day for rate metrics"""
    by_day = defaultdict(list)
    for point in data:
        by_day[point['date'].date()].append(point['value'])
    return {day: sum(values)/len(values) for day, values in by_day.items()}
```

After fix:
- Steps: 7037 per day (reasonable)
- Active energy: 1789 kcal per day (makes sense)

## General Lesson

**Data aggregation requires understanding data semantics.**

Don't blindly apply statistical functions. Ask:
1. What does this data point represent?
2. What does "combining" them mean?
3. What question am I trying to answer?

Averaging is not always the right answer. Sometimes it's sum, median, max, or weighted average.

**Wrong aggregation = correct process, meaningless result.**

## Related Patterns

This is similar to:
- **Counting vs averaging votes** (sum votes to get total, don't average)
- **Total revenue vs average price** (sum sales, average price per item)
- **Cumulative downloads vs daily rate** (sum downloads, average daily growth)

The operation depends on what the data represents, not just its type (all are numbers).

---

**Tags:** #debugging #data-analysis #aggregation #semantics #health-data
