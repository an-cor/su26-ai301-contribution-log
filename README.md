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
