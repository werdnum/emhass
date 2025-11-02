# EMHASS Fork Divergence Analysis Report
**Generated:** November 1, 2025
**Fork Point:** Commit `7878647` (November 10, 2024)
**Upstream:** davidusb-geek/emhass (master branch)
**Fork:** werdnum/emhass

## Executive Summary

This repository diverged from upstream approximately 1 year ago (November 10, 2024). Since then:
- **Our fork:** 6 commits adding targeted improvements
- **Upstream:** 590 commits with major refactoring and new features

**Key Findings:**
1. **Debugging Feature** - We have a unique LP problem export capability that upstream lacks (genuinely valuable)
2. **Dockerfile ARG Pattern** - Required for our CI infrastructure to inject pre-built base images (local customization, not for upstream)
3. **Thermal Mode Selection** - Nice-to-have API convenience, but functionally equivalent to upstream's penalty approach with high penalty_factor

**Fork Purpose:** Minimal local patches for CI requirements + contribution of generally-useful debugging features back to upstream.

---

## Fork Point Details

**Commit:** `7878647e556f2a63e4a7f493166b05b46b805b14`
**Date:** November 10, 2024
**Message:** "Merge pull request #370 from GeoDerp/patch-19"

---

## Changes in Our Fork (6 commits)

### 1. **Thermal Management Enhancements** (fccd9ed, 1ff13e1)
**Date:** June 29, 2024
**Status:** 🟡 MINOR CONVENIENCE - Functionally equivalent to upstream

**Our Implementation:**
- Dual-mode thermal management: 'constrain' and 'penalize'
- Advanced overshoot prevention with binary constraint logic
- Prevents heating/cooling from turning on if already past overshoot temperature
- Unit tests for thermal management

**Key Code Features:**
```python
# Mode selection (only in our fork)
if hc.get('mode', 'constrain') == 'constrain':
    # Hard constraint on temperature
elif hc['mode'] == 'penalize':
    # Penalty-based approach
```

**Upstream Comparison:**
- Upstream implemented similar thermal features in April 2025 (commit 2f0cf19)
- Upstream uses penalty-only approach (no 'constrain' mode)
- Upstream has similar overshoot logic
- Upstream has better logging

**Optimization Theory Note:**
From an optimization perspective, `constrain` ≈ `penalize` with very high penalty factor. As penalty_factor approaches infinity, the soft constraint effectively becomes a hard constraint. Users can achieve the same result with upstream by setting `penalty_factor: 10000` or similar.

**Verdict:** Nice API convenience (explicit mode is clearer than magic numbers), but not functionally critical. Can be dropped to simplify rebase. Users can achieve same behavior with high penalty values.

---

### 2. **Deferrable Load Handling Fixes** (bcc462b)
**Date:** June 29, 2024
**Status:** 🟡 PARTIALLY OBSOLETE - Upstream has equivalent/better

**Our Changes:**
- Pad `def_total_hours`, `def_start_timestep`, `def_end_timestep` with zeros if not specified for all loads
- Added check: `len(def_load_config) > k` to prevent index errors

```python
def_total_hours = def_total_hours + [0] * (num_deferrable_loads - len(def_total_hours))
def_start_timestep = def_start_timestep + [0] * (num_deferrable_loads - len(def_start_timestep))
def_end_timestep = def_end_timestep + [0] * (num_deferrable_loads - len(def_end_timestep))
```

**Upstream Status:**
- Upstream implemented identical padding logic (commit 2f0cf19, April 2025)
- Upstream added NEW features we don't have:
  - `def_total_timestep` parameter (alternative to def_total_hours)
  - `minimum_power_of_deferrable_loads` support
  - `operating_timesteps_of_each_deferrable_load` validation
  - Much more extensive unit tests

**Verdict:** Our changes are now redundant. Upstream version is superior.

---

### 3. **Dockerfile Base Image Customization** (c6ce621)
**Date:** May 8, 2024
**Status:** 🟢 REQUIRED INFRASTRUCTURE - Critical for CI pipeline

