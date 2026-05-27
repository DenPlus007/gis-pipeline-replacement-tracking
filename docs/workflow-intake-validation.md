# 🔄 Workflow — Pipeline Replacement Data Intake & GIS Update

Complete methodology for receiving, validating and loading
pipeline replacement data into GIS.

---

## Overview

Pipeline replacement projects involve a continuous cycle of data delivery
from contractors to the GIS team. Each replacement generates:

- A **CAD file** (.dwg) with the surveyed pipeline trace
- A **field survey form** (.xlsx) with the pipeline attributes

Both must be validated before any GIS update is performed.

---

## Stage 1 — Data Intake Protocol

### Delivery requirements (per pipeline)

Each delivery must consist of:

```
One CAD file (.dwg)     ← pipeline trace surveyed by GPS in field
One form (.xlsx)        ← attribute data for that pipeline
```

> **Important:** Batch deliveries (multiple pipelines in one file) are not accepted.
> Each pipeline must have its own individual CAD and form.
> Batch deliveries are returned to the contractor before review.

### Intake checklist

- [ ] One CAD per pipeline
- [ ] One form per pipeline
- [ ] CAD filename matches pipeline ID
- [ ] Form filename matches pipeline ID
- [ ] Files received via agreed channel (email / shared folder)
- [ ] Delivery logged in intake control table (date, contractor, batch number)

---

## Stage 2 — CAD Spatial Validation

See full detail in [`cad-validation-checklist.md`](cad-validation-checklist.md).

**Summary of checks:**
- Endpoint 1 displacement ≤ 5 m
- Endpoint 2 displacement ≤ 5 m
- Trace geometry integrity (no self-intersections, no zero-length segments)
- Vertex interval consistency
- Layout elements present and complete
- Nomenclature matches GIS database

---

## Stage 3 — Form Attribute Validation

### Key attribute rules

**OMEGA Integration field**

```
Total replacement  →  OMEGA = YES   (pipeline is treated as new)
Partial replacement →  OMEGA = NO    (existing pipeline, section modified)
```

This is the most frequent error. Contractors often set `OMEGA = NO`
for total replacements. Always verify replacement type before accepting.

**Data Origin field**

```
GPS survey performed in field  →  Data_Origin = Survey
No field survey (schematic)    →  Data_Origin = Schematic
```

If a `Survey_Date` is present, `Data_Origin` must be `Survey`.

**Pipeline Status field**

```
CAD received from field survey  →  Status = Definitive
CAD not yet received            →  Status = Active without CAO
```

**Survey Date field**

- Mandatory when `Data_Origin = Survey`
- Must contain the actual GPS survey date
- `No data` is not acceptable for surveyed replacements

### Form validation table structure

| Field | Expected | Received | Result |
|---|---|---|---|
| `OMEGA` | YES (total replacement) | NO | ❌ Error |
| `Survey_Date` | 2024-04-16 | No data | ❌ Error |
| `Data_Origin` | Survey | Schematic | ❌ Error |
| `Endpoint_2_ID` | MA4CG9 | MA2CD10 | ❌ Error |
| `Pipeline_Status` | Definitive | Active without CAO | ❌ Error |

---

## Stage 4 — Observation Report

When errors are found, an observation report is sent to the contractor before returning the files.

### Report structure

For each pipeline with errors, document:

```
Pipeline_ID   : [ID]
Delivery_Date : [date]
Source        : [CAD / Form / Both]
Inconsistency : [field or geometry element]
Observation   : [what was found]
Expected      : [what it should be]
Action        : [what the contractor must correct]
```

**Priority levels:**

| Priority | Condition | Action |
|---|---|---|
| 🔴 Critical | Endpoint displacement > 5m | Mandatory field re-survey |
| 🔴 Critical | Missing layout elements | Return before any review |
| 🟡 Medium | Wrong attribute value | Correction in form, no field work needed |
| 🟢 Low | Naming inconsistency | Clarification and correction |

---

## Stage 5 — GIS ABM Update

Once a pipeline is approved, the GIS update follows the ABM cycle:

### Alta — New pipeline loaded

1. Reproject CAD to GIS coordinate system
2. Extract polyline from CAD
3. Load into GIS pipeline feature class
4. Populate all attributes from validated form
5. Assign `Status = Definitive`
6. Assign `OMEGA = YES` (if total replacement)
7. Record in GIS control table

### Baja — Replaced pipeline retired

1. Select the original pipeline being replaced in GIS
2. Update `Status = Retired`
3. Record `Retirement_Date` and `Replaced_by` fields
4. Do not delete — maintain history in GIS

### Modificación — Attribute correction

1. Identify record in GIS by Pipeline_ID
2. Update only the fields that require correction
3. Document what was changed, by whom, and when
4. Log in the modification history table

---

## Stage 6 — Plan vs Execution Comparison

After each loading batch, update the comparison between
the replacement master plan and the GIS control table.

**Key metrics tracked:**

| Metric | Description |
|---|---|
| Total planned replacements | Count from master plan |
| Delivered to GIS | Count loaded and approved |
| Pending delivery | Not yet received from contractor |
| In review | Received but with open observations |
| Rejected | Returned — awaiting correction |
| Delivery rate (%) | Delivered / Total planned × 100 |

See [`scripts/compare_replacement_plan.py`](../scripts/compare_replacement_plan.py)
for automated comparison.

---

## Stage 7 — Project Follow-up Report

Periodic report sent to the project team:

- Total replacements per period
- Delivery rate by contractor / batch
- Open observations count
- Pipelines pending field re-survey
- GIS loading progress vs plan

---

## Common Issues Reference

| Issue | Root cause | Solution |
|---|---|---|
| Endpoint displacement > 5m | GPS precision in field | Re-survey endpoint in field |
| Wrong manifold ID | Outdated nomenclature used by contractor | Provide updated ID list |
| OMEGA = NO on total replacement | Misunderstanding of the rule | Capacity building + checklist |
| Missing survey date | Form not completed at field time | Enforce form completion protocol |
| Batch delivery (multiple pipelines per file) | Process not followed | Return batch, request individual files |
| Layout without legend or title block | CAD template not used | Provide mandatory CAD template |
