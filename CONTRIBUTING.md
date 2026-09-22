# Contributing to SkyWarden

Thank you for your interest in contributing to SkyWarden! We welcome contributions ranging from bug fixes and documentation enhancements to model training optimizations and new UI features.

---

## 1. Code of Conduct

We are committed to providing a friendly, safe, and welcoming environment for all contributors, regardless of experience level, background, or identity. Please maintain professional, respectful, and constructive communication in issues, PRs, and discussions.

---

## 2. Getting Started

### 2.1 Fork & Clone
1. Fork the repository to your GitHub profile.
2. Clone your fork locally:
   ```bash
   git clone https://github.com/<your-username>/SkyWarden.git
   cd SkyWarden
   ```
3. Set the upstream remote:
   ```bash
   git remote add upstream https://github.com/ashishsps20/SkyWarden.git
   ```

### 2.2 Environment Setup
Create and activate an isolated virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

---

## 3. Branching Strategy

Always develop on a dedicated feature or bugfix branch branched from `main`:

| Prefix | Usage | Example |
|---|---|---|
| `feature/` | New capabilities, models, or UI features | `feature/onnx-export` |
| `bugfix/` | Resolving an existing defect or regression | `bugfix/class-label-typo` |
| `docs/` | Documentation, guides, or docstring updates | `docs/architecture-diagrams` |
| `refactor/` | Code refactoring without changing functionality | `refactor/modularize-app` |
| `test/` | Adding or updating unit/integration tests | `test/preprocessing-tests` |

Example:
```bash
git checkout -b feature/dynamic-threat-config
```

---

## 4. Commit Message Guidelines

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<scope>): <short description>

[optional body explaining context and motivation]

[optional footer referencing issues, e.g., Closes #12]
```

### Allowed Types
- `feat`: A new feature or capability
- `fix`: A bug fix
- `docs`: Documentation changes only
- `style`: Changes that do not affect the meaning of the code (whitespace, formatting, missing semicolons)
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: A code change that improves performance
- `test`: Adding missing tests or correcting existing tests
- `chore`: Maintenance tasks, dependencies, build scripts

---

## 5. Coding Standards

- **Python Style Guide:** Follow [PEP 8](https://peps.python.org/pep-0008/).
- **Formatting:** Code should be formatted with `black` or `ruff` (line length 100).
- **Type Annotations:** Use Python type hints (`typing.Optional`, `typing.List`, `typing.Tuple`, etc.) for function signatures where practical.
- **Documentation:** All new functions and modules should include descriptive docstrings explaining inputs, return values, and expected exceptions.
- **Paths:** Avoid hardcoded OS-specific paths (e.g., `C:\...`). Always use Python's `pathlib.Path` or `os.path.join` for cross-platform compatibility.

---

## 6. Pull Request Submission Checklist

Before submitting your pull request, ensure:

- [ ] Branch is up to date with `upstream/main`.
- [ ] Code runs without syntax errors or unhandled exceptions.
- [ ] Any newly introduced dependencies are documented in `requirements.txt`.
- [ ] Associated documentation (`README.md`, `docs/`) has been updated where relevant.
- [ ] The PR description clearly explains the changes and references any open issues.
