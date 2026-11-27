# 「正規表現」

Regular expressions for Suru. It calls PCRE2 (libpcre2-8) through external functions.
Write `「正規表現」 を 読み込んで 使う` in your file. Every name starts with `正規`.

Patterns use the PCRE2 syntax (the same as Perl), read as UTF-8 with Unicode character classes.
`.` is one character (one Japanese character is one too), and `\w`, `\d` and `\b` count Japanese characters as well.
Put `(?i)` at the head of a pattern to ignore case, `(?m)` for multi-line, and `(?s)` to let `.` match a newline.
Positions start at 1 and count characters, not bytes.

A match is the record `正規一致`:

- `m の 正規全体`: the matched string.
- `m の 正規位置`: where the match starts (from 1, in characters).
- `m の 正規群`: the strings of the `( )` groups (「」 for a group that did not match).
- `m の 正規名前群`: the pairs of name and string of the `(?<名前>…)` groups, like `（「年」 と 「2026」）`.

The functions that run a pattern are actions. A pattern that cannot be read is a runtime error, which `試行する` can catch.

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
