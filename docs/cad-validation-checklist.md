# 📐 CAD Validation Checklist — Pipeline Replacements

Reference guide for reviewing CAD files submitted by contractors
for pipeline replacement projects.

---

## Purpose

Every CAD file received must be validated before loading into GIS.
This checklist ensures that geometry, layout and nomenclature meet
the required standards for spatial data quality and GIS compatibility.

---

## 1. Geometry Validation

### 1.1 Endpoint Displacement

The most critical check. Endpoint coordinates in the CAD must match
the reference position in the GIS database within the allowed tolerance.

| Endpoint | Max allowed displacement | Action if exceeded |
|---|---|---|
| Endpoint 1 (origin) | ≤ 5 m | Return to contractor for field correction |
| Endpoint 2 (destination) | ≤ 5 m | Return to contractor for field correction |

**How to measure:**

1. Load CAD and GIS layer in the same projected coordinate system
2. Identify Endpoint 1 and Endpoint 2 in the CAD
3. Measure distance to the corresponding facility in GIS
4. Record displacement value in the validation table

> Displacements between 5m and 10m may be accepted exceptionally
> with documented justification. Above 10m: mandatory field re-survey.

### 1.2 Trace Geometry

| Check | Pass condition |
|---|---|
| No self-intersections | Polyline does not cross itself |
| No zero-length segments | Every segment has measurable length |
| Vertex interval consistency | Spacing between vertices follows defined standard |
| Single polyline per pipeline | One polyline per replacement — no splits |
| Correct direction | From Endpoint 1 (origin) to Endpoint 2 (destination) |

### 1.3 Coordinate System

- CAD must use the same projected coordinate system as the GIS database
- If different: note the source CRS and reproject before displacement measurement
- WGS84 / UTM zone must be specified in the CAD layout

---

## 2. Layout Validation

Every CAD file must include a properly formatted layout (print sheet).
Files without layout are returned to the contractor before review.

### Mandatory layout elements

| Element | Description |
|---|---|
| **Title block** | Project name, company, pipeline ID, survey date, revision |
| **Legend** | All symbols used in the drawing explained |
| **Coordinate system** | Datum and projection explicitly stated |
| **Endpoint 1 label** | Coordinate label and facility ID at origin |
| **Endpoint 2 label** | Coordinate label and facility ID at destination |
| **Scale bar** | Graphic scale on the layout |
| **North arrow** | Orientation reference |

### Partial replacements — additional requirements

For partial replacements (section of existing pipeline):
- Progressive distance label at Endpoint 1
- Progressive distance label at Endpoint 2
- Clear indication of which section is being replaced

---

## 3. Nomenclature Validation

| Element | Validation rule |
|---|---|
| Pipeline ID | Must match exact ID in GIS database |
| Endpoint 1 facility ID | Cross-check against GIS facilities layer |
| Endpoint 2 facility ID | Cross-check against GIS facilities layer |
| Replacement type label | Total / Partial — consistent with form |

**Common nomenclature errors:**

- Manifold ID in CAD does not match GIS database (updated ID vs legacy ID)
- Pipeline ID uses old format instead of current naming convention
- Facility label refers to decommissioned equipment

---

## 4. Validation Table Structure

For each batch of CAD files received, fill in this table:

| Pipeline_ID | EP1_Disp_m | EP2_Disp_m | Layout_OK | Nomenclature_OK | Form_OK | Result | Observations |
|---|---|---|---|---|---|---|---|
| Example-001 | 1.2 | 3.8 | ✅ | ✅ | ✅ | Approved | — |
| Example-002 | 2.1 | 6.5 | ❌ | ✅ | ⚠️ | Returned | EP2 exceeds 5m; layout missing legend |
| Example-003 | 0.8 | 1.1 | ✅ | ⚠️ | ✅ | Returned | Manifold ID mismatch at EP2 |

**Result codes:**

| Code | Meaning |
|---|---|
| `Approved` | Meets all requirements — ready for GIS loading |
| `Returned` | Has critical errors — must be corrected by contractor |
| `Conditional` | Minor issues — accepted with documented observation |

---

## 5. Validation Workflow

```
Receive CAD + Form
      │
      ▼
Check layout completeness
      │
   Missing? ──► Return immediately (layout is prerequisite)
      │
      ▼
Load CAD in GIS environment
      │
      ▼
Measure EP1 displacement
Measure EP2 displacement
      │
   > 5m? ──► Flag for correction
      │
      ▼
Check trace geometry
(self-intersections, zero-length, intervals)
      │
      ▼
Cross-check nomenclature
(Pipeline ID, Facility IDs)
      │
      ▼
Validate form attributes
(see cad-validation-checklist.md → Form section)
      │
      ▼
Fill validation table
      │
      ▼
Send observation report to contractor
      │
      ▼
Approved? ──► Load into GIS (ABM)
```

---

## 6. After Approval — GIS Loading Steps

1. Reproject CAD to GIS coordinate system if needed
2. Extract polyline geometry from CAD
3. Load into GIS pipeline layer (Feature Class)
4. Populate attributes from validated form
5. Set pipeline status: `Definitive`
6. Set OMEGA integration flag: `YES` (total replacements)
7. Update status of replaced pipeline to `Retired`
8. Record in GIS control table: date loaded, batch, validated by

---

## References

- Spatial data delivery standard (internal document — not included in this repo)
- GIS database attribute dictionary (internal — not included)
- ABM operations guide: [`workflow-intake-validation.md`](workflow-intake-validation.md)
