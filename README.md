# su26-ai301-contribution-log
CodePath's AI Open Source Capstone living document that tracks everything from issue selection through pull request submissions

# OpenSearch k-NN Issue

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

### Expanded Testing / Validation Notes

This PR was documentation-only and changed only `CONTRIBUTING.md`. Because it did not modify runtime code, build logic, Java classes, or GitHub Actions YAML, unit tests were not applicable. Based on instructor guidance, unit tests are mainly required when testing executable code, logic, math, proofs, or behavior-changing implementation.

Instead, I validated the change with documentation-focused checks:

- Confirmed the diff only modified `CONTRIBUTING.md`.
- Ran `git diff --check HEAD~1 HEAD` to verify there were no whitespace or formatting errors.
- Reviewed the rendered Markdown to make sure the updated contributor instructions were readable.
- Verified that the previous guidance encouraging dummy pull request information was removed.
- Verified that the new wording clearly explains that stale or inaccurate changelog entries can affect generated release notes.
- Confirmed that no Java code, Gradle files, or workflow YAML files were changed, so the project test suite was not necessary for this scoped documentation change.

The PR was reviewed, approved by two reviewers, and merged upstream, which confirmed that the scoped validation was acceptable for this contribution.

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

### Week 5 Update

My pull request was merged into the upstream OpenSearch k-NN repository.

PR: https://github.com/opensearch-project/k-NN/pull/3380
Merged status: Merged
Merged by: `shatejas`
Final status: Completed / Merged

This PR contributed a focused documentation/process improvement to `CONTRIBUTING.md`. It clarified that release notes are generated from `CHANGELOG.md`, removed guidance encouraging dummy pull request information, and asked contributors to verify changelog entries before merge.

The original issue #2970 remains open because the broader release-note generation problem may still require additional enforcement or build-repo workflow changes. My PR addressed one concrete source of the problem by improving the contributor guidance around stale or placeholder changelog entries.

### Reflection

This contribution taught me that not every open-source fix is a runtime code change. Some issues are process bugs, documentation bugs, or release-engineering problems. I also learned how important it is to scope a contribution honestly: my PR did not claim to fully close #2970, but it did provide a small, reviewable improvement that maintainers accepted and merged.

### Next Steps

For the next part of the course, I plan to:

1. monitor issue #2970 in case maintainers want follow-up work around changelog enforcement, or
2. start a second contribution cycle with another OpenSearch k-NN issue.

---

# Final Status: OpenSearch k-NN

## Final Outcome

Repository: OpenSearch k-NN
Issue: #2970 – Incorrect changelog creating incorrect release notes
Issue Link: https://github.com/opensearch-project/k-NN/issues/2970
Pull Request: https://github.com/opensearch-project/k-NN/pull/3380
Final PR Status: Merged

My first contribution cycle is complete. PR #3380 was merged into the upstream OpenSearch k-NN repository. The contribution updated `CONTRIBUTING.md` to clarify changelog accuracy expectations, remove guidance encouraging dummy pull request information, and reduce the chance of stale changelog entries producing incorrect release notes.

The original issue remains open because the broader release-note generation problem may still require additional enforcement or build-repo workflow changes. My PR addressed one concrete source of the problem and was accepted as a focused documentation/process improvement.

## Final Reflection

This cycle taught me that open-source contributions are not always runtime code changes. Some valuable fixes improve release processes, contributor guidance, or project maintainability. I also learned how important it is to scope a PR honestly: my PR did not claim to fully close the issue, but it did address part of the problem in a way maintainers accepted and merged.

---

# Vector Issue

Repository: Vector
Project Link: https://github.com/vectordotdev/vector
Fork: https://github.com/an-cor/vector

Issue: #1065 – Only warn about small files when they are not empty
Issue Link: https://github.com/vectordotdev/vector/issues/1065

## Why I Chose This Issue

I chose Vector issue #1065 because it is related to observability, file-based log ingestion, and production log noise. The issue describes a case where Vector emits the warning `Currently ignoring file too small to fingerprint` for files that may be empty, which can create noisy logs in environments with many short-lived or empty log files. This matters because operators need warnings to indicate real problems, not harmless empty-file cases.

This issue seems like a good second contribution because it is labeled as a good first issue, is still open, has no active linked PR, and appears to be contained within the file source or file fingerprinting logic. It also connects well to data engineering and observability because Vector is a data pipeline tool used for log and metrics collection.

From reading the issue thread, my understanding is that non-empty files that are too small to fingerprint should still warn because they may represent a real ingestion problem. Empty files, however, should likely be handled without emitting the same warning because they can be common in production environments and may not require user action.

