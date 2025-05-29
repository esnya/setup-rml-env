# Contributor Guidelines

## Coding Conventions
- Use cross-platform CLI commands and prefer built-in shell features.
- Adopt concise naming; variables and functions should be self-descriptive.
- Favor modern language features available in the project.

## Commit Messages
Follow Conventional Commit with gitmoji:
```
<type>[optional scope]: <emoji> <description>

[optional body]

[optional footer(s)]
```
Examples of `<type>` include `feat`, `fix`, `docs`, `chore`.

## Testing with GitHub Actions
The workflow `.github/workflows/build-test.yml` runs on pushes or pull requests to branches that start with `v`. It can also be triggered manually:
- From GitHub UI: **Actions → Test Setup Action → Run workflow**.
- With GitHub CLI: `gh workflow run build-test.yml`.

Add any further instructions or pending tasks here as needed.
