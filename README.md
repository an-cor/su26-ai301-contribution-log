# su26-ai301-contribution-log
CodePath's AI Open Source Capstone living document that tracks everything from issue selection through pull request submissions

# Issue

Repository: k-NN

Issue: #2970 – Incorrect changelog creating incorrect release notes

Issue Link: https://github.com/opensearch-project/k-NN/issues/2970

# Why I Chose This Issue

I chose issue #2970 because it combines software engineering, release automation, and open-source development practices. The issue describes a problem where generated release notes can become inaccurate when the repository's CHANGELOG.md file is not kept synchronized with the commits that are included in a release. As a result, users may receive incomplete or misleading release notes even though the underlying code changes were successfully merged.

This issue interested me because it appears to have a well-defined scope while still requiring investigation into how the k-NN project handles releases and changelog generation. The issue is labeled as a good first issue, has no active pull request associated with it, and provides a concrete example of the problem through a specific release. I also want to gain experience working with CI/CD workflows, release engineering, and project maintenance practices, since these are important skills for software engineering and data platform roles.

From reading the issue discussion, my understanding is that the current release process can generate incorrect release notes when a changelog exists but is outdated. The maintainers appear to be considering either enforcing changelog updates or removing reliance on the changelog entirely. My next step is to confirm the intended direction with the maintainers before beginning implementation.

## Phase II: Reproduce & Plan

### Reproduction Process

#### Environment Setup

I cloned my fork of the OpenSearch k-NN repository locally and created a working branch for issue #2970.

Fork: https://github.com/an-cor/k-NN
Branch: https://github.com/an-cor/k-NN/tree/fix-issue-2970
Issue: https://github.com/opensearch-project/k-NN/issues/2970

Commands used:

```bash
git clone https://github.com/an-cor/k-NN.git
cd k-NN
git checkout -b fix-issue-2970
git push origin fix-issue-2970
```

During setup, I initially attempted to pull from `upstream`, but the upstream remote had not been added yet. The fix is to add the original OpenSearch k-NN repository as an upstream remote:

```bash
git remote add upstream https://github.com/opensearch-project/k-NN.git
git fetch upstream
```

I also searched the repository for release-note and changelog-related files using:

```bash
grep -R "release-notes" .
grep -R "CHANGELOG" .
grep -R "release notes" .
```

This showed that the relevant files include `CONTRIBUTING.md`, `CHANGELOG.md`, `.github/workflows/auto-release.yml`, `.github/workflows/draft-release-notes-workflow.yml`, `.github/draft-release-notes-config.yml`, and the `release-notes/` directory.

#### Steps to Reproduce / Investigate the Issue

1. Open issue #2970, "[BUG] Incorrect changelog creating incorrect release notes."
2. Review the issue description, which explains that release-note generation depends on either the changelog or commits if the changelog is missing.
3. Open the referenced commit `c53ffe2`, which generated release notes for version `2.19.4`.
4. Observe that the generated release note file only contains two release-note entries:

   * Fix the put mapping issue for already created index with flat mapper
   * Avoid opening graph file if graph is already loaded in memory
5. Review `CONTRIBUTING.md`, which states that OpenSearch k-NN expects contributors to update `CHANGELOG.md` so release notes can be built incrementally throughout development.
6. Compare this expectation against the issue report: if `CHANGELOG.md` exists but is stale or incomplete, the generated release notes can become incomplete or inaccurate.
7. Confirm that this is not a normal runtime bug. The bug is in the release/documentation workflow: the repository has a process that expects changelog accuracy, but stale changelog contents can still lead to incorrect generated release notes.

#### Expected Behavior

The release process should not generate incomplete or misleading release notes from an outdated changelog. Either changelog updates should be enforced before release-note generation, or the release-note process should avoid relying on a stale changelog and instead generate notes from commits or another reliable source.

#### Actual Behavior

The issue shows that release notes for `2.19.4` were generated from changelog information that did not match all commits included in the release. This can result in release notes that omit changes users or maintainers should know about.

#### Additional Investigation Notes

After creating my branch, I added the upstream OpenSearch k-NN repository as a remote, fetched upstream branches/tags, synced my local `main` branch with `upstream/main`, pushed the updated `main` to my fork, and rebased my working branch on top of the latest `main`.

Commands used:

```bash
git remote add upstream https://github.com/opensearch-project/k-NN.git
git fetch upstream
git checkout main
git pull upstream main
git push origin main
git checkout fix-issue-2970
git rebase main
git push --force-with-lease origin fix-issue-2970
```

I inspected the release-related workflow files and found the following:

```text
.github/workflows/changelog_verifier.yml
.github/workflows/draft-release-notes-workflow.yml
.github/workflows/auto-release.yml
.github/draft-release-notes-config.yml
CONTRIBUTING.md
CHANGELOG.md
release-notes/
```

The current release-note workflow uses `release-drafter/release-drafter` with `.github/draft-release-notes-config.yml` to generate draft release notes on pushes to `main`. The auto-release workflow publishes a release from a file matching:

```text
release-notes/opensearch-knn.release-notes-${tag}.md
```

I also found that `changelog_verifier.yml` already references `skip-changelog`, which matches the contribution guide’s statement that PRs without changelog entries should fail unless maintainers apply the `skip-changelog` label.

Important finding: when I ran:

```bash
find release-notes -iname "*2.19.4*"
```

there was no `2.19.4` release-note file present in the current branch. This matches the GitHub commit page for `c53ffe2`, which says that commit does not belong to a current branch. That means the issue’s reproduction is based on a historical release-note generation commit rather than a runtime bug that can be reproduced by running the k-NN plugin locally.


---

### Implementation Plan

