# SC404 – Big Data Driver Power Integrity Signoff in ICs
## Lab Report — Delhi Technological University, Department of Applied Physics

**Student Name:** Manan Agarwal
**Roll Number:** 23/EE/323
**Course Code:** SC404
**Course Title:** Big Data Driver Power Integrity Signoff in ICs
**Institute:** Delhi Technological University
**Tool Used:** Ansys RedHawk-SC (RHSC) 2025 R2.3
**Design:** galaxy\_cmplx (ASAP7 6-track)
**Date:** April 2026

---

## Table of Contents

1. [Run the RedHawk-SC Lab](#task-1-run-the-redhawk-sc-lab)
2. [Design Details and Layout](#task-2-design-details-and-layout)
3. [Design Input Coverage](#task-3-design-input-coverage)
4. [Grid Robustness Report](#task-4-grid-robustness-report)
5. [Heatmaps – BQM and SPR](#task-5-heatmaps--bqm-and-spr)
6. [Power Analysis Results](#task-6-power-analysis-results)
7. [Static Analysis Results](#task-7-static-analysis-results)
8. [EM (Electromigration) Analysis](#task-8-em-electromigration-analysis)
9. [RHSC Explorer – One Click for All Results](#task-9-rhsc-explorer--one-click-for-all-results)

---

## Task 1: Run the RedHawk-SC Lab

### 1.1 Identification Setup

To maintain traceability across all output artefacts, a dedicated run directory named `reports_23EE323_Manan` was created within the QuickStart Training workspace. The identification string `23EE323_Manan` was systematically appended to all RHSC analysis views and output log files, ensuring that every generated report can be unambiguously attributed to this submission.

### 1.2 (a) RHSC Command Used

The lab was initiated using the following command at the Unix prompt, which invokes the RedHawk-SC Python script for the `galaxy_cmplx` design and simultaneously captures a full transcript via `tee`:

```bash
redhawk_sc galaxy_cmplx.py | tee run_23_EE_323_Manan.log
```

[**INSERT FIGURE 1.1 HERE** — RHSC command at Unix prompt showing the run initiated with 16 workers on localhost]

Key environmental parameters captured from the run log are summarised below:

| Parameter | Value |
|---|---|
| Tool Version | RHSC 2025 R2.3, built on Nov 12 2025 |
| SeaScape Path | `/opt/seascape_release/2025_R2.3/linux_x86_64_rhel8/bin` |
| Work Directory | `.../Galaxy2/QuickStart_Training/gp.1` |
| Process ID (PID) | 15569 |
| Run Start Time | 2026-Apr-20 21:55:33 |
| Host | `vmdccc8ff768bce` |
| Parallel Workers | 16 workers on localhost (for the first view, `lv_tt#v1`) |

The use of 16 parallel workers on localhost reflects RHSC's cloud-scale scheduler architecture, which partitions analysis tasks across available CPU cores to accelerate the multi-view simulation pipeline.

---

### 1.3 (b) Total Runtime and Stage-by-Stage Timing

The complete wall-clock runtime for the analysis was **00:28:46.35 (28 minutes and 46 seconds)**. The table below provides a comprehensive breakdown of each pipeline stage, its corresponding view, and its start and completion timestamps.

[**INSERT FIGURE 1.3 HERE** — Stage-by-stage timing table from run.log]

| Stage | View | Start | Completed |
|---|---|---|---|
| LibertyView | `lv_tt#v1` | 00:00:14.10 | 00:00:24.81 |
| Input Design | `dv0#v1` | 00:00:19.23 | 00:01:13.49 |
| Modified Design | `dv#v1` | 00:01:13.50 | 00:01:15.55 |
| Extract From File | `evx#v1` | 00:01:14.62 | 00:01:33.39 |
| TimingView | `tv#v1` | 00:01:14.34 | 00:04:05.21 |
| Scenario | `scn_bqm` | 00:04:51.39 | 00:05:16.13 |
| Extract | `ev#v1` | 00:01:14.74 | 00:06:17.46 |
| Preprocess | `av_bqm` | 00:07:18.01 | 00:07:41.65 |
| Prepare DC | `av_bqm` | 00:07:41.65 | 00:08:00.52 |
| Simulate DC | `av_bqm` | 00:08:00.52 | 00:08:06.25 |
| Postprocess | `av_bqm` | 00:08:06.26 | 00:08:49.28 |
| Event Generation | `scn_curr` | 00:09:53.33 | 00:10:56.16 |
| Scenario | `scn_curr` | 00:09:53.23 | 00:17:48.01 |
| Switching Activity | `swa` | 00:22:26.93 | 00:22:46.52 |
| Power Calc | `pwr` | 00:22:26.99 | 00:22:52.08 |
| Scenario | `scn_static` | 00:22:28.42 | 00:23:03.91 |
| Preprocess | `av_static` | 00:23:07.83 | 00:23:37.21 |
| Prepare DC | `av_static` | 00:23:37.21 | 00:23:55.47 |
| Simulate DC | `av_static` | 00:23:55.47 | 00:24:01.46 |
| Postprocess | `av_static` | 00:24:01.46 | 00:24:47.70 |
| EM Check | `pem_dc` | 00:25:39.92 | 00:27:43.66 |
| **Run (Total)** | — | 00:00:00.00 | **00:28:46.35** |

**Maximum time view:** The `scn_curr` stage (Scenario/Event Generation for the transient current waveform) was the most computationally intensive, running from timestamp 00:09:53 to 00:17:48 — a duration of approximately **7 minutes and 55 seconds**. This is expected behaviour: generating accurate gate-level current waveforms requires RHSC to resolve millions of individual switching events across the design, convolving cell-level current templates with the VCD-derived activity data at each simulation time step.

**Minimum time view:** The `lv_tt#v1` (LibertyView) stage completed in approximately **10 seconds** (00:00:14 to 00:00:24). This stage simply parses and indexes the Liberty timing library, a task that is I/O-bound rather than compute-bound and therefore requires negligible processing time.

---

### 1.4 (c) Memory Statistics

Memory consumption was tracked at 5% completion intervals throughout the run. The key metrics are tabulated below.

[**INSERT FIGURE 1.4 HERE** — Scheduler memory progress table showing Total Memory, Peak Worker Memory, and Avg Worker Memory at 5% intervals]

| Metric | Value |
|---|---|
| Starting Total Memory (at 5% completion) | 19.577 GB |
| Peak Total Memory (at 100% completion) | 54.191 GB |
| Peak Worker Memory (at 100% completion) | 5.246 GB |
| Average Worker Memory (typical mid-run range) | ~2.0 – 2.5 GB |

Memory usage grew monotonically from 19.6 GB at run inception to 54.2 GB at completion, reflecting the incremental loading of analysis views, netlists, and simulation databases into memory. The asterisks (`*`) recorded at 30% and 85% completion indicate temporary local memory spikes caused by large intermediate data structures — such as the full event queue during transient simulation — which are subsequently released as the pipeline advances.

---

### 1.5 (d) Errors and Warnings

The run completed successfully with **no ERROR-level messages**. Two categories of WARNING messages were observed:

- **WARNING\<PYG.120\>** (fired 5 times, subsequently suppressed): Views `ev`, `sv`, `dv0`, `pkg_sv`, and `tv` were found to be present but incomplete in the existing database directory. RHSC automatically regenerated these views from scratch. This warning is characteristic of a QuickStart lab environment where a partial database from a prior execution already exists, and it carries no analytical consequence for the results.

- **WARNING\<PYG.119\>**: The existing Liberty view `lv_tt` already resided in the database and was automatically renamed to `lv_tt#v1` to prevent a naming conflict. This is an automatic, benign housekeeping action performed by the scheduler.

The absence of any ERROR-level messages confirms that all analysis views — LibertyView, DesignView, TimingView, extraction, BQM simulation, current simulation, static analysis, and EM check — completed without fatal issues and that the resulting signoff data is valid.

---

### 1.6 (e) Starting RHSC in GUI Mode with Scheduler (Unix Prompt)

To launch the RHSC graphical interface with the integrated scheduler from the Unix shell, the following command is used:

```bash
redhawk_sc -gui -gp /home/student/labs/avl_dtu_lab_training_20260214/Galaxy2/QuickStart_Training/gp.1
```

A full list of available command-line options can be retrieved by executing `redhawk_sc -help` at the Unix prompt.

---

### 1.7 (f) Starting RHSC GUI with Scheduler from the RHSC Console

When already operating within the RHSC Python console (`>>>`), the GUI and scheduler can be launched programmatically:

```python
>>> app.start_gui(with_scheduler=True)
```

To load an existing project (`gp`) directory directly into the GUI:

```python
>>> rhsc.open_design(gp_dir="<path_to_gp>", gui=True, scheduler=True)
```

---

## Task 2: Design Details and Layout

### 2.1 Design Overview

The design under analysis is **galaxy\_cmplx**, a complex system-on-chip test vehicle implemented on the **ASAP7 6-track** standard cell library — an open-source 7 nm predictive process design kit widely used in academic power integrity research. The RHSC Explorer Summary page aggregates all critical design parameters into a single dashboard view, providing an at-a-glance baseline before any detailed analysis is performed.

[**INSERT FIGURE 2.1 HERE** — RHSC Explorer Design Statistics panel showing galaxy\_cmplx key parameters and top-level floorplan view]

The complete set of design parameters is captured in the following table:

| Parameter | Value |
|---|---|
| Design Name | galaxy\_cmplx |
| Design Size | 793.588 µm × 300.500 µm |
| Total Instance Count (dv0#v1) | 1,057,974 |
| Number of Power Nets (dv0#v1) | 7 |
| Number of Ground Nets (dv0#v1) | 1 |
| Total Power (pwr) | 156.083 mWatts |
| Dominant Frequency | Not Available |
| Temperature | 115.0 °C (pem\_dc); 110.0 °C (ev\_bqm, ev#v1) |
| Minimum Supply Voltage | 0.7 V at execution\_unit\_0/register\_file\_0/VDD\_reg\_file (scn\_bqm) |
| Power Net / Voltage | VDD\_cmplx / 0.70 V |
| Ground Net / Voltage | VSS\_cmplx / 0.00 V |
| Technology | ASAP7 6-track |
| Run ID | reports\_23EE323\_Manan |

The design occupies a rectangular die of approximately 794 µm × 300 µm, containing over one million instances. The seven distinct power nets reflect a multi-voltage domain architecture, where different functional blocks operate at potentially different supply levels — a common strategy in modern SoCs for power optimisation.

---

### 2.2 Layout Display – Visibility Controls

The RHSC GUI layout viewer provides granular, per-category visibility control over every layer in the physical design. Individual categories — **All Metals**, **All Vias**, **Leaf Instances**, **Hier Instances**, and **Macro Instances** — as well as specific routing layers (M1–M7) and via layers (V1–V6) can each be toggled independently. The following figures demonstrate the diagnostic value of selectively enabling and disabling these layer groups.

[**INSERT FIGURE 2.2 HERE** — Layout with all layers enabled (metals, vias, leaf cells, hier cells, macro instances). Full design showing 12 tiles in a 3×4 grid of regfile\_4KB and SRAM\_16\_2048 blocks]

With all layers rendered simultaneously, the full tile structure of the design is visible: twelve tiles arranged in a 3×4 grid, comprising `regfile_4KB` and `SRAM_16_2048` macro blocks interspersed with standard-cell logic.

[**INSERT FIGURE 2.3 HERE** — Only Leaf Instances enabled; metals, vias, hier, and macro all hidden. SRAM\_16\_2048\_APACHECELL macro outlines are visible; all metal routing is stripped away]

Disabling metals and retaining only leaf instances reveals the bare outlines of the SRAM macros (`SRAM_16_2048_APACHECELL`), clearly delineating the physical footprint of the memory blocks without the visual noise of signal and power routing.

[**INSERT FIGURE 2.4 HERE** — Only Hier Instances and Macro Instances enabled. Pink/magenta outlines show sc\_w1Rf\_w2Sr, regfile\_4KB, SRAM\_16\_2048 hierarchical boundaries]

Enabling only hierarchical and macro instance boundaries exposes the block-level floorplan in its cleanest form. The pink/magenta outlines of `sc_w1Rf_w2Sr`, `regfile_4KB`, and `SRAM_16_2048` provide an unobstructed view of the physical partitioning — invaluable for correlating IR-drop hotspots with functional blocks during signoff review.

[**INSERT FIGURE 2.5 HERE** — All metals disabled; vias and instances still visible. Underlying via and instance layers exposed; tile structure identifiable from hier outlines]

[**INSERT FIGURE 2.6 HERE** — Both metals and vias disabled; only instance boundaries remain. Layout is sparse and macro names are easily readable]

With both metals and vias disabled, the layout reduces to a minimal representation of instance boundaries alone. This view is particularly useful for rapidly locating a specific instance identified in an IR-drop or EM report without the visual interference of the routing stack.

---

### 2.3 Changing Display Colours

Layer display colours in the RHSC GUI are fully customisable. The procedure is as follows: navigate to the **Layers** panel on the right-hand side of the GUI, click the **colour swatch** adjacent to the desired layer entry, and select a replacement colour from the **Select Color** dialog. The enable/disable checkbox for each entry controls visibility independently of the assigned colour, allowing the engineer to create a personalised colour scheme that maximises contrast for their specific analysis workflow.

[**INSERT FIGURE 2.7(a) HERE** — Hier instances displayed in default blue colour]

[**INSERT FIGURE 2.7(b) HERE** — Hier instances displayed in green (#2ae600) after colour change]

[**INSERT FIGURE 2.8 HERE** — Select Color dialog showing the full HSV colour picker with stipple pattern options. Chosen colour: #2ae600 (Hue 109, Sat 255, Val 230) applied to the Hier Instances layer]

The colour selector provides a full HSV (Hue-Saturation-Value) picker along with stipple pattern options, and applies universally to any layer in the stack, including individual metal layers (M1, M2, ...), via layers, leaf cells, and macro instances.

---

### 2.4 Additional Explorer Views – Power and Static Voltage Drop

Beyond the floorplan visualisation, the RHSC Explorer provides integrated summary panels for Power and Voltage Drop analysis. These are previewed here to establish context for the detailed per-task analysis in Tasks 6 and 7.

[**INSERT FIGURE 2.9 HERE** — RHSC Explorer Power Summary: total chip power 156.083 mW broken down by component (Internal 78.18 mW / 50.09%, Switching 75.16 mW / 48.16%, Leakage 2.741 mW / 1.76%) and by domain. VDD\_Cmplx dominates at 154.4 mW (98.95%)]

[**INSERT FIGURE 2.10 HERE** — RHSC Explorer Static Voltage Drop Summary (av\_static): Wire(Power) 3.333 mV, Wire(Ground) 2.441 mV, Package(Power) 7.153×10⁻⁶ V, Package(Ground) 6.63×10⁻⁶ V. Instance Voltage Histogram also shown]

[**INSERT FIGURE 2.11 HERE** — RHSC Explorer Dynamic Voltage Drop Summary (av\_bqm): worst drop Wire(Power) 66.69 mV, Wire(Ground) 36.21 mV. Error indicating instance voltages not saved; current waveforms plotted for all power domains]

The dynamic worst-case wire drop of **66.69 mV** on the power net is substantially larger than the static worst-case of **3.333 mV**, which is expected — the static analysis reflects the average DC current consumption, whereas the dynamic simulation captures the peak simultaneous switching of large flip-flop populations, which creates short-duration current spikes that cause significantly larger momentary voltage sag on the power delivery network.

---

## Task 3: Design Input Coverage

### 3.1 Working Directory and Report Files

Prior to any power or EM analysis, RHSC performs a comprehensive Design Input (DI) coverage check to validate the completeness and consistency of all input collateral — Liberty models, LEF abstracts, and SPEF parasitics. Incomplete or inconsistent inputs are among the most common sources of inaccurate power integrity results in production signoff flows.

[**INSERT FIGURE 3.1 HERE** — `pwd` and `ll` output in `reports_23EE323_Manan/data_integrity` showing all DI report files present with sizes and timestamps]

All DI report files were generated successfully in the path:

```
/home/student/labs/avl_dtu_lab_training_20260214/Galaxy2/QuickStart_Training/reports_23EE323_Manan/data_integrity/
```

Notable files and their sizes include: `dv#v1_lef_check.rpt`, `dv#v1_lib_check.rpt`, `spef_annotation_summary.rpt`, `inst_no_slew.rpt` (17,278 B), `inst_no_tw.rpt` (22,658 B), and `tv_di_summary.rpt`.

---

### 3.2 Coverage Table

[**INSERT FIGURE 3.2 HERE** — DI check summary table for lef\_check, lib\_check, lib\_nldm, and lib\_nlpm]

| Check | Category | Covered | Total | Coverage % |
|---|---|---|---|---|
| lef\_check | All (MBFF + Seq + PG + Comb + Decap + Filler) | 303 | 303 | **100.0000%** |
| lib\_check | All | 300 | 300 | **100.0000%** |
| lib\_nldm | Combinational | 282 | 284 | 99.2958% |
| lib\_nldm | **All** | **296** | **298** | **99.3289%** |
| lib\_nlpm | Combinational | 282 | 284 | 99.2958% |
| lib\_nlpm | **All** | **296** | **298** | **99.3289%** |

[**INSERT FIGURE 3.3(a) HERE** — LEF check report confirming 303/303 (100%) coverage with zero uncovered cells]

[**INSERT FIGURE 3.3(b) HERE** — lib\_nldm report showing 296/298 (99.33%) coverage; 2 uncovered cells are TIEHIxp5\_ASAP7\_6t\_L and TIELOxp5\_ASAP7\_6t\_L]

---

### 3.3 SPEF Annotation Coverage

[**INSERT FIGURE 3.4 HERE** — SPEF annotation summary: 933,019/933,019 nets annotated (100%)]

SPEF annotation achieved perfect 100% coverage across all 933,019 connected nets. The per-category breakdown is as follows:

- **Input ports:** 94/94 (100%)
- **Flip-flops:** 169,864/169,864 (100%)
- **Latches:** 272/272 (100%)
- **Combinational instances:** 762,883/762,883 (100%)
- **Memory, macro, pad, sequential clock, combinational clock, unknown:** 0/0 — no such instances exist in this design
- **Total: 933,019/933,019 (100%)**

---

### 3.4 Observations and Assessment

The two uncovered `lib_nldm`/`lib_nlpm` cells are **TIEHIxp5\_ASAP7\_6t\_L** and **TIELOxp5\_ASAP7\_6t\_L** — the standard tie-high and tie-low cells used to drive constant logic values. These cells contain no combinational timing arcs and therefore no characterised NLDM/NLPM switching power data exists for them by design. Their absence from the coverage table is expected and does not represent a deficiency in the input data.

The 100% SPEF annotation is a strong indicator of a high-quality extraction flow and means that all RC parasitics are fully accounted for in the power and IR-drop simulations. However, the non-zero file sizes of `inst_no_slew.rpt` (17,278 B) and `inst_no_tw.rpt` (22,658 B) indicate that a population of instances lacks characterised input slew rates and timing window data, respectively. These missing values force RHSC to rely on default or interpolated estimates, which can reduce the accuracy of dynamic current waveform generation and EM current density calculations. These files warrant careful review before any final production signoff tape-out.

---

## Task 4: Grid Robustness Report

### 4.1 Working Directory and Report Summary

The grid robustness analysis evaluates the structural integrity of the power delivery network at the physical level — identifying disconnected power pins, isolated nodes, and shorted power/ground connections. These defects, if left unresolved, render any IR-drop or EM analysis unreliable, as the simulator may be working with an inaccurate representation of the PDN topology.

The working directory is `reports_23EE323_Manan/grid_robustness_checks`.

[**INSERT FIGURE 4.1 HERE** — Grid robustness report directory showing `wc -l` counts, SPR values, and `grep` output for worst/best instances]

| Report File | Line Count |
|---|---|
| `disconnected_instance_pins.rpt` | 62 |
| `disconnected_nodes.rpt` | 1,421 |
| `shorted_nodes.rpt` | 7 |
| **Total** | **1,490** |

The 1,421 entries in `disconnected_nodes.rpt` and 62 in `disconnected_instance_pins.rpt` indicate a non-trivial number of floating nodes and unconnected power pins in the PDN. While some of these may be expected artefacts of the extraction flow (e.g., isolated fill cells with no active connections), the 7 shorted nodes represent genuine short circuits between power and ground rails that will cause anomalous current paths and corrupt voltage distribution analysis if not resolved.

---

### 4.2 Supply Path Resistance (SPR) Analysis

Supply Path Resistance (SPR) quantifies the total resistance between the nearest C4 bump and a given instance's power pin, traversing the full PDN stack. A high SPR is a direct predictor of elevated IR-drop — the voltage at an instance's supply pin is depressed proportional to the product of the drawn current and the supply path resistance (V_drop = I × R_SPR). Ideally, a robust PDN design targets SPR values well below 100 Ω for all instances.

**Total instances with SPR > 500 Ω: 3,417**

This figure — derived using the command `awk '$2 > 500 {count++} END {print count}' instance_pin_spr.rpt` — indicates that a significant fraction of the 1,057,974-instance design has supply path resistances that are at least five times the ideal threshold.

---

#### Worst SPR Instances (Highest Resistance)

| SPR (Ω) | Cell | Instance (abbreviated) |
|---|---|---|
| **1,117.096** | NAND3xp33 ASAP7 6t L | .../execution\_unit\_0/ctmTdsLR\_2\_2872 |
| 1,111.151 | OAI21xp5b ASAP7 6t SL | .../execution\_unit\_0/741 |
| 1,104.570 | INVx1 ASAP7 6t L | .../execution\_unit\_0/ctmTdsLR\_1\_2871 |
| 1,104.493 | INVx1 ASAP7 6t SL | .../dbg\_uart\_0/ctmTdsLR\_2\_1887 |
| 1,101.042 | INVx2 ASAP7 6t L | .../execution\_unit\_0/HFSINV\_4\_304 |

**Worst instance:** `sc4/wSrPgt_inst/openMSP430_pgt_inst/execution_unit_0/ctmTdsLR_2_2872` — **SPR = 1,117.096 Ω**

---

#### Best SPR Instances (Lowest Resistance)

| SPR (Ω) | Cell | Instance (abbreviated) |
|---|---|---|
| **433.691** | OAI21xp5b ASAP7 6t L | .../register\_file\_0/ctmTdsLR\_1\_2606 |
| 433.836 | INVx3 ASAP7 6t L | .../clock\_module\_0/HFSINV\_276\_1419 |
| 433.884 | BUFx5 ASAP7 6t SL | .../frontend\_0/HFSBUF\_103\_1613 |
| 433.885 | BUFx4 ASAP7 6t SL | .../multiplier\_0/HFSBUF\_32\_484 |
| 433.891 | O2A1O1Ixp33 ASAP7 6t SL | .../frontend\_0/ctmTdsLR\_1\_8417 |

**Best instance:** `sc4/wSrPgt_inst/openMSP430_pgt_inst/execution_unit_0/register_file_0/ctmTdsLR_1_26` — **SPR = 433.691 Ω**

---

### 4.3 Assessment

The SPR results reveal a systemic PDN weakness across this design. Even the *best* SPR value of 433.691 Ω is more than four times the ideal target of <100 Ω for a well-designed power grid. More critically, the worst instances — concentrated in the execution unit of the openMSP430 processor core — reach 1,117 Ω, over eleven times the target. This is primarily attributable to a combination of factors: the execution unit is a high-density logic block that may lack sufficient M5/M6/M7 power straps, and the cells deep within the block hierarchy are potentially several strap pitches away from the nearest PDN grid node. The direct consequence of such high SPR is that even a moderate drawn current will produce a significant voltage drop (V = I × R), which translates into reduced noise margins, potential setup-time violations in critical paths, and in the worst case, functional failure. These findings align precisely with the IR-drop and BQM results discussed in Tasks 5 and 7.

---

## Task 5: Heatmaps – BQM and SPR

### 5.1 BQM (Bump Quality Metric) Heatmap

The Bump Quality Metric (BQM) is a per-bump figure-of-merit that quantifies how effectively each C4 flip-chip bump is contributing current to the on-die power delivery network. Mathematically, BQM is derived from the ratio of current flowing through a given bump to the ideal uniform distribution that would exist if all bumps contributed equally. Hotter (red) regions in the BQM heatmap identify bumps operating under disproportionately high current stress, which accelerates bump fatigue and increases electromigration risk at the bump interface. Conversely, low-BQM (cool/blue) bumps are underutilised — they could, in principle, be removed or relocated without significantly degrading power delivery, though in practice they serve as reliability redundancy.

[**INSERT FIGURE 5.1 HERE** — BQM heatmap for VDD\_cmplx and VSS\_cmplx bump data overlaid on layout. Red blobs indicate high-stress bumps (BQM up to ~0.058 for dbg domain, ~0.045 for VDD\_eu domain)]

**Key observations from the BQM heatmap:**

- The most heavily stressed bumps (red/hot) are concentrated in the **upper-centre and lower-centre columns** of the floorplan, spatially coinciding with the high-activity `openMSP430` processor core region. This is consistent with the expected activity pattern: the CPU core has the highest dynamic power density and therefore draws the most current from nearby bumps.
- **Four distinct vertical red stripes** are discernible across the die width, indicating that current demand is non-uniform and peaks at specific bump columns rather than being distributed evenly. This non-uniformity suggests either that the bump grid pitch in those columns is insufficient for the local current density, or that the horizontal M6/M7 power strap density in intervening regions is inadequate to redistribute current laterally.
- The `sc4/wSrPgt_inst/openMSP430_pgt_inst/dbg_0/VDD_dbg` domain registers the highest BQM of **0.058** — the debug block, despite its relatively small physical area, draws a disproportionate current share from its supply bumps, suggesting the debug logic is either more active than expected or its bump assignment is sparse.
- **Blue/cool regions** corresponding to the `SRAM_16_2048` memory macro corners exhibit low BQM, consistent with SRAM being predominantly in a data-retention (low-switching) state during the simulation scenario. The memory bumps are lightly loaded relative to the processor bumps.

---

### 5.2 SPR (Supply Path Resistance) Heatmap

The SPR heatmap presents a spatial view of the supply path resistance across the die, computed per metal layer from the extraction view `ev#v1`. Unlike the scalar SPR values in Task 4, this visualisation reveals the *spatial distribution* of PDN weakness — identifying which die regions and which metal layers constitute the bottlenecks in current delivery.

[**INSERT FIGURE 5.2 HERE** — SPR heatmap per metal layer (SPR::M1, M2, M3, M4, M5 from ev#v1) overlaid with short markers (pink/magenta). Colour scale: blue (~20 Ω) to red (>350 Ω). Red hotspots cluster at top and middle of die]

**Key observations from the SPR heatmap:**

- **M1** exhibits the highest SPR values, reaching approximately **354 Ω**. This is physically intuitive: M1 is the lowest metal layer, furthest in the via stack from the C4 bumps, and is typically a thin, minimum-width routing layer. Instances whose supply current must traverse long M1 segments before reaching an M2 or M3 via will see the worst resistance.
- **M5** shows substantially lower SPR values, with the scale topping out at approximately **66 Ω**. M5 is a thick global routing layer that forms the primary horizontal backbone of the power grid. Its low resistance reflects its wider wire dimensions and direct connectivity to the global power straps.
- The **pink regions** overlaid on the SPR heatmap represent shorted nodes — locations where the power and ground rails are electrically connected due to a routing error or extraction artefact. The spatial distribution of these shorts correlates directly with the 7 entries in `shorted_nodes.rpt` from Task 4, confirming that the visual markers are not artefacts.
- The SPR hotspot locations (execution unit and register file regions) are spatially consistent with the worst-SPR instances identified quantitatively in Task 4, validating the cross-tool analysis.

---

### 5.3 Shorts-Only Overlay

[**INSERT FIGURE 5.3 HERE** — Shorts-only overlay with all other heatmaps disabled. Three distinct short locations visible at zoomed-in level. Affected cells: XNOR2xp5\_ASAP7\_6t\_R and XOR2x2\_ASAP7\_6t\_R]

When all other heatmap overlays are disabled, the three distinct short locations are clearly visible. The affected standard cells are `XNOR2xp5_ASAP7_6t_R` and `XOR2x2_ASAP7_6t_R`. The total of 7 entries in `shorted_nodes.rpt` confirms that these shorts are localised and limited in number. Nevertheless, they must be investigated and resolved in the physical design database before tape-out, as even a single power-to-ground short can create a low-impedance path that significantly distorts the PDN voltage distribution across the entire domain.

---

### 5.4 Comparative Assessment: BQM vs. SPR

The combined interpretation of the BQM and SPR heatmaps provides richer diagnostic insight than either metric in isolation:

- In regions where **SPR is high but BQM is low**, the nearest bump is not efficiently routing current to the consuming cells. This pattern points to a structurally under-designed PDN mesh in that region — potentially caused by insufficient power strap width, too few inter-layer vias, or a suboptimal bump placement that locates bumps too far from the logic they serve.
- The **execution unit region** of the `openMSP430` core presents the most concerning combination: both the highest SPR (Task 4 worst instances, up to 1,117 Ω) and elevated BQM (bumps in this region are among the most highly stressed). This dual signature means that the bumps are being pushed to their current-carrying limits *while simultaneously* the cells they are meant to supply are experiencing the worst supply path resistance. The cumulative effect is a region of the die that is simultaneously overconstrained at the bump level and undersupplied at the instance level — the highest-risk area for dynamic IR-drop violations, timing failures, and long-term reliability degradation.
- **Memory corners** exhibit low BQM and moderate SPR, a combination that is acceptable given the low switching activity of SRAM in this simulation scenario. The resistance to the nearest bump, while not ideal, is tolerable for the current magnitudes involved.

---

## Task 6: Power Analysis Results

### 6.1 Power Summary Report

[**INSERT FIGURE 6.1 HERE** — RedHawk-SC Power Summary Report for galaxy\_cmplx, created Mon 20 Apr 2026 22:18:45]

The power analysis was generated on Mon 20 Apr 2026 at 22:18:45. Of the 1,057,974 total design instances, **1,057,916 were successfully summarised**, with 46,783 omitted due to missing power characterisation data — yielding a **coverage of 95.58%**. The missing 4.42% is attributable primarily to the instances identified in the `inst_no_slew.rpt` and `inst_no_tw.rpt` files from Task 3.

---

### 6.2 Total Power Breakdown by Component

| Power Component | Value (W) | Percentage | Description |
|---|---|---|---|
| Clock Pin Power | 0.06122 W | 39.22% | Power dissipated in the capacitive load of clock signal pins |
| Internal Power | 0.07818 W | 50.09% | Energy consumed by internal node transitions within standard cells |
| Leakage Power | 0.002741 W | 1.76% | Static subthreshold and gate-oxide leakage; constant regardless of activity |
| Switching Power | 0.075162 W | 48.16% | Dynamic power from charging and discharging output load capacitances |
| **TOTAL** | **0.15608 W** | **100%** | **≈ 156 mW** |

The near-equal split between internal power (50.09%) and switching power (48.16%) is characteristic of a highly active synchronous design operating near its maximum clock frequency. Leakage power at 1.76% is consistent with the ASAP7 7 nm predictive process node, where subthreshold leakage is non-trivial but still subordinate to dynamic dissipation at high activity factors.

---

### 6.3 Breakdown by Cell Class (User-Defined Groups)

| Cell Class | Total Power (W) | Percentage | Instance Count | Note |
|---|---|---|---|---|
| Sequential Logic | 0.069869 | 44.76% | 169,856 | **Maximum power class** |
| Clock | 0.048096 | 30.81% | 14,822 | — |
| Combinational Logic | 0.037618 | 24.10% | 746,294 | — |
| Memory | 0.00050079 | 0.32% | 8 | — |
| Decap / Filler | 1.1712×10⁻⁹ | ≈ 0% | 80,153 | **Minimum power class** |
| **Total** | **0.15608** | **100%** | **1,011,133** | — |

---

### 6.4 Clock Domain Power

From the Frequency Domains section of the power report, the clock domain operating at **7.143×10⁸ Hz (714.3 MHz)** consumes **0.15541 W**, representing **99.57%** of the total design power. This single domain dominates the power budget because the main system clock drives the toggling of every sequential element in the design at the highest possible rate.

---

### 6.5 Maximum and Minimum Power Cell Classes

**Maximum power class — Sequential Logic (0.069869 W, 44.76%):** Sequential logic (flip-flops and latches) is invariably the largest power consumer in deeply pipelined clocked digital designs. On every active clock edge, each flip-flop performs four energy-consuming operations simultaneously: capturing the incoming data value, switching its internal master-slave latch nodes, driving its output to the new state, and contributing to the clock tree load. With 169,856 sequential instances toggling at 714.3 MHz, the cumulative power dissipation is substantial.

**Minimum power class — Decap/Filler Cells (1.1712×10⁻⁹ W, ≈0%):** Decoupling capacitor and filler cells are passive PDN elements that serve structural (filling DRC-required gaps) and electrical (local charge reservoirs that mitigate dynamic voltage droop) purposes. They have no switching activity and contribute only negligible subthreshold leakage, making their power consumption effectively zero on the scale of the overall design.

---

## Task 7: Static Analysis Results

### 7.1 Working Directory

[**INSERT FIGURE 7.1 HERE** — `pwd` and `ll` output in `reports_23EE323_Manan/static_analysis` showing all report files with sizes and timestamps]

The static IR-drop analysis outputs reside in:

```
.../QuickStart_Training/reports_23EE323_Manan/static_analysis/
```

| Report File | Size | Description |
|---|---|---|
| `inst_voltage.rpt` | 616,993 B | Per-instance supply voltage values |
| `node_voltage.rpt` | 3,394,088 B | Full PDN node voltage data |
| `bump_current.rpt` | 272,911 B | Per-bump DC current values |
| `bump_voltage.rpt` | 272,911 B | Per-bump voltage values |
| `switch_report.rpt` | 207,797 B | Switching activity summary |
| `layer_voltage.rpt` | 6,133 B | Per-metal-layer voltage statistics |
| `supply_currents.rpt` | 126 B | Total supply current summary |
| `demand_current.rpt` | 670 B | Total demand current summary |

---

### 7.2 Static IR-Drop – Instance Voltage Results

Static IR-drop analysis (`av_static`) solves the linearised DC circuit of the full PDN, injecting average current demands at each instance and computing the resulting voltage distribution. The worst-case (lowest voltage) and best-case (highest voltage) instance results are obtained by sorting the `inst_voltage.rpt` file:

```bash
# Worst (lowest voltage):
sort -k3 -n inst_voltage.rpt | head -n 15

# Best (highest voltage):
sort -k3 -nr inst_voltage.rpt | head -n 15
```

[**INSERT FIGURE 7.2 HERE** — Instance voltage report sorted to show worst (lowest) and best (highest) voltages side by side]

---

#### Worst Instance Voltages (Highest IR-Drop)

| Voltage (V) | PG Arc | Instance (abbreviated) |
|---|---|---|
| **0.6951** | VDD\_cmplx / VSS\_cmplx | sc3/wRfPF\_inst/pmem/ClkBuf\_inst\_Lvl1\_0\_76\_76\_152 |
| 0.6951 | VDD\_cmplx / VSS\_cmplx | sc2/wRf\_inst/pmem/ClkBuf\_inst\_Lvl1\_0\_76\_76\_152 |
| 0.6956 | VDD\_cmplx / VSS\_cmplx | sc3/wRfPF\_inst/pmem/ClkBuf\_inst\_Lvl1\_0\_152\_76\_228 |
| 0.6956 | VDD\_cmplx / VSS\_cmplx | sc2/wRf\_inst/pmem/293213 |
| 0.6957 | VDD\_cmplx / VSS\_cmplx | sc3/wRfPF\_inst/pmem/ClkBuf\_inst\_Lvl1\_76\_76\_152\_152 |

**Worst voltage: 0.6951 V — IR-drop = 0.70 – 0.6951 = 4.9 mV from nominal.**

These worst-case instances are clock buffer cells located inside the `pmem` (program memory) hierarchy, distributed across multiple design replicas (sc2, sc3).

---

#### Best Instance Voltages (Lowest IR-Drop)

| Voltage (V) | PG Arc | Instance (abbreviated) |
|---|---|---|
| **0.6971** | VDD\_cmplx / VSS\_cmplx | sc4/wRf\_inst/pmem/ClkBuf\_inst\_Lvl1\_304\_0\_380\_76 |
| 0.6971 | VDD\_cmplx / VSS\_cmplx | sc1/wRf\_inst/pmem/ClkBuf\_inst\_Lvl1\_304\_0\_380\_76 |
| 0.6971 | VDD\_cmplx / VSS\_cmplx | sc1/wRf\_inst/pmem/ClkBuf\_inst\_Lvl1\_304\_228\_380\_304 |
| 0.6971 | VDD\_cmplx / VSS\_cmplx | sc4/wRf\_inst/pmem/237896 |
| 0.6971 | VDD\_cmplx / VSS\_cmplx | sc1/wRf\_inst/pmem/237896 |

**Best voltage: 0.6971 V — only 2.9 mV drop from nominal.**

These are also clock buffers within the `pmem` hierarchy, but located in a better-supplied region of the PDN (in proximity to supply straps).

---

### 7.3 Assessment

The static voltage range across the design spans **0.6951 V to 0.6971 V**, a span of just **2 mV**. On a 0.70 V nominal supply, the worst-case drop of **4.9 mV** represents a deviation of approximately **0.7%**. While this appears small in absolute terms, it is consequential at the 7 nm technology node where timing margins are extremely tight. At this supply voltage level, even sub-millivolt variations can translate to detectable propagation delay differences in critical-path logic. The fact that the worst-affected instances are clock buffer cells within the `pmem` clock tree is particularly significant: clock tree buffers operating with depressed supply voltages will exhibit increased propagation delays, which manifests as clock skew — a timing violation that cannot be corrected by static timing analysis margin tuning alone. This finding suggests that the power strapping density in the `pmem` memory region requires reinforcement.

---

### 7.4 Top/Bottom 10 Instances – Combined Report

[**INSERT FIGURE 7.3 HERE** — Output of `23EE323_Manan_task7.rpt` showing TOP 10 WORST and TOP 10 BEST instance voltages with associated power data, frequency, and toggle rate]

[**INSERT FIGURE 7.4 HERE** — Bash script visible in vim confirming `23EE323_Manan_task7.rpt` as the output filename, establishing authorship]

---

### 7.5 Instance Voltage Histogram

[**INSERT FIGURE 7.5 HERE** — Instance Voltage Histogram from the RHSC Explorer Static Voltage Drop summary (av\_static.get\_instance\_voltage\_histogram()). Majority of instances cluster near nominal 0.70 V; long left tail toward lower voltages]

The instance voltage histogram, generated via `av_static.get_instance_voltage_histogram()`, provides a statistical view of the PDN performance across the full instance population. The distribution shows the vast majority of the 1,057,974 instances clustered near the nominal 0.70 V supply, confirming that the PDN is well-designed over most of the die area. However, the pronounced left tail — extending toward lower supply voltages — represents the population of instances in PDN-weak regions (consistent with the execution unit and memory clock tree instances identified in the worst-case analysis above). The y-axis, plotted on a log scale, reveals that while the absolute count of high-drop instances is small relative to the full design, their spatial concentration in high-activity functional blocks makes them disproportionately impactful on design reliability.

---

## Task 8: EM (Electromigration) Analysis

### 8.1 Working Directory

The electromigration analysis outputs reside in:

```
.../QuickStart_Training/reports_23EE323_Manan/powerem_analysis/
```

| Report File | Size | Description |
|---|---|---|
| `DC_EM_metal_report.rpt` | 620,321 B | Per-segment DC EM check results for all metal layers |
| `DC_EM_via_report.rpt` | 540,303 B | Per-via DC EM check results |
| `EM_static.violations` | 9,720 B | Consolidated list of EM violations |

---

### 8.2 Background: What is Electromigration?

Electromigration (EM) is a physical reliability failure mechanism in which sustained high-density DC current causes the directional displacement of metal atoms within a conductor. In integrated circuits, copper atoms are displaced along the direction of electron flow — a process driven by momentum transfer from conduction electrons — gradually depleting material from one region (forming a **void**) and depositing it elsewhere (forming a **hillock**). As voids grow within a metal wire or via, the effective cross-sectional area available for current flow decreases, increasing local resistance, until the void spans the full conductor width and creates an **open circuit**. A hillock, conversely, can penetrate adjacent dielectric layers and short to neighbouring conductors.

Power delivery nets are uniquely vulnerable to electromigration for two compounding reasons. First, they carry **continuous, unidirectional DC current** throughout the entire operational lifetime of the device — 24 hours a day, 7 days a week — rather than the bidirectional AC currents that partially mitigate EM in signal nets through alternating atomic flux. Second, the total current magnitudes in power rails are determined by the aggregate demand of potentially millions of logic cells simultaneously, which can be orders of magnitude higher than any individual signal net current. The consequence is that undersized power rail segments can fail EM constraints by enormous factors, as observed in this design. The practical effect is premature chip failure: a device rated for 10 years of operational lifetime may fail in months or even weeks if EM violations are not corrected before tape-out.

---

### 8.3 EM Violations

[**INSERT FIGURE 8.2 HERE** — Cached EM violations content from `EM_static.violations.txt`]

[**INSERT FIGURE 8.3 HERE** — Same violations displayed with terminal title bar confirming `reports_23EE323_Manan/powerem_analysis` path, establishing authorship]

All violations identified in this analysis are on the **VDD\_cmplx power net**. The five worst violations are tabulated below:

| Net | Layer | EM Constraint (A) | Worst Violation (A) | Ratio (violation/limit) |
|---|---|---|---|---|
| VDD\_cmplx | **M5** | 5.7273×10⁻⁵ | **0.23711** | **≈ 4,140×** |
| VDD\_cmplx | M7 | 9.3469×10⁻⁵ | 0.17081 | ≈ 1,828× |
| VDD\_cmplx | M5 | 1.2359×10⁻⁴ | 0.15654 | ≈ 1,267× |
| VDD\_cmplx | M7 | 9.3469×10⁻⁵ | 0.16971 | ≈ 1,815× |
| VDD\_cmplx | M5 | 1.2359×10⁻⁴ | 0.15604 | ≈ 1,263× |

**Worst Metal violation:** Layer M5, EM limit = 5.727×10⁻⁵ A, actual current = **0.23711 A** (**4,140× over limit**)

**Worst Via-level violation:** Layer M7, EM limit = 9.347×10⁻⁵ A, actual current = **0.17081 A** (**1,828× over limit**)

Coordinates of the violation cluster are located near `RealCoord(287.156500, ...)` and `RealCoord(304.034000, ...)`.

---

### 8.4 Violation Assessment – FAIL Analysis

**All listed violations are FAILs and are critically severe.** The following analysis explains why:

- **The violation ratios are not marginal.** Overages of 1,000× to 4,140× above the EM current limit are not boundary cases that can be addressed through minor geometry adjustments. These segments on the VDD\_cmplx power net are conducting current densities that are three to four orders of magnitude beyond what the copper metallurgy and barrier layer can sustain for the required device lifetime. At these stress levels, the mean-time-to-failure of the affected segments is likely measured in days to weeks of operation at nominal temperature, rather than the 10-year product lifetime requirement.

- **The physical failure mechanism is well-understood and inevitable.** At current densities this far above the EM threshold, metal atom migration will proceed rapidly, forming voids in the M5 and M7 power strap segments near the identified coordinates. As these voids grow, the local resistance of the power rail increases, further concentrating current into the narrowing conductor in a self-reinforcing positive-feedback loop. The eventual outcome is a complete open circuit in the VDD\_cmplx power rail — the chip permanently loses power to the downstream logic region.

- **Power nets are ineligible for EM waivers without fundamental redesign.** In a production signoff flow, signal net EM violations can sometimes be formally waived by demonstrating through statistical lifetime analysis (using tools such as Black's equation) that the low toggle rate of a given net results in an acceptable mean-time-to-failure. This option is categorically unavailable for power nets, which by definition carry unidirectional DC current at all times. Every EM FAIL on a power net requires a physical fix.

- **The most probable root cause is undersized power strap geometry.** The extreme violation ratios on M5 and M7 — both global power grid layers — strongly suggest that specific strap segments in these layers were not sized for the full current they are carrying. In the ASAP7 process, EM constraints are derived from the wire cross-sectional area and the current density limit of the copper process. A segment carrying 0.237 A against a limit of 5.7×10⁻⁵ A must have an effective current-carrying cross-section that is thousands of times too small — consistent with a minimum-width strap or an isolated via stack serving an excessively large current demand area.

- **Remediation options:** The two primary physical fixes are (1) **widening the violating strap segments** to increase their cross-sectional area proportionally to the excess current ratio, and (2) **adding parallel strap structures** to distribute the current load across multiple conductors. In practice, both techniques are applied together, accompanied by a layout review of the bump placement and strap routing in the `RealCoord(287, ...)` and `RealCoord(304, ...)` regions of the die.

---

## Task 9: RHSC Explorer – One Click for All Results

### 9.1 The RHSC Explorer Dashboard

The **RHSC Explorer** (also referred to as the *Signoff Explorer*) is the central signoff review interface integrated into the RedHawk-SC environment. Rather than requiring the engineer to individually open and interpret each of the nine analysis reports generated throughout this lab, the Explorer consolidates all major analysis results — heatmaps, violation tables, power summaries, and design hierarchy — into a single unified browser-based dashboard. The screenshots in Task 2 (Figures 2.9–2.11) were themselves generated directly from the Explorer's Summary, Power, Static Voltage Drop, and Dynamic Voltage Drop tabs, demonstrating that the Explorer is not merely a navigation aid but is the primary interface through which signoff conclusions are drawn.

---

### 9.2 How to Launch the RHSC Explorer

The Explorer can be accessed through multiple entry points:

**GUI menu (quickest method):** In the SeaScape GUI, navigate to `View → Explorer`.

[**INSERT FIGURE 9.1 HERE** — Launching RHSC Explorer from the SeaScape GUI menu: View → Explorer. The Explorer entry is visible directly below the Layout Window option (shortcut Ctrl+L)]

**GUI toolbar:** `Analysis → Explorer`, or click the Explorer toolbar button directly.

**RHSC Python console:**
```python
>>> app.open_explorer()
```

**Unix prompt (direct launch):**
```bash
redhawk_sc -gui -explorer -gp /path/to/gp_directory
```

---

### 9.3 Scope of the RHSC Explorer

The Explorer collates results from across the full analysis pipeline into a single browsable interface. Its coverage encompasses:

- **IR-drop heatmaps (Static and Dynamic)** — corresponding to Tasks 5 and 7
- **EM violation heatmaps** — corresponding to Task 8
- **BQM and SPR heatmaps** — corresponding to Tasks 4 and 5
- **Power breakdown charts** (by domain, cell class, and clock frequency) — corresponding to Task 6
- **Grid robustness summary** (opens, shorts, disconnected nodes) — corresponding to Task 4
- **Design hierarchy browser** for drilling from the full-chip view down to any individual instance
- **Clickable violation table** with cross-probe capability to the layout view
- **One-click collated HTML/PDF signoff report generation** for team review and tape-out approval

---

### 9.4 Utility and Significance in the Signoff Flow

The workflow efficiency gain delivered by the RHSC Explorer cannot be understated in the context of modern large-scale IC design. Without it, a signoff engineer must navigate disparate text reports, manage multiple GUI windows, and mentally correlate spatial violation locations with functional block identities — a process that is both time-consuming and error-prone for a design with over one million instances.

The Explorer resolves this by enabling the engineer to:

1. **Identify the worst die regions at a glance** through heatmap overlays, without parsing megabyte-scale text reports.
2. **Cross-probe from any violation directly to its physical layout location**, enabling instant visual identification of the affected metal segments, macro boundaries, and surrounding routing.
3. **Compare analysis types side-by-side** — for example, superimposing SPR and static IR-drop heatmaps to confirm that high-resistance PDN regions correlate with elevated voltage drop, as observed in the execution unit analysis of this report.
4. **Generate and share a single visual signoff report** in HTML or PDF format, replacing the need to distribute multiple text files across the design team for review.

The RHSC Explorer thus represents the convergence point of all individual analysis tasks: it is the tool through which a power integrity engineer makes the final go/no-go recommendation on a design's readiness for tape-out, informed by the full analytical breadth of the RedHawk-SC platform.

---

*End of Lab Report — SC404, 23/EE/323 Manan Agarwal, Delhi Technological University, April 2026*
