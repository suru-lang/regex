English | [日本語](README_jp.md)

# Regular expressions (a library for Suru)

A library for using [PCRE2](https://github.com/PCRE2Project/pcre2) regular expressions from [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese. It is not part of the standard library (`lib/`); use it as a kumu dependency.

## Installing

Suru 0.7.0 or later is required (the library writes procedures with 「以下の手順です … 以上です」).

Install PCRE2 (`libpcre2-8`, 10.30 or later) on the system.

| OS | How to install |
|---|---|
| macOS | `brew install pcre2` |
| Debian, Ubuntu | `sudo apt install libpcre2-dev` |
| Fedora | `sudo dnf install pcre2-devel` |

Add it to your project as a dependency.

```sh
kumu 追加 github.com/suru-lang/regex
```

If PCRE2 is not installed, kumu stops before building with 「システムのライブラリが足りません」 ("system libraries are missing") and shows how to install it (the `外部依存` and `導入案内` in this `組.suru`).

## Usage

```
「正規表現」 を 読み込んで 使う

日付表示する とは 以下の入出力手順です
  r とは 「今日は2026年11月23日です」 を 「(?<年>\d+)年(\d+)月」 で 正規検索した 結果
  r を 年表示する
  以上です

正規一致 の 有無 を 年表示する は 数 を 返す 入出力動作
（m という 有） を 年表示する とは （m から 「年」 を 正規名前取得する） を 表示する
無 を 年表示する とは 「見つかりません」 を 表示する

日付表示する
「a1b22c333」 を 「\d+」 から 「<$0>」 へ 正規置換してから 表示する     # a<1>b<22>c<333>
「あ、い、う」 を 「、」 で 正規分割してから 表示する                  # （「あ」 と 「い」 と 「う」）
```

With `「正規表現」 を 読み込む`, use the prefix, as in `正規表現内の正規検索する`.

| Syntax | Result |
|---|---|
| `s を p で 正規照合する` | Whether any part of s matches p (a boolean action) |
| `s を p で 正規検索する` | The first match: `m の 有` (m is a 正規一致) or `無` |
| `s を p で 正規全検索する` | All non-overlapping matches from the front (a list of 正規一致) |
| `s を p から r へ 正規置換する` | The string with every match replaced by r. `$0`, `$1` and `${名前}` in r are the match and the contents of ( ); `$$` is 「$」 |
| `s を p で 正規分割する` | The list of strings split at the matches |
| `p を 正規検査する` | 「」 if the pattern can be read; otherwise the reason, like 「2文字目: missing closing parenthesis」 ("character 2: …") |
| `m から 名前 を 正規名前取得する` | The string of `(?<名前>…)` (「」 if absent; a pure function) |
| `s を 正規逃避する` | A pattern that matches the text literally (「1+1」 → 「1\+1」; a pure function) |

Fields of a match `m` (type `正規一致`):

- `m の 正規全体`: the matched string
- `m の 正規位置`: the start position of the match (from 1, counted in characters)
- `m の 正規群`: the list of strings of ( ) groups (「」 for groups that didn't take part)
- `m の 正規名前群`: the list of names and strings of `(?<名前>…)` (`（（「年」 と 「2026」））`)

## Decisions

1. **Patterns are PCRE2 (the same syntax as Perl), read as UTF-8 with Unicode character classes.** `PCRE2_UTF` and `PCRE2_UCP` are set, so `.` matches one Japanese character, and `\w`, `\d` and `\b` match Japanese characters too. Case-insensitive (`(?i)`), multiline (`(?m)`) and dot-matches-newline (`(?s)`) are written at the head of the pattern.
2. **Positions count from 1, in characters.** To match the standard library's "positions count from 1", PCRE2's byte positions are converted to character counts.
3. **All are actions.** They use C memory, so matching, searching, replacing and splitting are actions (use them with `…した 結果` and `…してから`). A pattern that can't be read is the runtime error 「正規表現『(』を読めません(2文字目: missing closing parenthesis)」 ("can't read the regular expression 『(』 (…)"), catchable with `試行する`.
4. **The pattern is compiled every time and freed after use.** Each call runs `pcre2_compile` and frees with `pcre2_code_free` and the like when done, so no memory is left behind.
5. **After an empty match, it behaves like Perl.** When a full search or split hits an empty match, it looks for a non-empty match at the same place (`PCRE2_NOTEMPTY_ATSTART`), and otherwise advances one character.
6. **`${名前}` in replacements can use Japanese names.** PCRE2 replacements only read ASCII names, so `${年}` is rewritten to a number like `${1}` before being passed.
7. **No match is `無`.** The result of a search is Suru's basic `有無` (`m の 有` or `無`). `取出する`, `写像する` and `連鎖する` work on it as they are.
8. **C functions are called as foreign functions.** `pcre2_match_8` takes seven arguments, so it is called with the form `外部の「pcre2_match_8」（c と 対象 と … と 0）` (Suru's "Argument lists for foreign functions").

## Tests

```sh
kumu 試験
```

## License

MIT ([LICENSE](LICENSE)). PCRE2 is under the BSD license.
