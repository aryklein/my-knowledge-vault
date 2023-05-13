---
tags: neovim, cheatsheet
---

# Neovim cheat sheet

As a Neovim user, this document will focus on providing a reference for commands
and shortcuts that I often find challenging to remember. Please note that it is
not intended to be a comprehensive tutorial on how to use Neovim. Basic commands
won't be documented here.

---

# How to break a long line into multiple lines

When editing Markdown files or writing plain text documentation, I find the
following command extremely useful as it helps maintain lines within the
recommended limit of 80 characters while breaking them at word boundaries. To
enable automatic line breaking when you exceed the character limit while typing,
you can set the `textwidth` (`tw`) option:

```
:set textwidth=80
```

After setting the `textwidth` option, you can reformat specific lines by
highlighting the lines you want to modify and executing the following command:

```
gq
```

If you want to ensure that already present line breaks are not modified during
the reformatting process, you need to add the `w` option to the `formatoptions`
setting:

```
:set formatoptions+=w
```

By adding the `w` option to `formatoptions`, Neovim will intelligently split
long lines at word boundaries while preserving existing line breaks.
