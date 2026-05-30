---
name: test-case-generator
description: "Use when the user asks to generate test cases, write tests, create unit tests, add test coverage, or test a function/module/file. Also use when the user asks to generate test cases from requirements docs, PRD, user stories, API specs, or design documents. Supports two modes: code-driven (analyze source code) and requirement-driven (analyze docs). Also trigger on 'test this', 'write tests for', 'generate tests', 'add tests', '根据需求生成测试', '从文档生成测试用例'."
---

# Test Case Generator

Generate comprehensive test suites from **source code** or **requirement documents**. Auto-detects language, framework, and test runner from the project.

## Two Modes

| Mode | Input | Output |
|------|-------|--------|
| **Code-Driven** | Source file (`.ts`, `.py`, `.go`, etc.) | Unit/integration test code |
| **Requirement-Driven** | Doc file (`.md`, `.txt`, `.pdf`, Swagger/OpenAPI, user stories) | Functional test cases (code or table) |

Detect mode automatically:
- Input is a code file → Code-Driven
- Input is a doc/spec/API file → Requirement-Driven
- User says "根据需求" / "from requirements" → Requirement-Driven

## When to Use

- "Write tests for src/utils/parser.ts" → Code-Driven
- "Generate test cases from docs/requirements.md" → Requirement-Driven
- "根据 PRD 文档生成测试用例" → Requirement-Driven
- "给这个 API 文档写测试" → Requirement-Driven
- "Add tests for the UserService class" → Code-Driven
- "从 Swagger 文档生成接口测试" → Requirement-Driven

## When NOT to Use

- User is asking about existing test failures (use `systematic-debugging` instead)
- User wants to run existing tests (just run them directly)
- User is setting up a test framework from scratch (use `brainstorming` first)

---

## Mode A: Code-Driven

Analyze source code and generate runnable test code.

### Step 1: Detect Project Stack

Scan the project root for manifest files:

| File | Language | Default Test Framework |
|------|----------|----------------------|
| `package.json` | JS/TS | Jest (if `jest` in deps), Vitest (if `vitest`), Mocha (if `mocha`), else Jest |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python | pytest (if `pytest` in deps), else unittest |
| `go.mod` | Go | `go test` (standard) |
| `pom.xml` / `build.gradle` | Java | JUnit 5 (if `junit-jupiter`), else JUnit 4 |
| `Cargo.toml` | Rust | `cargo test` (standard) |
| `*.csproj` / `*.sln` | C# | xUnit (if `xunit` in deps), else NUnit |

If no manifest found, infer from file extensions.

### Step 2: Identify Test File Convention

Check existing test files for naming patterns:
- `*.test.js` / `*.spec.js` / `__tests__/*.js`
- `test_*.py` / `*_test.py` / `tests/*.py`
- `*_test.go` / `*_test.rs`
- `*Test.java` / `*Tests.cs`

### Step 3: Analyze Target Code

Read the target file and extract:
- **Functions/methods** — name, parameters, return type, complexity
- **Classes** — constructor, methods, properties, inheritance
- **API routes** — HTTP method, path, request/response shape
- **Exported symbols** — what's publicly accessible
- **Edge cases** — null checks, boundary conditions, error handling paths

### Step 4: Generate Test Cases

For each function/method:

#### Happy Path Tests
- Valid inputs with expected outputs
- Typical use cases
- Return value assertions

#### Boundary Tests
- Empty strings, arrays, objects
- Zero, negative numbers, MAX_SAFE_INTEGER
- Single-element collections
- Whitespace-only strings

#### Error/Exception Tests
- Invalid input types (string where number expected)
- Null/undefined inputs
- Missing required parameters
- Out-of-range values

#### Edge Cases
- Concurrent calls (if async)
- Idempotency (calling twice produces same result)
- State mutation side effects

#### Integration Tests (if applicable)
- Module interactions
- Database/API mocking
- Event emission/handling

### Step 5: Write Test File

