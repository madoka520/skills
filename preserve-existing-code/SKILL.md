---
name: preserve-existing-code
description: Preserve all existing code during implementation and refactoring. Use whenever editing source code under a strict no-deletion policy: comment out replaced code instead of deleting it, retain original logic for traceability, and mark every newly written code block with explicit Codex-added comments.
---

# Preserve Existing Code

Apply a strict append-and-comment editing policy for user-authored code. Preserve user-authored original code for traceability, but do not accumulate obsolete Codex-authored revisions.

## Mandatory Rules

1. Never delete user-authored pre-existing code.
2. Never overwrite user-authored code without retaining the user-authored original as comments.
3. Comment out user-authored code that must be disabled, replaced, or moved.
4. Mark every newly written code block with language-appropriate comments containing `Codex 新增开始` and `Codex 新增结束`.
5. Mark retained replaced user-authored code with a comment containing `原代码 - Codex 保留`.
6. When modifying a block that was originally written by Codex, keep only the latest Codex-authored version. Delete obsolete earlier Codex-authored variants and their old `Codex 新增开始` / `Codex 新增结束` wrappers instead of nesting preservation comments.
7. When a block contains both user-authored original code and later Codex edits, preserve only the user-authored original plus the latest Codex-authored replacement. Remove intermediate Codex-authored revisions.
8. Treat imports, types, configuration, styles, templates, tests, and scripts as code under this policy.
9. Do not remove user-authored dead code, unused imports, old comments, or duplicated logic, even when linters recommend removal.
10. Do not use formatters or bulk rewrites that erase or obscure the user-authored original code.
11. When adding new files under a Git repository, stage those new files with `git add` before finishing the task. Do not auto-commit or publish unless the user explicitly asks.

## Editing Workflow

Before editing:

1. Read the complete target block and identify every line that would normally be changed or removed.
2. Choose the smallest insertion point that preserves behavior and keeps the old implementation readable.
3. Determine the valid comment syntax for the file format.

When replacing code:

1. Determine whether the block being replaced is user-authored original code or an older Codex-authored revision.
2. If it is user-authored original code, add an `原代码 - Codex 保留` marker and comment out the complete original block without altering its content.
3. If it is an older Codex-authored revision, delete that older Codex revision and replace it with the latest Codex-authored version; do not preserve obsolete Codex code.
4. Add the latest replacement between `Codex 新增开始` and `Codex 新增结束` markers.
5. Keep the user-authored original block adjacent to its latest replacement unless the language makes that invalid.

When only adding code:

1. Leave surrounding code unchanged.
2. Wrap the new code in `Codex 新增开始` and `Codex 新增结束` markers.


## Repeated Modification Policy

When editing the same area multiple times:

1. Preserve the user's original code once.
2. Keep only the newest Codex-authored implementation.
3. Remove older Codex-authored attempts, including obsolete commented Codex blocks and obsolete `Codex 新增开始` / `Codex 新增结束` wrappers.
4. Do not create nested stacks of Codex preservation comments.
5. If authorship is unclear, treat the code as user-authored and preserve it.


## Import Cleanup Policy

For import-only changes:

1. Do not add `Codex 新增开始` / `Codex 新增结束` markers around imports.
2. Do not preserve old import lines as comments when only the module path, type source, or named imports change.
3. Delete obsolete imports directly, including old Codex-authored import comments.
4. Keep import sections clean and readable.
5. If an import line itself contains user-authored non-import logic or an important user comment, preserve that user-authored comment separately.
## Comment Examples

TypeScript, JavaScript, Go, Java, C, and similar languages:

```ts
// 原代码 - Codex 保留
// const result = legacyTransform(input)

// Codex 新增开始
const result = safeTransform(input)
// Codex 新增结束
```

Vue templates and HTML:

```html
<!-- 原代码 - Codex 保留 -->
<!-- <legacy-panel /> -->

<!-- Codex 新增开始 -->
<current-panel />
<!-- Codex 新增结束 -->
```

CSS and Less:

```css
/* 原代码 - Codex 保留 */
/* color: red; */

/* Codex 新增开始 */
color: var(--status-error);
/* Codex 新增结束 */
```

SQL:

```sql
-- 原代码 - Codex 保留
-- SELECT * FROM users;

-- Codex 新增开始
SELECT id, name FROM users;
-- Codex 新增结束
```

## Unsupported Formats

If the target format does not support comments, do not edit it silently. Stop and explain that the requested preservation markers cannot be represented safely. Ask for explicit permission before using an alternative such as an adjacent documentation file or a format-specific metadata field.

Do not insert invalid comments into JSON, generated binary files, lockfiles, images, or other comment-free formats.

## Exceptions

Only delete or replace existing code without comment preservation when the user explicitly overrides this skill for a specific file or block. A general request such as “refactor,” “clean up,” or “fix” is not an override.

Generated artifacts may be regenerated only when the user explicitly permits regeneration. Report that generated output may not preserve previous lines.

## Verification

Before finishing:

- Check the diff for deleted pre-existing lines.
- Confirm every disabled or replaced block remains as comments.
- Confirm every Codex-authored code block has start and end markers.
- Confirm comments use valid syntax and do not break parsing or compilation.
- Confirm new files created by Codex are staged in Git when they are inside a Git repository.
- Run relevant validation without applying automatic fixes or formatting rewrites.



