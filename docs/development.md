# Develop bt

Use Python 3.11 for the development and documentation tools. bt's runtime still supports Python 3.9 and later. Install [uv](https://docs.astral.sh/uv/) and a C compiler for the Cython extension, then create an environment:

```bash
uv venv --python 3.11
source .venv/bin/activate
make develop
make lint
make checks
make coverage
make build
```

On Windows, activate with `.venv\Scripts\activate` instead. Run `make help` for available targets. `make test` runs tests without coverage; `make benchmark` runs the separate backtest benchmarks. Type checking (`make check-types`) is advisory and not a CI gate.

Coverage measures the Python modules. The compiled `bt.core` extension is still tested, but excluded from coverage because it is built without Cython line tracing.

## Build the documentation

```bash
make docs-develop
make docs
make serve
```

Open <http://localhost:9087>. Yardang reads `[tool.yardang]` in `pyproject.toml` and builds `docs/source` into `docs/html`, using the installed Klink theme. The old Sphinx Makefiles, hand-maintained `conf.py`, vendored theme, and manual `make pages` deployment are no longer used.

`docs/requirements.txt` temporarily pins Yardang's existing-source support to a Git commit. That prerequisite must be available remotely before CI can install it. For local review with the updated Yardang checkout symlinked as `yardang`, use `uv pip install -e ./yardang klink==0.1.10` instead of `make docs-develop`.

Edit the existing reStructuredText pages in `docs/source`. Notebook examples use their checked-in reStructuredText exports and images; documentation builds do not execute notebooks or fetch market data. After editing notebooks, regenerate the exports with Klink's conversion helper:

```bash
uv pip install nbconvert
cd docs/source
python -c 'import klink; klink.convert_notebooks()'
```

This converts saved notebook outputs without executing cells, copies images into `_static`, and updates image paths and Klink CSS classes. Review and commit the changed notebooks, exports, and images together.

Pull requests build documentation and upload an HTML artifact. Successful pushes to `master` also publish to the existing `gh-pages` branch. Local builds never publish.

## Update the Copier template

`.copier-answers.yaml` records the Python variant of [python-project-templates/base](https://github.com/python-project-templates/base) and the exact template revision. From a clean branch, update with:

```bash
copier update --trust
```

Review the resulting diff and resolve conflicts before running the checks above. The Python Templates Copier Update GitHub App can propose updates automatically; installing that app is separate from this repository change.

bt intentionally retains its MIT license, runtime Python floor, package metadata and version, Cython build hook, native wheel matrix, top-level `tests` directory, benchmarks, and Ruff line length. Documentation retains its existing reStructuredText URLs and Klink styling instead of adopting the template's Markdown homepage and default theme. The docs workflow builds from source because bt's CI produces platform-specific wheel artifacts.