**Our Change:**
- Added build argument to specify custom base image:
```dockerfile
ARG base_image=ghcr.io/home-assistant/$TARGETARCH-base-$os_version:bookworm
FROM ${base_image} AS base
```

**Why This Matters - CI Pipeline Requirement:**
This enables our Concourse CI pipeline to inject pre-built base images as OCI tarballs:
```yaml
params:
  IMAGE_ARG_base_image: base-image/image.tar
```

This pattern supports:
- **Image caching** - Avoid re-downloading base images on every build
- **Security scanning** - Pre-scan and approve base images before use
- **Air-gapped builds** - Base image fetched through separate, controlled pipeline
- **Reproducibility** - Pin exact base image versions independent of registry state
- **Enterprise compliance** - Required for many production CI/CD environments

**Upstream Status:**
- Complete Dockerfile rewrite (multiple commits 2024-2025)
- Migrated from `pip` to `uv` package manager
- Changed from `setup.py` to `pyproject.toml`
- Changed data path from `/app/data/` to `/data/`
- Added `gunicorn` as production server
- Removed 32-bit ARM support
- Upstream uses hardcoded base image (no ARG pattern)

**Verdict:** MUST BE PRESERVED as local patch. This is a legitimate infrastructure customization for enterprise CI/CD. Not suitable for upstream contribution (too specific to our build environment). Must be reapplied to new Dockerfile structure during rebase.

**Migration Strategy:**
```dockerfile
# Add ARG at top of upstream's new Dockerfile:
ARG base_image=ghcr.io/home-assistant/$TARGETARCH-base-debian:bookworm

# Replace their FROM line:
FROM ${base_image} AS base

# Keep all other upstream changes (uv, gunicorn, /data/, etc.)
```

---

### 4. **Optimization Debugging Feature** (edc4678)
**Date:** August 3, 2024
**Status:** 🟢 UNIQUE VALUE - Useful debugging tool

**Our Changes:**
- Dump LP problem to disk after optimization
- Export problem definition and solution in multiple formats

```python
opt_model.writeLP(lp_debug_path / 'problem.lp')
opt_model.toJson(lp_debug_path / 'problem.json')
with open(lp_debug_path / "result.json", "w") as f:
    result_dump = {
        "vars": {v.name: v.varValue for v in opt_model.variables()},
        "constraints": {k: v.toDict() for k, v in opt_model.constraints.items()},
        "cost_fun": opt_model.objective.toDict(),
    }
    json.dump(result_dump, f, default=repr, indent=2)
```

**Upstream Status:**
- No equivalent debugging feature exists
- Upstream has added extensive logging but not problem/solution export

**Verdict:** Valuable debugging capability. Should be preserved and potentially made configurable (e.g., only enabled when debug flag is set).

---

### 5. **Unit Tests for Thermal Management** (1ff13e1)
**Date:** June 29, 2024
**Status:** 🟡 NEEDS ADAPTATION

**Our Changes:**
- Added 242 lines of thermal management tests
- Removed sync conflict file

**Upstream Status:**
- Test file completely refactored (552 lines vs our 273 lines)
- Extensive new tests for:
  - Deferrable load validation
  - Minimum power constraints
  - Operating timesteps
  - DST handling
- Code formatted with ruff

**Verdict:** Our tests are valuable but need significant adaptation to work with upstream's refactored test framework.

---

### 6. **Merge Commit** (ef96338)
**Date:** November 14, 2024
**Status:** N/A - Merge commit

---

## Major Upstream Changes Since Fork (590 commits)

### Code Quality & Infrastructure
- **Massive refactoring:** optimization.py grew from 894 → 1682 lines (+788 lines)
- **Code formatting:** Migrated to ruff formatter (consistent style)
- **Type hints:** Better typing throughout (Python 3.10+ syntax: `int | None`)
- **Build system:** setup.py → pyproject.toml + uv package manager
- **Docker:** Complete rewrite with gunicorn, uv, modern practices
- **Testing:** Significantly expanded test coverage

