# EMHASS Fork Divergence Analysis Report
**Generated:** November 1, 2025
**Fork Point:** Commit `7878647` (November 10, 2024)
**Upstream:** davidusb-geek/emhass (master branch)
**Fork:** werdnum/emhass

## Executive Summary

This repository diverged from upstream approximately 1 year ago (November 10, 2024). Since then:
- **Our fork:** 6 commits adding targeted improvements
- **Upstream:** 590 commits with major refactoring and new features

**Key Finding:** Our thermal management implementation (June 2024) is MORE ADVANCED than upstream's later implementation (April 2025), as we support both 'constrain' and 'penalize' modes while upstream only supports 'penalize' mode.

---

## Fork Point Details

**Commit:** `7878647e556f2a63e4a7f493166b05b46b805b14`
**Date:** November 10, 2024
**Message:** "Merge pull request #370 from GeoDerp/patch-19"

---

## Changes in Our Fork (6 commits)

### 1. **Thermal Management Enhancements** (fccd9ed, 1ff13e1)
**Date:** June 29, 2024
**Status:** 🟢 UNIQUE VALUE - More advanced than upstream

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
- BUT: Upstream ONLY supports penalty mode (no 'constrain' mode option)
- Upstream has similar overshoot logic
- Upstream has better logging

**Verdict:** Our implementation is MORE FLEXIBLE. Should be preserved and potentially contributed back to upstream.

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
**Status:** 🔴 COMPLETELY OBSOLETE

**Our Change:**
- Added build argument to specify custom base image:
```dockerfile
ARG base_image=ghcr.io/home-assistant/$TARGETARCH-base-$os_version:bookworm
FROM ${base_image} AS base
```

**Upstream Status:**
- Complete Dockerfile rewrite (multiple commits 2024-2025)
- Migrated from `pip` to `uv` package manager
- Changed from `setup.py` to `pyproject.toml`
- Changed data path from `/app/data/` to `/data/`
- Added `gunicorn` as production server
- Removed 32-bit ARM support
- Our change no longer applies due to complete restructure

**Verdict:** Completely obsolete. Cannot be ported to new Dockerfile structure.

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
**Status:** 🔴 COMPLETE REWRITE

Cannot merge - completely different structure:
- Old: pip-based, /app/data/
- New: uv-based, /data/, gunicorn
- Our base_image ARG is now meaningless

### tests/test_optimization.py
**Status:** 🟡 SIGNIFICANT CHANGES

- Reformatted and expanded
- New test patterns and fixtures
- Our thermal tests would need adaptation

---

## Change Categorization

### 🟢 Changes Still Useful & Should Be Preserved (2)

1. **Thermal Management Mode Selection**
   - Our 'constrain' mode option is MORE CAPABLE than upstream
   - Provides flexibility upstream lacks
   - Well-tested in our fork
   - **Action:** Definitely preserve, consider contributing back

2. **Optimization Debugging Feature**
   - Unique debugging capability
   - No equivalent in upstream
   - Very useful for troubleshooting
   - **Action:** Preserve, make configurable

### 🟡 Changes Needing Adaptation (2)

1. **Unit Tests for Thermal Management**
   - Core logic is valuable
   - Needs adaptation to upstream's test framework
   - Must work with their refactored code
   - **Action:** Adapt and integrate

2. **Thermal Overshoot Logic Details**
   - Both versions have similar logic
   - Ours may have subtle improvements
   - Need careful comparison to see if any details are better
   - **Action:** Compare carefully during rebase

### 🔴 Changes Now Obsolete (2)

1. **Dockerfile Base Image ARG**
   - Dockerfile completely rewritten
   - Our change no longer applicable
   - **Action:** Discard

2. **Deferrable Load Padding**
   - Upstream has identical logic plus more features
   - Our version is now redundant
   - **Action:** Discard, use upstream version

---

## Rebase Strategy Recommendations

### Option A: Clean Rebase (RECOMMENDED)
**Complexity:** High
**Outcome:** Clean history, modern codebase

