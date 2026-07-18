# rhit-neuro Project Lineage — Reconstruction Log

**Working dir:** `/home/brooklyn/xilinxWork/test`
**Started:** 2026-07-17 (repo-archaeology session)
**Author:** automated analysis (Claude) + brooklyn

This file is the running log for reconstructing the git lineage across the Rose-Hulman
(rhit-neuro) neuromorphic / Caravel-FPGA project ecosystem, so the history can later be
cleaned (build-artifact removal) and stitched back together with proper submodules.

It supersedes / extends the earlier `caravel_migration_findings.md`, which only covered
the Caravel-FPGA tapeout sub-thread. This document covers **all 16 rhit-neuro repos**
present in `rhit-neuro/` plus external upstreams in `outside-sources/`.

> **READMEs are not authoritative.** Every lineage edge below is backed by *git evidence*
> (shared commit SHAs, shared file-blob hashes, root-commit dates/subjects), with README
> claims noted only as corroboration or contradiction.

---

## Method (how each edge was derived)

All heavy lifting was offloaded to local `git`; no large binaries were read into context.

1. **Metadata dump** — `_analysis/repo_metadata.txt`: remotes, HEAD, all branches
   (local+remote) with tip date/SHA/subject, commit counts, tags.
2. **Git-ancestry links** — `_analysis/shas/*.sha` = every commit SHA per repo across all
   refs. Pairwise `comm -12` → repos that literally **share commits** = true
   clone-and-continue ancestry.
3. **Copy-based lineage** — `_analysis/blobs/*.blob` = every *file-blob* hash per repo
   (content-addressed, repo-independent). Pairwise intersection → repos that share
   identical file contents even with **no shared commit history** (the dominant pattern
   here — files were copied / "uploaded", not cloned).
4. **Direction** — root-commit dates + root-commit subject lines + tip dates.
5. **Provenance claims** — grep of all READMEs for external repo URLs and lineage
   keywords; external upstreams cloned (blobless) into `outside-sources/`.

Key raw artifacts live under `_analysis/`.

---

## TL;DR — the ecosystem is 4 parallel "tracks" + shared upstreams

The rhit-neuro work is a multi-year (2017→2026) undergraduate/research program with
**yearly cohort hand-offs**. History is *mostly discontinuous*: each cohort typically
started a fresh GitHub repo and **copied files in** rather than cloning, so git ancestry
is broken at almost every generation boundary. Only **2 pairs share real commit SHAs**;
everything else is linked by identical file blobs.

Four functional tracks:

- **Track A — Neuron simulation software (C++/HLS)** — the oldest thread (2017).
- **Track B — NeuroML model translation (Python)** — 2023→2025.
- **Track C — Caravel FPGA harness / programmer (Verilog + Vivado)** — 2022→2026.
- **Track D — Custom accelerator RTL modules** (synapse, DMA, NPU, LUT, SD, FP) —
  2022→2026, which feed **into** Track C's integration repos.

Plus external upstreams (efabless Caravel, bol-edu lab, ZipCPU, LiteX, etc.) in
`outside-sources/`.

---

## Repo inventory (16 rhit-neuro + upstreams)

| Repo | HEAD branch | First commit | Tip | #commits | Track |
|---|---|---|---|---|---|
| `parallel-neuro-simulation` | master | 2017-12-17 | 2020-12-15 | 264 | A |
| `fixed-neuro-sim` | master | 2021-01-04 | 2022-03-26 | 11 | A |
| `two-zedboard-sim` | main | 2021-05-20 | 2021-05-20 | 3 | A |
| `24-25_parallel-neuro-simulation` | main | 2025-04-09 | 2025-04-30 | 4 | A |
| `DECA-NeuroML_23-24` | main | 2024-04-03 | 2024-04-24 | 11 | B |
| `24-25_neuroml` | main | 2024-04-03 | 2025-05-13 | 26 | B |
| `caravel_FPGA` | main | 2022-10-21 | 2024-01-30 | 15 | C |
| `DECA-Caravel-Programmer-22-23` | main | 2022-10-21 | 2024-02-08 | 27 | C |
| `deca-caravel-programmer-vexriscv` | main | 2024-03-25 | 2024-04-24 | 27 | C |
| `Caravel_FPGA_2025_-DEPRECATED-_` | Our_Userspace | 2025-03-18 | 2025-05-21 | 34 | C |
| `Caravel_FPGA_2026` | main | 2025-05-07 | 2026-05-22 | 56 | C |
| `Caravel_Accelerator_Modules` | main | 2022-12-14 | 2023-03-10 | 24 | D |
| `deca-synapse-model-2324` | main | 2024-03-31 | 2024-04-24 | 6 | D |
| `DECA-wbdma-23-24` | main | 2024-04-16 | 2024-04-24 | 8 | D |
| `24-25_npu` | main | 2024-10-29 | 2025-05-06 | 69 | D |
| `24-25_Synaptic_Communication_Loop` | main | 2025-05-30 | 2026-01-20 | 13 | D |
| _(upstream)_ `outside-sources/caravel_mgmt_soc_litex` | main | — | 2024-01-03 | 601 | C |

