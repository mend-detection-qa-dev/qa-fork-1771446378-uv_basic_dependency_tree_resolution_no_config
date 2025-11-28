**Category:** Core Dependencies
**Priority:** P0
**UV Version:** All (0.1.0+)
**Complexity:** Simple

**Feature Dependencies:**
- Basic dependency resolution (see `research/uv_features.md#dependency-resolution`)
- Lock file parsing (see `research/uv_features.md#lock-file-management`)
- pyproject.toml PEP 621 compliance (see `research/uv_features.md#pep-621-support`)

**Inspired By:**
- GitHub Project: gustavocadev/example-fastapi-uv-ruff (see `research/github_projects.md#project-1`)
- Documentation: https://docs.astral.sh/uv/concepts/projects/
- Pattern: Simple dependencies with minimal transitive depth (see `research/patterns.md#pattern-1`)

**Real-World Relevance:** CRITICAL
- Found in 17/17 analyzed projects (100%)
- Core functionality required by all UV projects
- Blocks all dependency tree operations if broken

### Objective

Verify that the dependency tree builder can parse a basic UV project with standard PyPI dependencies and accurately resolve the complete dependency tree including all transitive dependencies.
