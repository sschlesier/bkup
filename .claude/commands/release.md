---
allowed-tools: Bash(git *), Bash(gh *), Bash(curl *), Bash(shasum *), Bash(date *), Bash(rm *), Read, Edit, Write, AskUserQuestion, Read(/Users/scotts/src/homebrew-tap/**), Edit(/Users/scotts/src/homebrew-tap/**), Write(/Users/scotts/src/homebrew-tap/**)
---

# Release bkup

Perform a full release of the bkup project: generate release notes from commits, update CHANGES.md, tag and push, create a GitHub release, and update the homebrew-tap formula.

The bkup repo is at the current working directory. The homebrew-tap is at `../homebrew-tap` relative to the bkup repo root.

**Important:** Do not use `$()` subshell substitution — it triggers extra permission prompts. Always run the inner command first, read its output, then use the literal value in the next call.

---

## Step 1: Gather current state

Run in parallel:
- `git status --porcelain` — uncommitted changes in bkup repo
- `git -C ../homebrew-tap status --porcelain` — uncommitted changes in tap
- `git describe --tags --abbrev=0` — last version tag

If either working tree has uncommitted changes, stop and tell the user to commit or stash first.

## Step 2: Collect commits since last tag

Using the tag value from Step 1, run:
```
git log <LAST_TAG>..HEAD --oneline
```

Summarize each user-facing change as a bullet point. Omit internal-only changes (CI tweaks, refactors with no behavior change, chore commits) unless significant. Use the conventional commit type to guide this — `feat:` and `fix:` are user-facing; `chore:`, `docs:`, `refactor:` usually are not.

## Step 3: Suggest version bump and confirm with user

Based on the changes, suggest a semver increment:
- **patch** — bug fixes, minor tweaks
- **minor** — new features or non-breaking additions
- **major** — breaking changes requiring user action

Use `AskUserQuestion` to present the draft changelog and confirm the version. Include the suggested bump (marked recommended) and the other two options.

Wait for the user's response before continuing.

## Step 4: Review and approve changelog

Get today's date:
```
date +%Y-%m-%d
```

Show the user the new CHANGES.md section that will be inserted:

```
## v{VERSION} ({TODAY})

- bullet one
- bullet two
```

Use `AskUserQuestion` with options **Looks good** and **Edit** (with "Other" for feedback). If the user wants edits, apply them and show the updated text again. Repeat until approved.

## Step 5: Update CHANGES.md

Read `CHANGES.md`. Insert the new version section immediately after the `# Changelog` heading line, with a blank line before the first existing `## ` section. Use the Edit tool.

## Step 6: Commit, tag, and push

Run each command separately and in order:
```
git add CHANGES.md
git commit -m "chore: release v{VERSION}"
git tag v{VERSION}
git push origin main
git push origin v{VERSION}
```

## Step 7: Create GitHub release

Write the release notes bullet points to `.release-notes.md` in the project root using the Write tool.

Then run:
```
gh release create v{VERSION} --repo sschlesier/bkup --title v{VERSION} --notes-file .release-notes.md
```

Then remove the temp file:
```
rm .release-notes.md
```

## Step 8: Compute sha256 of the release tarball

Run these in order:
```
curl -fsSL -o .release.tar.gz https://github.com/sschlesier/bkup/archive/refs/tags/v{VERSION}.tar.gz
shasum -a 256 .release.tar.gz
rm .release.tar.gz
```

The sha256 is the first field of the `shasum` output.

## Step 9: Update the homebrew formula

Read `../homebrew-tap/Formula/bkup.rb`.

Use the Edit tool to:
1. Replace the `url` line with the new version URL
2. Replace the `sha256` line with the new checksum

## Step 10: Commit and push the homebrew tap

Run each command separately:
```
git -C ../homebrew-tap add Formula/bkup.rb
git -C ../homebrew-tap commit -m "feat: bump bkup to v{VERSION}"
git -C ../homebrew-tap push origin main
```

Report that the release is complete, showing the version and the GitHub release URL returned by `gh`.
