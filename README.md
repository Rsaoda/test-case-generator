# 测试用例生成器 - Claude Code Skill

一个 Claude Code 技能，从**源代码**或**需求文档**生成全面的测试套件。支持多种语言和框架，智能自动检测。

## 两种模式

| 模式 | 输入 | 输出 |
|------|------|------|
| **代码驱动** | 源代码文件（`.ts`、`.py`、`.go` 等） | 可运行的单元/集成测试代码 |
| **需求驱动** | 文档文件（`.md`、`.txt`、`.pdf`、Swagger/OpenAPI） | 结构化测试用例表 + 可选的可运行代码 |

## 生成内容

### 从代码生成
- **正向功能测试** — 有效输入、预期输出
- **边界测试** — 空值、边界数字、单元素集合
- **异常测试** — 无效类型、空输入、缺少参数
- **边界情况** — 并发调用、幂等性、副作用
- **集成测试** — 模块交互、依赖模拟

### 从需求文档生成
- **功能测试** — 核心功能验证
- **数据验证测试** — 必填字段、格式、长度、类型校验
- **业务规则测试** — 权限控制、状态流转、计算规则
- **异常场景测试** — 未授权访问、资源不存在、冲突场景
- **用户旅程测试** — 跨功能的端到端流程

## 支持的语言和框架

| 语言 | 测试框架 |
|------|---------|
| JavaScript / TypeScript | Jest, Vitest, Mocha |
| Python | pytest, unittest |
| Go | go test |
| Java | JUnit 5, JUnit 4 |
| Rust | cargo test |
| C# | xUnit, NUnit |
| E2E | Playwright, Cypress |

## 安装

### 通过 Skills CLI
```bash
npx skills add rullerzhou-afk/test-case-generator@test-case-generator -g -y
```

### 手动安装
```bash
git clone https://github.com/Rsaoda/test-case-generator.git ~/.claude/skills/test-case-generator
```

## 使用方式

### 代码驱动（分析代码生成测试）

```
给 src/utils/parser.ts 写测试
```

```
为 UserService 类生成测试用例
```

```
给 calculateDiscount 函数添加单元测试
```

### 需求驱动（从需求文档生成测试用例）

```
根据 docs/requirements.md 生成测试用例
```

```
从 PRD 文档生成测试用例
```

```
从这个 Swagger 文档生成接口测试用例
```

```
根据用户故事生成测试用例
```

技能会自动检测输入类型并切换对应模式。

## 工作流程

### 代码驱动流程
1. **检测技术栈** — 扫描 `package.json`、`requirements.txt`、`go.mod` 等
2. **识别命名规范** — 发现已有测试文件的命名模式
3. **分析代码** — 提取函数、类、路由、导出
4. **生成测试** — 按 AAA 模式（准备-执行-断言）创建测试
5. **输出** — 写入测试文件并提供摘要

### 需求驱动流程
1. **解析文档** — 读取 Markdown、纯文本、PDF 或 Swagger/OpenAPI 文件
2. **提取需求** — 识别功能需求、业务规则、验收标准
3. **分类场景** — 归类为功能、验证、边界、业务规则、异常、用户旅程
4. **生成测试用例** — 输出包含步骤和预期结果的结构化测试用例表
5. **可选生成代码** — 如检测到测试框架，可同时生成可运行的测试代码

## 示例：代码驱动

输入：
```javascript
function divide(a, b) {
  if (b === 0) throw new Error("Division by zero");
  return a / b;
}
```

输出：
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

## 示例：需求驱动

输入（PRD 摘录）：
> 用户注册功能：邮箱格式校验，密码不少于8位，注册成功发送验证邮件

输出：
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

## 许可证

MIT
