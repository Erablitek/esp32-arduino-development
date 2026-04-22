# Contributing to esp32-arduino-development

## Versioning rules (Semantic Versioning 2.0.0)

This skill follows [SemVer](https://semver.org/). The version is maintained in
**two places that must always stay in sync**:

1. `SKILL.md` frontmatter: `version: X.Y.Z`
2. `CHANGELOG.md` `## [X.Y.Z] - YYYY-MM-DD` section
3. Git tag: `vX.Y.Z` (created at release time)

### When to bump which number

Given a change, answer these questions in order:

**Is it a breaking change?** (existing projects will stop working, e.g., schema
change in `project.json`, removal of a trigger phrase, renamed fields, changed
default behavior that affects correctness)
→ Bump MAJOR: `1.x.y` → `2.0.0`

**Is it a new feature?** (new action, new optional field, new supported board,
new command — but existing projects keep working)
→ Bump MINOR: `1.2.x` → `1.3.0`

**Is it a fix or improvement?** (bug fix, typo, clarification, refactoring
documentation, improved error messages)
→ Bump PATCH: `1.2.0` → `1.2.1`

### Pre-release versions (optional)

For work-in-progress that should not be considered stable:
- `1.3.0-alpha.1` — early draft
- `1.3.0-beta.1` — feature-complete but testing
- `1.3.0-rc.1` — release candidate

## Changelog discipline

Every change that touches `SKILL.md`, `templates/`, or `references/` MUST be
logged in `CHANGELOG.md` under the `## [Unreleased]` section before committing.

Categories (from Keep a Changelog):
- **Added** — new features
- **Changed** — changes in existing functionality
- **Deprecated** — soon-to-be removed features
- **Removed** — features that have been removed
- **Fixed** — bug fixes
- **Security** — security-related changes

## Release process

When ready to release a new version:

1. Move entries from `## [Unreleased]` to a new `## [X.Y.Z] - YYYY-MM-DD` section
2. Update `version:` in `SKILL.md` frontmatter
3. Update `date:` in `SKILL.md` frontmatter
4. Update the compare links at the bottom of `CHANGELOG.md`
5. Commit with message: `Release vX.Y.Z`
6. Tag the commit: `git tag -a vX.Y.Z -m "Release vX.Y.Z"`
7. Push both the commit and the tag: `git push && git push --tags`

## Commit message convention

Not enforced, but recommended for readability:

- `feat: ...` — new feature
- `fix: ...` — bug fix
- `docs: ...` — documentation only
- `refactor: ...` — internal restructure, no behavior change
- `chore: ...` — maintenance (CI, deps, tooling)
- `release: vX.Y.Z` — version release commit

## When Claude modifies the skill

If Claude (via Claude Code or similar) is asked to modify this skill, Claude
should:

1. Make the requested changes
2. Determine the appropriate version bump (patch / minor / major) based on the rules above
3. Update `SKILL.md` frontmatter `version:` field
4. Add an entry under `## [Unreleased]` in `CHANGELOG.md` with the correct category
5. If the user explicitly says "release this version" or similar, follow the release process above
6. Otherwise, leave the changes in `Unreleased` for later grouping
