<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🐍 Python Project/Package Names

Extracts the Python project name and derives the Python package name.

## python-project-name-action

The action probes the project for metadata in the following order,
selecting the first present file (and, for `setup.cfg`, falling
through to `setup.py` when the file is present but contains no
`[metadata] name`):

1. `pyproject.toml` — `[project] name` (PEP 621, preferred)
2. `setup.cfg` — `[metadata] name` (legacy pbr / setuptools projects)
3. `setup.py` — `name="…"` literal (legacy setuptools)

If the chosen source is present but its name field is missing or
empty, and no later source remains to try, the action fails rather
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
