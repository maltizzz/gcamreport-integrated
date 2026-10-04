# gcamreport-integrated

This repository groups the report-package variants used with GCAM. The current
integration brings GCAM v9.1 support into the main `gcamreport` fork while
keeping the related variants available for comparison and testing.

## Repository layout

Each package lives twice: an `original` checkout that tracks the reference
remote and is never edited, and a `forked` checkout where the work happens.

| Directory | Remote / branch | Purpose |
| --- | --- | --- |
| [`gcamreport-core/original`](gcamreport-core/original) | [bc3LC/gcamreport](https://github.com/bc3LC/gcamreport) `gcam-core` | Upstream package, reference only. |
| [`gcamreport-core/forked`](gcamreport-core/forked) | [maltizzz/gcamreport](https://github.com/maltizzz/gcamreport) `pj-kaist` | Fork used for the v9.1 integration and the pull requests to upstream. |
| [`gcamreport-kaist/original`](gcamreport-kaist/original) | [GCAM-KAIST/gcamreport-kaist](https://github.com/GCAM-KAIST/gcamreport-kaist) `main` | KAIST team repository, reference only. |
| [`gcamreport-kaist/forked`](gcamreport-kaist/forked) | GCAM-KAIST/gcamreport-kaist `pj-kaist` | Personal working copy of the KAIST repository. Its `R/`, `inst/` and `data/` are synced from `gcamreport-core/forked`; the KAIST pipeline lives in [`kaist-pj/`](gcamreport-kaist/forked/kaist-pj/README.md). Pushing to the KAIST `main` from this checkout is disabled. |
| [`gcamreport-temp`](gcamreport-temp) | [maltizzz/gcamreport_temp](https://github.com/maltizzz/gcamreport_temp) `main` | Scratch checkout used during the v9.1 work and the KAIST u0909 debugging. Kept for its logs under `docs/`, `debug/` and `tests/`. |

Each directory is a Git submodule (see [.gitmodules](.gitmodules)). Run
commands for a package from inside that package's directory, not from this
repository's root. `Testrun.R` at the root is a local, gitignored launcher for
the Shiny UI and still points at the old `forked/gcamreport` layout.

## Status (October 2026)

The v9.1 work described below was merged into upstream `bc3LC/gcamreport`
(`gcam-core`, PR #90 from `pj-kaist`). `gcamreport-core/forked` is at the same
commit as `gcamreport-core/original`, and `gcamreport-kaist/forked` carries the
same `R/` and `inst/` (plus the GCAM9.1 mapping, query and template folders
that `gcamreport-kaist/original` does not have yet). All five checkouts report
package version 1.0.3; `available_GCAM_versions` includes `v9.1`.

Current KAIST work is the six-step KMIP pipeline in
`gcamreport-kaist/forked/kaist-pj/`, run against the two GCAM v9.1 u0909
scenario databases. See its [README](gcamreport-kaist/forked/kaist-pj/README.md).

## GCAM v9.1 support

The package (upstream `gcam-core`, and both `forked` checkouts) supports:

- GCAM v7.0, v7.1, v7.2, v8.2, and v9.1
- GCAMEurope 7.2 and 8.7
- ScenarioMIP and CMIP7 workflows

Use v9.1 explicitly when generating a report:

```r
gcamreport::generate_report(
  project = "path/to/project",
  GCAM_version = "v9.1"
)
```

The default version remains `v8.2` for compatibility with existing users.

## What changed for v9.1

GCAM v9.1 introduced new names and structures that were not present in the
original v8.2 mapping tables. The integration adds corresponding mappings and
package data for:

- Disaggregated residential and commercial building sectors
- `SMR` and `large reactor` nuclear technologies
- Hydrogen delivery modes: `H2 LDV`, `H2 MHDV`, and `LH2`
- Hybrid hydrogen electrolysis
- KLEAM agriculture labor and capital markets
- New transport, final-energy, water, emissions, and refrigerant combinations
- GCAM v9.1-specific data objects and version-selection logic

The changes are concentrated in `inst/extdata` and its package data. The report-generation and query code did not require a broad rewrite.

## Validation

The integrated package was tested against the GCAM v9.1 Windows release
database with all regions and variables enabled:

- `generate_report()` completed successfully.
- The result contained 88,739 rows and 15 columns.
- The result covered 33 regions and 2,847 variables.
- No `NA` or `Inf` values were present.
- The Shiny UI launched successfully with `GCAM_version = "v9.1"`.

A USA-only test can fail on the district-heat query because that region has no
district-heat rows. The same failure occurs with v8.2 and is a pre-existing
single-region limitation, not a v9.1 mapping failure. Use all regions for the
full validation run.

## Follow-up fixes (September 2026)

A second review against GCAM v9.1's own `Main_queries.xml` found problems that
the first validation run did not catch, because they produced wrong or missing
values rather than errors. All paths below are inside `gcamreport-core/forked`
(and, since the merge, in upstream as well).

- **Hydrogen query names.** GCAM v9.1 renamed `H2 wholesale dispensing` and
  `H2 retail dispensing` to `H2 LDV`, `H2 MHDV`, and `LH2`. The v9.1 query file
  still used the old names, so on-site hydrogen production and hydrogen prices
  to transport were missing. Five queries in
  [`queries_gcamreport_general.xml`](gcamreport-core/forked/inst/extdata/queries/GCAM9.1/queries_gcamreport_general.xml)
  now use the new names.
- **Transport units.** GCAM v9.1 reports transport service in billion pass-km
  and ton-km; v8.2 used million. The package assumed million, so transport
  energy service, vehicle sales, and vehicle stock were 1000 times too small. A
  new helper, `harmonize_trn_service_units()` in
  [`R/functions.R`](gcamreport-core/forked/R/functions.R), converts billion to million
  before these calculations. Older versions are unaffected.
- **On-site hydrogen mapping.** After the query fix, on-site hydrogen
  production (`onsite production` under `H2 LDV`, `H2 MHDV`, `LH2`) had no
  mapping row and stopped the report. Six rows were added to
  [`capacity_map.csv`](gcamreport-core/forked/inst/extdata/mappings/GCAM9.1/capacity_map.csv),
  following the old forecourt rows: electrolysis to `Secondary Energy|Hydrogen|Other`,
  natural gas steam reforming to `Gas` and `Fossil`.
- **Building energy prices.** The updated `en_price_map.csv` introduced 53
  new price variables, mostly building end uses, with no matching rows in
  [`en_demand_price_map.csv`](gcamreport-core/forked/inst/extdata/mappings/GCAM9.1/en_demand_price_map.csv),
  which stopped the report. The rows were added using the existing rule: the
  weighting variable is the price name without `Price|`.
- **Untracked v9.1 query files.** The `.gitignore` rule `queries_*` matched the
  v9.1 query XML files and their `data/queries_*_v9.1.rda` copies, so they had
  never been committed. Exceptions were added to `.gitignore`.

The package data objects built from these files were regenerated. After the
fixes, a full v9.1 report (all regions, Reference scenario, up to 2050)
completes. Compared with the first validation run:

- Transport energy service, sales, and stock values are 1000 times larger, now
  at realistic magnitudes (for example, about 78 trillion pass-km worldwide in
  2021).
- World `Secondary Energy|Hydrogen` in 2050 rises from 10.9 to 15.3 EJ/yr
  because on-site production is now counted.
- The template update adds building end-use variables and removes the
  `Emissions|...|Biomass|Traditional` variables, giving 88,961 rows and 2,887
  variables.

Known limitation: four prices have no World average because no consumption
variable exists to weight them: `Lighting|Electricity` for Residential,
Commercial, and Residential and Commercial, and `Commercial|Cooking|Liquids`.
GCAM counts lighting electricity under Appliances.

## Development notes

The v9.1 work followed this process:

1. Compare GCAM v8.2 and v9.1 sector, technology, commodity, and market names.
2. Clone the fork's GCAM8.2 mapping, query, and template directories for v9.1.
3. Add v9.1-specific mapping rows by cloning the equivalent existing reporting
   categories.
4. Rebuild package data with the `saveDataFiles_*.R` scripts.
5. Run a full-region report and fix any remaining strict-join gaps.
6. Verify report output and launch the Shiny UI.

The temporary checkout contains the detailed development history:

- [`gcamreport-temp/docs/CLAUDE_DEVELOPMENT_LOG.md`](gcamreport-temp/docs/CLAUDE_DEVELOPMENT_LOG.md):
  chronological log of the v9.1 mapping work and its validation.
- [`gcamreport-temp/debug/ERROR_LOG.md`](gcamreport-temp/debug/ERROR_LOG.md):
  the first v9.1 mapping error and the `v9.1` vs `v.9.1` key pitfall.
- [`gcamreport-temp/debug/case1_chemical_feedback_Sector/Error_Log/README.md`](gcamreport-temp/debug/case1_chemical_feedback_Sector/Error_Log/README.md):
  the eight fixes needed for the KAIST u0909 scenarios (`chemical feedstocks`
  sequestration rows, KAIST-only markets, the empty-`ignore` CO2 price bug).
  These are re-applied at runtime by `kaist-pj/core/functions.R`
  (`kaist_overrides`) and `kaist-pj/core/gcamreport_patch.R` in
  `gcamreport-kaist/forked`.
- [`gcamreport-temp/tests/README.md`](gcamreport-temp/tests/README.md) and
  [`gcamreport-temp/tests/log/README.md`](gcamreport-temp/tests/log/README.md):
  how the testthat suite is run and why five tests failed on 2026-09-14.
- [`gcamreport-temp/R/FUNCTIONS.md`](gcamreport-temp/R/FUNCTIONS.md): a
  function-by-function catalog of the package's `R/` folder.

## Working with submodules

Check the state of every nested repository with:

```bash
git submodule status --recursive
```

Commit changes in the repository where they were made, then update the parent
repository's gitlink:

```bash
cd gcamreport-kaist/forked
git add <files>
git commit -m "Describe the package change"
git push origin pj-kaist

cd ../..
git add gcamreport-kaist/forked
git commit -m "Update gcamreport-kaist/forked"
git push
```

The `original` checkouts are never committed to; update them with
`git pull` inside the submodule and then record the new gitlink here.

The parent repository tracks only each submodule's commit, not the files inside
that submodule.