---

## Confirmed git-ancestry links (shared commit SHAs — real `git clone` continuity)

Only **two** exist in the entire ecosystem:

### 1. `DECA-NeuroML_23-24` → `24-25_neuroml`  (clean clone-and-continue) ✅
`24-25_neuroml` contains **all 11** of `DECA-NeuroML_23-24`'s commits as its base, then
continues (14 more commits, 2025-01 → 2025-05). This is the *only* clean generational
hand-off in the whole ecosystem that preserved history.
- Shared tip of old repo: `fecfffb` (2024-04-24) → first new commit `78ada33` (2025-01-09).

### 2. `caravel_FPGA` ⟷ `DECA-Caravel-Programmer-22-23`  (siblings from common trunk) ✅
Both share the identical trunk `ba027da "Initial commit"` (2022-10-21) → `019156a
"fixed git lolzzz"` (2023-01-25), **10 commits**, then **fork**:
- `caravel_FPGA` continues: `80e1ace` → `6cb9b98` → `56f5c65` → `34e0d22` (to 2024-01-30;
  latest work on branch `Bilinear_as_fifo_temp_test`).
- `DECA-Caravel-Programmer-22-23` continues: `48044c9` → `dda8378` → … → `e79d239`
  (to 2024-02-08; latest on branch `bitstream_created`).
- They are **the same original 22-23 Caravel-on-FPGA Vivado project pushed to two GitHub
  repos**, then diverged. (Blob overlap 8327 — essentially identical object stores.)

**Everything else below is copy-based (identical blobs, zero shared commits).**

---

## Track A — Neuron simulation software (C++/RISC-V HLS)

Oldest thread. Ordering from root dates + root subjects + blob overlap:

```
parallel-neuro-simulation      (2017-12-17; GitLab import "Import project template", 264 commits)
        │  copied  (fixed-neuro-sim root commit subject literally = "Copied parallel-neuro-simulation files")
        ▼
fixed-neuro-sim                (2021-01-04)
        │  README: "Fork of fixed-neuro-sim that uses two connected ZedBoards"
        ▼
two-zedboard-sim               (2021-05-20)
        ⋮  (gap)
24-25_parallel-neuro-simulation (2025-04-09)  — 24-25 cohort revival of the sim line
```

**Evidence**
- Blob overlap within cluster: fixed↔two-zedboard **438**, 24-25↔fixed **413**,
  24-25↔two-zedboard **394**, 24-25↔parallel **305**, fixed↔parallel **302**,
  parallel↔two-zedboard **289**. Strong copied-content family.
- **No shared commit SHAs** among any of them → every hop was a file copy, not a clone.
  (parallel-neuro-simulation originates on GitLab — has GitLab-runner MR-merge branch
  names like `35-investigate-soft-lut-performance`, `13-add-command-line-option-…`.)
- Direction note: `fixed-neuro-sim`'s *tip* date (2022-03) is later than
  `two-zedboard-sim`'s (2021-05), but its *root* (2021-01-04, "Copied
  parallel-neuro-simulation files") precedes two-zedboard, and two-zedboard's README
  declares itself the fork → order above is correct; the late fixed-neuro-sim tip is just
  a README/CMake fix.