### New Features
1. **HiGHS Solver Support** - Modern, faster LP solver option
2. **InfluxDB Integration** - Historical data retrieval from InfluxDB
3. **Deferrable Load Enhancements:**
   - Minimum power constraints
   - Timestep-based scheduling (not just hours)
   - Sequence-based deferrable loads improvements
4. **DST Handling** - Comprehensive daylight saving time fixes
5. **Open-Meteo Integration** - Weather forecast improvements
6. **Multi-threading** - Configurable `num_threads` for solver
7. **Thermal Management** - Penalty-based approach (less flexible than ours)

### Bug Fixes (Selection)
- Operating timesteps validation formulas
- None handling in deferrable loads
- CSV forecast handling for multiple days
- Battery SOC constraints
- Hybrid inverter curtailment
- Timezone handling improvements
- Weather forecast caching

---

## Detailed File-Level Analysis

### src/emhass/optimization.py
**Lines changed:**
- Fork point: 894 lines
- Our version: 957 lines (+63 from fork point, +7%)
- Upstream: 1682 lines (+788 from fork point, +88%)

**Conflict Areas:**
- Thermal management code (lines 400-500): Both modified
- Deferrable load initialization: Both modified (identical padding logic)
- Constructor: Upstream added num_threads support
- Solver selection: Upstream added HiGHS

**Merge Difficulty:** 🔴 HIGH
- File has been completely reformatted (ruff)
- Major structural changes throughout
- Our debugging feature would need integration
- Our thermal mode selection needs to be added to their code

### Dockerfile
**Status:** 🟢 REWRITE WITH REQUIRED PATCH

Upstream completely rewrote the Dockerfile:
- Old: pip-based, /app/data/
- New: uv-based, /data/, gunicorn

**Our Required Changes:**
- Must reapply base_image ARG pattern to new structure
- This is a local customization for our CI infrastructure
- Simple 2-line addition to upstream's new Dockerfile

**Migration:** Accept upstream's complete rewrite, then add ARG pattern at the top

### tests/test_optimization.py
**Status:** 🟡 SIGNIFICANT CHANGES

- Reformatted and expanded
- New test patterns and fixtures
- Our thermal tests would need adaptation

---

## Change Categorization

### 🟢 Changes MUST Be Preserved (2)

1. **Optimization Debugging Feature** (edc4678)
   - Unique debugging capability for exporting LP problems
   - No equivalent in upstream
   - Extremely useful for troubleshooting infeasible optimizations
   - **Action:** Preserve, make configurable, contribute to upstream

2. **Dockerfile Base Image ARG** (c6ce621)
   - **CRITICAL for CI infrastructure** - enables OCI tarball injection
   - Required for enterprise build pipelines (caching, security, air-gap)
   - Not suitable for upstream (too specific to our environment)
   - **Action:** Preserve as local patch, reapply to new Dockerfile

### 🟡 Changes Needing Adaptation (1)

1. **Unit Tests for Thermal Management** (1ff13e1)
   - Core test logic is valuable
   - Needs adaptation to upstream's refactored test framework
   - Must work with their reformatted code (ruff)
   - **Action:** Adapt and integrate selected tests

### 🟠 Changes Nice-to-Have But Optional (1)

1. **Thermal Management Mode Selection** (fccd9ed)
   - API convenience: explicit `mode='constrain'` vs `penalty_factor=10000`
   - Functionally equivalent to upstream's penalty approach
   - Adds code complexity for minimal benefit
   - **Action:** DROP to simplify rebase (users can use high penalty_factor)

### 🔴 Changes Now Obsolete (1)

1. **Deferrable Load Padding** (bcc462b)
   - Upstream has identical logic plus more features
   - Our version is completely redundant
   - **Action:** Discard, use upstream version

---

## Rebase Strategy Recommendations

### RECOMMENDED: Clean Rebase with Minimal Patches
**Complexity:** Medium
**Outcome:** Modern codebase with essential local customizations

**Steps:**
1. **Rebase onto upstream/master**
   ```bash
   git fetch upstream
   git rebase upstream/master
   ```

