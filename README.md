## Test T-P1-001: Basic Dependency Tree Resolution

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

### Setup

1. Create directory: `fixtures/phase_1/basic_dependency_tree_resolution/`
2. Create `pyproject.toml` with basic dependencies
3. Run `uv lock` to generate `uv.lock`
4. Run dependency tree builder on the fixture

### Input Files

**Location:** `fixtures/phase_1/basic_dependency_tree_resolution/`

**pyproject.toml:**
```toml
[project]
name = "test-basic-deps"
version = "0.1.0"
description = "Basic dependency tree test"
requires-python = ">=3.8"
dependencies = [
    "requests>=2.31.0",
    "click>=8.1.0",
    "pydantic>=2.0.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

**Expected uv.lock structure (partial):**
```toml
version = 1
requires-python = ">=3.8"

[[package]]
name = "requests"
version = "2.31.0"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "certifi" },
    { name = "charset-normalizer" },
    { name = "idna" },
    { name = "urllib3" },
]

[[package]]
name = "certifi"
version = "2024.7.4"
source = { registry = "https://pypi.org/simple" }

[[package]]
name = "charset-normalizer"
version = "3.3.2"
source = { registry = "https://pypi.org/simple" }

[[package]]
name = "click"
version = "8.1.7"
source = { registry = "https://pypi.org/simple" }

[[package]]
name = "pydantic"
version = "2.7.1"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "annotated-types" },
    { name = "pydantic-core" },
    { name = "typing-extensions" },
]

# ... additional transitive dependencies
```

### Expected Behavior

The dependency tree builder should:

1. **Parse pyproject.toml:**
   - Extract project name: "test-basic-deps"
   - Extract version: "0.1.0"
   - Extract Python requirement: ">=3.8"
   - Identify 3 direct dependencies: requests, click, pydantic

2. **Parse uv.lock:**
   - Extract all package entries
   - Map dependency relationships
   - Preserve version information
   - Identify source registries

3. **Build Dependency Tree:**
   - Create root node for project
   - Add direct dependencies as first-level children
   - Add transitive dependencies as subsequent levels
   - Calculate total dependency count
   - Calculate maximum depth

4. **Classify Dependencies:**
   - Mark requests, click, pydantic as "direct" + "runtime"
   - Mark all others as "transitive" + "runtime"
   - Map "required_by" relationships for transitive deps

**Expected Output Structure:**

```json
{
  "project": {
    "name": "test-basic-deps",
    "version": "0.1.0",
    "python_version": ">=3.8"
  },
  "dependencies": {
    "direct": [
      {
        "name": "requests",
        "version": "2.31.0",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "dependencies": ["certifi", "charset-normalizer", "idna", "urllib3"]
      },
      {
        "name": "click",
        "version": "8.1.7",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "dependencies": []
      },
      {
        "name": "pydantic",
        "version": "2.7.1",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "dependencies": ["annotated-types", "pydantic-core", "typing-extensions"]
      }
    ],
    "transitive": [
      {
        "name": "certifi",
        "version": "2024.7.4",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "required_by": ["requests"],
        "depth": 2
      },
      {
        "name": "charset-normalizer",
        "version": "3.3.2",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "required_by": ["requests"],
        "depth": 2
      },
      {
        "name": "idna",
        "version": "3.7",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "required_by": ["requests"],
        "depth": 2
      },
      {
        "name": "urllib3",
        "version": "2.2.1",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "required_by": ["requests"],
        "depth": 2
      },
      {
        "name": "annotated-types",
        "version": "0.6.0",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "required_by": ["pydantic"],
        "depth": 2
      },
      {
        "name": "pydantic-core",
        "version": "2.18.2",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "required_by": ["pydantic"],
        "depth": 2
      },
      {
        "name": "typing-extensions",
        "version": "4.11.0",
        "type": "runtime",
        "source": "https://pypi.org/simple",
        "required_by": ["pydantic"],
        "depth": 2
      }
    ]
  },
  "summary": {
    "total_dependencies": 10,
    "direct_count": 3,
    "transitive_count": 7,
    "max_depth": 2,
    "python_version": ">=3.8"
  }
}
```

### Success Criteria

- [x] All 3 direct dependencies identified correctly
- [x] All transitive dependencies discovered (minimum 7)
- [x] Correct dependency relationships mapped (required_by field accurate)
- [x] Version constraints preserved exactly as in uv.lock
- [x] No missing dependencies
- [x] No false dependencies (dependencies not in uv.lock)
- [x] Dependency depth calculated correctly (max depth = 2)
- [x] Dependency type classification accurate (runtime vs dev)
- [x] Source registry information preserved
- [x] Output format matches expected schema

### Version Notes

**UV 0.1.x - 0.3.x:**
- Lock file format v1
- Basic dependency structure

**UV 0.4.x - 0.6.x:**
- Lock file format may include additional metadata
- `requires-dist` field added

**UV 0.7.x - 0.9.x:**
- Enhanced lock file with distribution metadata
- May include `sdist` and `wheel` information

**Test Requirement:** Tool must handle all UV versions correctly.

### Edge Cases

1. **Missing uv.lock:**
   - Tool should fail gracefully with clear error message
   - Error: "uv.lock file not found. Run 'uv lock' to generate."

2. **Mismatched versions:**
   - If pyproject.toml specifies "requests>=2.31.0" but uv.lock has 2.30.0
   - Tool should report warning but use uv.lock version (it's the source of truth)

3. **Circular dependencies:**
   - Though rare in Python, should be detected
   - Tool should not infinite loop

4. **Empty dependencies:**
   - If a package has no transitive dependencies (like click)
   - Should show dependencies: [] not null or undefined

### Mend Integration

**Validation Requirement:** Compare dependency tree output with Mend scan results.

**Mend Configuration (for comparison):**

**whitesource.config:**
```properties
python.resolveDependencies=true
python.installVirtualenv=true
python.resolveHierarchyTree=true
python.path=python3.11
```

**Comparison Test:**
```bash
# Generate requirements.txt for Mend
uv pip compile pyproject.toml -o requirements.txt

# Run Mend scan
mend dep --dir fixtures/phase_1/basic_dependency_tree_resolution/

# Compare results
# - Total dependency count should match
# - Direct dependencies should match exactly
# - Transitive dependencies should match (versions may differ slightly)
```

**Expected Mend Behavior:**
- Should detect all 3 direct dependencies
- Should detect all transitive dependencies
- Versions should match UV's resolution
- Tree structure should be equivalent

**If Mend differs:**
- Document discrepancies in test report
- Identify which dependencies differ and why
- Reference `research/mend_integration.md#edge-case-1` for known UV/Mend differences

**See:** `research/mend_integration.md#testing-implications` for detailed Mend test scenarios.