- `24-25_parallel-neuro-simulation` closest to `fixed-neuro-sim` by blob overlap.

**External deps referenced (vendored C++ libs — candidates for submodule remediation):**
`nlohmann/json`, `google/protobuf`, `mariokonrad/marnav`, and **`IBM/rocc-software`**
(the latter is *already* a proper submodule in `parallel-neuro-simulation`
— see `.gitmodules`, only real submodule found in the whole ecosystem).

---

## Track B — NeuroML model translation (Python)

```
DECA-NeuroML_23-24   (2024-04-03, 11 commits)
        │  ✅ real git clone-and-continue (shares all 11 SHAs)
        ▼
24-25_neuroml        (continues 2025-01 → 2025-05)
```
The one clean hand-off. External references are all *model sources*, not vendored code:
`NeuroML/NeuroML2`, `OpenSourceBrain/*`, `RonCalabreseLab/Leech-8Cell-Tutorial-NeuroML`.

---

## Track C — Caravel FPGA harness / programmer (Verilog + Vivado)

```
efabless/caravel ───┐ (harness RTL: caravel.v, housekeeping.v, mgmt core)
efabless/caravel_pico ─┤
efabless/caravel_mgmt_soc_litex ─┤ (mgmt SoC / LiteX)
efabless/Caravel_on_FPGA ────────┴──► THE FPGA-flow parent (caravel.v, adapters, sim, firmware, Vivado IP)
        │
        │  (two early rhit sub-threads, both from the efabless material)
        ├─ caravel_FPGA ≡ DECA-Caravel-Programmer-22-23  (2022-10-21; shared trunk→fork, link #2)
        │        22-23 Caravel-on-FPGA Vivado bring-up (Arty/Nexys). Latest work NOT on main:
        │        caravel_FPGA→ branch Bilinear_as_fifo_temp_test; Programmer→ branch bitstream_created
        │
        └─ deca-caravel-programmer-vexriscv (2024-03-25) — README: "built from 'labi' Vivado
                 project" of  bol-edu/caravel-soc_fpga-lab  (overlap 144); VexRiscv FPGA lab track
        │
        ▼
Caravel_FPGA_2025_-DEPRECATED-_ (2025-03-18, root "moving to rhit-neuro")
        │   ★ real work on branch **Our_Userspace** (ahead=14 of main), NOT main (main=blinking LED)
        │   - vendored hand-copy of caravel_mgmt_soc_litex first added here: commit 25e7dd67
        │     (Simar Dhillon, 2025-04-29, on Our_Userspace); it is a splice of efabless/caravel +
        │     Caravel_on_FPGA + caravel_mgmt_soc_litex (findings §8–§9)
        │   - blob overlap: caravel 518, Caravel_on_FPGA 105, caravel_mgmt_soc_litex 682
        ▼  copied, NOT cloned (blob overlap 1223, zero shared SHAs)
Caravel_FPGA_2026 (2025-05-07 → 2026-05-22)  ★ current head-of-dev repo
        - main = "Add files via upload" (history broken by uploads)
        - unmerged feature branches: gallegos_dev (FreeRTOS), PortBoards (Zedboard XDC),
          brooklyn-onboarding (gitignore/cleanup, strictly ahead of main)
        - blob overlap: caravel 517, caravel_mgmt_soc_litex 680, Caravel_on_FPGA 35
```

**Notes**
- `Caravel_FPGA_2025` root subject **"moving to rhit-neuro"** ⇒ repo was relocated from
  another account/org into the rhit-neuro org (name history: README references
  `Caravel_FPGA_2025.git` and `Caravel_NPU_FPGA_2025.git`).
- `Caravel_FPGA_2026` is a **copy** of 2025 (no shared SHAs; blob overlap 1223), not a
  clone. This is the biggest history-break to repair.
- Accelerator RTL from Track D flows *into* 2025/2026 (24-25_npu ↔ 2025 = 93 blobs,
  ↔ 2026 = 77) — the NPU/synapse modules were integrated into the FPGA build.
- The vendored `caravel_mgmt_soc_litex/` folder inside 2025/2026 is a **hand-assembled
  splice** (see `caravel_migration_findings.md` §8–§9), not a clean clone of the
  efabless upstream.