2. **For each commit during rebase:**
   - **fccd9ed (thermal mode):** DROP - not worth the complexity
   - **bcc462b (deferrable padding):** DROP - upstream has same + better
   - **c6ce621 (Dockerfile ARG):** KEEP - modify to work with new Dockerfile
   - **1ff13e1 (thermal tests):** ADAPT - update to new test framework
   - **edc4678 (debug export):** KEEP - make configurable
   - **ef96338 (merge):** DROP - merge commit

3. **Dockerfile conflict resolution:**
   ```dockerfile
   # Take upstream's entire new Dockerfile, then add at top:
   ARG base_image=ghcr.io/home-assistant/$TARGETARCH-base-debian:bookworm
   FROM ${base_image} AS base
   # ... rest of upstream's content
   ```

4. **Debug feature integration:**
   ```python
   # Add configuration option in optim_conf
   if self.optim_conf.get('debug_export_problem', False):
       lp_debug_path = self.emhass_conf['data_path']
       opt_model.writeLP(lp_debug_path / 'problem.lp')
       # ... rest of debug code
   ```

5. **Test thoroughly:**
   - All upstream tests must pass
   - CI pipeline with base image injection must work
   - Debug export must work when enabled

**Advantages:**
- Clean integration with upstream
- Access to all 590 commits of improvements (HiGHS, InfluxDB, DST fixes, etc.)
- Minimal maintenance burden going forward
- Only 2 local patches to maintain

**Result:** A clean fork that:
- Stays close to upstream (easy to sync)
- Preserves critical CI infrastructure
- Adds optional debugging capability
- Can contribute debug feature back to community

### Alternative: Contribute Debug Feature First
**If you have time:** Create PR to upstream with debug feature before rebasing

**Advantages:**
- Community benefit
- One less local patch to maintain
- Recognition for contribution

**Process:**
1. Create clean branch from upstream/master
2. Add debug export as configurable feature
3. Submit PR to davidusb-geek/emhass
4. After acceptance, rebase becomes even simpler (only Dockerfile ARG patch)

---

## Specific Rebase Challenges

### Challenge 1: Dockerfile Base Image ARG
**Location:** Dockerfile line 1-5

**Problem:**
- Upstream completely rewrote Dockerfile (pip→uv, /app/data/→/data/, gunicorn)
- Need to preserve ARG pattern for CI pipeline
- Must work with new structure

**Solution:**
```dockerfile
# At the very top of upstream's new Dockerfile, add:
ARG base_image=ghcr.io/home-assistant/$TARGETARCH-base-debian:bookworm

# Then change their FROM line from:
FROM ghcr.io/home-assistant/$TARGETARCH-base-debian:bookworm AS base

# To:
FROM ${base_image} AS base

# Keep everything else from upstream unchanged
```

**Testing:**
- Verify CI can pass `IMAGE_ARG_base_image: base-image/image.tar`
- Confirm OCI tarball injection still works
- Test both default (no ARG) and custom base image paths

### Challenge 2: Debugging Feature Integration
**Location:** optimization.py end of perform_optimization()

**Problem:**
- Need to add debugging code to upstream's reformatted file
- Should be configurable (off by default)
- Must respect new file structure (/data/ instead of /app/data/)

**Solution:**
```python
# Add after optimization completes, before returning:
if self.optim_conf.get('debug_export_problem', False):
    lp_debug_path = self.emhass_conf['data_path']
    self.logger.info(f"Exporting optimization problem to {lp_debug_path}")

    # Export LP problem in multiple formats
    opt_model.writeLP(lp_debug_path / 'problem.lp')
    opt_model.toJson(lp_debug_path / 'problem.json')

    # Export solution
    with open(lp_debug_path / "result.json", "w") as f:
        result_dump = {
            "status": self.optim_status,
            "objective_value": plp.value(opt_model.objective),
            "vars": {v.name: v.varValue for v in opt_model.variables()},
            "constraints": {k: v.toDict() for k, v in opt_model.constraints.items()},
            "cost_fun": opt_model.objective.toDict(),
        }
        json.dump(result_dump, f, default=repr, indent=2)
```

