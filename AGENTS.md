# Working on Hemx skills

Each skill lives in `skills/<name>/SKILL.md`; its frontmatter `name` must match
the directory. Keep it self-contained and specific about when it applies.
Teach application decisions rather than repeating the framework documentation
or introducing project-specific policy.

Describe released APIs, checked against public READMEs, Rustdoc, source and
tests. Do not depend on private examples, local paths or unpublished behavior.

## Checks

- Run `git diff --check` and check the frontmatter name.
- Check each API reference against the supported release.
- Try a request that should use the skill and a similar one that should not.
- Check that a broader request does not lead it to invent unrelated features.

## Git

Use Conventional Commits: `<type>(<scope>): <summary>`, and branch names such as
`docs/update-guidance`. Versioned releases use annotated SemVer tags (`v1.2.0`).
Commit, push and publish only when requested.
