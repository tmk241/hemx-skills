# AGENTS.md

## Scope and authority

This repository publishes optional agent skills for applications built with
Hemx. Keep each skill self-contained under `skills/<name>/SKILL.md` and keep its
frontmatter name equal to its directory name.

The public Hemx README, crate READMEs, public APIs, Rustdoc, and tests are the
authority for supported framework behavior. Do not make skill guidance depend on
unpublished documentation, private examples, local repository paths, or
unreleased APIs.

Skills provide reusable judgment, not project-local policy or substitute
framework documentation. Keep routing narrow, preserve Hemx's public contracts,
and prefer the smallest instruction change that alters a real decision or
refusal.

## Verification

For skill changes:

- run `git diff --check`;
- confirm the frontmatter name matches the skill directory;
- review one positive trigger, the nearest negative trigger, and one tempting
  sprawl case;
- verify every named Hemx mechanism against the public Hemx source.

## Git conventions

Use Conventional Commits: `<type>(<scope>): <summary>`.
When branch-based delivery is used, name branches `<type>/<short-kebab-slug>`.
When publishing a versioned release, use an annotated SemVer tag such as
`v1.2.0`.

These naming rules do not grant authority to commit, push, rewrite history, or
publish a release.
