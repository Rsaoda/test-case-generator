# Test Case Generator - Claude Code Skill

A Claude Code skill that automatically analyzes your code and generates comprehensive, runnable test suites. Supports multiple languages and frameworks with intelligent auto-detection.

## What It Does

Point it at any function, class, module, or API endpoint — it reads the code, understands the structure, and generates a complete test file with:

- **Happy path tests** — valid inputs, expected outputs
- **Boundary tests** — empty values, edge numbers, single-element collections
- **Error tests** — invalid types, null inputs, missing parameters
- **Edge cases** — concurrency, idempotency, side effects
- **Integration tests** — module interactions with proper mocking

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
# Clone into your skills directory
git clone https://github.com/rullerzhou-afk/test-case-generator.git ~/.claude/skills/test-case-generator
```

## Usage

Just ask Claude Code to write tests:

```
Write tests for src/utils/parser.ts
```

```
Generate test cases for the UserService class
```

```
Add unit tests for the calculateDiscount function
```

```
Test this API endpoint
```

The skill auto-detects your project's language and test framework, then generates a properly formatted test file at the conventional location.

## How It Works

1. **Detect Stack** — Scans `package.json`, `requirements.txt`, `go.mod`, etc.
2. **Find Conventions** — Identifies existing test file naming patterns
3. **Analyze Code** — Extracts functions, classes, routes, exports
4. **Generate Tests** — Creates tests following AAA pattern (Arrange-Act-Assert)
5. **Output** — Writes test file and provides a summary

## Example Output

For a function like:
```javascript
function divide(a, b) {
  if (b === 0) throw new Error("Division by zero");
  return a / b;
}
```

Generated tests:
```javascript
describe('divide', () => {
  it('should return correct quotient for positive numbers', () => {
    expect(divide(10, 2)).toBe(5);
  });

  it('should return negative quotient for mixed signs', () => {
    expect(divide(-10, 2)).toBe(-5);
  });

  it('should return zero when numerator is zero', () => {
    expect(divide(0, 5)).toBe(0);
  });

  it('should throw when dividing by zero', () => {
    expect(() => divide(10, 0)).toThrow('Division by zero');
  });

  it('should handle floating point division', () => {
    expect(divide(7, 2)).toBeCloseTo(3.5);
  });
});
```

## License

MIT