1. **Imports** — use the detected framework's syntax
2. **Describe/Group** — organize by function or class
3. **Naming** — `should [expected behavior] when [condition]`
4. **Arrange-Act-Assert** — follow AAA pattern
5. **Mocking** — mock external dependencies
6. **No flaky tests** — avoid time-dependent or order-dependent assertions

---

## Mode B: Requirement-Driven

Analyze requirement documents and generate functional test cases.

### Step 1: Parse Requirement Document

Support these formats:
- **Markdown** (`.md`) — parse headings, tables, lists, code blocks
- **Plain text** (`.txt`) — parse numbered requirements, bullet points
- **PDF** (`.pdf`) — extract text content
- **Swagger/OpenAPI** (`.json`, `.yaml`) — parse endpoints, schemas, examples
- **User stories** — extract "As a... I want... So that..." patterns

Extract from the document:
- **Functional requirements** — what the system should do
- **Business rules** — constraints, validations, calculations
- **User scenarios** — workflows, user journeys
- **API specifications** — endpoints, request/response formats
- **Acceptance criteria** — conditions for "done"
- **Edge cases mentioned** — explicit boundary conditions

### Step 2: Classify Test Scenarios

For each requirement, generate test scenarios in these categories:

#### Functional Tests (正向功能)
- Core functionality works as described
- Expected outputs for valid inputs
- Complete workflow end-to-end

#### Validation Tests (数据验证)
- Required field validation
- Format validation (email, phone, ID, etc.)
- Length/range validation
- Type validation

#### Boundary Tests (边界值)
- Minimum/maximum values
- Empty/null inputs
- Single item vs bulk operations
- Special characters

#### Business Rule Tests (业务规则)
- Permission/role-based access
- State transitions (e.g., order status: pending → paid → shipped)
- Calculation rules (discount, tax, shipping)
- Conflict resolution (duplicate orders, concurrent edits)

#### Negative Tests (异常场景)
- Invalid credentials
- Unauthorized access
- Resource not found
- Concurrent modification conflicts
- Network/timeout failures

#### User Journey Tests (用户旅程)
- Complete user workflow from start to finish
- Cross-feature interactions
- Multi-role scenarios

### Step 3: Generate Test Case Output

For requirement-driven mode, generate a structured test case document:

```markdown
# 测试用例: [需求名称]

## 需求摘要
[简要描述需求内容]

## 测试用例列表

### TC-001: [测试场景名称]
- **前置条件**: [测试前需要满足的条件]
- **测试步骤**:
  1. [步骤1]
  2. [步骤2]
  3. [步骤3]
- **预期结果**: [期望的输出/行为]
- **测试类型**: [功能/边界/异常/业务规则]
- **优先级**: [P0/P1/P2]

### TC-002: ...
```

### Step 4 (Optional): Generate Runnable Test Code

If the user also wants runnable test code (not just test case table):
- Detect the project's test framework (same as Code-Driven mode)
- Convert test case table into executable test functions
- Include mock data and setup/teardown

---

## Framework-Specific Patterns (Code-Driven)

### Jest / Vitest (JS/TS)
```javascript
import { describe, it, expect, vi } from 'vitest';
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

---

## Output

### Code-Driven Output
1. Create the test file at the conventional path
2. List all generated test cases with a summary table
3. Suggest running the tests with the appropriate command
4. Note any ungenerated tests as TODOs

### Requirement-Driven Output
1. Create a test case document (Markdown table or structured list)
2. Optionally generate runnable test code
3. Summary: total test cases by category and priority
4. Coverage gap analysis — requirements with no test cases

## Quality Checklist

Before finalizing:
- [ ] Every public function/requirement has at least one test
- [ ] Error paths are covered
- [ ] No hardcoded values that should be mocked
- [ ] Tests are independent (no shared mutable state)
- [ ] Test names clearly describe the scenario
- [ ] File follows project's existing test conventions
- [ ] (Requirement mode) All acceptance criteria have corresponding test cases
- [ ] (Requirement mode) Business rules are explicitly tested
