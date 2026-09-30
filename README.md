<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🐍 Python Project/Package Names

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

Without `python3`, or where it cannot parse the file (`tomllib` needs
Python 3.11+), a best-effort line match stands in: scoped to the
same TOML tables for `pyproject.toml`, and taking the first quoted
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
| PATH_PREFIX         | False    | Path/directory to Python project code |

<!-- markdownlint-enable MD013 -->

## Outputs

<!-- markdownlint-disable MD013 -->

| Output Variable       | Mandatory | Value                                                            |
| --------------------- | --------- | ---------------------------------------------------------------- |
| `python_project_file` | Yes       | File used to extract metadata: pyproject.toml/setup.cfg/setup.py |
| `python_project_name` | Yes       | Extracted from pyproject.toml, setup.cfg or setup.py             |
| `python_package_name` | Yes       | Derived from the project name (dashes become underscores)        |
| `source`              | Yes       | Source file used (pyproject.toml, setup.cfg or setup.py)         |

The action also exports the same values as environment variables
(lowercase names) for use in later steps within the same job.

<!-- markdownlint-enable MD013 -->