**External upstreams referenced:** `efabless/caravel`, `efabless/caravel_mgmt_soc_litex`
(in `outside-sources/` ✅), `efabless/Caravel_on_FPGA`, `efabless/caravel_pico`,
`efabless/caravel_board`, `Cloud-V/DFFRAM`, `bol-edu/caravel-soc_fpga-lab`,
`bol-edu/caravel-soc`, the LiteX ecosystem (`enjoy-digital/litex`, `litex-hub/*`,
`m-labs/migen`).

---

## Track D — Custom accelerator RTL modules

These are the project's own IP (synapse model, DMA, NPU, LUT/MAC, SD, FP units),
developed per-cohort and eventually integrated into Track C.

```
Caravel_Accelerator_Modules (2022-12-14, 22-23 cohort)
        │
        ├── deca-synapse-model-2324   (2024-03-31)   synapse / leech-heart model
        ├── DECA-wbdma-23-24          (2024-04-16)   Wishbone DMA (from ZipCPU/zipcpu zipdma)
        │
        ▼
24-25_npu (2024-10-29, 69 commits)  — integrates FP/LUT/DMA/SD/synaptic modules
        │   module dev branches: LUT_Module, Memory_Module, SD_Card_Interface_Module,
        │                        DMA_Module, b3-wishbone, Synaptic_Module
        │
24-25_Synaptic_Communication_Loop (2025-05-30)  — Zynq/ZedBoard Ethernet+SPI comm loop
```

**Evidence / cross-links (blob overlaps)**
- deca-caravel-programmer-vexriscv ↔ deca-synapse-model-2324 = **66**
- 24-25_Synaptic_Communication_Loop ↔ deca-synapse-model-2324 = **34**
- 24-25_npu ↔ Caravel_Accelerator_Modules = **27**
- 24-25_npu ↔ Caravel_FPGA_2025 = **93**, ↔ Caravel_FPGA_2026 = **77** (integration edge)

**External RTL origins referenced (vendored — candidates for submodule/attribution):**
- `ZipCPU/zipcpu` (zipdma) → DMA_Module / DECA-wbdma-23-24
- `ZipCPU/sdspi` → SD_Card_Interface_Module
- `akilm/FPU-IEEE-754` → FloatMultiply / FloatingCompare
- `nishthaparashar/Floating-Point-ALU-in-Verilog` → FP add/sub/mul
- `jiacaiyuan/i2c_slave` → deca-synapse-model I2C

---

## External upstreams (in `outside-sources/`)

All cloned blobless (`--filter=blob:none`) to stay lightweight. "Overlap" = # of exactly
identical file-blobs shared with rhit repos (see `_analysis/blobs/UP_*.blob`).

| Repo | Role / what was copied from it | Strongest rhit overlap | Status |
|---|---|---|---|
| `efabless/caravel` | **Harness RTL origin** (`caravel.v`, `housekeeping.v`, mgmt core, DFFRAM views) | mgmt_soc_litex **595**, FPGA_2025 **518**, FPGA_2026 **517** | ✅ |
| `efabless/Caravel_on_FPGA` | **True parent of the rhit Caravel-FPGA project** — FPGA flow, `_antmicro`/`_FPGA` adapter files, `debug_uart_script.py`, sim TBs, firmware, Vivado `clk_wiz` IP | FPGA_2025 **105**, FPGA_2026 **35**, vexriscv **13** | ✅ |
| `efabless/caravel_mgmt_soc_litex` | Base of vendored mgmt-core folder (splice, see findings §9) | (self) | ✅ |
| `efabless/caravel_pico` | PicoRV32 mgmt-core variant blobs merged into vendored copy | mgmt_soc_litex **249**, FPGA_2026 **31** | ✅ |
| `efabless/caravel_board` | Board bring-up firmware / programming scripts | mgmt_soc_litex **45**, FPGA_2026 **38** | ✅ |
| `bol-edu/caravel-soc_fpga-lab` | **Origin of `deca-caravel-programmer-vexriscv`** ("labi" Vivado lab) | vexriscv **144**, FPGA_2025 **54**, synapse-2324 **28** | ✅ |
| `Cloud-V/DFFRAM` | DFFRAM macro RTL | mgmt_soc_litex **3**, FPGA **3** | ✅ |
| `ZipCPU/zipcpu` | zipdma → `DMA_Module` / `DECA-wbdma-23-24` | 24-25_npu **7**, FPGA_2026 **7** | ✅ |
| `ZipCPU/sdspi` | SD-card SPI → `SD_Card_Interface_Module` | 24-25_npu **7**, FPGA_2026 **4** | ✅ |
| `akilm/FPU-IEEE-754` | FP multiply/compare RTL | (low; renamed) | ✅ |
| `nishthaparashar/Floating-Point-ALU-in-Verilog` | FP add/sub/mul RTL | (low; renamed) | ✅ |
| `jiacaiyuan/i2c_slave` | I2C slave → `deca-synapse-model-2324` | (low) | ✅ |

