# Changelog

All notable changes to this project are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- Adopt the `palewire/python-open-source-template` development and agentic
  tooling: `AGENTS.md`, an expanded `Makefile`, PEP 735 `dependency-groups`,
  `ty` type checking, `deptry`, `zizmor`, `check-manifest`, and expanded
  pre-commit hooks (secrets/spelling/workflow-lint checks).
- Add CodeQL and OpenSSF Scorecard security-scanning workflows, issue/PR
  templates, and `CODEOWNERS`.

### Changed

- Replace `mypy` with [`ty`](https://github.com/astral-sh/ty) as the static
  type checker.

### Fixed

- Map the `multiple-columns` API type to `MultipleColumnChart` in `get_chart()`.
  The factory keyed the class under `d3-bars-split`, so real multiple-column
  charts fell back to `BaseChart` and split-bar charts raised a chart type
  mismatch instead of falling back.
- Keep chart settings that `BaseChart` does not model when a fetched chart is
  saved again. `serialize_model()` merged its section defaults over the raw API
  metadata one level deep, so an unchanged `get()` then `update()` reset
  `describe` number formatting (`number-format`, `number-divisor`,
  `number-prepend`, `number-append`), forced `visualize.sharing.auto` to
  `False` and replaced `publish.blocks` wholesale, dropping blocks such as
  `edit-in-datawrapper`. Sections are now merged deeply, with values set on the
  chart still taking precedence.
- Accept annotation numbers the Datawrapper editor stores as strings or
  decimals: `RangeAnnotation.opacity` (`"10"`) and `TextAnnotation.dx` and
  `dy` (`"3"`, `-29.5`). Charts with such annotations loaded with `get()` but
  failed validation when serialized again, so `update()` raised.
- Send each column's `number-format` in `ColumnFormatList`. The serializer
  dropped it along with the default values, so a column's number or date
  format was lost on create, and on every `update()` of a fetched chart.
- Read chart data separated by semicolons, tabs or pipes, not only commas.
  `Datawrapper.get()` parsed every CSV response with a comma, so charts with
  semicolon-separated data could not be fetched, and tab-separated data (what
  the editor stores when you paste from a spreadsheet) came back as a single
  column. `get()` and `BaseChart.deserialize_data()` now share a
  `read_csv()` helper that sniffs the delimiter from the first lines.
- Stop turning off the theme's logo and the team's footer settings on new
  charts. `serialize_model()` always sent `logo`, `get-the-data`,
  `download-*` and `embed` with the model defaults (all `False`), which
  override what the team and theme would otherwise apply. It now sends only
  the publish blocks that were set or read from the API.
- Name columns with a blank header `X.1`, `X.2`, ... as Datawrapper does,
  instead of pandas' `Unnamed: 1`, so a chart's settings that refer to them
  (`axes`, `column-format`) still match after `update()`.
- Keep the text of fetched data that `update()` uploads again. Codes with
  leading zeros (`05001`) became numbers, labels with spaces (`2025 `) lost
  them, so settings such as `highlighted-series` no longer matched, and cells
  reading `NA` or `N/A` were emptied. Such columns now stay text, and only
  empty cells are missing values.

### Removed

- Remove the unused `rich` and `exceptiongroup` runtime dependencies.
- Remove the `mypy` optional dependency group (superseded by the `ty` entry
  in the `dev` dependency group).

### Security
