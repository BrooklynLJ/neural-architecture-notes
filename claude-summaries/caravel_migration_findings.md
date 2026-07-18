# Caravel FPGA → ChipFoundry Tapeout Migration — Research Findings

Compiled from a repo-archaeology session on 2026-07-17. All commit SHAs, branch names,
and blob hashes below were captured directly from `git` (via `git ls-tree`, `git log`,
`git show`, `git rev-parse`) against shallow/blobless clones — file *contents* of large
binary artifacts (GDS, LEF, MAG, etc.) were never read, only their paths, types, and
presence/absence across repos.

---

## 0. Repositories referenced, with exact versions inspected

| # | Repo (owner/name) | Branch | Commit SHA inspected | Commit date | Role in this investigation |
|---|---|---|---|---|---|
| 1 | `efabless/caravel` | `main` | `27cbe49c90ba5362ad52c9968dd98e035c30c74f` | 2024-11-04 | Original ASIC harness (heavy, full hardened GDS) |
| 2 | `efabless/caravel-lite` | `main` | `5593d992bbeb5608b7524bc279d91237371612f1` | 2024-09-12 | Stripped-GDS variant of #1 |
| 3 | `efabless/caravel_mgmt_soc_litex` | `main` | `503eda0790085712ffef7f4ad8934c7daed3237f` | 2024-01-03 | Upstream drop-in management-core macro (also cloned **full history**, unshallow, for version-pinning in §9) |
| 4 | `chipfoundry/caravel_user_project` | `main` | `b510613cec367828966b37583f9090ac5ddb6491` | 2026-04-23 | Currently-maintained ChipFoundry template (successor ecosystem to #1–#3) |
| 5 | `antmicro/caravel` | `main` | `38492d9da4071eef2b9a32029fa9d4778a698cc1` | 2022-11-25 | Reference fork checked for origin of `_antmicro`-suffixed FPGA files (no match found — see §8) |
| 6 | `rhit-neuro/Caravel_FPGA_2026` | `main` | `f4fbc6c912a477afda3ec27bf933990e926e1179` | 2026-05-15 | Current head-of-development FPGA project (per user, this branch/repo is canonical) |
| 7 | `rhit-neuro/Caravel_FPGA_2026` | `brooklyn-onboarding` | `eb190cf2e66a1f8ad49c1145ad442f2818f809de` | 2026-05-22 | Cleaned-up version of #6, provided by user for this analysis |
| 8 | `rhit-neuro/Caravel_FPGA_2025_-DEPRECATED-_` | `Our_Userspace` | `b0bf94a4fe243b71227ec7e1c4b6c6ceda7c6a0f` (branch tip) | 2025-05-20 | Predecessor repo; contains the original commit that first added `caravel_mgmt_soc_litex/` |

Not independently deep-dived (referenced only): `chipfoundry/caravel`, `chipfoundry/caravel-lite`
(both referenced by name/URL inside `chipfoundry/caravel_user_project`'s `Makefile`, not cloned).

---

## 1. The ecosystem map

```
efabless/caravel  ───────────────┐
efabless/caravel-lite ───────────┤  original Efabless ecosystem (harness + mgmt core)
efabless/caravel_mgmt_soc_litex ─┘

        │  ChipFoundry (Umbralogic Technologies) acquired Efabless's
        │  IP/assets in September 2025 — see §2
        ▼

chipfoundry/caravel ──────────────┐
chipfoundry/caravel-lite ─────────┤  forks of the above, now the maintained lineage
chipfoundry/caravel_user_project ─┘  (uses the `cf` CLI, not raw Makefiles)

(separately, unrelated fork lineage used for FPGA emulation)
antmicro/caravel — FPGA-portable fork of efabless/caravel (swaps SkyWater-specific
  hard IP for FPGA-synthesizable equivalents: chip_io_antmicro.v-style modules)

rhit-neuro/Caravel_FPGA_2025_-DEPRECATED-_ (Our_Userspace branch)
        │  original repo; hand-assembled harness + mgmt core + FPGA project
        ▼
rhit-neuro/Caravel_FPGA_2026 (main branch = current head of dev)
        │  cleanup pass
        ▼
rhit-neuro/Caravel_FPGA_2026 (brooklyn-onboarding branch)
```

---

## 2. Business context: ChipFoundry's acquisition of Efabless

Per web search (Sept 3, 2025 press release, Semiconductor Digest, ChipFoundry FAQ):

- **Umbralogic Technologies LLC, d/b/a ChipFoundry, acquired Efabless's IP/technology
  assets** (15 issued patents + 3 pending) in September 2025.
- ChipFoundry is run in part by people from the original Efabless team (e.g. Mohamed
  Kassem), and continues the chipIgnite shuttle program.
- This is why `chipfoundry/caravel`, `chipfoundry/caravel-lite`, and
  `chipfoundry/caravel_user_project` exist as forks of the `efabless/*` originals and
  are the actively-updated lineage — Efabless itself is no longer the operating shuttle
  provider.

---

## 3. `efabless/caravel` vs `efabless/caravel-lite`

Diffed `verilog/rtl/` between commit `27cbe49c` (caravel) and `5593d992` (caravel-lite):
**file lists are identical** — same RTL source in both.

The actual difference is in pre-hardened physical views:

- `caravel/gds/` (commit `27cbe49c`): ~40 `.gds.gz` files — full hardened views:
  `caravel.gds.gz`, `caravel_core.gds.gz`, `housekeeping.gds.gz`,
  `mgmt_protect_hv.gds.gz`, etc.
- `caravel-lite/gds/` (commit `5593d992`): only 5 files —
  `openframe_project_wrapper_empty.gds.gz`, `user_project_wrapper_empty.gds.gz`,
  `user_analog_project_wrapper_empty.gds.gz`, plus `antenna_on_gds.tcl` and
  `gds2mag-all.sh`.

Same pattern in `lef/`, `mag/`, `def/`. **`caravel-lite` is "lite" purely in repo
weight** — the harness GDS is installed separately at setup time rather than committed.

---

## 4. `efabless/caravel_mgmt_soc_litex` (upstream, commit `503eda07`)

Top-level: `.gitignore`, `.readthedocs.yaml`, `LICENSE`, `Makefile`, `README.md`,
`def/`, `docs/`, `ef_io.list`, `gds/`, `lef/`, `lib/`, `litex/`, `lvs/`, `mag/`,
`maglef/`, `openlane/`, `scripts/`, `signoff/`, `spi/`, `sram_roerrors`, `verilog/`.

README (full text, commit `503eda07`):
```
To install litex library dependencies: cd litex && make setup
To build the caravel mgmt soc: cd litex && make
To simulate: cd sim && make clean sim
```
Also documents ECO fixes done on DFFRAM views for antenna cleanup.

`verilog/rtl/` at commit `503eda07` contains **only management-core-side files**:
`RAM128.v`, `RAM256.v`, `VexRiscv_LiteDebug.v`, `VexRiscv_MinDebug.v`,
`VexRiscv_MinDebugCache.v`, `defines.v`, `ibex/` (LowRISC ibex core — a LiteX-supported
alternate CPU option), `ibex_all.v`, `mgmt_core.v`, `mgmt_core.w_rst_init_modification.v`,
`mgmt_core_wrapper.v`, `picorv32.v` (another LiteX-supported alternate CPU).

**It has never contained harness-level RTL.** Confirmed via
`git log --all -- verilog/rtl/housekeeping.v` on the full-history clone: **zero commits,
ever.** Same for `caravel.v`, `caravel_core.v`, `gpio_control_block.v`, `pads.v`, etc.
— none of these have ever existed in this repo's history.

`caravel/README.rst` (commit `27cbe49c`) explicitly names this repo as
*"the default instantiation for the management core"* for the harness.

---

## 5. `chipfoundry/caravel_user_project` (commit `b510613c`) vs. the classic flow

1. **`cf` CLI, not raw Makefiles.** The top-level `Makefile` still exists but every
   target routes through a `check-deprecated` guard:
   > "This Makefile target is deprecated and requires confirmation. Use the `cf` CLI
   > instead... `pip install cf-cli`"
   The real workflow: `cf login → cf init → cf setup → cf harden <macro> →
   cf gpio-config → cf precheck → cf verify`.

2. **Dependencies are pulled at setup time, not vendored.** `Makefile` logic:
   ```makefile
   CARAVEL_LITE?=1
   ifeq ($(CARAVEL_LITE),1)
   CARAVEL_NAME := caravel-lite
   CARAVEL_REPO := https://github.com/chipfoundry/caravel-lite
   else
   CARAVEL_NAME := caravel
   CARAVEL_REPO := https://github.com/chipfoundry/caravel
   endif
   export MCW_ROOT?=$(PWD)/mgmt_core_wrapper
   ```
   `cf setup` fetches `chipfoundry/caravel-lite` (or `chipfoundry/caravel`) plus
   `mgmt_core_wrapper` into `dependencies/` — same mechanism as the old flow, repointed
   at ChipFoundry's forks and orchestrated by the CLI.

3. **`.cf/repo.json`** (provenance tracking file):
   ```json
   { "version": "v1", "changes": ["README.md", "Makefile", "openlane/Makefile"] }
   ```

4. **Only the project's own macro ships in-repo.** `gds/`, `lef/`, `def/`, `mag/`
   contain just `user_proj_example.*` and `user_project_wrapper.*` — no harness GDS at
   all, consistent with point 2.

5. Standard layout otherwise: `verilog/rtl/` (`user_project_wrapper.v`,
   `user_proj_example.v`, `user_defines.v`, `uprj_netlists.v`), `verilog/dv/`,
   `verilog/gl/`, `verilog/includes/`, `openlane/user_proj_example/`,
   `openlane/user_project_wrapper/`.

---

## 6. `rhit-neuro/Caravel_FPGA_2026`, `main` branch (commit `f4fbc6c9`)

Top level: `CARAVEL/`, `C_program/`, `Documents/`, `FreeRTOS Setup.docx`, `MATLAB/`,
`Mastering-the-FreeRTOS-Real-Time-Kernel.v1.1.0.pdf`, `Micropython_scripts/`,
`README.md`, `SpinalHDL_Scala_files/`, `caravel_mgmt_soc_litex/`,
`debug_uart_script.py`, `elf_file/`, `hex_file/`, `sim/`, `testing-car-on-fpga.runs/`.

- **`CARAVEL/`** — a Vivado project (`CARAVEL.xpr` + `.cache/`, `.gen/`, `.hw/`,
  `.ip_user_files/`, `.runs/`, `.sim/`, `.srcs/`, plus `vivado.log`/`vivado.jou`)
  targeting a Nexys A7 board.
  - `CARAVEL.srcs/sources_1/imports/src/` — harness/mgmt-core Verilog, including
    FPGA/Antmicro-adapted variants: `chip_io_FPGA.v`, `chip_io_antmicro.v`,
    `mprj_io_antmicro.v`, `constant_block_antmicro.v`, alongside standard-named
    `caravel.v`, `caravel_core.v`, `housekeeping.v`, `mgmt_core.v`,
    `mgmt_core_wrapper.v`, `VexRiscv.v`, `RAM128.v`, `RAM256.v`.
  - `CARAVEL.srcs/sources_1/imports/imports/` — the custom accelerator RTL:
    `TopLevel/SynapticModule.v`, `TopLevel/LUT.v`, `TopLevel/UserSpaceWBSYSCON.v`,
    `TopLevel/WishboneDevice.v`, `LUT_Module/` (LUT-based MAC unit + floating-point
    add/compare/multiply + indexing unit), `DMA_Module/` (ZipDMA-derived Wishbone DMA),
    `SD_Card_Interface_Module/`.
- **`caravel_mgmt_soc_litex/`** — a full vendored *copy* (not a submodule) of the
  upstream repo structure — see §8 for what it actually contains and where it came
  from.
- **`sim/`, `elf_file/`, `hex_file/`, `C_program/`, `Micropython_scripts/`** — firmware
  build artifacts (`gpio_mgmt.elf`, `our_userspace.hex`, `our_userspace.c`,
  `flash.py`/`main.py`).
- **`SpinalHDL_Scala_files/VexRiscvCachedWishboneForSim.scala`** — SpinalHDL generator
  config for the simulation VexRiscv core.
- **`Documents/`, `MATLAB/`, FreeRTOS PDF** — course/reference material.

---

## 7. `rhit-neuro/Caravel_FPGA_2026`, `brooklyn-onboarding` branch (commit `eb190cf2`)

Cleaned-up version of §6. Top level drops `elf_file/` and
`testing-car-on-fpga.runs/`; `CARAVEL/` no longer carries Vivado-generated junk
(`.cache/`, `.gen/`, `.hw/`, `.ip_user_files/`, `.runs/`, `.sim/` are gone — only
`CARAVEL.gen/`, `CARAVEL.srcs/`, `CARAVEL.xpr` remain).

README now documents an explicit Vivado git-merge workflow (feature branches →
`main_integration` → cherry-pick `*.v`/`*.sv`/`*.xdc`/`*.tcl` → verify bitstream →
merge to `main`) and a recommended `.gitignore`:
```
.Xil/
*.runs/
*.cache/
*.sim/
*.hw/
*.jou
*.log
*.str
```

### 7a. Vivado source management — are files "referenced" or "copied"?

Inspected `CARAVEL/CARAVEL.xpr` (blob `f68f7356523ff337aaabaeb7aa8d8a80cbf6b875`, 90
registered `<File Path=...>` entries) directly. **Every path uses
`$PSRCDIR/sources_1/...`** — Vivado's project-relative variable, pointing at files
*inside* the Vivado project tree, not at a canonical location elsewhere in the repo.

Confirmed by diff:
- `caravel_mgmt_soc_litex/verilog/rtl/housekeeping.v` vs.
  `CARAVEL/CARAVEL.srcs/sources_1/imports/src/housekeeping.v` — **byte-identical**.
- `caravel_mgmt_soc_litex/verilog/rtl/mgmt_core_wrapper.v` vs. the same file under
  `CARAVEL.srcs/.../imports/src/` — differ by exactly one line (an instance-name
  comment: `VexRiscV core (` vs `mgmt_core core (`).

This is Vivado's default "Import" behavior (Copy sources into project = checked):
files get physically duplicated into `<project>.srcs/sources_1/imports/<original
folder name>/...` and tracked from there, disconnected from their origin.

**`sources_1/new/`** contains files with no canonical copy anywhere else in the repo —
created directly via Vivado's "New Source" flow:
`SM_Reg_File.v`, `SM_g_accumulator.v`, `SM_h_accumulator.v`, `SM_Mem_Mapping.v`,
`SM_Packet_Generator.v`, `SM_I_SYN_Accumulator.v`, `SM_Reg_File_Static.v`,
`SM_Reg_File_Synapse_tn1.v`, plus **two files present on disk but NOT in the active
`.xpr` source list** (orphaned/unwired): `FloatMultiplyIEEE754.v` and
`SynapticModule_Standalone.v`.

**Implication:** moving the Vivado project into a subdirectory alone will not make it
reference shared canonical RTL. To get there: extract the `new/` files as canonical
sources first, then in Vivado use *Add Sources → Add or create → uncheck "Copy sources
into project"* so `.xpr` stores relative paths (e.g. `../../rtl/...`) instead of
importing duplicates, then delete the `imports/` copies.

---

## 8. What `caravel_mgmt_soc_litex/` inside the rhit-neuro repo actually is

Not a copy of one thing. Three distinct roles bundled into one folder:

1. **Origin of the harness/mgmt-core RTL used in the Vivado build** (see §7a diff
   evidence) — despite upstream `efabless/caravel_mgmt_soc_litex` never having
   contained harness files (§4). This vendored copy's `verilog/rtl/` contains BOTH
   the upstream mgmt-core files AND harness files AND FPGA/Antmicro-adapted files —
   a merge from at least two, likely three, source repos.
2. **RISC-V firmware/toolchain scaffolding** — `verilog/dv/firmware/` (`crt0_vex.S`,
   `crt0_ibex.S`, `linker_vex.ld`, `link_ibex.ld`, `defs.h`, `csr-defs.h`, IRQ headers)
   and `verilog/dv/generated/` (`csr.h`, `regions.ld`, `soc.h` — LiteX-generated memory
   map headers). Confirmed real: diffed `C_program/our_userspace.c`
   (blob differs, 338 vs 325 lines) against
   `caravel_mgmt_soc_litex/verilog/dv/tests-caravel/gpio_mgmt/our_userspace.c` — the
   project's firmware is a direct extension of this template (same `#include
   <defs.h>`, same structure, with project-specific register addresses like
   `#define LUT1 (*(uint32_t *)0x30501000)` added on top).
3. **Pure ASIC-hardening baggage, unused by the FPGA build**: `gds/`, `lef/`, `mag/`,
   `maglef/`, `lvs/`, `signoff/`, `def/`, `openlane/`, and the hardening half of
   `litex/`'s Makefile. Vivado never touches Sky130 GDS/LEF or runs OpenLane.

**Origin of the `_antmicro`-suffixed files could not be pinned.** Checked
`antmicro/caravel` (commit `38492d9d`, `verilog/rtl/`): it has `chip_io.v`,
`chip_io_alt.v`, `gpio_control_block.v`, `housekeeping.v`, `caravel.v` — but **no**
`chip_io_antmicro.v`, `mprj_io_antmicro.v`, `chip_io_FPGA.v`, or
`constant_block_antmicro.v`. These filenames don't match this fork's current state
(2022-11-25 snapshot); a web search for the exact filenames also returned no direct
hit. Origin remains unidentified — likely a different/older/unindexed fork.

---

## 9. Version-pinning investigation

### 9a. The commit that first added `caravel_mgmt_soc_litex/` to the project

Repo: `rhit-neuro/Caravel_FPGA_2025_-DEPRECATED-_`, branch `Our_Userspace`.

```
commit 25e7dd677955aaaddef6b8a689870a34ab99ff9f
Author: Simar Dhillon <dhillos@rose-hulman.edu>
Date:   Tue Apr 29 11:38:27 2025 -0400
    caravel code compilation folder
```

835 files added in this single commit — the entire upstream `caravel_mgmt_soc_litex`
repo structure (docs images, `gds/`, `lef/`, `lvs/`, `openlane/`, `litex/`, `verilog/`),
already including the harness + `_antmicro`/`_FPGA` files described in §8 (i.e., these
were present from the very first commit, not added later by the rhit-neuro team on top
of a clean upstream copy).

**Evidence this was assembled by hand (browser download), not `git clone`:**
- `caravel_mgmt_soc_litex/verilog/rtl/FPGA_POR.v:Zone.Identifier` and similar
  `:Zone.Identifier` files alongside most `.v` files — Windows "mark-of-the-web"
  metadata created when a file is downloaded via browser and extracted on Windows.
- `caravel_mgmt_soc_litex/verilog/rtl/io_buf copy.v` — an OS-level "Copy" duplicate
  (note the space before "copy").
- `caravel_mgmt_soc_litex/verilog/rtl/mgmt_core.v:Zone.Identifier` exists with **no**
  corresponding `mgmt_core.v` — it was renamed to
  `mgmt_core.w_rst_init_modification.v`, leaving the marker file orphaned.

### 9b. Blob-hash comparison against upstream `efabless/caravel_mgmt_soc_litex` full history

Compared file blobs from the vendored copy (as of `brooklyn-onboarding`,
commit `eb190cf2`) against every commit that ever touched the corresponding path in
upstream's full history (`efabless_caravel_mgmt_soc_litex_full`, unshallow clone).

| File | Result |
|---|---|
| `README.md` | Byte-identical to upstream content introduced by commit `69ec83d9` (2022-10-28); unchanged in upstream through current `main` HEAD |
| `Makefile` | Byte-identical to upstream content introduced by commit `075ab4f6` (2022-10-11); unchanged through current HEAD |
| `def/RAM128.def`, `def/mgmt_core_wrapper.def`, `openlane/Makefile`, `litex/caravel.py`, `verilog/includes/includes.rtl.caravel` | Byte-identical to current upstream `main` HEAD (`503eda07`) |
| `verilog/rtl/mgmt_core.w_rst_init_modification.v` | Byte-identical to upstream. Added by commit `d0d0d914` ("update to mgmt core to fix dbg_uart register reset"), **Dec 8, 2021**; never modified since, including at current HEAD |
| `verilog/rtl/mgmt_core_wrapper.v` | Matches upstream commit **`d4d7d5a1`, Jan 29, 2023** ("update `mgmt_core_wrapper` rtl: remove pass through signals as the soc will be flattened in caravel core") almost exactly. Only 2 diffs: `` `default_nettype none `` → commented out + `` `default_nettype wire `` added (an FPGA/Vivado compatibility tweak), and one instance-name comment (`mgmt_core core (` → `VexRiscV core (`). Checked against every other commit touching this file (25 commits total, spanning Nov 2021–Jan 2023) — no other candidate matches as closely. |
| `verilog/rtl/RAM128.v` | **Does not match upstream at any point in its history.** Upstream's `RAM128.v` has exactly one commit ever, `202d918f` ("update DFFRAM names", 2022-10-11), unchanged since — completely different module signature (no `COLS` parameter, different port order) than the vendored copy. Came from a different source. |
| `verilog/rtl/RAM256.v`, `verilog/rtl/defines.v` | Also do not match upstream current or historical content (differences include `` `RAM_BLOCKS`` 2 vs 1, `` `DM_INIT`` 3'b110 vs 3'b001, an added `` `OPENFRAME_IO_PADS`` define not present upstream) |

**Conclusion:** there is no single upstream commit this maps to. It's a splice:
- Tooling/physical-design side (`def/`, `openlane/`, `litex/`, `includes/`, `README.md`,
  `Makefile`) matches upstream's state as of **late 2022**, still current today (those
  files haven't changed upstream since).
- The mgmt-core wrapper RTL matches upstream as of **January 29, 2023**
  (commit `d4d7d5a1`), with 2 manual FPGA-compatibility edits.
- RAM models (`RAM128.v`, `RAM256.v`) and `defines.v` came from an unidentified
  different source — never matched anything in `caravel_mgmt_soc_litex` history.
- Harness RTL (`caravel.v`, `housekeeping.v`, etc.) and the `_antmicro`/`_FPGA` files
  came from a separate, unidentified source entirely (§8) — `caravel_mgmt_soc_litex`
  has never contained these files at any commit.
- The whole assembly shows signs of manual browser-download-and-copy (§9a), not a
  clean `git clone`, so a single clean "version" genuinely doesn't exist to point to.

---

## 10. Answers to the two specific claims checked

**Claim: "the Vivado portion could just be another directory, with verilog files moved
and added back to the project as references."**
→ Correct *direction*, not yet true. Today Vivado holds physical duplicates
(`$PSRCDIR/sources_1/imports/...`), not references. Making this work requires
extracting the Vivado-only `new/` files as canonical sources, then re-adding all
sources in Vivado with "Copy sources into project" unchecked. See §7a.

**Claim: "the entire `caravel_mgmt_soc_litex` directory was included solely for
RISC-V compilation, and maybe won't be necessary now."**
→ Not confirmed. It bundles three things (§8): (1) the actual origin of harness/mgmt-
core RTL used in the current Vivado build, (2) genuine RISC-V firmware/toolchain
scaffolding (real, keep this), (3) unused ASIC-hardening tooling (safe to drop once
migrating to the ChipFoundry template, which will pull its own management core via
`cf setup`). Don't delete the whole folder — the firmware headers and linker scripts in
it are load-bearing for current firmware builds.

---

## 11. Migration recommendations (for the FPGA-first, tapeout-later plan)

1. **Extract the real IP** — everything under `imports/imports/` (`TopLevel/`,
   `LUT_Module/`, `DMA_Module/`, `SD_Card_Interface_Module/`) plus the Vivado-only
   `sources_1/new/` files (`SM_*.v`) — into a canonical top-level RTL location, shared
   between Vivado and (eventually) the tapeout template.
2. **Re-home the harness dependency** — stop vendoring a hand-modified copy of harness
   RTL; either pick one canonical copy now, or plan to source it fresh via `cf setup`
   when the tapeout-track repo is stood up.
3. **Keep firmware scaffolding** (`verilog/dv/firmware/`, `verilog/dv/generated/`,
   `includes/`) — actively used by `C_program/` builds.
4. **Drop ASIC-only content** (`gds/`, `lef/`, `mag/`, `maglef/`, `lvs/`, `signoff/`,
   `def/`, `openlane/`) from the FPGA dev repo — free to re-fetch later from
   ChipFoundry's dependency chain.
5. **Re-point Vivado at canonical sources** — Add Sources with "Copy sources into
   project" unchecked, so `.xpr` stores relative paths instead of importing duplicates;
   verify the two orphaned `new/` files (`FloatMultiplyIEEE754.v`,
   `SynapticModule_Standalone.v`) are intentionally unused before discarding them.
6. **Treat the FPGA/Vivado flow and the future tapeout flow as separate tracks** that
   share RTL but not tooling — Vivado for iteration/verification now,
   `chipfoundry/caravel_user_project` + `cf` CLI for the eventual TinyTapeout submission.