> **Findings §8 open question RESOLVED:** the `_antmicro`/`_FPGA` harness files
> (`chip_io_antmicro.v`, `mprj_io_antmicro.v`, `constant_block_antmicro.v`,
> `chip_io_FPGA.v`, `FPGA_POR.v`) originate in **`efabless/Caravel_on_FPGA`**
> (`Caravel/src/`), *not* `antmicro/caravel`. The rhit copies were then **modified**
> (Arty A7 → Nexys A7 port: exact blob hashes differ from Caravel_on_FPGA `main` HEAD),
> but `caravel.v` and ~35 other files (firmware, sim TBs, `clk_wiz` IP, `.bit`) still
> match byte-for-byte. Inside the rhit repos the modified adapters are duplicated
> identically into **both** `CARAVEL.srcs/.../imports/src/` and
> `caravel_mgmt_soc_litex/verilog/rtl/`.

**Not cloned (incidental citations, not vendored code):** LiteX ecosystem
(`enjoy-digital/litex`, `litex-hub/*`, `m-labs/migen`), C++ libs referenced by Track A
(`nlohmann/json`, `google/protobuf`, `mariokonrad/marnav`), NeuroML model sources
(`NeuroML/NeuroML2`, `OpenSourceBrain/*`), `Xilinx/embeddedsw`. `IBM/rocc-software` is
already a real submodule in `parallel-neuro-simulation`. These are candidates for the
submodule-remediation phase but were not needed for lineage.

---

## Branch catalog (all branches; ⚑ = dangling/unmerged w/ unique work, ★ = true head-of-dev)

Ahead/behind computed vs each repo's `origin/main` (or `origin/master`). Full data:
`_analysis/branch_topology.txt`. **Key theme: in several repos `main` is NOT current.**

### Repos where the newest work is NOT on `main` ⚠️
| Repo | ★ real head-of-dev | vs main | Notes |
|---|---|---|---|
| `Caravel_FPGA_2025_-DEPRECATED-_` | **`Our_Userspace`** | +14 / −5 | The actual FPGA userspace work + vendored mgmt_soc_litex. `main` = blinking-LED baseline only. (HEAD is correctly on Our_Userspace.) |
| `caravel_FPGA` | **`Bilinear_as_fifo_temp_test`** | +2 / −0 | Strictly ahead of main (synthesis fix, 2024-01-30). main is stale. |
| `DECA-Caravel-Programmer-22-23` | **`bitstream_created`** | +12 / −3 | 12 unique commits (schematic cleanup, warning fixes, "modify files to match found github repo"). Diverged from main. |

### Repos with dangling unmerged feature branches (real work never merged) ⚑
| Repo | Branch | vs main | Content — needs stitch/rescue decision |
|---|---|---|---|
| `Caravel_FPGA_2026` | ⚑ `gallegos_dev` | +9 / −20 | **FreeRTOS integration** (task scheduler, docs) — never merged to main. |
| `Caravel_FPGA_2026` | ⚑ `PortBoards` | +5 / −19 | **Zedboard XDC / board-port** (remapped mprj_io, removed 7-seg) — never merged. |
| `Caravel_FPGA_2026` | ★ `brooklyn-onboarding` | +7 / −0 | Cleanup branch (big .gitignore, board file, mgmt cleanup); strictly ahead of main; merges main. User-provided cleaned branch. |
| `24-25_npu` | ⚑ `LUT_Module` | +11 / −47 | Per-module dev branch; 11 unique commits (LUT + WB interface). Mostly superseded by main integration but verify before drop. |
| `24-25_npu` | ⚑ `Memory_Module` | +3 / −43 | Per-module dev branch. |
| `24-25_npu` | ⚑ `DMA_Module`, `SD_Card_Interface_Module`, `b3-wishbone` | +1 each | Per-module dev branches, near-fully merged. |
| `24-25_npu` | `Synaptic_Module` | +0 / −43 | Fully merged into main. |
| `fixed-neuro-sim` | ⚑ `cuda` | +6 / −2 | Abandoned CUDA-acceleration experiment (2021). |