**Steps:**
1. Create new branch from upstream/master
2. Cherry-pick thermal mode selection logic
3. Cherry-pick debugging feature (with config option)
4. Adapt thermal tests to new framework
5. Resolve conflicts carefully in optimization.py
6. Update all code to match ruff formatting
7. Test extensively

**Advantages:**
- Clean integration with upstream
- Access to all new features (HiGHS, InfluxDB, etc.)
- Future merges will be easier
- Code quality improvements from upstream

**Disadvantages:**
- Significant work required
- Extensive testing needed
- May break existing workflows temporarily

### Option B: Selective Port
**Complexity:** Medium
**Outcome:** Keep our branch, port specific upstream features

**Steps:**
1. Stay on our branch
2. Manually port specific upstream features we want
3. Keep our thermal implementation as-is
4. Keep debugging feature as-is

**Advantages:**
- Less disruptive
- Keep full control
- Faster short-term

**Disadvantages:**
- Growing divergence over time
- Miss out on many upstream improvements
- Future synchronization becomes harder

### Option C: Contribute Back Then Rebase
**Complexity:** High
**Outcome:** Best long-term, helps community

**Steps:**
1. Create PR to upstream adding 'constrain' mode option
2. Create PR to upstream adding debugging feature
3. Wait for upstream acceptance
4. Then do clean rebase from upstream
5. All our improvements are now in upstream

**Advantages:**
- Benefits entire community
- Cleanest long-term solution
- Recognition for our work
- Simplest maintenance going forward

**Disadvantages:**
- Requires upstream approval
- May involve modifications based on review
- Takes longer

---

## Specific Rebase Challenges

### Challenge 1: Thermal Management Code
**Location:** optimization.py lines 400-500

**Problem:**
- Both versions modified same code section
- Upstream reformatted everything with ruff
- Need to add our mode selection to their penalty-only code

**Solution:**
```python
# Add this to upstream's thermal section:
mode = hc.get('mode', 'penalize')  # Default to penalize for backward compatibility

if mode == 'constrain':
    # Use our hard constraint logic
    constraints.update({"constraint_defload{}_temperature_{}".format(k, Id):
        plp.LpConstraint(
            e = predicted_temp[Id],
            sense = plp.LpConstraintGE if sense == 'heat' else plp.LpConstraintLE,
            rhs = desired_temperatures[Id],
        )
    })
elif mode == 'penalize':
    # Keep upstream's penalty logic (which matches ours)
    penalty_factor = hc.get("penalty_factor", 10)
    # ... rest of penalty code
```

### Challenge 2: Debugging Feature Integration
**Location:** optimization.py end of perform_optimization()

**Problem:**
- Need to add our debugging code without breaking upstream's cleaner structure
- Should be configurable, not always-on

**Solution:**
```python
# Add to optim_conf:
if optim_conf.get('debug_export', False):
    lp_debug_path = self.emhass_conf['data_path']
    opt_model.writeLP(lp_debug_path / 'problem.lp')
    # ... rest of debug code
```

### Challenge 3: File Structure Changes
**Problem:**
- /app/data/ → /data/
- setup.py → pyproject.toml
- pip → uv

**Solution:**
- Accept all upstream changes
- Update our local development setup
- Update any documentation we have

---

## Testing Requirements After Rebase

### Critical Tests
1. ✅ Thermal management with mode='constrain'
2. ✅ Thermal management with mode='penalize'
3. ✅ Deferrable loads with missing timestep config
4. ✅ Debug export functionality (when enabled)
5. ✅ All existing upstream tests still pass

### Integration Tests
1. ✅ Real optimization with thermal loads
2. ✅ Battery + deferrable loads
3. ✅ Performance benchmarks (ensure no regression)

---

## Estimated Effort

| Task | Effort | Risk |
|------|--------|------|
| Set up upstream rebase | 1-2 hours | Low |
| Resolve optimization.py conflicts | 4-6 hours | High |
| Adapt unit tests | 2-3 hours | Medium |
| Update to ruff formatting | 1 hour | Low |
| Testing & validation | 4-6 hours | High |
| **Total** | **12-18 hours** | **High** |

