<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🐍 Python Project/Package Names

> [!WARNING]
> Deprecated: this action gets no further development. The maintainers
> will archive this repository; references to its existing tags and
> commits keep working after that.
> [`lfreleng-actions/build-metadata-action`](https://github.com/lfreleng-actions/build-metadata-action)
> replaces it: see
> [Migrating to build-metadata-action](#migrating-to-build-metadata-action).

Extracts the Python project name and derives the Python package name.

## python-project-name-action

The action probes the project for metadata in the following order,
selecting the first source that yields a project name. The action
skips a present source that carries no usable name and continues with
the next one:

1. `pyproject.toml` — `[project] name` (PEP 621, preferred), or
   `[tool.poetry] name` when the file has no `[project] name`
2. `setup.cfg` — `[metadata] name` (legacy pbr / setuptools projects)
3. `setup.py` — the literal `name` keyword passed to `setup()`
   (legacy setuptools)

The action parses `pyproject.toml` with `tomllib` and reads `setup.py`
with the `ast` module, so any valid quoting or spacing works, and a
`name` in an unrelated TOML table (e.g. `[[tool.uv.index]]`) or an
identifier such as `package_name` does not count. For `setup.py`, the
action resolves imports and accepts a `setup()` call from a packaging
module (`setuptools`, `distutils.core`, `numpy.distutils.core` or
`skbuild`) under any alias; a call such as `helper.setup(name="x")`
does not count. A `setup.py` name that is not a string literal (e.g.
`name=NAME`) yields no name.

The action runs these parsers with a Python 3.11 or newer interpreter
it sets up itself through `actions/setup-python`, rather than whichever
`python3` comes first on the caller's `PATH`, and leaves the caller's
`PATH` and environment unchanged. Where a parser cannot read the file
(e.g. a `setup.py` in Python 2 syntax), or the action cannot set up
that interpreter or run it in the job's container, a best-effort line
match stands in, never the caller's `python3`. That match reads the
same TOML tables for `pyproject.toml`, and takes the first quoted
whole-word `name=` in `setup.py`. For every source, the action
rejects any name holding characters other than letters, digits, `.`,
`_` and `-`, and skips a file that holds a raw NUL byte, which none
of the three formats allows.

This tolerates a `pyproject.toml` that contains a `[build-system]`
table (PEP 517) but no `[project] name` — the common pbr / setuptools
layout — by falling through to `setup.cfg` and then `setup.py` rather
than aborting. If no source yields a name, the action fails rather
than emitting an empty value.

The `setup.cfg` and `setup.py` paths exist to support legacy projects
(e.g. those still using `pbr`) that have not migrated to PEP 621
metadata in `pyproject.toml`.

## Usage Example

An example workflow step using this action:

```yaml
steps:
    # Code checkout performed in earlier step
    - name: Fetch Python project/package name
      uses: lfreleng-actions/python-project-name-action@main
```

##  Inputs

<!-- markdownlint-disable MD013 -->

| Variable Name       | Required | Description                           |
| ------------------- | -------- | ------------------------------------- |
| path_prefix         | False    | Path/directory to Python project code |

<!-- markdownlint-enable MD013 -->

## Outputs

<!-- markdownlint-disable MD013 -->

| Output Variable       | Mandatory | Value                                                            |
| --------------------- | --------- | ---------------------------------------------------------------- |
| `python_project_file` | Yes       | File used to extract metadata: pyproject.toml/setup.cfg/setup.py |
| `python_project_name` | Yes       | Extracted from pyproject.toml, setup.cfg or setup.py             |
| `python_package_name` | Yes       | Derived from the project name (dashes become underscores)        |
| `source`              | Yes       | Source file used (pyproject.toml, setup.cfg or setup.py)         |

<!-- markdownlint-enable MD013 -->

## Migrating to build-metadata-action

Use `lfreleng-actions/build-metadata-action` v0.13.5 or later. Earlier
releases can take a `setup.py` name from the wrong identifier
([build-metadata-action#227](https://github.com/lfreleng-actions/build-metadata-action/issues/227)).

Replace the step in the usage example above with:

<!-- markdownlint-disable MD013 -->

```yaml
- name: Fetch Python project/package name
  id: metadata
  uses: lfreleng-actions/build-metadata-action@0000000000000000000000000000000000000000  # REPLACE: vX.Y.Z
  with:
    project_type: 'python'
    use_version_extract: 'false'
    include_environment: 'false'
    artifact_upload: 'false'
    output_format: ''
    python_offline_mode: 'true'
```

<!-- markdownlint-enable MD013 -->

Pin the action to the commit SHA of the release you choose, in place of
the placeholder. Setting `project_type` stops a `package.json` or
`go.mod` beside the Python files from naming another project. The other
five inputs keep the step as quiet as this action: no version lookup,
no probe of the runner, no uploaded artefact, no job summary and no
query for current Python releases. Add `path_prefix` where the project
does not sit at the repository root.

The step sets up Go and builds the action's binary on each run, so
expect it to take longer than this action did.

### Output mapping

<!-- markdownlint-disable MD013 -->

| This action           | `build-metadata-action`  |
| --------------------- | ------------------------ |
| `python_project_name` | `project_name`           |
| `python_package_name` | `python_package_name`    |
| `python_project_file` | `python_metadata_source` |
| `source`              | `python_metadata_source` |

<!-- markdownlint-enable MD013 -->

### Differences

1. **The package name.** This action derives `python_package_name` by
   replacing dashes in the project name with underscores. In
   `build-metadata-action`, `python_package_name` keeps the name as
   the project declares it: `my-cool-pkg` stays `my-cool-pkg`.
2. **A project with no name.** This action fails, while
   `build-metadata-action` succeeds and leaves `project_name` empty. A
   caller that needs a name has to check for one.
3. **Characters in the name.** This action rejects a name holding any
   character other than letters, digits, `.`, `_` and `-`, then tries
   the next file or fails. `build-metadata-action` reports the name as
   the project declares it: `name = "bad name!"` gives `bad name!`. A
   caller that relied on that check has to make it itself, and should
   pass the output to a script through `env:`, never by expanding it
   inside a `run:` block.
