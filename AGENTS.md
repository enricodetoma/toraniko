# AGENTS.md

## Project Overview

Toraniko is a Python library for estimating characteristic multi-factor equity risk models. It uses NumPy for numerical routines and Polars for tabular and lazy query operations. The supported Python range is 3.10 through 3.x.

The main package is `toraniko/`:

- `model.py`: cross-sectional factor-return and residual-return estimation.
- `styles.py`: momentum, size, and value factor-score construction.
- `math.py`: cross-sectional normalization, winsorization, exponential weights, and covariance helpers.
- `utils.py`: Polars data cleaning, smoothing, universe selection, and dummy-variable helpers.
- `main.py`: the higher-level `FactorModel` orchestration API.
- `config.py` and `config.ini`: user configuration loading and defaults.
- `meta.py`: API lifecycle decorators.
- `tests/`: the pytest suite.

Read the public examples and expected input schemas in `README.md` before changing user-facing behavior.

## Setup and Commands

Run commands from the repository root unless noted otherwise.

```bash
python -m pip install -r dev-requirements.txt
python -m pytest -q
```

Useful focused test commands:

```bash
python -m pytest toraniko/tests/test_model.py -q
python -m pytest toraniko/tests/test_math.py -q
python -m pytest toraniko/tests/test_utils.py -q
```

Formatting and lint configuration lives in `pyproject.toml`. The project uses a 120-character line length.

```bash
python -m black --check toraniko
python -m ruff check toraniko
```

Do not assume Black or Ruff is installed by `dev-requirements.txt`; install tooling separately when needed.

## Implementation Conventions

- Keep the package dependency-light. Do not add a runtime dependency when NumPy, Polars, or the standard library is sufficient.
- Preserve the existing public function signatures and default column names unless the change explicitly requires an API break.
- Use type hints and NumPy-style docstrings for public functions.
- Prefer Polars expressions and lazy operations over row-wise Python loops or conversion to pandas.
- Functions accepting both `pl.DataFrame` and `pl.LazyFrame` should preserve their documented return behavior. Test both input forms where applicable.
- Validate required columns and argument ranges near the public API boundary. Match the existing exception style when extending nearby code.
- Keep imports package-absolute (`from toraniko...`) and avoid import-time work.
- Add comments only for non-obvious numerical reasoning or data-shape constraints.

## Numerical and Data Rules

- Treat `date` and `symbol` as the default observation keys. Check sort order and join cardinality when modifying time-series or cross-sectional operations.
- Maintain deterministic column ordering for factor exposures and returned factor series.
- Preserve null and NaN semantics deliberately. Polars nulls and floating-point NaNs are not interchangeable; several helpers normalize between them explicitly.
- Do not replace rank-deficiency handling in factor estimation with a plain unconstrained inverse. The market factor is spanned by sector exposures, so the sector-return constraint is part of the model definition.
- Market-cap weighting, sector constraints, style residualization, winsorization, and residual-return calculation must remain aligned to the same asset rows.
- Avoid silent look-ahead bias in rolling features. Momentum, smoothing, and covariance changes must respect lags, window boundaries, and per-symbol ordering.
- Exercise empty, all-NaN, singular, and undersized-window inputs when changing numerical helpers. Runtime warnings for explicitly tested empty/all-NaN winsorization currently do not fail the suite.
- Use tolerant numerical assertions (`numpy.testing` or Polars testing helpers) for floating-point results; use exact assertions only when exactness is meaningful.

## Tests

- Add or update focused tests with every behavioral change.
- Put tests alongside the relevant module suite under `toraniko/tests/`.
- Cover eager and lazy Polars inputs when the API supports both.
- For model changes, test output shapes and labels as well as numerical values and model identities or constraints.
- For bug fixes, include a regression test that fails without the fix.
- Run the full suite before finishing. The current baseline is 74 passing tests.

## Configuration and Packaging

- Keep versions synchronized between `pyproject.toml`, `setup.py`, and `toraniko/__init__.py` when making a release change.
- Keep runtime requirements synchronized between `pyproject.toml` and `requirements.txt`.
- `toraniko/config.ini` is packaged as the sample copied by the `toraniko-init` entry point. Update parsing, defaults, documentation, and tests together when its schema changes.
- Do not create or overwrite a user's `~/.toraniko/config.ini` during tests.

## Scope and Safety

- Keep changes narrow; some higher-level orchestration and covariance features are intentionally marked TODO or unimplemented.
- Do not implement unrelated TODOs as part of another fix.
- Do not commit generated caches, local configuration, environments, or build artifacts.
- Preserve unrelated working-tree changes and never rewrite user data as part of tests or examples.
