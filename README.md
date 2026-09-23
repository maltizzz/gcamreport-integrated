# gcamreport-integrated

This repository groups the report-package variants used with GCAM. The current
integration brings GCAM v9.1 support into the main `gcamreport` fork while
keeping the related variants available for comparison and testing.

## Repository layout

| Directory | Remote | Purpose |
| --- | --- | --- |
| [`gcamreport-fork`](gcamreport-fork) | [maltizzz/gcamreport](https://github.com/maltizzz/gcamreport) | Main report package. This is where the integrated v9.1 changes live. |
| [`gcamreport-kaist`](gcamreport-kaist) | [GCAM-KAIST/gcamreport-kaist](https://github.com/GCAM-KAIST/gcamreport-kaist) | KAIST report variant. |
| [`gcamreport-temp`](gcamreport-temp) | [maltizzz/gcamreport_temp](https://github.com/maltizzz/gcamreport_temp) | Temporary development and comparison checkout used during the v9.1 work. |

Each directory is a Git submodule. Run commands for a package from inside that
package's directory, not from this repository's root.

## Clone and initialize

```bash
git clone https://github.com/maltizzz/gcamreport-integrated.git
cd gcamreport-integrated
git submodule update --init --recursive
```

The nested submodule definitions are in [.gitmodules](.gitmodules). If a folder
appears with an unclickable arrow on a web page, check that this file has been
committed and pushed in the repository that contains the nested submodule.

## GCAM v9.1 support

The main package in `gcamreport-fork` supports:

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

The changes are concentrated in `gcamreport-fork/inst/extdata` and its package
data. The report-generation and query code did not require a broad rewrite.

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
values rather than errors. All paths below are inside `gcamreport-fork`.

- **Hydrogen query names.** GCAM v9.1 renamed `H2 wholesale dispensing` and
  `H2 retail dispensing` to `H2 LDV`, `H2 MHDV`, and `LH2`. The v9.1 query file
  still used the old names, so on-site hydrogen production and hydrogen prices
  to transport were missing. Five queries in
  [`queries_gcamreport_general.xml`](gcamreport-fork/inst/extdata/queries/GCAM9.1/queries_gcamreport_general.xml)
  now use the new names.
- **Transport units.** GCAM v9.1 reports transport service in billion pass-km
  and ton-km; v8.2 used million. The package assumed million, so transport
  energy service, vehicle sales, and vehicle stock were 1000 times too small. A
  new helper, `harmonize_trn_service_units()` in
  [`R/functions.R`](gcamreport-fork/R/functions.R), converts billion to million
  before these calculations. Older versions are unaffected.
- **On-site hydrogen mapping.** After the query fix, on-site hydrogen
  production (`onsite production` under `H2 LDV`, `H2 MHDV`, `LH2`) had no
  mapping row and stopped the report. Six rows were added to
  [`capacity_map.csv`](gcamreport-fork/inst/extdata/mappings/GCAM9.1/capacity_map.csv),
  following the old forecourt rows: electrolysis to `Secondary Energy|Hydrogen|Other`,
  natural gas steam reforming to `Gas` and `Fossil`.
- **Building energy prices.** The updated `en_price_map.csv` introduced 53
  new price variables, mostly building end uses, with no matching rows in
  [`en_demand_price_map.csv`](gcamreport-fork/inst/extdata/mappings/GCAM9.1/en_demand_price_map.csv),
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

The temporary checkout contains the detailed development history in
[`gcamreport-temp/README.md`](gcamreport-temp/README.md) and
[`gcamreport-temp/development_log/ERROR_LOG.md`](gcamreport-temp/development_log/ERROR_LOG.md).

## Working with submodules

Check the state of every nested repository with:

```bash
git submodule status --recursive
```

Commit changes in the repository where they were made, then update the parent
repository's gitlink:

```bash
cd gcamreport-fork
git add <files>
git commit -m "Describe the package change"
git push

cd ..
git add gcamreport-fork
git commit -m "Update gcamreport-fork"
git push
```

The parent repository tracks only each submodule's commit, not the files inside
that submodule.
