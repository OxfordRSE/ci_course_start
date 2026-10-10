# OxRSE Continuous Integration course

This repository contains a small Python package for summarising experimental
measurements. Over the course you will use free cloud services to automate:

- unit testing on multiple Python versions
- unit testing on multiple operating systems
- static analysis
- coverage testing
- documentation generation and deployment

## The package

```text
src/ci_course/
    io.py       read measurements from a CSV file
    stats.py    summarise a column of measurements
    report.py   write a summary report
    cli.py      the ci-course command
tests/
```

## Setting up

To make sure all dependencies are installed, we recommend creating a new
virtual environment. From the directory containing this file:

```bash
python3 -m venv venv
```

Activate the virtual environment:

Linux / macOS:

```bash
source venv/bin/activate
```

Windows cmd.exe:

```bash
venv\Scripts\activate.bat
```

Windows PowerShell:

```bash
venv\Scripts\Activate.ps1
```

Windows using Git Bash:

```bash
source venv/Scripts/activate
```

Upgrade the build tools and install this project:

```bash
pip install --upgrade pip setuptools wheel
pip install -e ".[dev,docs]"
```

The `-e` matters: it installs the project in editable mode, so that the
package you import is the source tree you are editing.

## Checking it works

```bash
pytest          # run the tests
ruff check .    # run the linter
ci-course       # summarise the bundled sample data
```
