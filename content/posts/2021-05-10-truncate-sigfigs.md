---
title: Truncate to specified number of significant figures
date: 2021-05-10
draft: false
---

It's a simple request. You have a number, e.g. 1.25, and want to truncate it to say 2 significant figures. You specifically need to truncate instead of rounding, i.e. you need to get 1.2 instead of 1.3. How do you do that quickly in Python? Check out this code snippet:

<script src="https://gist.github.com/jmbhughes/e24ed7bcfa370094ef47bf09f4601c3e.js"></script>

It's based on [this](https://stackoverflow.com/a/3413529/10046967) and [this](https://stackoverflow.com/a/39165933/10046967) on stackoverflow.