---

## Recommendations

### Immediate Actions (Priority 1)
1. ✅ **Create this report** (DONE)
2. 🔲 **Backup current branch** - Tag as `pre-rebase-backup`
3. 🔲 **Create test scenarios** - Document current working configurations
4. 🔲 **Review upstream changelog** - Understand all breaking changes

### Short-term Actions (Priority 2)
1. 🔲 **Start with Option C** - Contribute back to upstream
   - Create PR for thermal mode selection
   - Create PR for debug export (as optional feature)
2. 🔲 **Engage with upstream maintainers** - Discuss our improvements
3. 🔲 **Set up test environment** - For validating rebase

### Long-term Strategy
1. 🔲 **Complete rebase after PRs accepted**
2. 🔲 **Establish regular sync schedule** - Don't let it diverge again
3. 🔲 **Consider becoming upstream contributor** - Prevent future divergence

---

## Risk Assessment

### High Risks
- **Breaking existing configurations** - Upstream has breaking changes
- **Regression in thermal management** - Complex merge area
- **Loss of debugging capability** - Must preserve carefully

### Medium Risks
- **Test failures** - Extensive refactoring upstream
- **Performance changes** - New solver options may behave differently
- **Time investment** - Significant effort required

### Low Risks
- **Data loss** - Git history is preserved
- **Reversibility** - Can always roll back with git

---

## Conclusion

This fork has valuable improvements, particularly:
1. **Thermal management flexibility** (constrain vs penalize modes)
2. **Debugging capabilities** (problem/solution export)

However, we're significantly behind upstream (590 commits, 1 year). The recommended path is:

1. **Contribute our improvements back to upstream**
2. **Then perform a clean rebase**
3. **Establish regular sync schedule**

This approach:
- Benefits the community
- Keeps us up-to-date with improvements
- Reduces long-term maintenance burden
- Provides recognition for our work

The alternative (staying diverged) will only make future integration harder and prevent us from benefiting from upstream's many improvements (HiGHS solver, InfluxDB, DST fixes, better testing, etc.).

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

### Our Thermal Mode Selection (To Preserve)
```python
if len(desired_temperatures) > I and desired_temperatures[I]:
    if hc.get('mode', 'constrain') == 'constrain':
        constraints.update({"constraint_defload{}_temperature_{}".format(k, I):
            plp.LpConstraint(
                e = predicted_temp[I],
                sense = plp.LpConstraintGE if sense == 'heat' else plp.LpConstraintLE,
                rhs = desired_temperatures[I],
            )
        })
    elif hc['mode'] == 'penalize':
        penalty_factor = hc.get('penalty_factor', 10)
        if penalty_factor < 0:
            raise ValueError("penalty_factor must be positive")
        penalty_value = (predicted_temp[I] - desired_temperatures[I]) * penalty_factor * sense_coeff
        penalty_var = plp.LpVariable("defload_{}_thermal_penalty_{}".format(k, I),
                                     cat='Continuous', upBound=0)
        constraints.update({
            "constraint_defload{}_penalty_{}".format(k, I):
            plp.LpConstraint(e = penalty_var - penalty_value, sense = plp.LpConstraintLE, rhs = 0)
        })
        opt_model.setObjective(opt_model.objective + penalty_var)
```

### Our Debug Export (To Preserve)
```python
lp_debug_path = self.emhass_conf['data_path']
opt_model.writeLP(lp_debug_path / 'problem.lp')
opt_model.toJson(lp_debug_path / 'problem.json')
with open(lp_debug_path / "result.json", "w") as f:
    vars_dump = {v.name: v.varValue for v in opt_model.variables()}
    result_dump = {
        "vars": vars_dump,
        "constraints": {k: v.toDict() for k, v in opt_model.constraints.items()},
        "cost_fun": opt_model.objective.toDict(),
    }
    json.dump(result_dump, f, default=repr, indent=2)
```

---

*End of Report*
