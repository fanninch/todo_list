# Todo List CLI — Monthly Developer Project

Welcome to the Todo List CLI challenge designed for developers. This project guides you through creating a real-world, testable command-line application, iterating on it over a month, and releasing an installable Python package (.whl) at the end.

At the beginning of the month, you will receive this README containing the requirements and expectations. Build your implementation in your own repository. At the end of the month, a reference solution will be published.

---

## Project Overview
You will build a command-line todo application that can manage multiple named todo lists. The tool should allow you to:
- Create any number of todo lists.
- Create/delete lists.
- Add items to a list.
- Mark items as completed on a list.
- Remove items from a list.
- Retrieve and display an entire list.
- Retrieve and display completed tasks from a list.
- Retrieve and display items finished on a list (same as completed tasks; include an explicit command for it).
- Persist data on disk so lists and items survive between runs.
- Package the project into a pip-installable wheel (.whl) with a console entry point.

Data persistence must use JSON files on disk. The on-disk format must be deterministic and stable across runs and machines.

---

## Learning Goals
- Practice designing a small but complete CLI tool with clear user experience.
- Gain experience with file I/O (JSON) for persistence.
- Learn basic Python packaging to produce a .whl and expose a console script.
- Write maintainable code with structure, docstrings, and helpful error messages.
- Write unit tests with pytest and target at least 80% code coverage.

---

## Functional Requirements (Must-Haves)
1. Multiple lists
   - Users can create, list, and delete named todo lists (e.g., "work", "home").
2. Item management
   - Add a new item to a specified list with a human-readable description.
   - Remove an item by ID or index.
   - Mark an item as completed by ID or index.
3. Retrieval
   - Display an entire list with item statuses.
   - Display only completed items for a list.
   - Display items finished on a list (alias of completed items, but provide a separate command name).
4. Persistence
   - All lists and items must be saved on disk and reloaded on application start.
   - JSON must be used as the on-disk format. The format should be deterministic and stable across runs.
5. Packaging
   - The project must build into a wheel (.whl) and be pip-installable.
   - Installation should provide a CLI executable (e.g., `todo`) on PATH.

---

## Non-Functional Requirements
- The CLI must exit with non-zero status codes on errors and print informative messages.
- Commands and flags should be discoverable via `--help`.
- Avoid destructive actions without confirmation or a `--yes`/`-y` flag.
- The code should be organized into modules (not a single monolithic file).
- Include basic type hints and docstrings for public functions.

---

## Using AI Assistance
Developers are encouraged to use AI assistants to help with:
- Writing individual functions or snippets (e.g., parsing, formatting, small utilities).
- Drafting or refining docstrings and inline documentation.
- Discovering which packages or standard-library modules to use for a given task.

Important limitation:
- Do not use advanced AI tools (e.g., Cursor, Junie) to design the project itself. Architectural decisions, overall CLI design, data models, persistence layout, and test strategy must be your own work.

Suggested practice:
- Treat the AI like a coding copilot or reference librarian. Keep design and reasoning in your own hands, but feel free to ask for small implementations or examples.

---

## Suggested CLI Specification
Use a root command named `todo` (or `todolist`). Below is a suggested interface you may copy or adapt.

- `todo init`  
  Initialize the data store (create the folder/file or database on first run). Safe to run multiple times.

- `todo list-lists`  
  Show all existing list names.

- `todo create-list <list_name>`  
  Create a new list.

- `todo delete-list <list_name> [-y|--yes]`  
  Delete a list (ask for confirmation unless `--yes` is provided).

- `todo add <list_name> <description>`  
  Add a new item with the given description to the list. Returns the new item ID.

- `todo remove <list_name> <item_id_or_index>`  
  Remove a specific item from the list.

- `todo completed <list_name> <item_id_or_index>`  
  Mark a specific item as completed.

- `todo show <list_name> [--all|--pending|--completed]`  
  Display items in a list. Default is `--all`.

- `todo show-completed <list_name>`  
  Display only completed items for a list.

- `todo finished <list_name>`  
  Display items finished on a list (explicit alias for completed items).

- `todo version`  
  Print the application version.

Tips:
- Use stable IDs per item (e.g., an integer counter per list). Showing indices in `show` is helpful.
- Support `--json` output if you want to enable scripting (optional).

---

## Data Persistence
You must use JSON as the on-disk format. A simple, beginner-friendly approach is:
- JSON files stored in a user data directory, e.g.,
  - Windows: `%APPDATA%/todo_list` or a folder in the project directory during development.
  - Linux/macOS: `~/.local/share/todo_list` or similar.
