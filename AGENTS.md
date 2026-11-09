# Rules for agents working in thedavidweng repositories

These rules apply to every repository owned by `thedavidweng`. A repository's
own `AGENTS.md` can add to them.

## Commits

- Author and committer: `thedavidweng <95214375+thedavidweng@users.noreply.github.com>`.
- No `Co-authored-by` or other trailers.
- No force-push or history rewrite without the owner's approval.

## Pull requests

### Review comments

CodeRabbit (`coderabbitai[bot]`) and Kilo Code Review (`kilo-code-bot[bot]`)
review every PR. Before merging:

1. Read every comment from both: inline comments, review bodies (including
   CodeRabbit's collapsed "Additional comments" and "Outside diff range"
   sections), and summary comments.
2. Judge each one critically. Check the claim against the code, docs or a
   test. A bot being confident is not evidence.
3. Fix real issues with new commits on the PR branch.
4. Do not reply to noise. Record why it was dismissed in your report to the
   owner instead.
5. After pushing a fix, wait for the re-review and for CI to finish.

A CodeRabbit check that reports "Review rate limited" means it did not review
that commit. Say so in the report rather than counting it as a review.

### Merging

- Merge only when every check is green: nothing pending, failing or
  `action_required`.
- Never bypass required reviews, branch protection or rulesets with admin
  rights. If a PR only lacks a required review the author cannot give, leave
  it open and report it.
- Squash merge with an explicit subject and body, so GitHub adds no
  `Co-authored-by` trailers:

  ```sh
  gh pr merge <number> --squash --subject "<type>(<scope>): <summary> (#<number>)" --body ""
  ```

  Then check the squash commit: author `thedavidweng`, no trailers.

### Screenshots for UI changes

A PR that changes anything a user can see (app UI, web pages, CLI/TUI output
layout, generated images) must include screenshots in its body, uploaded as
user attachments with `gh`'s `--attach` flag (gh 2.102.0 or later). Do not
link images hosted elsewhere and do not commit screenshots to the repository.

`--attach` exists on `gh pr create`, `gh pr edit`, `gh pr comment`,
`gh issue create`, `gh issue edit` and `gh issue comment`. Format:
`--attach '<file>#<alt text>'`, repeatable, up to 50 files per command.

```sh
# New PR: the images are appended to the body.
gh pr create --title "feat(ui): ..." --body-file body.md \
  --attach './before.png#Settings page before' \
  --attach './after.png#Settings page after'

# Existing PR: without --body the current body is kept and the images are appended.
gh pr edit 23 --attach './after.png#Settings page after'
```

If the body already references the file, for example `![Before](./before.png)`,
`gh` rewrites that reference to the uploaded asset instead of appending it.
Video files render as a player and take no alt text. If some uploads fail,
`gh` still creates or updates the PR with the rest and exits non-zero, so
check the exit status.

## Dependabot

Every repository with Dependabot uses the same auto-merge workflow:
copy `templates/dependabot-automerge.yml` from this repository to
`.github/workflows/dependabot-automerge.yml`. It calls the reusable
`.github/workflows/dependabot-automerge.yml`.

- Eligible: semver patch and minor updates of any dependency (production,
  development or indirect), and GitHub Actions pin refreshes.
- Left for review: majors, unclassified updates, the deny list
  (`googleapis/release-please-action`, `dependabot/fetch-metadata`) and any
  ecosystem the repository passes in `skip-ecosystems`.
- Token: `GITHUB_TOKEN` only, never a PAT.
- It approves the PR when a review is required, then enables GitHub's native
  auto-merge (`--squash --match-head-commit`). GitHub merges only after every
  required status check passes.
- It does nothing when the base branch has no required status checks, because
  auto-merge would then merge immediately.
- Repository settings it needs: "Allow auto-merge" on, and "Allow GitHub
  Actions to create and approve pull requests" on when the branch requires a
  review.

Native auto-merge does not wait for review bots. A CodeRabbit or Kilo finding
on a Dependabot PR does not block the merge by itself, so check open
Dependabot PRs against the review rules above when you work in a repository.

## Releases

Repositories that use release-please call `.github/workflows/release-please.yml`
from this repository. It mints a token from the `thedavidweng-release`
GitHub App (`vars.RELEASE_APP_ID`, `secrets.RELEASE_APP_PRIVATE_KEY`), so
release PRs and tags trigger CI and release workflows. Do not add PATs.