#### Understand

The current problem is that `CHANGELOG.md` is treated as a release-note source, but if it is not kept up to date, the generated release notes become inaccurate. The issue author states that the expected behavior is to either force changelog entries or remove reliance on the changelog entirely.

#### Match

The repository already documents changelog expectations in `CONTRIBUTING.md`. It says changelog entries are intended to be maintained incrementally and that PRs without changelog entries should fail validation unless a `skip-changelog` label is applied. This suggests that the likely fix should align with existing repository policy rather than inventing a new release process.

#### Plan

My current plan is:

1. Inspect the existing GitHub Actions workflows related to release notes and changelog validation.
2. Identify whether the repository already has a changelog validation workflow and whether it is missing, disabled, incomplete, or not covering the affected release path.
3. Determine whether the smallest safe fix is to:

   * enforce changelog updates during PR validation, or
   * update release-note generation so it does not rely on stale changelog contents.
4. Prefer the smallest fix that matches existing maintainer guidance in `CONTRIBUTING.md`.
5. Add or update validation so that release-note generation cannot silently use an outdated changelog.
6. Run the relevant validation locally if possible.
7. If the intended direction is still ambiguous, follow up on the GitHub issue with a short summary of findings and ask maintainers to confirm the preferred implementation path.

#### Updated Files Likely Involved

Based on my investigation, the most likely files involved are:

```text
.github/workflows/changelog_verifier.yml
.github/workflows/draft-release-notes-workflow.yml
.github/draft-release-notes-config.yml
.github/workflows/auto-release.yml
CONTRIBUTING.md
CHANGELOG.md
```

The most likely implementation path is to improve or adjust changelog validation rather than changing k-NN search code. The existing repository policy already expects changelog entries for user-facing changes, and the existing workflow already has support for `skip-changelog`. Because of that, the safest fix is likely to strengthen the validation/release workflow so stale or missing changelog entries cannot silently produce incomplete release notes.


#### Verification Plan

To verify the fix, I plan to:

1. Run the relevant changelog or release-note validation workflow locally if possible.
2. Create a test scenario where a release-note-relevant change exists without a matching changelog entry.
3. Confirm the validation catches the missing or stale changelog condition.
4. Confirm that cases with a valid changelog entry still pass.
5. Confirm that documentation-only or skipped changelog cases behave according to the repository’s contribution rules.


## Phase III: Build

### Implementation Notes

For Phase III, I implemented a small, scoped documentation/process change for OpenSearch k-NN issue #2970.

Based on my Phase II investigation, the issue appears to be a release/changelog workflow problem rather than a k-NN runtime bug. The release notes can become incorrect when `CHANGELOG.md` exists but contains stale or inaccurate information. I chose a minimal implementation path by updating `CONTRIBUTING.md` instead of changing Java code or modifying GitHub Actions workflows.

The previous changelog instructions told contributors to add changelog entries with dummy pull request information and update the entry later. That process can lead to incorrect release notes if the placeholder entry is never corrected. My change removes the dummy PR guidance and clarifies that contributors should verify the changelog entry references the correct pull request and accurately describes the change before merge.

### Code Changes

Development branch: https://github.com/an-cor/k-NN/tree/fix-issue-2970

Commit: https://github.com/an-cor/k-NN/commit/fdc638d2

Pull request: https://github.com/opensearch-project/k-NN/pull/3380

Files changed:

```text
CONTRIBUTING.md
```

Summary of change:

* Added a note that release notes are generated from `CHANGELOG.md`.
* Clarified that stale or inaccurate changelog entries can result in incorrect release notes.
* Removed wording that encouraged dummy pull request information.
* Updated the contributor workflow to emphasize verifying the correct PR reference before merge.

### Testing Strategy

Because this was a documentation-only change, I did not run the Java or Gradle test suite. The change does not modify k-NN runtime code, build logic, or GitHub Actions YAML.

Validation performed:

```bash
git diff --check HEAD~1 HEAD
git status
```

I intentionally avoided YAML validation because the final implementation only touched `CONTRIBUTING.md`. No workflow files were modified.

### Challenges Faced

The main challenge was scoping the issue appropriately. The issue could potentially lead to a larger release-engineering change, such as modifying changelog validation or changing how release notes are generated. Since I only had limited time, I chose a small, reviewable documentation/process fix that directly addresses one concrete source of the problem: stale or placeholder changelog entries.

I also had to be careful not to claim the PR fully solves the entire release-note generation problem. Instead, I framed it as a focused improvement that addresses part of issue #2970 and can be reviewed independently.

## Phase IV: Submit & Iterate

### Pull Request

PR Link: https://github.com/opensearch-project/k-NN/pull/3380

### PR Summary

I submitted a focused documentation/process change to `CONTRIBUTING.md` for OpenSearch k-NN issue #2970. The PR clarifies that release notes are generated from `CHANGELOG.md`, removes guidance encouraging dummy pull request information, and asks contributors to verify changelog entries before merge.

### Current Status

Status: Iterating / Awaiting maintainer action

The PR received approvals from two reviewers. Most checks are passing or skipped appropriately for a documentation-only change. The remaining blocker is the `Changelog Verifier` workflow, which failed because the PR does not update `CHANGELOG.md`.

Since this is a documentation-only PR, I commented on the PR asking whether a maintainer can apply the `skip-changelog` label or whether they prefer that I add a changelog entry.

### Maintainer Feedback / Next Steps

- PR opened: https://github.com/opensearch-project/k-NN/pull/3380
- Review received: two approvals
- Current blocker: `Changelog Verifier` requires either a `CHANGELOG.md` update or the `skip-changelog` label
- Next step: wait for maintainer guidance or label application