+++
title = 'Adding codespell as a pre-commit and CI'
date = 2024-06-15
draft = false

[taxonomies]
tags = ["programming", "python"]
+++

I have some repos for work on GitHub that are private and were accumulating spelling errors in both the documentation and the code.
I decided to add [codespell](https://github.com/codespell-project/codespell) to them to catch spelling errors.
I'm not sure if I did it the best way, but it worked so that's what mattered. Here are notes of what I set up.

I added codespell to the pre-commit hook and to the GitHub Action for continuous integration. I did both because I wanted to make sure
we caught spelling errors even if a contributor wasn't complying with the pre-commit. When the repo is public and we can add the
pre-commit action via GitHub, I may remove the CI step because it would be extra redundant. Although, it's pretty fast so it doesn't really matter.

## Adding to pre-commit

To add to the pre-commit I just added a section to my `.pre=commit-config.yaml` file:

```yaml
- repo: https://github.com/codespell-project/codespell
    rev: v2.1.0
    hooks:
      - id: codespell
        files: ^.*\.(py|c|h|md|rst|yml)$
        args: ["--ignore-words", ".codespellignore" ]
```

This addition forces it to run on all Python, C, Header, Markdown, Restructured Test, and YAML files. It ignores specific words that have
been defined in the `.codespellignore` file, one word per line.

## Adding to GitHub Action

To add to the GitHub Action CI, I added the following two sections to the `steps` part of the Action:

```yaml
- name: Run codespell on source code
    uses: codespell-project/actions-codespell@v2
    with:
        skip: '*.fits'
        ignore_words_file: .codespellignore
        path: punchbowl
- name: Run codespell on documentation
    uses: codespell-project/actions-codespell@v2
    with:
    skip: '*.fits'
    ignore_words_file: .codespellignore
    path: docs/source
```

I'm not sure if there's a way to make it run on both the `punchbowl` and `docs/source` directories with one call, but I did it the fast route.
I realize now that the pre-commit and GitHub Action check different files. The pre-commit checks only files with certain extensions while the
Action checks all file in those directories that are not FITS files. We might want to change that to be consistent. At least they share the same
`.codespellignore` so we don't have differences there.
