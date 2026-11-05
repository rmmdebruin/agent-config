# Agent setup

## Current setup
- **Rules:** `AGENTS.md` in this repo. A hook in each migrated repo loads the latest version at every session start.
- **Where agents run:** in the cloud by default. Local sessions (VS Code, desktop app on Local) get the same rules, and the sandbox from `user-settings.json` limits them to `~/repos`.
- **Checks:** CI runs on every PR. Agents open PRs, and I merge.

## Start a new repo

```bash
gh repo create <name> --private --template rmmdebruin/repo-template
```

Start a cloud session on it, describe what to build, and ask it to rename the `app` package.

## Develop

### Easy feature
Plain prompt in Auto:

```text
<What you want>. Done when <how to tell>.
```

### Hard feature
**Session 1, with me present:**

```text
I want to add <feature>. Interview me on the hard parts until you could write the spec, then write it to docs/features/<slug>/spec.html and open a PR with only the spec. Don't build anything yet.
```

Merge the spec PR.

**Session 2, a new session in Auto:**

```text
/goal Build docs/features/<slug>/spec.html. Done when every requirement is implemented, a PR is open against main, CI is green and the evidence is in the PR. Or stop after 40 turns and report what is left.
```

Review the green PR and merge it.
