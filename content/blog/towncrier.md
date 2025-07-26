+++
title = 'Towncrier is a cleaner changelog utility'
date = 2025-07-25
draft = false
description = "Gone are the days of persistent merge conflicts with each changelog update. Now you can combined your changelog entries automatically!"

[taxonomies]
tags = ["programming", "python"]

[extra]
toc = false
+++

`towncrier` ([docs](https://towncrier.readthedocs.io/en/stable/index.html)) is a utility to produce changelogs from projects.
Instead of maintaining a changelog as a file that gets merge conflicts when different feature branches merge, you create
news fragments for each PR. These accumulate into the changelog at release.

To set it up, do the following things.

Add the following snippet to your pyproject.toml file:

```toml
[tool.gilesbot]
[tool.gilesbot.pull_requests]
enabled = true

[tool.gilesbot.towncrier_changelog]
enabled = true
changelog_skip_label = "no-changelog-entry-needed" # default is none
changelog_noop_label = "skip-changelog-checks"
whatsnew_label = "whatsnew-needed"

[tool.towncrier]
package = "punchbowl"
filename = "CHANGELOG.rst"
directory = "changelog/"
issue_format = "`#{issue} <https://github.com/punch-mission/punchbowl/pull/{issue}>`__"
title_format = "{version} ({project_date})"

[[tool.towncrier.type]]
directory = "breaking"
name = "Breaking Changes"
showcontent = true

[[tool.towncrier.type]]
directory = "deprecation"
name = "Deprecations"
showcontent = true

[[tool.towncrier.type]]
directory = "removal"
name = "Removals"
showcontent = true

[[tool.towncrier.type]]
directory = "feature"
name = "New Features"
showcontent = true

[[tool.towncrier.type]]
directory = "bugfix"
name = "Bug Fixes"
showcontent = true

[[tool.towncrier.type]]
directory = "doc"
name = "Documentation"
showcontent = true

[[tool.towncrier.type]]
directory = "trivial"
name = "Internal Changes"
showcontent = true
```

Be sure to replace `punchbowl` with your package name and user in the `package` and `issue_format` entries.

Then, enable giles as a bot if you want PR checks by [going here](https://github.com/apps/giles).
This will check each PR for a changelog update. If a PR doesn't need a changelog, then you can add the `no-changelog-entry-needed` label.

Now anytime you make a PR and have change, create a file in the `changelog` folder that ends in one of the following:

- breaking
- deprecation
- removal
- feature
- bugfix
- doc
- trivial

So for example, you might make a file `38.bugfix` if PR 38 is a bugfix.
In the file, you write a description of what that change is, e.g. "Fixed the bug where the refresh button didn't display."

Then, when you want to release simply run `towncrier` to populate the `CHANGELOG.rst`.

I'm starting to use this with PUNCH software to make development and maintenance easier.