- One `lists.json` index mapping list names to metadata and file paths.
- One JSON file per list, e.g., `work.json` with a structure like:
  ```json
  {
    "name": "work",
    "next_id": 7,
    "items": [
      {"id": 1, "text": "Email team", "completed": true, "created_at": "2025-10-01T08:10:00Z", "completed_at": "2025-10-01T09:00:00Z"},
      {"id": 2, "text": "Write report", "completed": false, "created_at": "2025-10-01T08:15:00Z", "completed_at": null}
    ]
  }
  ```

---

## Project Structure (suggested)
```
project_root/
  todo_list/
    __init__.py
    cli.py          # argparse or click entry points
    storage.py      # file persistence (JSON)
    models.py       # domain models, dataclasses
    services.py     # operations on lists & items
    version.py      # __version__ = "0.1.0"
  tests/            # required — include comprehensive pytest unit tests
  pyproject.toml    # packaging config (recommended)
  README.md
  LICENSE
```

---

## Packaging and Installation
You must be able to build a wheel and install it locally.

Option A: pyproject.toml (recommended)
- Use PEP 621 with setuptools:
  - pyproject.toml (minimal example):
    ```toml
    [build-system]
    requires = ["setuptools>=68", "wheel"]
    build-backend = "setuptools.build_meta"

    [project]
    name = "todo-list-cli"
    version = "0.1.0"
    description = "A simple CLI todo manager supporting multiple lists"
    authors = [{name = "Your Name"}]
    readme = "README.md"
    requires-python = ">=3.9"
    classifiers = [
      "Programming Language :: Python :: 3",
      "License :: OSI Approved :: MIT License",
      "Operating System :: OS Independent",
    ]
    dependencies = []

    [project.scripts]
    todo = "todo_list.cli:main"
    ```

Build and install locally:
- Create a virtual environment (optional but recommended).
- Build: `python -m build` or `python -m pip wheel .` or `python -m pip install build && python -m build`.
- Install: `pip install dist/todo_list_cli-0.1.0-py3-none-any.whl` (adjust filename as produced).

Option B: setup.cfg/setup.py
- You can alternatively use setup.cfg/setup.py with `entry_points = {"console_scripts": ["todo=todo_list.cli:main"]}` and build with `python -m build` or `python setup.py bdist_wheel`.

After install, ensure the `todo` command is available: `todo --help`.

---

## Example Usage
Below are illustrative examples of how your CLI could behave.

- Initialize storage:
  - `todo init`

- Create lists:
  - `todo create-list work`
  - `todo create-list home`

- Add items:
  - `todo add work "Email team about Q4 goals"`
  - `todo add home "Buy groceries"`

- Show list:
  - `todo show work`

- Mark completed:
  - `todo completed work 1`

- Show only completed (two ways):
  - `todo show-completed work`
  - `todo finished work`

- Remove an item:
  - `todo remove home 2`

- Delete a list:
  - `todo delete-list home --yes`

---

## Milestones (Suggested Month Plan)
- Week 1: Project scaffold, packaging config, `todo init`, data folder creation, `list-lists`, `create-list`.
- Week 2: Add `add`, `show`, basic persistence.
- Week 3: Add `completed`, `remove`, `show-completed`, `finished` alias, confirmations and error codes.
- Week 4: Polish UX, help messages, tests (optional), finalize packaging, produce wheel, README polish.

---

## Acceptance Criteria
- All functional requirements above are implemented.
- Data is persisted on disk and reloaded correctly.
- The wheel builds successfully and installs via pip.
- The `todo` command runs and supports `--help` with documented commands.
- README includes usage examples and packaging instructions.
- Comprehensive unit tests are present with at least 80% line coverage across the codebase.

---

## Stretch Goals (Optional)
- Add `--json` output mode for commands that display data.
- Support priorities, due dates, tags, and filtering.
- Implement simple unit tests for core operations.
- Add `export`/`import` commands.

---

## Getting Started (Quick Setup)
1. Clone your project repository.
2. Create and activate a virtual environment.
3. Implement the CLI according to this README.
4. Build the wheel and install it locally.
5. Verify commands with `todo --help` and the examples above.

Good luck, and have fun building! 🎯



---

## Testing
Developers must create comprehensive unit tests using pytest and target a minimum of 80% line coverage.

Recommended commands:
- Run tests: `pytest -q`
- Measure coverage with coverage.py:
  - `python -m pip install coverage`
  - `coverage run -m pytest`
  - `coverage report -m` (aim for >= 80%)

Alternatively, using pytest-cov:
- `python -m pip install pytest-cov`
- `pytest --cov=todo_list --cov-report=term-missing`

Guidelines:
- Place tests under the `tests/` directory.
- Use fixtures and monkeypatching to isolate filesystem and environment (e.g., set `APPDATA` to a temp directory).
- Test both success paths and error handling (non-zero exit codes, user messages, and logging hints).
- Keep tests deterministic: avoid relying on actual timestamps beyond format validation.