**Configuration:**
Add to config documentation that users can enable with:
```json
{
  "optim_conf": {
    "debug_export_problem": true
  }
}
```

### Challenge 3: Thermal Tests Adaptation
**Location:** tests/test_optimization.py

**Problem:**
- Upstream refactored test file completely
- New test patterns and structure
- Code reformatted with ruff

**Solution:**
- Take upstream's test file as base
- Port our thermal-specific test cases to their structure
- Follow their naming conventions and setup patterns
- Ensure tests work with penalty-based approach (not constrain mode)

---

## Testing Requirements After Rebase

### Critical Tests
1. ✅ All upstream unit tests pass
2. ✅ Debug export functionality (when `debug_export_problem: true`)
3. ✅ Debug export OFF by default (when config not set)
4. ✅ CI pipeline with base image injection works
5. ✅ Default Dockerfile build (no custom base image)

### Integration Tests
1. ✅ Real optimization with thermal loads (penalty-based)
2. ✅ Battery + deferrable loads
3. ✅ HiGHS solver (new upstream feature)
4. ✅ New `/data/` path structure works correctly
5. ✅ Performance benchmarks (ensure no regression from upstream changes)

---

## Estimated Effort

| Task | Effort | Risk |
|------|--------|------|
| Set up upstream rebase | 1 hour | Low |
| Dockerfile ARG reapplication | 1 hour | Low |
| Debug feature integration | 2-3 hours | Medium |
| Adapt selected unit tests | 2-3 hours | Medium |
| Testing & validation | 3-4 hours | Medium |
| **Total** | **8-12 hours** | **Medium** |

**Effort Reduction Factors:**
- Dropping thermal mode selection saves ~4 hours (no complex merge)
- Dropping deferrable load padding saves ~1 hour (redundant code)
- Dockerfile ARG is simple 2-line patch, not complex rewrite
- Only ~2 local patches to maintain going forward

---

## Recommendations

### Immediate Actions (Priority 1)
1. ✅ **Create this report** (DONE)
2. 🔲 **Backup current branch** - Tag as `pre-rebase-backup`
3. 🔲 **Document CI pipeline requirements** - Ensure base image ARG pattern is preserved
4. 🔲 **Review upstream v0.13.5 changelog** - Understand breaking changes

### Short-term Actions (Priority 2)
1. 🔲 **Perform clean rebase**
   - Follow strategy outlined above
   - Keep only: Dockerfile ARG + debug export
   - Drop: thermal mode, deferrable padding
2. 🔲 **Validate CI pipeline** - Ensure base image injection still works
3. 🔲 **Test debug export** - Verify problem export functionality

### Long-term Strategy
1. 🔲 **Optional: Contribute debug feature to upstream**
   - Create PR with `debug_export_problem` as configurable option
   - Benefits community, reduces our maintenance burden
2. 🔲 **Establish regular sync schedule** - Quarterly merges from upstream
3. 🔲 **Maintain minimal patch set** - Only 1-2 local patches (Dockerfile ARG + possibly debug if not upstreamed)
4. 🔲 **Document local customizations** - Clear README noting which changes are local-only

---

## Risk Assessment

### Medium Risks
- **CI pipeline breakage** - Base image ARG must be correctly reapplied
- **Path structure changes** - /app/data/ → /data/ may affect workflows
- **Performance changes** - New solver options and refactored code may behave differently

### Low Risks
- **Debug feature** - Simple addition, doesn't affect core logic
- **Test failures** - Accepting upstream tests wholesale
- **Data loss** - Git history is preserved
- **Reversibility** - Can always roll back with git tags
- **Config breakage** - Minimal configuration changes on our side

**Risk Mitigation:**
- Tag current state before rebase: `git tag pre-rebase-backup`
- Test CI pipeline thoroughly before deploying
- Keep backup of current Docker image

---

## Conclusion

This fork has **2 valuable changes** that must be preserved:

