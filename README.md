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
