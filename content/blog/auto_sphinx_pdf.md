+++
title = "Automatically building Sphinx PDF in a GitHub Action"
date = 2026-01-13
draft = false
description = "I created a GitHub workflow to automatically build documentation and upload the PDF on pull requests in GitHub."

[taxonomies]
tags = ["programming", "python"]
+++

I have a project at work where I need to automatically build the PDF version of Sphinx documentation. 
Since our code is managed on GitHub, I thought it was easiest to use a GitHub action on each pull request. 
I tried several different pre-built actions but none of them worked properly or were maintained. 
The final solution was to just install everything in the action:

```yaml
name: Documentation PDF Generation
on:
  push:

jobs:
  texlive:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.13'
      - name: Install Python dependencies
        run: |
          python -m pip install --upgrade pip
          pip install ".[docs]"
      - name: Install Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install mermaid-cli
        run: npm install -g @mermaid-js/mermaid-cli
      - name: Build docs
        run: |
          sudo apt-get update
          sudo apt-get install texlive-latex-recommended texlive-latex-extra texlive-fonts-recommended latexmk
          cd docs/
          make latexpdf
      - name: Upload PDF
        uses: actions/upload-artifact@v4
        with:
          name: documentation
          path: docs/_build/latex/*.pdf
```

We also install Mermaid since we have some Mermaid plots in our documentation.
It uploads the PDF as an artifact to the GitHub Action for easy download. 
It certainly saves time instead of pulling, building the docs manually each time, and then inspecting.
