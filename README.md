# Lera2026 benchmark family (TDVRPTW and TDVRP, TD-IGP-Gehring&Homberger, 200–1000 customers)

Benchmark family repository of [MAMUT-routing](https://github.com/ANR-MAMUT/MAMUT-routing) (ANR MAMUT, ANR-22-CE22-0016), mounted there as `benchmarks/Lera2026`. It holds the 2 problem variants of the family, TDVRPTW and its time-window-free twin TDVRP. Self-contained: instances, arc-category sidecars and best-known solutions ship together. The time-dependent travel times are stored as a compact IGP specification (`igp-profile` model) and materialize deterministically on load. No arrival-time-function sidecars are committed (a complete n=1000 ATF sidecar would weigh ~60–90 MB gzipped).

The family is named in honour of **Gonzalo Lera-Romero**, whose open-source exact TDVRPTW solver lineage (Lera-Romero et al. 2020) anchors modern work on the problem. The name is purely honorific. Lera-Romero is not an author of this family, and the family is distinct from the Lera-Romero branch-price-and-cut code vendored elsewhere in the MAMUT ecosystem.

## What this family is, and what it is not

This family does for time-dependent routing what Gehring & Homberger (1999) did for Solomon's VRPTW testbed. It scales the classic IGP time-dependent model (Ichoua, Gendreau, Potvin 2003) from 100 to 200–1000 customers. Base data are the **Gehring & Homberger VRPTW instances** (n = 200, 400, 600, 800, 1000, 60 per size) exactly as curated in MAMUT-routing's `Sintef2008` family (origin `GehHom1999`): coordinates, demands, service times, fleet sizes, capacities and time windows carried over verbatim except for the minimal time-window repair described below. Travel speeds follow **IGP 2003 Table 1** (3 arc categories, average speed ≈ 1) placed in the **Dabia et al. (2013) five-period skeleton** (rush periods at 20–30% and 70–80% of the original horizon, profile `[mid, rush, mid, rush, mid]` per category). This period placement protects G&H's early tight deadlines and was adopted after a full feasibility audit.

It is therefore **not** the raw G&H testbed (results are not comparable with static VRPTW literature), and **not** the historical IGP 2003 experimental setup either (that used Solomon-100 with soft time windows and 3 equal periods). The Solomon-based IGP testbed at n ≤ 100 lives in the `Dabia2013` family of MAMUT-routing.

## The TDVRP variant

The TDVRP variant is the **time-window-free twin** of the TDVRPTW variant. Its instances are identical, with the `time_windows` array dropped. Capacity, fleet, service times, horizon and all `td` content, including both sha256 values, are identical to the TDVRPTW sibling. The horizon is the TDVRPTW-repaired one, so the twins stay exactly comparable. The time-window repair section below describes where that horizon comes from.

## 2 tiers, the canonical S2 core and the S1/S3 intensity ladder

Scenario = rush slowdown factor a from IGP 2003 Table 1 (rush speed = midday speed ÷ a per category):

- **`S2` (a = 2), the canonical core**: all 300 G&H bases. Every G&H instance has exactly one canonical TD twin (the Dabia2013 philosophy).
- **`S1` (a = 1.5) and `S3` (a = 4), the intensity ladder**: the deterministic subset `_1`, `_5`, `_10` of each class (C1/C2/R1/R2/RC1/RC2), 18 per size, 90 per scenario. A base's S1/S2/S3 triplet shares one arc-category assignment, so the 3 instances differ only in the speed values. This gives a controlled congestion ladder on identical geometry. S3 is the extreme tier. Its time-window repair is substantial (see below), so treat it as a stress benchmark.

480 instances per variant, named `Lera-<GH name>-S<k>` under `TDVRPTW/S<k>/n=<customers>/` and `TDVRP/S<k>/n=<customers>/`.

## Minimal time-window repair (read before comparing anything)

G&H deadlines assume unit travel speed. Under IGP speeds (slow category ≈ 0.33–0.67 at S2) some customers become unreachable by **any** route, which is ill-posed under hard time windows. Each instance therefore carries the **minimal deterministic repair**. A customer deadline is lifted to the earliest possible time-dependent arrival (`l_i' = max(l_i, ceil(α_0i(0)))`), and the depot deadline to the worst singleton return. The 5 periods stay anchored to the *original* horizon (the last period extends), so repairs never change travel times. Repair magnitudes are recorded per instance in `metadata.tw_repair` (S2 lifts ~a few deadlines per affected instance, S3 substantially more). Instances are exactly reproducible from `Sintef2008` + this rule. The TDVRP variant drops the repaired time windows but keeps the repaired horizon.

## Layout

- `TDVRPTW/S<k>/n=<customers>/` and `TDVRP/S<k>/n=<customers>/` hold the instances (`<Name>.vrp.json`) and their best-known solutions (`<Name>.bks.Duration.json`) of each variant and scenario, for example `TDVRPTW/S2/n=200/Lera-C1_2_10-S2.vrp.json` and `TDVRP/S1/n=200/Lera-C1_2_10-S1.bks.Duration.json`.
- `sidecars/n=<customers>/` holds the arc-category sidecars, without the scenario level, for example `sidecars/n=200/Lera-C1_2_10.igp.json.gz`. Each sidecar is stored once and shared by every instance of its base name, in each scenario where the base appears and in both variants. Each instance references its sidecar by a path relative to the repository root, with the SHA-256 of the sidecar content.
- `mamut-collection.json` at the root marks the repository as a family collection (`"layout_version": 2`).
- `history/` holds the commit maps of the former repositories (see the history section below).

## Format (`igp-profile` model)

- `<Name>.vrp.json`: the instance in the MAMUT shape, with depot 0, coordinates/demands/service times/time windows/fleet from the G&H base (no time windows in the TDVRP variant), `horizon` `[0, repaired depot due date]`, and a `td` block `{model: "igp-profile", time_periods, speeds, categories_path, categories_sha256, atf_sha256}`. The stored `time_periods` floats are canonical.
- `<Base>.igp.json.gz`: the arc-category sidecar (`format: "mamut-td-igp-categories"`), an explicit symmetric category matrix over the complete graph as digit strings (seeded uniform, seed recipe in instance metadata), hashed over its uncompressed canonical bytes. It is shared by all instances of a base name (see Layout).
- **No ATF sidecar**: the canonical arrival-time functions (one non-decreasing continuous PWL function per arc, exact IGP consolidation with the last period's speed extended beyond the horizon) are **materialized on load** by `mamut_routing_lib.td` (≥ the `igp-profile`-capable release, see MAMUT-routing) from coordinates (Euclidean `sqrt(dx*dx+dy*dy)`), categories, periods and speeds. `atf_sha256` pins the materialized canonical bytes exactly as a committed sidecar's hash would. Integrity is therefore verifiable offline from published data alone.
- `<Name>.bks.Duration.json`: best known solution under the `Duration` objective. Costs are always the canonical checker's output (`mamut_routing_lib.td.check_td_solution`).

Population pipeline: `populate_td_lera2026` v1 (curation tooling maintained outside this repository, links will be published with the benchmark paper). Every instance was cross-validated against the direct forward Ichoua loop and hard-audited for post-repair singleton feasibility.

## Library compatibility

This layout is read by `mamut-routing-lib` 0.13 and later, which is in development and not released yet. `mamut-routing-lib` 0.12 and earlier read the former layout, which stays available read-only in the archived former repositories [MAMUT-routing-TDVRP-Lera2026](https://github.com/ANR-MAMUT/MAMUT-routing-TDVRP-Lera2026) and [MAMUT-routing-TDVRPTW-Lera2026](https://github.com/ANR-MAMUT/MAMUT-routing-TDVRPTW-Lera2026).

## Provenance and licensing

Underlying data: Gehring & Homberger (1999) VRPTW instances (as curated in MAMUT-routing `Sintef2008`); time-dependent model: Ichoua, Gendreau, Potvin (2003), EJOR 144(2), 379–396; period skeleton: Dabia et al. (2013). MAMUT-authored curation artifacts are distributed under the MIT license (see [LICENSE](LICENSE)). This notice does not relicense the underlying third-party benchmark definitions.

## Citation and benchmark authorship

The authors, in citation order, are Adrien Pichon, Florian Rascoussier, Romain Billot, Lina Fahed, Alexandru-Liviu Olteanu, Marc Sevaux and Christine Solnon. The 2 lead authors are ordered alphabetically by surname, followed by the scientific supervisors in alphabetical order. Their ORCID identifiers are recorded in [CITATION.cff](CITATION.cff).

The listed authors propose Lera2026 as a generated TDVRPTW and TDVRP benchmark. Their contribution comprises the construction of time-dependent instances from the Gehring and Homberger bases, seeded arc-category assignments, the selection of congestion scenarios and the deterministic feasibility-repair procedure. This repository publishes the resulting instances with time windows and their validated reference solutions. It also provides the time-window-free twins, retaining the same routing attributes, travel-time functions and repaired horizons to support paired comparisons. Cite this repository for that benchmark contribution, Gehring and Homberger (1999) for the underlying VRPTW data, Ichoua, Gendreau and Potvin (2003) for the speed model, and Dabia et al. (2013) for the five-period structure. The honorific family name does not attribute its construction to Gonzalo Lera-Romero. Individual solution-contributor credits are recorded with the solutions. Record the exact Git commit used in experiments.

## History of this repository

Until November 2026 the 2 variants lived in 2 repositories, `MAMUT-routing-TDVRPTW-Lera2026` and `MAMUT-routing-TDVRP-Lera2026`. Their histories are imported here. Each history was rewritten under the directory of its problem type (`TDVRPTW/` or `TDVRP/`), and a merge commit joins the 2. The rewrite gave every former commit a new identifier. `history/` holds 1 map per former repository, `history/commit-map-MAMUT-routing-TDVRPTW-Lera2026.tsv` and `history/commit-map-MAMUT-routing-TDVRP-Lera2026.tsv`. Each map is tab-separated, with the former commit identifier and the new one on each line, after a header line. See [CHANGELOG.md](CHANGELOG.md) for the reorganisation.
