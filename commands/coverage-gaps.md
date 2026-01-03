---
name: coverage-gaps
description: Find untested files and low coverage areas in your codebase
---

# Coverage Gap Analyzer

Identify files with missing or insufficient test coverage.

## Usage

`/coverage-gaps [threshold]` - Find files below coverage threshold (default: 50%)

## Instructions

When invoked, follow these steps:

### Step 1: Detect Test Framework

Check for these configuration files to identify the test framework:

```bash
ls -la jest.config.* vitest.config.* pytest.ini pyproject.toml setup.cfg karma.conf.* .rspec phpunit.xml go.mod Cargo.toml 2>/dev/null
```

| Config File | Framework | Test Pattern |
|-------------|-----------|--------------|
| `jest.config.*` | Jest (JS/TS) | `*.test.ts`, `*.spec.ts`, `__tests__/` |
| `vitest.config.*` | Vitest (JS/TS) | `*.test.ts`, `*.spec.ts` |
| `pytest.ini`, `pyproject.toml` | Pytest (Python) | `test_*.py`, `*_test.py` |
| `go.mod` | Go testing | `*_test.go` |
| `Cargo.toml` | Rust | `tests/`, `#[test]` in source |
| `.rspec` | RSpec (Ruby) | `*_spec.rb` |
| `phpunit.xml` | PHPUnit | `*Test.php` |

### Step 2: Find Existing Coverage Reports

Look for coverage output in common locations:

```bash
find . -maxdepth 3 -type f \( -name "lcov.info" -o -name "coverage.xml" -o -name "cobertura.xml" -o -name ".coverage" -o -name "coverage.json" \) 2>/dev/null
```

Also check directories:
```bash
ls -d coverage/ .nyc_output/ htmlcov/ target/coverage/ 2>/dev/null
```

### Step 3: Analyze Coverage

#### Option A: Parse Existing Coverage Report

If an `lcov.info` file exists, parse it:
```bash
grep -E "^SF:|^LH:|^LF:" coverage/lcov.info | paste - - - | awk -F'[:|]' '{file=$2; hit=$4; total=$6; pct=(total>0)?hit/total*100:0; if(pct<THRESHOLD) print pct"% - "file}' | sort -n
```

If `coverage.xml` (Cobertura) exists:
```bash
grep -E 'filename=|line-rate=' coverage.xml | paste - - | awk '{...}'
```

#### Option B: Map Source Files to Tests (No Coverage Report)

Find source files:
```bash
find src lib app -type f \( -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.go" -o -name "*.rb" \) 2>/dev/null | grep -v -E "(test|spec|__test__|_test)"
```

Find test files:
```bash
find . -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" -o -name "*_test.*" \) 2>/dev/null
```

Map each source file to potential test files by name convention:
- `src/utils/auth.ts` → `src/utils/auth.test.ts` or `tests/utils/auth.test.ts`
- `lib/payments.py` → `tests/test_payments.py` or `lib/test_payments.py`

### Step 4: Get File Metadata

For files with gaps, gather additional context:

```bash
wc -l <file>                           # Line count
git log -1 --format="%ar" -- <file>    # Last modified
git log --oneline -- <file> | wc -l    # Commit frequency
```

### Step 5: Format Report

Present gaps in this format:

```
COVERAGE GAPS (threshold: {threshold}%)

No tests found:
• src/utils/validation.ts (247 lines, modified 3 days ago)
• src/api/payments.ts (189 lines, modified today) ⚠️

Low coverage (<{threshold}%):
• src/auth/oauth.ts — 34% covered
• src/db/migrations.ts — 12% covered

Summary:
─────────────────────────────
Files with no tests:     12
Files below threshold:    5
Total files analyzed:    87
─────────────────────────────

Suggestion: Start with {highest_priority_file} ({reason})
```

### Step 6: Prioritization Logic

Rank files by priority using this scoring:
1. **Recently modified** (+3 points if modified in last 7 days)
2. **Large file** (+2 points if >200 lines)
3. **High commit frequency** (+2 points if >10 commits)
4. **Critical path** (+3 points if contains: auth, payment, security, user)

Suggest the highest-scored file first.

## Example Output

```
COVERAGE GAPS (threshold: 50%)

No tests found:
• src/utils/validation.ts (247 lines, modified 3 days ago)
• src/api/payments.ts (189 lines, modified today) ⚠️
• src/helpers/format.ts (45 lines, modified 2 weeks ago)

Low coverage (<50%):
• src/auth/oauth.ts — 34% covered
• src/db/migrations.ts — 12% covered

Summary:
─────────────────────────────
Files with no tests:      3
Files below threshold:    2
Total files analyzed:    28
─────────────────────────────

Suggestion: Start with src/api/payments.ts (recently modified + no tests + critical path)
```

## Fallback Behavior

- If no test framework detected: "No test framework found. Looking for source files without corresponding test files..."
- If no coverage report exists: "No coverage report found. Run your test suite with coverage enabled, or I'll analyze by file naming convention."
- If codebase is empty or no source files: "No source files found in standard directories (src/, lib/, app/)"

## Tips for Users

After running this command, you can ask:
- "Generate a test file for src/api/payments.ts"
- "What should I test first in oauth.ts?"
- "Create a coverage improvement plan"
