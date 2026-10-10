# Maintainer notes: exercise sequence

These notes give teachers and TAs an overview of the student's progression, the planted faults and their fixes.
Detailed student instructions, including how to inspect logs, belong in the practical and lecture.

## 1. Lint findings

### What a student should observe

On the first CI run, lint fails before the tests run:

- `src/ci_course/cli.py`: unused `import sys` (F401).
- `src/ci_course/io.py`: mutable default `errors=[]` (B006).
- `src/ci_course/io.py`: bare `except` (E722).

The shared default list accumulates errors across calls, but the sample data contains no unparseable rows, so the tests miss this bug.

### What a student needs to do to fix it

Remove the unused import and catch parsing errors with `except ValueError:`.
Give each call its own default error list:

```python
def load_measurements(path=None, errors=None):
    if errors is None:
        errors = []
```

## 2. Sample data missing from the installed package

### What a student should observe

Once lint passes, five tests fail with `FileNotFoundError` for the sample CSV inside `site-packages`.
The editable local install passes because it exposes the source data; CI's non-editable install omits it.
The `src` layout ensures the tests exercise the installed package.

### What a student needs to do to fix it

Include the CSV in the package by adding this to `pyproject.toml`:

```toml
[tool.setuptools.package-data]
ci_course = ["data/*.csv"]
```

## 3. Reports cannot be overwritten on Windows

### What a student should observe

When the OS matrix is introduced, `test_replaces_an_existing_report` fails with `FileExistsError` on Windows but passes on Ubuntu and macOS.
Native Windows users may already have seen this locally.
`os.rename` replaces an existing destination on POSIX but fails on Windows.

### What a student needs to do to fix it

In `write_report` in `src/ci_course/report.py`, replace `os.rename(tmp, path)` with:

```python
os.replace(tmp, path)
```

Keep this fault tied to the OS so changes to the Python matrix cannot make it disappear.

## 4. Missing values reported as a mean of zero

### What a student should observe

Coverage reveals an untested branch in `summarise` in `src/ci_course/stats.py`.
A test for entirely missing values exposes summary statistics of zero, incorrectly implying measured values exist.

### What a student needs to do to fix it

Add a test for the missing-values case and make the empty-data branch return `nan` for the statistics or raise an appropriate exception.
Keep `n` as zero if returning a summary.

## 5. A public function with no docstring

### What a student should observe

During the documentation section, `ruff check --select D src/` reports D103 for `count_missing` in `src/ci_course/stats.py`.
The function is exported and tested but absent from Sphinx's generated API reference.

### What a student needs to do to fix it

Add a docstring in the package's numpydoc style and rebuild the documentation to see the function appear.
Like code coverage, this check establishes completeness without judging quality.

## 6. Documentation previews

### What a student should observe

A successful documentation build does not establish that the rendered text, formatting and links are correct.
Published Pages documentation reflects `main`, so it does not show proposed PR changes.

### What a student needs to do to fix it

Use `docs_preview.yml` to upload the PR's rendered Sphinx HTML as `documentation-preview` for review before merging.
The Pages workflow publishes after merging.
The later gated deployment exercise reuses built HTML in a separate job after approval.

## 7. Caching and concurrency

### What a student should observe

Repeated runs download the same dependencies, and successive commits can leave obsolete CI runs active.
Timing differences will be small here; the practical and lecture should explain that larger downloads or expensive builds can make savings significant.

### What a student needs to do to fix it

Enable pip caching in the `setup-python` step:

```yaml
cache: pip
cache-dependency-path: pyproject.toml
```

Cancel superseded runs with workflow-level concurrency:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

The Python versions, Operating systems, Coverage and Docstrings workflows provide the reference examples.

## Maintaining the reference files

The workflows and documentation in `.cheat/` are maintained by hand, with no automated check against the practical.
When commands, job names, dependencies or faults change, update the matching practical examples and notes here.
Keep solutions out of the student README.
