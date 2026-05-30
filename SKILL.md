---
name: test-case-generator
description: "Use when the user asks to generate test cases, write tests, create unit tests, add test coverage, or test a function/module/file. Analyzes code and auto-generates comprehensive test suites with the correct framework. Also use when the user says 'test this', 'write tests for', 'generate tests', or 'add tests'."
---

# Test Case Generator

Analyze code and generate comprehensive, runnable test suites. Auto-detects language, framework, and test runner from the project.

## When to Use

- User asks to "write tests", "generate test cases", "add tests for X"
- User wants to improve test coverage for a file or module
- User says "test this function/class/API"
- Reviewing code and suggesting test scenarios

## When NOT to Use

- User is asking about existing test failures (use `systematic-debugging` instead)
- User wants to run existing tests (just run them directly)
- User is setting up a test framework from scratch (use `brainstorming` first)

## Process

### Step 1: Detect Project Stack

Scan the project root for manifest files to identify language and framework:

| File | Language | Default Test Framework |
|------|----------|----------------------|
| `package.json` | JS/TS | Jest (if `jest` in deps), Vitest (if `vitest`), Mocha (if `mocha`), else Jest |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python | pytest (if `pytest` in deps), else unittest |
| `go.mod` | Go | `go test` (standard) |
| `pom.xml` / `build.gradle` | Java | JUnit 5 (if `junit-jupiter`), else JUnit 4 |
| `Cargo.toml` | Rust | `cargo test` (standard) |
| `*.csproj` / `*.sln` | C# | xUnit (if `xunit` in deps), else NUnit |

If no manifest found, infer from file extensions of the target file.

### Step 2: Identify Test File Convention

Check existing test files in the project for naming patterns:
- `*.test.js` / `*.spec.js` / `__tests__/*.js`
- `test_*.py` / `*_test.py` / `tests/*.py`
- `*_test.go` / `*_test.rs`
- `*Test.java` / `*Tests.cs`

If no existing tests, use the framework's default convention.

### Step 3: Analyze Target Code

Read the target file and extract:
- **Functions/methods** — name, parameters, return type, complexity
- **Classes** — constructor, methods, properties, inheritance
- **API routes** — HTTP method, path, request/response shape
- **Exported symbols** — what's publicly accessible
- **Edge cases** — null checks, boundary conditions, error handling paths

### Step 4: Generate Test Cases

For each function/method, generate tests in this order:

#### 4a. Happy Path Tests
- Valid inputs with expected outputs
- Typical use cases
- Return value assertions

#### 4b. Boundary Tests
- Empty strings, arrays, objects
- Zero, negative numbers, MAX_SAFE_INTEGER
- Single-element collections
- Whitespace-only strings

#### 4c. Error/Exception Tests
- Invalid input types (string where number expected)
- Null/undefined inputs
- Missing required parameters
- Out-of-range values

#### 4d. Edge Cases
- Concurrent calls (if async)
- Idempotency (calling twice produces same result)
- State mutation side effects

#### 4e. Integration Tests (if applicable)
- Module interactions
- Database/API mocking
- Event emission/handling

### Step 5: Write Test File

Generate the test file following these rules:

1. **Imports** — use the detected framework's syntax
2. **Describe/Group** — organize by function or class
3. **Naming** — `should [expected behavior] when [condition]`
4. **Arrange-Act-Assert** — follow AAA pattern
5. **Mocking** — mock external dependencies (DB, API, filesystem)
6. **No flaky tests** — avoid time-dependent or order-dependent assertions

## Framework-Specific Patterns

### Jest / Vitest (JS/TS)
```javascript
import { describe, it, expect, vi } from 'vitest'; // or jest
import { functionUnderTest } from './module';

describe('functionUnderTest', () => {
  it('should return expected result for valid input', () => {
    expect(functionUnderTest('input')).toBe('expected');
  });

  it('should throw when given invalid input', () => {
    expect(() => functionUnderTest(null)).toThrow('error message');
  });
});
```

### pytest (Python)
```python
import pytest
from module import function_under_test

class TestFunctionUnderTest:
    def test_returns_expected_for_valid_input(self):
        assert function_under_test("input") == "expected"

    def test_raises_on_invalid_input(self):
        with pytest.raises(ValueError, match="error message"):
            function_under_test(None)
```

### Go
```go
func TestFunctionUnderTest(t *testing.T) {
    t.Run("returns expected for valid input", func(t *testing.T) {
        got := FunctionUnderTest("input")
        if got != "expected" {
            t.Errorf("FunctionUnderTest() = %v, want %v", got, "expected")
        }
    })

    t.Run("panics on invalid input", func(t *testing.T) {
        defer func() {
            if r := recover(); r == nil {
                t.Error("expected panic")
            }
        }()
        FunctionUnderTest(nil)
    })
}
```

### JUnit 5 (Java)
```java
@Test
void returnsExpectedForValidInput() {
    assertEquals("expected", functionUnderTest("input"));
}

@Test
void throwsOnInvalidInput() {
    assertThrows(IllegalArgumentException.class, () -> functionUnderTest(null));
}
```

## Output

1. Create the test file at the conventional path
2. List all generated test cases with a summary table
3. Suggest running the tests with the appropriate command
4. If any test cases could not be generated (e.g., missing mocks), note them as TODOs in the file

## Quality Checklist

Before finalizing, verify:
- [ ] Every public function/method has at least one test
- [ ] Error paths are covered
- [ ] No hardcoded values that should be mocked
- [ ] Tests are independent (no shared mutable state)
- [ ] Test names clearly describe the scenario
- [ ] File follows project's existing test conventions
