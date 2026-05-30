# Test Case Generator - Claude Code Skill

A Claude Code skill that generates comprehensive test suites from **source code** or **requirement documents**. Supports multiple languages and frameworks with intelligent auto-detection.

## Two Modes

| Mode | Input | Output |
|------|-------|--------|
| **Code-Driven** | Source file (`.ts`, `.py`, `.go`, etc.) | Runnable unit/integration test code |
| **Requirement-Driven** | Doc file (`.md`, `.txt`, `.pdf`, Swagger/OpenAPI) | Structured test case table + optional runnable code |

## What It Generates

### From Code
- **Happy path tests** — valid inputs, expected outputs
- **Boundary tests** — empty values, edge numbers, single-element collections
- **Error tests** — invalid types, null inputs, missing parameters
- **Edge cases** — concurrency, idempotency, side effects
- **Integration tests** — module interactions with proper mocking

### From Requirements
- **Functional tests** — core functionality verification
- **Validation tests** — required fields, format, length, type checks
- **Business rule tests** — permissions, state transitions, calculations
- **Negative tests** — unauthorized access, not found, conflict scenarios
- **User journey tests** — end-to-end workflows across features

## Supported Languages & Frameworks

| Language | Frameworks |
|----------|-----------|
| JavaScript / TypeScript | Jest, Vitest, Mocha |
| Python | pytest, unittest |
| Go | go test |
| Java | JUnit 5, JUnit 4 |
| Rust | cargo test |
| C# | xUnit, NUnit |
| E2E | Playwright, Cypress |

## Installation

### Via Skills CLI
```bash
npx skills add rullerzhou-afk/test-case-generator@test-case-generator -g -y
```

### Manual Installation
```bash
git clone https://github.com/rullerzhou-afk/test-case-generator.git ~/.claude/skills/test-case-generator
```

## Usage

### Code-Driven (分析代码生成测试)

```
Write tests for src/utils/parser.ts
```

```
Generate test cases for the UserService class
```

```
Add unit tests for the calculateDiscount function
```

### Requirement-Driven (从需求文档生成测试用例)

```
根据 docs/requirements.md 生成测试用例
```

```
Generate test cases from the PRD document at docs/prd.md
```

```
从这个 Swagger 文档生成接口测试用例
```

```
根据用户故事生成测试用例
```

The skill auto-detects the input type and switches mode accordingly.

## How It Works

### Code-Driven Flow
1. **Detect Stack** — Scans `package.json`, `requirements.txt`, `go.mod`, etc.
2. **Find Conventions** — Identifies existing test file naming patterns
3. **Analyze Code** — Extracts functions, classes, routes, exports
4. **Generate Tests** — Creates tests following AAA pattern (Arrange-Act-Assert)
5. **Output** — Writes test file and provides a summary

### Requirement-Driven Flow
1. **Parse Document** — Reads markdown, text, PDF, or Swagger/OpenAPI files
2. **Extract Requirements** — Identifies functional requirements, business rules, acceptance criteria
3. **Classify Scenarios** — Groups into functional, validation, boundary, business rule, negative, journey
4. **Generate Test Cases** — Outputs structured test case table with steps and expected results
5. **Optional Code** — Can also generate runnable test code if framework is detected

## Example: Code-Driven

Input:
```javascript
function divide(a, b) {
  if (b === 0) throw new Error("Division by zero");
  return a / b;
}
```

Output:
```javascript
describe('divide', () => {
  it('should return correct quotient for positive numbers', () => {
    expect(divide(10, 2)).toBe(5);
  });

  it('should return negative quotient for mixed signs', () => {
    expect(divide(-10, 2)).toBe(-5);
  });

  it('should throw when dividing by zero', () => {
    expect(() => divide(10, 0)).toThrow('Division by zero');
  });
});
```

## Example: Requirement-Driven

Input (PRD excerpt):
> 用户注册功能：邮箱格式校验，密码不少于8位，注册成功发送验证邮件

Output:
```markdown
# 测试用例: 用户注册

## TC-001: 正常注册
- **前置条件**: 邮箱未被注册
- **测试步骤**: 1. 输入有效邮箱 2. 输入8位以上密码 3. 点击注册
- **预期结果**: 注册成功，收到验证邮件
- **测试类型**: 功能
- **优先级**: P0

## TC-002: 邮箱格式错误
- **前置条件**: 无
- **测试步骤**: 1. 输入 "abc" 作为邮箱 2. 点击注册
- **预期结果**: 提示"邮箱格式不正确"
- **测试类型**: 验证
- **优先级**: P0

## TC-003: 密码不足8位
- **前置条件**: 无
- **测试步骤**: 1. 输入有效邮箱 2. 输入 "1234567" 3. 点击注册
- **预期结果**: 提示"密码不少于8位"
- **测试类型**: 边界
- **优先级**: P0

## TC-004: 邮箱已被注册
- **前置条件**: 邮箱已存在
- **测试步骤**: 1. 输入已注册邮箱 2. 点击注册
- **预期结果**: 提示"该邮箱已注册"
- **测试类型**: 异常
- **优先级**: P1
```

## License

MIT