### Merged / dead branches (safe to archive once history is stitched)
| Repo | Branch | Status |
|---|---|---|
| `Caravel_FPGA_2025_-DEPRECATED-_` | `BlinkingLED` (+0/−1), `Old_Attempt` (+0/−13) | merged / superseded |
| `24-25_Synaptic_Communication_Loop` | `aster_dev` (+0/−1) | merged into main |
| `caravel_FPGA` | `no-gitignore` (+2/−1) | diverged dead-end (early gitignore attempt) |
| `parallel-neuro-simulation` | 9 GitLab-era issue branches (2018–2019) | mix of merged + long-dead; `13-…-2`, `20-…`, `35-…` merged (+0); `multi-run-comparison` (+9), `test-for-data-memo` (+3), `12-allow-…-remote-debug` (+8), `18-…` (+3), `45-…` (+1), `13-…` (+2) are ancient dangling. `master` canonical. |

### Repos that are clean single-branch (`main`/`master` is canonical)
`24-25_neuroml`, `24-25_parallel-neuro-simulation`, `Caravel_Accelerator_Modules`,
`deca-caravel-programmer-vexriscv`, `DECA-NeuroML_23-24`, `deca-synapse-model-2324`,
`DECA-wbdma-23-24`, `two-zedboard-sim`.

---

## Resolved this session
- ✅ `_antmicro`/`_FPGA` harness files → **`efabless/Caravel_on_FPGA`** (findings §8 mystery).
- ✅ Harness RTL (`caravel.v`, `housekeeping.v`) origin → **`efabless/caravel`** (+ Caravel_on_FPGA).
- ✅ Full branch topology / which branch is canonical per repo (table above).
- ✅ `deca-caravel-programmer-vexriscv` ← `bol-edu/caravel-soc_fpga-lab` (overlap 144).
- ✅ All README-referenced code-origin upstreams cloned into `outside-sources/`.

## Open questions / next steps (for the cleanup + stitch phases)

1. **Naming vs identity:** confirm `24-25_npu` == the "Caravel_NPU_FPGA_2025" and
   `Caravel_FPGA_2025` == "Caravel_FPGA_2025.git" referenced in the 2026 README (repos were
   renamed when moved into the rhit-neuro org — root subject "moving to rhit-neuro").
2. Map exact file-level flow of Track D modules (LUT/DMA/SD/FP/synapse) into Track C 2025/2026.
3. **Decide fate of dangling branches** before stitching: rescue `gallegos_dev` (FreeRTOS) and
   `PortBoards` (Zedboard) into 2026 history; verify `24-25_npu` module branches are fully
   superseded; archive GitLab-era `parallel-neuro-simulation` branches.
4. **Cleanup targets (task #1 — history rewrite):** Vivado `*.runs/ *.cache/ *.gen/ *.sim/
   *.hw/ *.ip_user_files/ .Xil/ *.jou *.log`, committed bitstreams (`*.bit`), ELF/hex build
   outputs, and ASIC parasitics (`gds/ lef/ mag/ maglef/ def/ lvs/ signoff/`) in the vendored
   `caravel_mgmt_soc_litex/`. (Quantify sizes next.)
5. **Submodule-remediation candidates (task #3):** replace vendored copies with submodules
   pinned to the upstreams now in `outside-sources/` — esp. the `caravel_mgmt_soc_litex/`
   folder (splice; hard to submodule cleanly), ZipCPU DMA/SD, FP units, and the C++ libs in
   Track A (`nlohmann/json`, `protobuf`, `marnav`).
