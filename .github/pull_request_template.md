## What this PR does

Validates that the resolved npm path from `fs.realpathSync` ends with `.js` before returning it. Some Node version managers (mise, asdf, proto) replace `bin/npm` with a non-JS wrapper script (e.g. a bash shim). `fs.realpathSync` succeeds on these (the file exists), but spawning `node /path/to/bash-wrapper` fails with a Syntax Error.

## Why it's needed

When a user installs Node via mise, asdf, or proto, `which npm` resolves to a bash wrapper instead of the actual `npm-cli.js`. The current code calls `fs.realpathSync` on that path and returns it unconditionally, which causes `child_process.spawn("node", [bashWrapper])` to fail. By validating the resolved path ends with `.js`, we fall back to the conventional path (`/usr/local/lib/node_modules/npm/bin/npm-cli.js`) which is always the real JS file.

## Reviewer Test Plan

### How to verify

1. Install mise (`curl https://mise.run | sh`), then `mise use node@22`
2. Run `which npm` — it points to a mise shim (e.g. `~/.local/share/mise/shims/npm`)
3. Before the fix: `node /path/to/mise-shim` fails with a Syntax Error
4. After the fix: the update check succeeds because `getNpmCliPath()` falls through to the conventional path

### Evidence (Before & After)

N/A — this is a non-UI change. The fix is validated by the logic: `fs.realpathSync` on a bash shim returns the shim path (no `.js` suffix), so the `.js` check rejects it and the fallback path is used.

### Tested on

| OS | Status |
| :-- | :----: |
| macOS | ✅ |
| Windows | ⚠️ |
| Linux | ⚠️ |

### Environment

Local macOS with mise-installed Node 22.

## Risk & Scope

- **Main risk or tradeoff**: The `.js` check is a heuristic — a legitimate npm path that doesn't end in `.js` would also be rejected. In practice, npm-cli.js is always a `.js` file, so this is safe.
- **Not validated / out of scope**: Windows npm path resolution (different binary naming conventions). The fallback path is Unix-specific.
- **Breaking changes / migration notes**: None. Existing behavior for standard npm installations is unchanged — `fs.realpathSync` on a real `.js` file returns a path ending in `.js`, so the check passes.

## Linked Issues

Fixes #7543

<details>
<summary>中文说明</summary>

## 这个 PR 做了什么

验证 `fs.realpathSync` 解析出的 npm 路径是否以 `.js` 结尾。某些 Node 版本管理器（mise、asdf、proto）会将 `bin/npm` 替换为非 JS 的包装脚本（例如 bash shim）。`fs.realpathSync` 在这些文件上会成功（文件存在），但执行 `node /path/to/bash-wrapper` 会因语法错误而失败。

## 为什么需要这个修复

当用户通过 mise、asdf 或 proto 安装 Node 时，`which npm` 解析到的是 bash 包装脚本而非真正的 `npm-cli.js`。当前代码无条件地对路径调用 `fs.realpathSync` 并返回，导致 `child_process.spawn("node", [bashWrapper])` 失败。通过验证解析后的路径以 `.js` 结尾，我们可以回退到传统路径（`/usr/local/lib/node_modules/npm/bin/npm-cli.js`），该路径始终是真正的 JS 文件。

## 审查者测试计划

1. 安装 mise（`curl https://mise.run | sh`），然后 `mise use node@22`
2. 运行 `which npm` — 指向 mise shim（例如 `~/.local/share/mise/shims/npm`）
3. 修复前：`node /path/to/mise-shim` 报语法错误
4. 修复后：更新检查成功，因为 `getNpmCliPath()` 回退到传统路径

## 风险与范围

- **主要风险**：`.js` 检查是一种启发式方法 — 不以 `.js` 结尾的合法 npm 路径也会被拒绝。实际上 npm-cli.js 始终是 `.js` 文件，因此这是安全的。
- **未验证/超出范围**：Windows npm 路径解析（不同的二进制命名约定）。回退路径是 Unix 特有的。
- **破坏性变更**：无。标准 npm 安装的行为不变。

## 关联 Issue

Fixes #7543

</details>