## Phase I Status

Status: Phase I Complete

Completed Phase I tasks:

* Selected Vector issue #1065 as my second contribution cycle.
* Forked the Vector repository to my GitHub account.
* Left a comment on the issue expressing interest and summarizing my understanding of the problem.
* Started documenting the issue in this Contribution README.

## Phase II: Reproduce & Plan

### Environment Setup

I cloned my fork of the Vector repository locally, added the upstream repository, fetched upstream branches and tags, created a working branch, and pushed it to my fork.

Fork: https://github.com/an-cor/vector
Branch: https://github.com/an-cor/vector/tree/fix-small-empty-file-warning
Issue: https://github.com/vectordotdev/vector/issues/1065

Commands used:

```bash
cd ~/Documents/codingprojects/CodePath/ai2
git clone https://github.com/an-cor/vector.git
cd vector
git remote add upstream https://github.com/vectordotdev/vector.git
git fetch upstream
git checkout -b fix-small-empty-file-warning
git push origin fix-small-empty-file-warning
```

### Initial Code Search

To locate the warning mentioned in the issue, I searched the repository for the current log message and fingerprinting-related code.

Commands used:

```bash
grep -R "Currently ignoring file too small to fingerprint" .
grep -R "too small to fingerprint" .
grep -R "fingerprint" src lib | head -100
```

The warning appears in:

```text
src/internal_events/file.rs
```

The search also showed likely related file-source and fingerprinting areas:

```text
src/sources/file.rs
src/sources/kubernetes_logs/mod.rs
lib/file-source/src/file_server.rs
lib/file-source-common/src/checkpointer.rs
```

### Problem Summary

The current issue is that Vector can emit a warning when a file is too small to fingerprint. This warning is useful for non-empty files because it can explain why Vector is not tailing or fingerprinting a file. However, the same warning can become noisy when the file is empty, especially in production environments with many short-lived jobs or log files.

### Expected Behavior

Vector should still warn when a non-empty file is too small to fingerprint, because that may indicate a real ingestion issue. Empty files should not emit the same warning, because empty files are often harmless and can create unnecessary log noise.

### Actual Behavior

The issue reports that Vector can emit the warning:

```text
Currently ignoring file too small to fingerprint.
```

for cases where the file may be empty or not actionable. This can pollute Vector’s own logs and make it harder for operators to distinguish meaningful warnings from expected empty-file behavior.

### Implementation Plan

My current plan is:

1. Inspect `src/internal_events/file.rs` to understand how the warning event is emitted.
2. Trace where that internal event is called from the file source/fingerprinting logic.
3. Identify whether the caller has access to file size, file metadata, or read-length information.
4. Update the logic so the warning is emitted only for non-empty files that are too small to fingerprint.
5. Preserve the existing warning behavior for non-empty files.
6. Add or update tests around the file source/fingerprinting path if there are existing nearby tests.
7. Run the smallest relevant Rust test target I can identify for the changed file source/fingerprinting behavior.

### Files Likely Involved

Likely files to inspect or modify:

```text
src/internal_events/file.rs
lib/file-source/src/file_server.rs
src/sources/file.rs
src/sources/kubernetes_logs/mod.rs
```

The exact file list may change after I trace the call path for the warning event.

### Verification Plan

To verify the fix, I plan to:

1. Create or identify a test case for an empty file that is too small to fingerprint.
2. Confirm the empty-file case does not emit the warning.
3. Confirm a non-empty file that is too small to fingerprint still emits the warning.
4. Run targeted tests for the file source or fingerprinting module.
5. Run formatting/check commands required by Vector’s contribution guide if available.


## Cycle 2 Phase III: Build

### Implementation Notes

I implemented a small fix for Vector issue #1065. The warning was emitted through the file-source-common fingerprinter path when fingerprinting returned `UnexpectedEof`. I updated the logic to record the file size from metadata and skip the checksum warning when the file is empty, while preserving the existing warning for non-empty files that are too small to fingerprint.

### Code Changes

Branch: https://github.com/an-cor/vector/tree/fix-small-empty-file-warning  
Commit: https://github.com/an-cor/vector/commit/e8f82d1f3a  
Pull Request: https://github.com/vectordotdev/vector/pull/25897

Files changed:

- `lib/file-source-common/src/fingerprinter.rs`
- `changelog.d/1065-empty-file-fingerprint-warning.fix.md`

### Testing Strategy

Validation performed:

```bash
cargo test -p file-source-common no_error_on_empty_file
cargo test -p file-source-common fingerprinter
cargo fmt --check
git diff --check