# sed

`sed` is a powerful command-line utility used for parsing and transforming text
in files or streams. It can perform basic text manipulations like searching,
finding and replacing, inserting, and deleting lines of text based on patterns.
sed is particularly useful for automated editing tasks and processing large
amounts of text data efficiently.

This document contains useful sed commands that help solve specific problems. It
is not a tutorial on how to use sed.
## Remove blank lines at the end of a file

```bash
sed -i -e :a -e '/^\n*$/{$d;N;ba' -e '}' file
```
