---
tags:
  - linux
  - cheatsheet
---

# AWK

`AWK` is a versatile programming language for working with data files,
particularly files that are organized into records (lines) and fields (columns).

Here's the basic structure of an AWK command:

```bash
awk 'pattern { action }' filename
```

**Pattern**: This is optional. When provided, AWK will search the input for
lines that match the pattern, and for each matching line, it will perform the
action. If no pattern is provided, AWK will perform the action for every line.

**Action**: This is what AWK does when it finds a line that matches the pattern.
If no action is specified, the default action is to print the line.

**Filename**: The name of the file AWK will process. If not specified, AWK will
read from the standard input.

This document is a very basic introduction to AWK. There's much more you can do
with it, including using conditionals and loops, defining functions, and more.

Let's break down a few examples to understand the basics

### Print all lines in a file:

```bash
awk '{ print }' filename
```

In this case, we didn't specify a pattern, so AWK will perform the action
(`print`) for every line. The print action without any parameters will print the
entire line.

### Print specific fields:

In AWK, fields are referenced as `$1`, `$2`, `$3`, etc., where `$1` is the first
field, `$2` is the second, and so on. `$0` refers to the entire line.

```bash
awk '{ print $1, $3 }' filename
```

This command will print the first and third fields of each line.

### Use a pattern to filter lines:

```bash
awk '/pattern/ { print $1, $3 }' filename
```

This command will print the first and third fields of each line that matches the
pattern. For example, if '`pattern`' were '`error`', AWK would only print the
first and third fields of lines that contain the word 'error'.

### Use variables and arithmetic:

AWK supports variables and can perform arithmetic. Here's a simple example:

```bash
awk '{ sum += $1 } END { print sum }' filename
```

In this case, AWK will go through each line, adding the value of the first field
to the '`sum`' variable. After it has processed all lines, it will print the
`sum`.

### Built-in variables:

AWK has some built-in variables that you can use:

- `NR`: The current line number.
- `NF`: The number of fields in the current line.
- `FS`: The field separator (default is white space).
- `OFS`: The output field separator (default is a space).

Here's an example that uses some built-in variables:

```bash
awk '{ print NR, $0, NF }' filename
```

This command will print the line number, the entire line, and the number of
fields in each line.