1. **Debugging Feature** - LP problem export capability (unique, should be contributed to upstream)
2. **Dockerfile ARG Pattern** - CI infrastructure requirement (local patch, not for upstream)

We're significantly behind upstream (590 commits, 1 year), but the path forward is clear:

### Recommended Approach: Clean Rebase with Minimal Patches

**What to Keep:**
- Dockerfile base image ARG (2-line patch for CI)
- Debug export feature (configurable, off by default)

**What to Drop:**
- Thermal mode selection (functionally equivalent to high penalty_factor)
- Deferrable load padding (upstream has identical + better)

**Benefits of This Approach:**
- ✅ Access to 590 upstream improvements (HiGHS, InfluxDB, DST fixes, etc.)
- ✅ Minimal ongoing maintenance (only 1-2 local patches)
- ✅ Easy future syncs with upstream
- ✅ Modern codebase (ruff formatting, better tests, type hints)
- ✅ CI infrastructure preserved
- ✅ Optional: Contribute debug feature back to community

**Estimated Effort:** 8-12 hours (reduced from initial 12-18 by dropping thermal mode)

**This is a healthy fork pattern:** Maintaining minimal infrastructure patches while staying close to upstream, with optional contributions back to the community. Not a competing project, just local customizations for enterprise requirements.

---

## Appendix A: Commit Lists

### Our Commits (Most Recent First)
```
ef96338 (2024-11-14) Merge in current patches
edc4678 (2024-08-03) Dump problem and solution after optimisation for debugging
c6ce621 (2024-05-08) Update Dockerfile to allow specifying base image as build arg
1ff13e1 (2024-06-29) Unit tests for thermal management
bcc462b (2024-06-29) Fixes for def_start_timestep, def_end_timestep handling
fccd9ed (2024-06-29) Updates for thermal management to address feasibility issues
```

### Key Upstream Commits (Selection)
```
9aae831 (2025-10-30) Updated CHANGELOG
2f0cf19 (2025-04-27) Allows thermal and standard loads to be scheduled
671e4e9 (2025-08-26) Merge: deferrable_min_power
0859bc4 (2025-09-11) Merge: operating-timesteps-validation-formula
7b6493c (2025-10-30) Merge: influxdb fixes
d0d21ec (2025-05-15) HiGHS solver, fix glpk solver, config num_threads
```

### Latest Upstream Version
Tag: `v0.13.5`

---

## Appendix B: Code Snippets for Reference

### Debug Export Feature (To Preserve and Contribute)
```python
# Add to end of perform_optimization(), make it configurable
if self.optim_conf.get('debug_export_problem', False):
    lp_debug_path = self.emhass_conf['data_path']
    self.logger.info(f"Exporting optimization problem to {lp_debug_path}")

    opt_model.writeLP(lp_debug_path / 'problem.lp')
    opt_model.toJson(lp_debug_path / 'problem.json')

    with open(lp_debug_path / "result.json", "w") as f:
        result_dump = {
            "status": self.optim_status,
            "objective_value": plp.value(opt_model.objective),
            "vars": {v.name: v.varValue for v in opt_model.variables()},
            "constraints": {k: v.toDict() for k, v in opt_model.constraints.items()},
            "cost_fun": opt_model.objective.toDict(),
        }
        json.dump(result_dump, f, default=repr, indent=2)
```

### Dockerfile Base Image ARG (To Preserve as Local Patch)
```dockerfile
# Add at top of upstream's new Dockerfile:
ARG base_image=ghcr.io/home-assistant/$TARGETARCH-base-debian:bookworm

# Modify FROM line:
FROM ${base_image} AS base

# This allows CI to inject: IMAGE_ARG_base_image: base-image/image.tar
```

### User Documentation: Achieving Hard Constraints Without Mode Selection
For users who want hard temperature constraints (equivalent to our dropped 'constrain' mode):
```json
{
  "def_load_config": [
    {
      "thermal_config": {
        "penalty_factor": 10000,  // High value ≈ hard constraint
        "desired_temperatures": [20, 20, 20, ...]
      }
    }
  ]
}
```

---

*End of Report*
