---
name: quickfix
description: Make the smallest possible fix (1 to a few lines) for a build/compile error, check it only by rebuilding plus a simple smoke run, and stop and report back if a real fix needs bigger changes. Manual only, run with /quickfix.
disable-model-invocation: true
argument-hint: "[error output, file, or build command]"
---

# Quickfix

The user wants the build working again with the **smallest diff possible**. This is not the time to fix the root cause properly, refactor, or tidy up. Small diffs are fast to review and safe to commit. If a small diff can't fix the problem, the user wants to hear that quickly and make the call themselves, rather than get a big change they didn't ask for.

Input from the user (may be empty): $ARGUMENTS

## 1. Get the error

- If the input has error output, start from that. If it names a build command, use that command.
- Otherwise, find the project's build command from what is in the project: `package.json` scripts, `Makefile`, `Cargo.toml`, `*.sln`/`*.csproj`, `go.mod`, `CMakeLists.txt`, `pyproject.toml`, `build.gradle`/`pom.xml`, and so on. Run it and read the output.
- Start with the **first** error. Later errors are often knock-on effects of the first one and may go away once it's fixed.

## 2. Read only what you need

Open the file at the reported line, plus enough of the code around it to understand the error. Look up a symbol's definition only if the error is about that symbol. Don't explore the whole codebase. The error message usually tells you where to look.

## 3. Decide: is this a quick fix?

**Quick** (go ahead): typo or wrong name, a missing import/`using`/`include`, a wrong type or a needed cast/conversion, missing punctuation or brackets, a call that needs updating after a signature change (one or two call sites), a missing null/undefined guard, a missing or wrong line in a manifest or config file, a wrong path.

**Not quick** (stop, go to step 6). Any of these means stop:
- The fix touches more than ~3 files or more than ~10 lines.
- It changes a public interface or signature that many places use.
- It needs a dependency upgrade across major versions, or a migration.
- It needs a design decision: there are several reasonable fixes and they behave differently.
- The error comes from the environment, not the code (missing SDK, wrong toolchain version, broken install). Report what's missing. Don't install or reconfigure things.
- The broken code is a half-finished feature, so "fixing" it means guessing what it was meant to do.

These limits are rough. Use judgment: a 12-line change that is obviously mechanical is fine, and a 2-line change to core logic you don't understand isn't.

## 4. Apply the fix

- Change only what the error needs. Don't reformat, rename, or "improve" nearby code.
- Don't **hide** the error to get a green build: no `@ts-ignore`, `any`, `#pragma warning disable`, `#[allow(...)]`, `// eslint-disable`, `--force`, no deleting or commenting out the broken code, and no turning off checks in the build config. These make the build pass while the problem stays. If hiding the error really is the only quick option, don't do it. List it as an option in the stop report and let the user choose.

## 5. Check the fix (build + smoke run only)

1. **Rebuild** with the same command.
   - Your fix caused a new error: adjust the fix.
   - A different error that was already there shows up: handle it the same way (steps 2–4) if it's quick too. After about **3 rounds** of fixing errors, if the build still fails, stop and report. The problem is bigger than it looked.
2. **Smoke run**, if it makes sense: start the program with simple input (`--help`, a sample file, the default entry point, a single request) and confirm it doesn't crash right away. Skip it when it doesn't apply, for example for a library, something that needs outside services or credentials, or a long-running server with no obvious quick check. Say that you skipped it.

Don't write new tests and don't run the full test suite. The user wants a fast loop. If tests matter, they'll run them.

## 6. Report

**Fixed:** keep it short.

```
Fixed: <file:line>: <what was wrong, in one line>
Change: <the changed lines, or a brief diff>
Verified: build ✓ (<command>) · smoke run ✓ <what you ran> | skipped (<why>)
```

If you fixed more than one error, add one `Fixed:` line for each.

**Stopped (not a quick fix):** first undo any edits you made, so the code is back the way you found it. Then report:

```
Not a quick fix: <error, file:line>
Cause: <what's actually wrong, 1–2 sentences>
Why it isn't quick: <which limit it crosses: files/lines touched, needs a design decision, environment problem, ...>
Options:
  1. <approach>: <rough size, e.g. "~40 lines across 5 files">
  2. <approach>: <rough size / tradeoff>
```

Then wait for the user to choose. Don't start the bigger fix on your own.
