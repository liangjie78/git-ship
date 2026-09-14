<p align="center">
  <img src="./assets/readme/hero.svg" width="100%" alt="git-ship turns scoped working-tree changes into a guarded delivery workflow">
</p>

<p align="center">
  <a href="./README.zh-CN.md">简体中文</a>
</p>

Git Ship takes a clearly scoped working-tree change through the repository's existing GitHub workflow.
The default mode remains fully autonomous:

```text
working tree → scope guard → latest default branch → task branch → commit
→ local checks → PR → CI repair → merge → cleanup
```

Use `ship --review` when you want the same preparation to stop at the PR for human review.

<p align="center">
  <img src="./assets/readme/workflow.svg" width="100%" alt="The guarded stages of the git-ship workflow">
</p>

## Why use it

- Ships all the way to a real merge by default, instead of stopping at PR creation.
- Establishes the task boundary before writing Git state and stages explicit paths only.
- Uses the repository's real default branch and existing validation commands.
- Reads CI failures, fixes safe in-scope causes, and waits for the next check run.
- Preserves unrelated changes, existing branches, local-only commits, and user-owned stashes.
- Stops only for ambiguity or external blockers such as authentication, permissions, required approval, or unavailable services.
- Supports an explicit review mode without making review mode the default.

## Install

Install the fork for a user-level Codex Skill:

```bash
npx skills add liangjie78/git-ship -g -a codex
```

Alternatively, clone it into a directory where your agent discovers Skills:

```bash
git clone https://github.com/liangjie78/git-ship.git ~/.agents/skills/git-ship
```

The entry point is `SKILL.md`. Reload the host after installation if it caches Skills.

## Use

Finish the change and leave it in the working tree, then ask your agent:

```text
ship
```

This invokes the full path: commit, push, PR, CI repair, merge, and cleanup. You can provide a branch or commit message:

```text
Ship this as feat/add-export with commit "feat(export): add markdown export"
```

For a PR handoff without merge or cleanup:

```text
ship --review
```

Run `ship` again later to continue an existing task PR through the full delivery path. Ordinary requests to only commit,
push, or create a PR do not trigger the complete Ship workflow.

## Workflow

| Stage | Default `ship` | `ship --review` | Guard |
| --- | --- | --- | --- |
| Inspect | Establish scope and repository state | Same | Stops on ambiguous or mixed ownership |
| Sync | Fetch the real default branch and preserve scoped changes | Same | Does not overwrite local-only commits |
| Branch | Create or reuse a task branch | Same | Does not reuse an unrelated branch |
| Commit | Stage explicit paths and run hooks | Same | Never uses `git add -A` |
| Verify | Run documented checks and repair in-scope failures | Same local checks | Does not invent or weaken checks |
| Publish | Push and create or update a PR | Same | Requires normal GitHub write access |
| CI | Wait, diagnose, repair, push, and wait again | Stop at PR | No bypass or fake pass |
| Merge | Follow repository policy and merge | Do not merge | No admin or protection bypass |
| Cleanup | Delete only the run-owned merged branch and return to the base branch | Do not clean | Preserve pre-existing branches |

## Requirements

- Git
- [GitHub CLI](https://cli.github.com/) authenticated with `gh auth login`
- A repository with a discoverable default branch and normal push/PR permissions
- Validation commands documented by the repository, when available

When no trustworthy validation command exists, Git Ship reports that fact instead of inventing one. A required human approval,
missing permission, unavailable service, unresolved scope, or repeated failure without new evidence is an external stop.

## Safety model

An explicit `ship` invocation authorizes commit, normal push, PR, in-scope repair, merge, and cleanup for the current task.
It does not authorize unrelated file changes or deletion of user-owned branches. `ship --review` changes only the stopping point;
it never silently changes the default mode.

Git Ship never force-pushes, hard-resets, cleans the working tree, skips hooks or tests, weakens valid assertions, uses an admin merge,
or bypasses branch protection. It preserves the original state when it must stop.

## Customize

Fork this repository and update `SKILL.md` to match team-specific branch naming, merge policy, PR templates, or required checks.
Keep the scope guard, explicit staging, non-destructive recovery, review switch, and external permission boundaries intact.

## Contributing

Keep changes focused, describe the behavior and failure path, and verify both default full delivery and `--review` behavior.

## License

[MIT](./LICENSE)
