# Changelog: Lera2026 benchmark family

All notable changes to the Lera2026 family are recorded here. The entry "2026-11: family repository" records the creation of this repository. The sections after it hold the changelogs of the 2 former repositories. They keep every entry, fact, date, count and commit reference, with the wording edited for style and the headings moved 2 levels down.

## 2026-11-26. Improved Duration records

- Florian Rascoussier improved the total route duration for 16 TDVRPTW instances with Kayros 2.0.0.dev0. [Kayros development revision `dba5688`](https://github.com/0nyr/kayros/commit/dba56886167c854d2ae36b627128d6d419f811f4)
- Replaying those routes without time windows on the corresponding TDVRP instances retained all existing TDVRP incumbents. The native and Python Fraction evaluators independently checked the exact durations and the earliest minimizing departure for each route under the edition `r1` profile, `td-binary64-exact-v1`.

## 2026-11: family repository

- The TDVRPTW and TDVRP variants of Lera2026, kept until now in the 2 repositories `MAMUT-routing-TDVRPTW-Lera2026` and `MAMUT-routing-TDVRP-Lera2026`, form 1 repository, `MAMUT-routing-Lera2026`, mounted in MAMUT-routing at `benchmarks/Lera2026`. The histories of both former repositories are imported, each under the directory of its problem type, and joined by a merge commit. `history/` holds the maps from the former to the new commit identifiers.
- Instances and best-known solutions keep their former path after the problem type, for example `TDVRPTW/S2/n=200/Lera-C1_2_10-S2.vrp.json`.
- The arc-category sidecars move to `sidecars/`, stored once: 300 sidecars kept, 660 duplicate copies removed. The same category file served both variants and several scenarios, so the scenario level `S1`/`S2`/`S3` is dropped under `sidecars/`, for example `sidecars/n=200/Lera-C1_2_10.igp.json.gz`. In 960 instances the sidecar path field now holds a path relative to the repository root. No sidecar content changed, and the sidecar SHA-256 recorded in each instance is unchanged.
- In each of the 960 instances, 2 metadata strings that named a former path now name the new one. `metadata.generator.source` changes from `MAMUT-routing benchmarks/VRPTW/Sintef2008 (GehHom1999)` to `MAMUT-routing benchmarks/Sintef2008/VRPTW (GehHom1999)`, and `metadata.base_instance` from `Sintef2008/n=<N>/<name>` to `Sintef2008/VRPTW/n=<N>/<name>`.
- The SHA-256 of the 960 instance files changed because of these rewritten fields. No instance identifier changed, and no best-known solution changed.
- `mamut-collection.json` at the root marks the repository as a family collection (`"layout_version": 2`).
- This layout is read by `mamut-routing-lib` 0.13 and later, which is in development and not released yet. `mamut-routing-lib` 0.12 and earlier read the former layout, which stays available read-only in the 2 archived former repositories.
- The README, LICENSE, CHANGELOG and CITATION.cff files of the 2 variants are replaced by 1 file each at the root, and a NOTICE file is added.

## Former TDVRPTW repository

### Changelog of the Lera2026 TDVRPTW BKS

All notable changes to the curated `Lera2026` TDVRPTW best-known solutions (BKS) are recorded here. The objective is **Duration** (duration minimization), and the depot departure time of each route is a decision variable. Costs are the authoritative output of the canonical checker (`mamut_routing_lib.td.check_td_solution`): exact IEEE-754 double arithmetic, no epsilon thresholds, routes in canonical order (sorted by first customer), total summed in that order. Any strict improvement is therefore real. Reminder: instances carry a minimal time-window repair relative to the raw Gehring & Homberger data (see README.md), so results are not comparable with static VRPTW literature.

#### 2026-09-24

**Re-priced under the `td-fold/2` checker contract (mamut-routing-lib 0.12.0), no route changed.** mamut-routing-lib 0.12.0 replaces the TD checker's route fold (checker contract `td-fold/1` -> `td-fold/2`). Waiting and service at a vertex are now applied exactly to the accumulated arrival times instead of being composed through a ratio interpolation, the departure window restricts the first arc without interpolation, and travel on slope-one pieces is computed by addition. Under `td-fold/1` a ready time could be off by an ulp, and on stepwise travel-time functions such an ulp could land past a step and read its upper branch. All 480 BKS were re-priced with `mamut-routing bks reprice-td`: 477 files were rewritten, 92 of them because the cost moved (by at most 4.4e-16 relative, float rounding only), the other 385 only to refresh `route_durations` / `route_departure_times` by ulps. Each BKS whose cost moved records `metadata.repriced` (previous cost, checker, contract, date).

#### 2026-07-08

480 of 480 BKS improved (mean -8.98%, largest single improvement -25.88%) by a 20,808-run anytime-strategy head-to-head campaign on Grid'5000: kayros 0.4.0.dev0 (TD-ILS, TD-ACO+LS, and an ACO-then-ILS budget split, all over the granular time-dependent local search), per-size time limits 300/600/900/1200/1800 s (n=200..1000), seeds {42, 123, 456}, single-threaded runs. Improve-only fold: for each instance the campaign-best solution was re-priced by the canonical checker before writing (checker cost authoritative). Stored BKS marked proven optimal were left untouched. Every instance of the family improved. The single-trajectory TD-ILS supersedes the initial TD-ACO seeding across the board at these sizes.

#### 2026-07-07 (family created)

480 instances generated by `populate_td_lera2026` v1 (IGP 2003 Table 1 speeds, Dabia-2013 five-period skeleton, minimal TW repair, cross-validated against the exact forward Ichoua loop). Initial BKS seeding by kayros TD-ACO+LS under a fixed protocol (per-size time limits 300/600/900/1200/1800 s, seeds {42, 123, 456}) on Grid'5000.

Seeding outcome (2026-07-07): full coverage, with all 480 instances holding a checker-validated `Duration` BKS (2880 runs, 960 instance-cells x 3 seeds, 0 failures, best of seeds kept per instance).

## Former TDVRP repository

### Changelog of the Lera2026 TDVRP BKS

All notable changes to the curated `Lera2026` TDVRP best-known solutions (BKS) are recorded here. The objective is **Duration** (duration minimization), and the depot departure time of each route is a decision variable. Costs are the authoritative output of the canonical checker (`mamut_routing_lib.td.check_td_solution`): exact IEEE-754 double arithmetic, no epsilon thresholds, routes in canonical order (sorted by first customer), total summed in that order. Any strict improvement is therefore real. Reminder: instances carry a minimal time-window repair relative to the raw Gehring & Homberger data (see README.md), so results are not comparable with static VRPTW literature.

#### 2026-09-24

**Re-priced under the `td-fold/2` checker contract (mamut-routing-lib 0.12.0), no route changed.** mamut-routing-lib 0.12.0 replaces the TD checker's route fold (checker contract `td-fold/1` -> `td-fold/2`). Waiting and service at a vertex are now applied exactly to the accumulated arrival times instead of being composed through a ratio interpolation, the departure window restricts the first arc without interpolation, and travel on slope-one pieces is computed by addition. Under `td-fold/1` a ready time could be off by an ulp, and on stepwise travel-time functions such an ulp could land past a step and read its upper branch. All 480 BKS were re-priced with `mamut-routing bks reprice-td`: 477 files were rewritten, 186 of them because the cost moved (by at most 9.5e-16 relative, float rounding only), the other 291 only to refresh `route_durations` / `route_departure_times` by ulps. Each BKS whose cost moved records `metadata.repriced` (previous cost, checker, contract, date).

#### 2026-07-08

473 of 480 BKS improved (mean -1.86%, largest single improvement -5.59%) by a 20,808-run anytime-strategy head-to-head campaign on Grid'5000: kayros 0.4.0.dev0 (TD-ILS, TD-ACO+LS, and an ACO-then-ILS budget split, all over the granular time-dependent local search), per-size time limits 300/600/900/1200/1800 s (n=200..1000), seeds {42, 123, 456}, single-threaded runs. Improve-only fold: for each instance the campaign-best solution was re-priced by the canonical checker before writing (checker cost authoritative). Stored BKS marked proven optimal were left untouched.

#### 2026-07-07 (family created)

480 instances generated by `populate_td_lera2026` v1 (IGP 2003 Table 1 speeds, Dabia-2013 five-period skeleton, minimal TW repair, cross-validated against the exact forward Ichoua loop). Initial BKS seeding by kayros TD-ACO+LS under a fixed protocol (per-size time limits 300/600/900/1200/1800 s, seeds {42, 123, 456}) on Grid'5000.

Seeding outcome (2026-07-07): full coverage, with all 480 instances holding a checker-validated `Duration` BKS (2880 runs, 960 instance-cells x 3 seeds, 0 failures, best of seeds kept per instance).
