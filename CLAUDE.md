# CLAUDE.md

TheAlgorithms/Python is an educational collection of ~1,400 standalone algorithm implementations, one algorithm per file, grouped by topic. There is no app to run and nothing to build: each file is its own unit and its doctests are its tests. Clear and correct beats fast.

## Layout

- Topic directories at the root (`sorts/`, `graphs/`, `maths/`, `ciphers/`, ...). Every directory has an `__init__.py`; ruff's INP rule fails a directory without one. Put new files in an existing directory.
- `project_euler/problem_NNN/solN.py` (zero-padded to 3 digits): each has a `solution()` whose default arguments are the problem's input. Answers are checked by `scripts/validate_solutions.py` against SHA-256 hashes in `scripts/project_euler_answers.json`, never by doctests.
- `tests/test_sorts.py` runs every general comparison sort against one shared battery. A new general-purpose sort gets added to its `SORTS` tuple.
- `scripts/` is CI tooling (DIRECTORY.md builder, filename validator, PR-closing bots), not algorithms.

## Commands (from the repo root)

uv manages everything. `.python-version` is `3.15t` (free-threaded); uv downloads it on first run.

```bash
uv run --group=test pytest sorts/bubble_sort.py            # one file; pyproject already adds --doctest-modules
uv run python -m doctest -v sorts/bubble_sort.py           # same doctests, verbose
uv run --group=test --with=pytest-run-parallel pytest --iterations=8 --parallel-threads=auto --ignore-gil-enabled sorts/   # CI mode
uv run --group=test pytest scripts/validate_solutions.py -k problem_001   # check one Euler answer
uvx ruff check sorts/bubble_sort.py && uvx ruff format sorts/bubble_sort.py
uvx pre-commit run --files sorts/bubble_sort.py            # all hooks; first run takes ~5 min to install
```

## Conventions

- Every function has type hints and a docstring with doctests for valid and invalid input. New algorithms cite a Wikipedia or similar URL in the docstring. Expected outputs are pasted from a real run.
- Return results; don't `print`, plot, or call `input()` inside the algorithm.
- Bad input raises `ValueError`/`TypeError`. An f-string message can't go straight into `raise` (ruff EM102): write `msg = f"..."` then `raise ValueError(msg)`.
- Target is Python 3.15, so use PEP 695 generics (`def sort[T: Comparable](items: list[T]) -> list[T]`), `list[int]`, and `X | None`. ruff's UP rules flag the old forms.
- Files usually end with `if __name__ == "__main__":` plus `import doctest` and `doctest.testmod()`.
- Filenames are lowercase snake_case with no hyphens or spaces. Spell names out: `greatest_common_divisor`, not `gcd`.
- CI runs each test 8 times across threads on free-threaded Python, so don't keep mutable state at module level.

## Guardrails

- Don't run plain `uv run pytest`. The `test` group isn't installed by default, so uv falls through to whatever pytest is on PATH, which runs under a different Python. The pytest header should say Python 3.15.
- Don't use `uvx pytest` (AGENTS.md suggests it). It runs without the project's dependencies, so any file that imports numpy fails with ModuleNotFoundError.
- Don't run pytest on the whole repo. `digital_image_processing/` and some `computer_vision/` files need cv2, which isn't installed on 3.15t, so collection errors out. `project_euler/` is slow and `web_programming/` hits the network. Test the files you changed, or copy the `--ignore` list from `.github/workflows/build.yml`.
- Don't run `uv pip install .`: it's a flat layout with no build-system, so the build fails. `uv sync --group=test` is the setup command.
- Never hand-edit `DIRECTORY.md` or `docs/hacktober_2026_prep.md` (CI regenerates them) or `uv.lock` (the uv-lock hook regenerates it after you change dependencies in `pyproject.toml`).
- On Windows the `validate-filenames` pre-commit hook fails with exit 9009 because its `python3` shebang hits the Microsoft Store alias. Run `uv run python scripts/validate_filenames.py` instead.
- Keep a PR to one algorithm file, and never change code and its doctests in the same PR. The algorithms-keeper bot closes PRs where no "Describe your change" box is ticked.
