# Maintainer notes: the planted failures

This repository contains deliberate faults. Each one is placed to fire at a
particular point in the course and teach one thing. They are listed here so
that a maintainer can find them without reading the whole package.

Nothing in this directory is visible to students working through the course,
but it is copied into their repository when they use the template, so do not
put anything here that gives the game away in a filename.

## 1. Sample data is not installed with the package

Fires on the first CI run, on every platform.

`src/ci_course/data/sample_measurements.csv` is not declared as package data,
so a non-editable install does not include it. The README tells students to
install with `pip install -e`, which exposes the source tree, so their tests
pass locally. The workflow runs `pip install .`, which does not, so five tests
fail with a `FileNotFoundError` naming a path inside `site-packages`.

The `src` layout is what makes this work: with a flat layout the source tree
would shadow the installed package and the tests would pass in CI too.

Fix:

```toml
[tool.setuptools.package-data]
ci_course = ["data/*.csv"]
```

## 2. Reports cannot be overwritten on Windows

Fires when the matrix is expanded across operating systems.

`write_report` in `src/ci_course/report.py` writes to a temporary file and then
calls `os.rename` to move it into place. On POSIX that silently replaces any
existing file. On Windows `os.rename` raises `FileExistsError` if the
destination exists, so `test_replaces_an_existing_report` fails on
`windows-latest` and passes on Ubuntu and macOS.

Fix: use `os.replace`, which has the POSIX behaviour everywhere.

Do not enable ruff's `PTH` rules on this repository. `PTH104` flags this exact
line, suggesting `Path.rename()`, which draws attention to the planted bug
before the matrix stage reaches it and proposes a fix that does not work:
`Path.rename` wraps `os.rename` and fails on Windows in the same way. The
working equivalent is `Path.replace`.

This is gated on the operating system rather than the Python version on
purpose. A failure that needs an old Python disappears when that version
leaves the matrix, which is how an earlier version of this exercise rotted.

Most students are on macOS or on WSL, so they will meet this in CI rather than
locally. A student on native Windows will see it on their own machine first.

## 3. A column of entirely missing values reports a mean of zero

Fires during the code coverage section.

`summarise` in `src/ci_course/stats.py` returns zeros when every value is
missing. No test covers that branch, so coverage points at it, and writing the
test to cover it shows the result is wrong: a mean of zero is a measurement
claim, not an absence of one.

Fix: return `nan`, or raise. Worth discussing rather than just patching.

Line 28, the standard deviation of a single value, is also uncovered and is a
second, milder, talking point.

## 4. Three lint findings

Fires on the first CI run, before the tests, because the lint step runs first.

- `src/ci_course/cli.py`: `import sys` is unused (F401).
- `src/ci_course/io.py`: `errors=[]` is a mutable default argument (B006). It
  accumulates across calls. The sample data contains no unparseable rows, so
  this never bites in testing: the linter finds a bug the tests cannot.
- `src/ci_course/io.py`: a bare `except` (E722).

## 5. A public function with no docstring

Fires during the documentation section.

`count_missing` in `src/ci_course/stats.py` is exported, tested, and has no
docstring. `ruff check --select D src/` reports exactly one finding, D103,
naming the function and line.

The point is not the failing check. Sphinx `automodule` only documents members
that have a docstring, so `count_missing` is silently absent from the generated
API reference: a public function that the documentation does not mention, with
nothing anywhere to say so. Build the docs before and after fixing it and the
function appears.

Fix: write the docstring, in the same numpydoc style as the rest of the
package.

This is the documentation counterpart to the code coverage stage before it.
Code coverage asks which lines the tests run, this asks which public things
are documented, and neither says the thing it measures is any good. Worth
saying out loud twice.

Two details that matter if you change the configuration:

- Scope the command to `src/`. Run over the whole repository it reports 18
  findings, because the tests have no docstrings and do not need any.
- Keep `convention = "numpy"` set in `pyproject.toml`. It is what makes the
  full `D` set usable, by disabling the rules that contradict numpydoc. Without
  it the same command reports 15 findings here, 14 of them blank line
  placement in docstrings that are perfectly good.
- The stage therefore checks both that a docstring exists and that it is
  shaped like the others. A student who answers the challenge with a one line
  docstring is told what a numpydoc one needs.

## 6. Optional: the version matrix and `requires-python`

`pyproject.toml` declares `requires-python = ">=3.12"`. Adding 3.11 to the
version matrix makes `pip install` refuse the package with a clear error. Both
halves live in this repository, so this cannot silently start passing, but it
does need bumping as supported versions move on.
