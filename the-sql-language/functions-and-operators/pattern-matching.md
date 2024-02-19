---
description: 版本：11
---

# 9.7. 特徵比對

PostgreSQL 提供了三種不同的特徵比對方法：傳統的 SQL LIKE 運算子，最新的 SIMILAR TO 運算子（於 SQL：1999 中加入）和 POSIX 樣式的正規表示式。除了基本的「這個字串符合這個樣式嗎？」運算子之外，還可以使用函數來提取或替換符合的子字串，以及在配對的位置拆分字串。

**提醒**\
如果您的特徵比對需求超出此範圍，請考慮在 Perl 或 Tcl 中撰寫使用者定義的函數。

#### 注意

雖然大多數正規表示式搜尋可以非常快速地執行，但是完成正規表示式需要花費大量的時間和記憶體來處理。要特別注意從各種來源接受正規表示式的搜尋方式。如果必須這樣做，建議強制限制執行語句執行時間。

使用 SIMILAR TO 方式的搜尋具有相同的安全隱憂，因為 SIMILAR TO 提供了許多與 POSIX 樣式的正規表示式相同功能。

LIKE 搜尋比其他兩個選項要簡單得多，在使用可能惡意的來源時更安全。

## 9.7.1. `LIKE`

```
string LIKE pattern [ESCAPE escape-character]
string NOT LIKE pattern [ESCAPE escape-character]
```

The `LIKE` expression returns true if the _`string`_ matches the supplied _`pattern`_. (As expected, the `NOT LIKE` expression returns false if `LIKE` returns true, and vice versa. An equivalent expression is `NOT (`_`string`_ LIKE _`pattern`_).)

If _`pattern`_ does not contain percent signs or underscores, then the pattern only represents the string itself; in that case `LIKE` acts like the equals operator. An underscore (`_`) in _`pattern`_ stands for (matches) any single character; a percent sign (`%`) matches any sequence of zero or more characters.

Some examples:

```
'abc' LIKE 'abc'    true
'abc' LIKE 'a%'     true
'abc' LIKE '_b_'    true
'abc' LIKE 'c'      false
```

`LIKE` pattern matching always covers the entire string. Therefore, if it's desired to match a sequence anywhere within a string, the pattern must start and end with a percent sign.

To match a literal underscore or percent sign without matching other characters, the respective character in _`pattern`_ must be preceded by the escape character. The default escape character is the backslash but a different one can be selected by using the `ESCAPE` clause. To match the escape character itself, write two escape characters.

#### Note

If you have [standard\_conforming\_strings](https://www.postgresql.org/docs/11/runtime-config-compatible.html#GUC-STANDARD-CONFORMING-STRINGS) turned off, any backslashes you write in literal string constants will need to be doubled. See [Section 4.1.2.1](https://www.postgresql.org/docs/11/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS) for more information.

It's also possible to select no escape character by writing `ESCAPE ''`. This effectively disables the escape mechanism, which makes it impossible to turn off the special meaning of underscore and percent signs in the pattern.

The key word `ILIKE` can be used instead of `LIKE` to make the match case-insensitive according to the active locale. This is not in the SQL standard but is a PostgreSQL extension.

The operator `~~` is equivalent to `LIKE`, and `~~*` corresponds to `ILIKE`. There are also `!~~` and `!~~*` operators that represent `NOT LIKE` and `NOT ILIKE`, respectively. All of these operators are PostgreSQL-specific.

There is also the prefix operator `^@` and corresponding `starts_with` function which covers cases when only searching by beginning of the string is needed.

## 9.7.2. `SIMILAR TO` Regular Expressions

```
string SIMILAR TO pattern [ESCAPE escape-character]
string NOT SIMILAR TO pattern [ESCAPE escape-character]
```

The `SIMILAR TO` operator returns true or false depending on whether its pattern matches the given string. It is similar to `LIKE`, except that it interprets the pattern using the SQL standard's definition of a regular expression. SQL regular expressions are a curious cross between `LIKE` notation and common regular expression notation.

Like `LIKE`, the `SIMILAR TO` operator succeeds only if its pattern matches the entire string; this is unlike common regular expression behavior where the pattern can match any part of the string. Also like `LIKE`, `SIMILAR TO` uses `_` and `%` as wildcard characters denoting any single character and any string, respectively (these are comparable to `.` and `.*` in POSIX regular expressions).

In addition to these facilities borrowed from `LIKE`, `SIMILAR TO` supports these pattern-matching metacharacters borrowed from POSIX regular expressions:

* `|` denotes alternation (either of two alternatives).
* `*` denotes repetition of the previous item zero or more times.
* `+` denotes repetition of the previous item one or more times.
* `?` denotes repetition of the previous item zero or one time.
* `{`_`m`_`}` denotes repetition of the previous item exactly _`m`_ times.
* `{`_`m`_`,}` denotes repetition of the previous item _`m`_ or more times.
* `{`_`m`_`,`_`n`_`}` denotes repetition of the previous item at least _`m`_ and not more than _`n`_ times.
* Parentheses `()` can be used to group items into a single logical item.
* A bracket expression `[...]` specifies a character class, just as in POSIX regular expressions.

Notice that the period (`.`) is not a metacharacter for `SIMILAR TO`.

As with `LIKE`, a backslash disables the special meaning of any of these metacharacters; or a different escape character can be specified with `ESCAPE`.

Some examples:

```
'abc' SIMILAR TO 'abc'      true
'abc' SIMILAR TO 'a'        false
'abc' SIMILAR TO '%(b|d)%'  true
'abc' SIMILAR TO '(b|c)%'   false
```

The `substring` function with three parameters, `substring(`_`string`_ from _`pattern`_ for _`escape-character`_), provides extraction of a substring that matches an SQL regular expression pattern. As with `SIMILAR TO`, the specified pattern must match the entire data string, or else the function fails and returns null. To indicate the part of the pattern that should be returned on success, the pattern must contain two occurrences of the escape character followed by a double quote (`"`). The text matching the portion of the pattern between these markers is returned.

Some examples, with `#"` delimiting the return string:

```
substring('foobar' from '%#"o_b#"%' for '#')   oob
substring('foobar' from '#"o_b#"%' for '#')    NULL
```

## 9.7.3. POSIX Regular Expressions

[Table 9.14](https://www.postgresql.org/docs/11/functions-matching.html#FUNCTIONS-POSIX-TABLE) lists the available operators for pattern matching using POSIX regular expressions.

#### **Table 9.14. Regular Expression Match Operators**

| Operator | Description                                         | Example                    |
| -------- | --------------------------------------------------- | -------------------------- |
| `~`      | Matches regular expression, case sensitive          | `'thomas' ~ '.*thomas.*'`  |
| `~*`     | Matches regular expression, case insensitive        | `'thomas' ~* '.*Thomas.*'` |
| `!~`     | Does not match regular expression, case sensitive   | `'thomas' !~ '.*Thomas.*'` |
| `!~*`    | Does not match regular expression, case insensitive | `'thomas' !~* '.*vadim.*'` |

POSIX regular expressions provide a more powerful means for pattern matching than the `LIKE` and `SIMILAR TO` operators. Many Unix tools such as `egrep`, `sed`, or `awk` use a pattern matching language that is similar to the one described here.

A regular expression is a character sequence that is an abbreviated definition of a set of strings (a _regular set_). A string is said to match a regular expression if it is a member of the regular set described by the regular expression. As with `LIKE`, pattern characters match string characters exactly unless they are special characters in the regular expression language — but regular expressions use different special characters than `LIKE` does. Unlike `LIKE` patterns, a regular expression is allowed to match anywhere within a string, unless the regular expression is explicitly anchored to the beginning or end of the string.

Some examples:

```
'abc' ~ 'abc'    true
'abc' ~ '^a'     true
'abc' ~ '(b|d)'  true
'abc' ~ '^(b|c)' false
```

The POSIX pattern language is described in much greater detail below.

The `substring` function with two parameters, `substring(`_`string`_ from _`pattern`_), provides extraction of a substring that matches a POSIX regular expression pattern. It returns null if there is no match, otherwise the portion of the text that matched the pattern. But if the pattern contains any parentheses, the portion of the text that matched the first parenthesized subexpression (the one whose left parenthesis comes first) is returned. You can put parentheses around the whole expression if you want to use parentheses within it without triggering this exception. If you need parentheses in the pattern before the subexpression you want to extract, see the non-capturing parentheses described below.

Some examples:

```
substring('foobar' from 'o.b')     oob
substring('foobar' from 'o(.)b')   o
```

The `regexp_replace` function provides substitution of new text for substrings that match POSIX regular expression patterns. It has the syntax `regexp_replace`(_`source`_, _`pattern`_, _`replacement`_ \[, _`flags`_ ]). The _`source`_ string is returned unchanged if there is no match to the _`pattern`_. If there is a match, the _`source`_ string is returned with the _`replacement`_ string substituted for the matching substring. The _`replacement`_ string can contain `\`_`n`_, where _`n`_ is 1 through 9, to indicate that the source substring matching the _`n`_'th parenthesized subexpression of the pattern should be inserted, and it can contain `\&` to indicate that the substring matching the entire pattern should be inserted. Write `\\` if you need to put a literal backslash in the replacement text. The _`flags`_ parameter is an optional text string containing zero or more single-letter flags that change the function's behavior. Flag `i` specifies case-insensitive matching, while flag `g` specifies replacement of each matching substring rather than only the first one. Supported flags (though not `g`) are described in [Table 9.22](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-EMBEDDED-OPTIONS-TABLE).

Some examples:

```
regexp_replace('foobarbaz', 'b..', 'X')
                                   fooXbaz
regexp_replace('foobarbaz', 'b..', 'X', 'g')
                                   fooXX
regexp_replace('foobarbaz', 'b(..)', 'X\1Y', 'g')
                                   fooXarYXazY
```

The `regexp_match` function returns a text array of captured substring(s) resulting from the first match of a POSIX regular expression pattern to a string. It has the syntax `regexp_match`(_`string`_, _`pattern`_ \[, _`flags`_ ]). If there is no match, the result is `NULL`. If a match is found, and the _`pattern`_ contains no parenthesized subexpressions, then the result is a single-element text array containing the substring matching the whole pattern. If a match is found, and the _`pattern`_ contains parenthesized subexpressions, then the result is a text array whose _`n`_'th element is the substring matching the _`n`_'th parenthesized subexpression of the _`pattern`_ (not counting “non-capturing” parentheses; see below for details). The _`flags`_ parameter is an optional text string containing zero or more single-letter flags that change the function's behavior. Supported flags are described in [Table 9.22](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-EMBEDDED-OPTIONS-TABLE).

Some examples:

```
SELECT regexp_match('foobarbequebaz', 'bar.*que');
 regexp_match
--------------
 {barbeque}
(1 row)

SELECT regexp_match('foobarbequebaz', '(bar)(beque)');
 regexp_match
--------------
 {bar,beque}
(1 row)
```

In the common case where you just want the whole matching substring or `NULL` for no match, write something like

```
SELECT (regexp_match('foobarbequebaz', 'bar.*que'))[1];
 regexp_match
--------------
 barbeque
(1 row)
```

The `regexp_matches` function returns a set of text arrays of captured substring(s) resulting from matching a POSIX regular expression pattern to a string. It has the same syntax as `regexp_match`. This function returns no rows if there is no match, one row if there is a match and the `g` flag is not given, or _`N`_ rows if there are _`N`_ matches and the `g` flag is given. Each returned row is a text array containing the whole matched substring or the substrings matching parenthesized subexpressions of the _`pattern`_, just as described above for `regexp_match`. `regexp_matches` accepts all the flags shown in [Table 9.22](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-EMBEDDED-OPTIONS-TABLE), plus the `g` flag which commands it to return all matches, not just the first one.

Some examples:

```
SELECT regexp_matches('foo', 'not there');
 regexp_matches
----------------
(0 rows)

SELECT regexp_matches('foobarbequebazilbarfbonk', '(b[^b]+)(b[^b]+)', 'g');
 regexp_matches
----------------
 {bar,beque}
 {bazil,barf}
(2 rows)
```

#### Tip

In most cases `regexp_matches()` should be used with the `g` flag, since if you only want the first match, it's easier and more efficient to use `regexp_match()`. However,`regexp_match()` only exists in PostgreSQL version 10 and up. When working in older versions, a common trick is to place a `regexp_matches()` call in a sub-select, for example:

```
SELECT col1, (SELECT regexp_matches(col2, '(bar)(beque)')) FROM tab;
```

This produces a text array if there's a match, or `NULL` if not, the same as `regexp_match()`would do. Without the sub-select, this query would produce no output at all for table rows without a match, which is typically not the desired behavior.

The `regexp_split_to_table` function splits a string using a POSIX regular expression pattern as a delimiter. It has the syntax `regexp_split_to_table`(_`string`_, _`pattern`_ \[, _`flags`_ ]). If there is no match to the _`pattern`_, the function returns the _`string`_. If there is at least one match, for each match it returns the text from the end of the last match (or the beginning of the string) to the beginning of the match. When there are no more matches, it returns the text from the end of the last match to the end of the string. The _`flags`_ parameter is an optional text string containing zero or more single-letter flags that change the function's behavior. `regexp_split_to_table` supports the flags described in [Table 9.22](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-EMBEDDED-OPTIONS-TABLE).

The `regexp_split_to_array` function behaves the same as `regexp_split_to_table`, except that `regexp_split_to_array` returns its result as an array of `text`. It has the syntax `regexp_split_to_array`(_`string`_, _`pattern`_ \[, _`flags`_ ]). The parameters are the same as for `regexp_split_to_table`.

Some examples:

```
SELECT foo FROM regexp_split_to_table('the quick brown fox jumps over the lazy dog', '\s+') AS foo;
  foo   
-------
 the    
 quick  
 brown  
 fox    
 jumps 
 over   
 the    
 lazy   
 dog    
(9 rows)

SELECT regexp_split_to_array('the quick brown fox jumps over the lazy dog', '\s+');
              regexp_split_to_array             
-----------------------------------------------
 {the,quick,brown,fox,jumps,over,the,lazy,dog}
(1 row)

SELECT foo FROM regexp_split_to_table('the quick brown fox', '\s*') AS foo;
 foo 
-----
 t         
 h         
 e         
 q         
 u         
 i         
 c         
 k         
 b         
 r         
 o         
 w         
 n         
 f         
 o         
 x         
(16 rows)
```

As the last example demonstrates, the regexp split functions ignore zero-length matches that occur at the start or end of the string or immediately after a previous match. This is contrary to the strict definition of regexp matching that is implemented by `regexp_match` and `regexp_matches`, but is usually the most convenient behavior in practice. Other software systems such as Perl use similar definitions.

### **9.7.3.1. Regular Expression Details**

PostgreSQL's regular expressions are implemented using a software package written by Henry Spencer. Much of the description of regular expressions below is copied verbatim from his manual.

Regular expressions (REs), as defined in POSIX 1003.2, come in two forms: _extended_ REs or EREs (roughly those of `egrep`), and _basic_ REs or BREs (roughly those of `ed`). PostgreSQL supports both forms, and also implements some extensions that are not in the POSIX standard, but have become widely used due to their availability in programming languages such as Perl and Tcl. REs using these non-POSIX extensions are called _advanced_ REs or AREs in this documentation. AREs are almost an exact superset of EREs, but BREs have several notational incompatibilities (as well as being much more limited). We first describe the ARE and ERE forms, noting features that apply only to AREs, and then describe how BREs differ.

#### Note

PostgreSQL always initially presumes that a regular expression follows the ARE rules. However, the more limited ERE or BRE rules can be chosen by prepending an _embedded option_ to the RE pattern, as described in [Section 9.7.3.4](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-METASYNTAX). This can be useful for compatibility with applications that expect exactly the POSIX 1003.2 rules.

A regular expression is defined as one or more _branches_, separated by `|`. It matches anything that matches one of the branches.

A branch is zero or more _quantified atoms_ or _constraints_, concatenated. It matches a match for the first, followed by a match for the second, etc; an empty branch matches the empty string.

A quantified atom is an _atom_ possibly followed by a single _quantifier_. Without a quantifier, it matches a match for the atom. With a quantifier, it can match some number of matches of the atom. An _atom_ can be any of the possibilities shown in [Table 9.15](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-ATOMS-TABLE). The possible quantifiers and their meanings are shown in [Table 9.16](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-QUANTIFIERS-TABLE).

A _constraint_ matches an empty string, but matches only when specific conditions are met. A constraint can be used where an atom could be used, except it cannot be followed by a quantifier. The simple constraints are shown in [Table 9.17](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-CONSTRAINTS-TABLE); some more constraints are described later.

#### **Table 9.15. Regular Expression Atoms**

| Atom            | Description                                                                                                                                                                                                                                    |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `(`_`re`_`)`    | (where _`re`_ is any regular expression) matches a match for _`re`_, with the match noted for possible reporting                                                                                                                               |
| `(?:`_`re`_`)`  | as above, but the match is not noted for reporting (a “non-capturing” set of parentheses) (AREs only)                                                                                                                                          |
| `.`             | matches any single character                                                                                                                                                                                                                   |
| `[`_`chars`_`]` | a _bracket expression_, matching any one of the _`chars`_ (see [Section 9.7.3.2](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-BRACKET-EXPRESSIONS) for more detail)                                                        |
| `\`_`k`_        | (where _`k`_ is a non-alphanumeric character) matches that character taken as an ordinary character, e.g., `\\` matches a backslash character                                                                                                  |
| `\`_`c`_        | where _`c`_ is alphanumeric (possibly followed by other characters) is an _escape_, see [Section 9.7.3.3](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-ESCAPE-SEQUENCES) (AREs only; in EREs and BREs, this matches _`c`_) |
| `{`             | when followed by a character other than a digit, matches the left-brace character `{`; when followed by a digit, it is the beginning of a _`bound`_ (see below)                                                                                |
| _`x`_           | where _`x`_ is a single character with no other significance, matches that character                                                                                                                                                           |

An RE cannot end with a backslash (`\`).

#### Note

If you have [standard\_conforming\_strings](https://www.postgresql.org/docs/11/runtime-config-compatible.html#GUC-STANDARD-CONFORMING-STRINGS) turned off, any backslashes you write in literal string constants will need to be doubled. See [Section 4.1.2.1](https://www.postgresql.org/docs/11/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS) for more information.

#### **Table 9.16. Regular Expression Quantifiers**

| Quantifier           | Matches                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------- |
| `*`                  | a sequence of 0 or more matches of the atom                                                  |
| `+`                  | a sequence of 1 or more matches of the atom                                                  |
| `?`                  | a sequence of 0 or 1 matches of the atom                                                     |
| `{`_`m`_`}`          | a sequence of exactly _`m`_ matches of the atom                                              |
| `{`_`m`_`,}`         | a sequence of _`m`_ or more matches of the atom                                              |
| `{`_`m`_`,`_`n`_`}`  | a sequence of _`m`_ through _`n`_ (inclusive) matches of the atom; _`m`_ cannot exceed _`n`_ |
| `*?`                 | non-greedy version of `*`                                                                    |
| `+?`                 | non-greedy version of `+`                                                                    |
| `??`                 | non-greedy version of `?`                                                                    |
| `{`_`m`_`}?`         | non-greedy version of `{`_`m`_`}`                                                            |
| `{`_`m`_`,}?`        | non-greedy version of `{`_`m`_`,}`                                                           |
| `{`_`m`_`,`_`n`_`}?` | non-greedy version of `{`_`m`_`,`_`n`_`}`                                                    |

The forms using `{`_`...`_`}` are known as _bounds_. The numbers _`m`_ and _`n`_ within a bound are unsigned decimal integers with permissible values from 0 to 255 inclusive.

_Non-greedy_ quantifiers (available in AREs only) match the same possibilities as their corresponding normal (_greedy_) counterparts, but prefer the smallest number rather than the largest number of matches. See [Section 9.7.3.5](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-MATCHING-RULES) for more detail.

#### Note

A quantifier cannot immediately follow another quantifier, e.g., `**` is invalid. A quantifier cannot begin an expression or subexpression or follow `^` or `|`.

#### **Table 9.17. Regular Expression Constraints**

| Constraint      | Description                                                                                     |
| --------------- | ----------------------------------------------------------------------------------------------- |
| `^`             | matches at the beginning of the string                                                          |
| `$`             | matches at the end of the string                                                                |
| `(?=`_`re`_`)`  | _positive lookahead_ matches at any point where a substring matching _`re`_ begins (AREs only)  |
| `(?!`_`re`_`)`  | _negative lookahead_ matches at any point where no substring matching _`re`_ begins (AREs only) |
| `(?<=`_`re`_`)` | _positive lookbehind_ matches at any point where a substring matching _`re`_ ends (AREs only)   |
| `(?<!`_`re`_`)` | _negative lookbehind_ matches at any point where no substring matching _`re`_ ends (AREs only)  |

Lookahead and lookbehind constraints cannot contain _back references_ (see [Section 9.7.3.3](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-ESCAPE-SEQUENCES)), and all parentheses within them are considered non-capturing.

### **9.7.3.2. Bracket Expressions**

A _bracket expression_ is a list of characters enclosed in `[]`. It normally matches any single character from the list (but see below). If the list begins with `^`, it matches any single character _not_ from the rest of the list. If two characters in the list are separated by `-`, this is shorthand for the full range of characters between those two (inclusive) in the collating sequence, e.g., `[0-9]` in ASCII matches any decimal digit. It is illegal for two ranges to share an endpoint, e.g., `a-c-e`. Ranges are very collating-sequence-dependent, so portable programs should avoid relying on them.

To include a literal `]` in the list, make it the first character (after `^`, if that is used). To include a literal `-`, make it the first or last character, or the second endpoint of a range. To use a literal `-` as the first endpoint of a range, enclose it in `[.` and `.]` to make it a collating element (see below). With the exception of these characters, some combinations using `[` (see next paragraphs), and escapes (AREs only), all other special characters lose their special significance within a bracket expression. In particular, `\` is not special when following ERE or BRE rules, though it is special (as introducing an escape) in AREs.

Within a bracket expression, a collating element (a character, a multiple-character sequence that collates as if it were a single character, or a collating-sequence name for either) enclosed in `[.` and `.]`stands for the sequence of characters of that collating element. The sequence is treated as a single element of the bracket expression's list. This allows a bracket expression containing a multiple-character collating element to match more than one character, e.g., if the collating sequence includes a `ch` collating element, then the RE `[[.ch.]]*c` matches the first five characters of `chchcc`.

#### Note

PostgreSQL currently does not support multi-character collating elements. This information describes possible future behavior.

Within a bracket expression, a collating element enclosed in `[=` and `=]` is an _equivalence class_, standing for the sequences of characters of all collating elements equivalent to that one, including itself. (If there are no other equivalent collating elements, the treatment is as if the enclosing delimiters were `[.` and `.]`.) For example, if `o` and `^` are the members of an equivalence class, then `[[=o=]]`, `[[=^=]]`, and `[o^]` are all synonymous. An equivalence class cannot be an endpoint of a range.

Within a bracket expression, the name of a character class enclosed in `[:` and `:]` stands for the list of all characters belonging to that class. Standard character class names are: `alnum`, `alpha`, `blank`,`cntrl`, `digit`, `graph`, `lower`, `print`, `punct`, `space`, `upper`, `xdigit`. These stand for the character classes defined in ctype. A locale can provide others. A character class cannot be used as an endpoint of a range.

There are two special cases of bracket expressions: the bracket expressions `[[:<:]]` and `[[:>:]]` are constraints, matching empty strings at the beginning and end of a word respectively. A word is defined as a sequence of word characters that is neither preceded nor followed by word characters. A word character is an `alnum` character (as defined by ctype) or an underscore. This is an extension, compatible with but not specified by POSIX 1003.2, and should be used with caution in software intended to be portable to other systems. The constraint escapes described below are usually preferable; they are no more standard, but are easier to type.

### **9.7.3.3. Regular Expression Escapes**

_Escapes_ are special sequences beginning with `\` followed by an alphanumeric character. Escapes come in several varieties: character entry, class shorthands, constraint escapes, and back references. A `\` followed by an alphanumeric character but not constituting a valid escape is illegal in AREs. In EREs, there are no escapes: outside a bracket expression, a `\` followed by an alphanumeric character merely stands for that character as an ordinary character, and inside a bracket expression, `\` is an ordinary character. (The latter is the one actual incompatibility between EREs and AREs.)

_Character-entry escapes_ exist to make it easier to specify non-printing and other inconvenient characters in REs. They are shown in [Table 9.18](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-CHARACTER-ENTRY-ESCAPES-TABLE).

_Class-shorthand escapes_ provide shorthands for certain commonly-used character classes. They are shown in [Table 9.19](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-CLASS-SHORTHAND-ESCAPES-TABLE).

A _constraint escape_ is a constraint, matching the empty string if specific conditions are met, written as an escape. They are shown in [Table 9.20](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-CONSTRAINT-ESCAPES-TABLE).

A _back reference_ (`\`_`n`_) matches the same string matched by the previous parenthesized subexpression specified by the number _`n`_ (see [Table 9.21](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-CONSTRAINT-BACKREF-TABLE)). For example, `([bc])\1` matches `bb` or `cc` but not `bc`or `cb`. The subexpression must entirely precede the back reference in the RE. Subexpressions are numbered in the order of their leading parentheses. Non-capturing parentheses do not define subexpressions.

#### **Table 9.18. Regular Expression Character-entry Escapes**

| Escape           | Description                                                                                                                                                                    |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `\a`             | alert (bell) character, as in C                                                                                                                                                |
| `\b`             | backspace, as in C                                                                                                                                                             |
| `\B`             | synonym for backslash (`\`) to help reduce the need for backslash doubling                                                                                                     |
| `\c`_`X`_        | (where _`X`_ is any character) the character whose low-order 5 bits are the same as those of _`X`_, and whose other bits are all zero                                          |
| `\e`             | the character whose collating-sequence name is `ESC`, or failing that, the character with octal value `033`                                                                    |
| `\f`             | form feed, as in C                                                                                                                                                             |
|                  | newline, as in C                                                                                                                                                               |
|                  | carriage return, as in C                                                                                                                                                       |
|                  | horizontal tab, as in C                                                                                                                                                        |
| `\u`_`wxyz`_     | (where _`wxyz`_ is exactly four hexadecimal digits) the character whose hexadecimal value is `0x`_`wxyz`_                                                                      |
| `\U`_`stuvwxyz`_ | (where _`stuvwxyz`_ is exactly eight hexadecimal digits) the character whose hexadecimal value is `0x`_`stuvwxyz`_                                                             |
| `\v`             | vertical tab, as in C                                                                                                                                                          |
| `\x`_`hhh`_      | (where _`hhh`_ is any sequence of hexadecimal digits) the character whose hexadecimal value is `0x`_`hhh`_ (a single character no matter how many hexadecimal digits are used) |
| `\0`             | the character whose value is `0` (the null byte)                                                                                                                               |
| `\`_`xy`_        | (where _`xy`_ is exactly two octal digits, and is not a _back reference_) the character whose octal value is `0`_`xy`_                                                         |
| `\`_`xyz`_       | (where _`xyz`_ is exactly three octal digits, and is not a _back reference_) the character whose octal value is `0`_`xyz`_                                                     |

Hexadecimal digits are `0`-`9`, `a`-`f`, and `A`-`F`. Octal digits are `0`-`7`.

Numeric character-entry escapes specifying values outside the ASCII range (0-127) have meanings dependent on the database encoding. When the encoding is UTF-8, escape values are equivalent to Unicode code points, for example `\u1234` means the character `U+1234`. For other multibyte encodings, character-entry escapes usually just specify the concatenation of the byte values for the character. If the escape value does not correspond to any legal character in the database encoding, no error will be raised, but it will never match any data.

The character-entry escapes are always taken as ordinary characters. For example, `\135` is `]` in ASCII, but `\135` does not terminate a bracket expression.

#### **Table 9.19. Regular Expression Class-shorthand Escapes**

| Escape | Description                                   |
| ------ | --------------------------------------------- |
| `\d`   | `[[:digit:]]`                                 |
| `\s`   | `[[:space:]]`                                 |
| `\w`   | `[[:alnum:]_]` (note underscore is included)  |
| `\D`   | `[^[:digit:]]`                                |
| `\S`   | `[^[:space:]]`                                |
| `\W`   | `[^[:alnum:]_]` (note underscore is included) |

Within bracket expressions, `\d`, `\s`, and `\w` lose their outer brackets, and `\D`, `\S`, and `\W` are illegal. (So, for example, `[a-c\d]` is equivalent to `[a-c[:digit:]]`. Also, `[a-c\D]`, which is equivalent to `[a-c^[:digit:]]`, is illegal.)

#### **Table 9.20. Regular Expression Constraint Escapes**

| Escape | Description                                                                                                                                                                        |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `\A`   | matches only at the beginning of the string (see [Section 9.7.3.5](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-MATCHING-RULES) for how this differs from `^`) |
| `\m`   | matches only at the beginning of a word                                                                                                                                            |
| `\M`   | matches only at the end of a word                                                                                                                                                  |
| `\y`   | matches only at the beginning or end of a word                                                                                                                                     |
| `\Y`   | matches only at a point that is not the beginning or end of a word                                                                                                                 |
| `\Z`   | matches only at the end of the string (see [Section 9.7.3.5](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-MATCHING-RULES) for how this differs from `$`)       |

A word is defined as in the specification of `[[:<:]]` and `[[:>:]]` above. Constraint escapes are illegal within bracket expressions.

#### **Table 9.21. Regular Expression Back References**

| Escape     | Description                                                                                                                                                                                                                  |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `\`_`m`_   | (where _`m`_ is a nonzero digit) a back reference to the _`m`_'th subexpression                                                                                                                                              |
| `\`_`mnn`_ | (where _`m`_ is a nonzero digit, and _`nn`_ is some more digits, and the decimal value _`mnn`_ is not greater than the number of closing capturing parentheses seen so far) a back reference to the _`mnn`_'th subexpression |

#### Note

There is an inherent ambiguity between octal character-entry escapes and back references, which is resolved by the following heuristics, as hinted at above. A leading zero always indicates an octal escape. A single non-zero digit, not followed by another digit, is always taken as a back reference. A multi-digit sequence not starting with a zero is taken as a back reference if it comes after a suitable subexpression (i.e., the number is in the legal range for a back reference), and otherwise is taken as octal.

### **9.7.3.4. Regular Expression Metasyntax**

In addition to the main syntax described above, there are some special forms and miscellaneous syntactic facilities available.

An RE can begin with one of two special _director_ prefixes. If an RE begins with `***:`, the rest of the RE is taken as an ARE. (This normally has no effect in PostgreSQL, since REs are assumed to be AREs; but it does have an effect if ERE or BRE mode had been specified by the _`flags`_ parameter to a regex function.) If an RE begins with `***=`, the rest of the RE is taken to be a literal string, with all characters considered ordinary characters.

An ARE can begin with _embedded options_: a sequence `(?`_`xyz`_`)` (where _`xyz`_ is one or more alphabetic characters) specifies options affecting the rest of the RE. These options override any previously determined options — in particular, they can override the case-sensitivity behavior implied by a regex operator, or the _`flags`_ parameter to a regex function. The available option letters are shown in [Table 9.22](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-EMBEDDED-OPTIONS-TABLE). Note that these same option letters are used in the _`flags`_ parameters of regex functions.

#### **Table 9.22. ARE Embedded-option Letters**

| Option | Description                                                                                                                                                   |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `b`    | rest of RE is a BRE                                                                                                                                           |
| `c`    | case-sensitive matching (overrides operator type)                                                                                                             |
| `e`    | rest of RE is an ERE                                                                                                                                          |
| `i`    | case-insensitive matching (see [Section 9.7.3.5](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-MATCHING-RULES)) (overrides operator type)  |
| `m`    | historical synonym for `n`                                                                                                                                    |
| `n`    | newline-sensitive matching (see [Section 9.7.3.5](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-MATCHING-RULES))                           |
| `p`    | partial newline-sensitive matching (see [Section 9.7.3.5](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-MATCHING-RULES))                   |
| `q`    | rest of RE is a literal (“quoted”) string, all ordinary characters                                                                                            |
| `s`    | non-newline-sensitive matching (default)                                                                                                                      |
| `t`    | tight syntax (default; see below)                                                                                                                             |
| `w`    | inverse partial newline-sensitive (“weird”) matching (see [Section 9.7.3.5](https://www.postgresql.org/docs/11/functions-matching.html#POSIX-MATCHING-RULES)) |
| `x`    | expanded syntax (see below)                                                                                                                                   |

Embedded options take effect at the `)` terminating the sequence. They can appear only at the start of an ARE (after the `***:` director if any).

In addition to the usual (_tight_) RE syntax, in which all characters are significant, there is an _expanded_ syntax, available by specifying the embedded `x` option. In the expanded syntax, white-space characters in the RE are ignored, as are all characters between a `#` and the following newline (or the end of the RE). This permits paragraphing and commenting a complex RE. There are three exceptions to that basic rule:

* a white-space character or `#` preceded by `\` is retained
* white space or `#` within a bracket expression is retained
* white space and comments cannot appear within multi-character symbols, such as `(?:`

For this purpose, white-space characters are blank, tab, newline, and any character that belongs to the _`space`_ character class.

Finally, in an ARE, outside bracket expressions, the sequence `(?#`_`ttt`_`)` (where _`ttt`_ is any text not containing a `)`) is a comment, completely ignored. Again, this is not allowed between the characters of multi-character symbols, like `(?:`. Such comments are more a historical artifact than a useful facility, and their use is deprecated; use the expanded syntax instead.

_None_ of these metasyntax extensions is available if an initial `***=` director has specified that the user's input be treated as a literal string rather than as an RE.

### **9.7.3.5. Regular Expression Matching Rules**

In the event that an RE could match more than one substring of a given string, the RE matches the one starting earliest in the string. If the RE could match more than one substring starting at that point, either the longest possible match or the shortest possible match will be taken, depending on whether the RE is _greedy_ or _non-greedy_.

Whether an RE is greedy or not is determined by the following rules:

* Most atoms, and all constraints, have no greediness attribute (because they cannot match variable amounts of text anyway).
* Adding parentheses around an RE does not change its greediness.
* A quantified atom with a fixed-repetition quantifier (`{`_`m`_`}` or `{`_`m`_`}?`) has the same greediness (possibly none) as the atom itself.
* A quantified atom with other normal quantifiers (including `{`_`m`_`,`_`n`_`}` with _`m`_ equal to _`n`_) is greedy (prefers longest match).
* A quantified atom with a non-greedy quantifier (including `{`_`m`_`,`_`n`_`}?` with _`m`_ equal to _`n`_) is non-greedy (prefers shortest match).
* A branch — that is, an RE that has no top-level `|` operator — has the same greediness as the first quantified atom in it that has a greediness attribute.
* An RE consisting of two or more branches connected by the `|` operator is always greedy.

The above rules associate greediness attributes not only with individual quantified atoms, but with branches and entire REs that contain quantified atoms. What that means is that the matching is done in such a way that the branch, or whole RE, matches the longest or shortest possible substring _as a whole_. Once the length of the entire match is determined, the part of it that matches any particular subexpression is determined on the basis of the greediness attribute of that subexpression, with subexpressions starting earlier in the RE taking priority over ones starting later.

An example of what this means:

```
SELECT SUBSTRING('XY1234Z', 'Y*([0-9]{1,3})');
Result: 123
SELECT SUBSTRING('XY1234Z', 'Y*?([0-9]{1,3})');
Result: 1
```

In the first case, the RE as a whole is greedy because `Y*` is greedy. It can match beginning at the `Y`, and it matches the longest possible string starting there, i.e., `Y123`. The output is the parenthesized part of that, or `123`. In the second case, the RE as a whole is non-greedy because `Y*?` is non-greedy. It can match beginning at the `Y`, and it matches the shortest possible string starting there, i.e., `Y1`. The subexpression `[0-9]{1,3}` is greedy but it cannot change the decision as to the overall match length; so it is forced to match just `1`.

In short, when an RE contains both greedy and non-greedy subexpressions, the total match length is either as long as possible or as short as possible, according to the attribute assigned to the whole RE. The attributes assigned to the subexpressions only affect how much of that match they are allowed to “eat” relative to each other.

The quantifiers `{1,1}` and `{1,1}?` can be used to force greediness or non-greediness, respectively, on a subexpression or a whole RE. This is useful when you need the whole RE to have a greediness attribute different from what's deduced from its elements. As an example, suppose that we are trying to separate a string containing some digits into the digits and the parts before and after them. We might try to do that like this:

```
SELECT regexp_match('abc01234xyz', '(.*)(\d+)(.*)');
Result: {abc0123,4,xyz}
```

That didn't work: the first `.*` is greedy so it “eats” as much as it can, leaving the `\d+` to match at the last possible place, the last digit. We might try to fix that by making it non-greedy:

```
SELECT regexp_match('abc01234xyz', '(.*?)(\d+)(.*)');
Result: {abc,0,""}
```

That didn't work either, because now the RE as a whole is non-greedy and so it ends the overall match as soon as possible. We can get what we want by forcing the RE as a whole to be greedy:

```
SELECT regexp_match('abc01234xyz', '(?:(.*?)(\d+)(.*)){1,1}');
Result: {abc,01234,xyz}
```

Controlling the RE's overall greediness separately from its components' greediness allows great flexibility in handling variable-length patterns.

When deciding what is a longer or shorter match, match lengths are measured in characters, not collating elements. An empty string is considered longer than no match at all. For example: `bb*`matches the three middle characters of `abbbc`; `(week|wee)(night|knights)` matches all ten characters of `weeknights`; when `(.*).*` is matched against `abc` the parenthesized subexpression matches all three characters; and when `(a*)*` is matched against `bc` both the whole RE and the parenthesized subexpression match an empty string.

If case-independent matching is specified, the effect is much as if all case distinctions had vanished from the alphabet. When an alphabetic that exists in multiple cases appears as an ordinary character outside a bracket expression, it is effectively transformed into a bracket expression containing both cases, e.g., `x` becomes `[xX]`. When it appears inside a bracket expression, all case counterparts of it are added to the bracket expression, e.g., `[x]` becomes `[xX]` and `[^x]` becomes `[^xX]`.

If newline-sensitive matching is specified, `.` and bracket expressions using `^` will never match the newline character (so that matches will never cross newlines unless the RE explicitly arranges it) and `^`and `$` will match the empty string after and before a newline respectively, in addition to matching at beginning and end of string respectively. But the ARE escapes `\A` and `\Z` continue to match beginning or end of string _only_.

If partial newline-sensitive matching is specified, this affects `.` and bracket expressions as with newline-sensitive matching, but not `^` and `$`.

If inverse partial newline-sensitive matching is specified, this affects `^` and `$` as with newline-sensitive matching, but not `.` and bracket expressions. This isn't very useful but is provided for symmetry.

### **9.7.3.6. Limits And Compatibility**

No particular limit is imposed on the length of REs in this implementation. However, programs intended to be highly portable should not employ REs longer than 256 bytes, as a POSIX-compliant implementation can refuse to accept such REs.

The only feature of AREs that is actually incompatible with POSIX EREs is that `\` does not lose its special significance inside bracket expressions. All other ARE features use syntax which is illegal or has undefined or unspecified effects in POSIX EREs; the `***` syntax of directors likewise is outside the POSIX syntax for both BREs and EREs.

Many of the ARE extensions are borrowed from Perl, but some have been changed to clean them up, and a few Perl extensions are not present. Incompatibilities of note include `\b`, `\B`, the lack of special treatment for a trailing newline, the addition of complemented bracket expressions to the things affected by newline-sensitive matching, the restrictions on parentheses and back references in lookahead/lookbehind constraints, and the longest/shortest-match (rather than first-match) matching semantics.

Two significant incompatibilities exist between AREs and the ERE syntax recognized by pre-7.4 releases of PostgreSQL:

* In AREs, `\` followed by an alphanumeric character is either an escape or an error, while in previous releases, it was just another way of writing the alphanumeric. This should not be much of a problem because there was no reason to write such a sequence in earlier releases.
* In AREs, `\` remains a special character within `[]`, so a literal `\` within a bracket expression must be written `\\`.

### **9.7.3.7. Basic Regular Expressions**

BREs differ from EREs in several respects. In BREs, `|`, `+`, and `?` are ordinary characters and there is no equivalent for their functionality. The delimiters for bounds are `\{` and `\}`, with `{` and `}` by themselves ordinary characters. The parentheses for nested subexpressions are `\(` and `\)`, with `(` and `)` by themselves ordinary characters. `^` is an ordinary character except at the beginning of the RE or the beginning of a parenthesized subexpression, `$` is an ordinary character except at the end of the RE or the end of a parenthesized subexpression, and `*` is an ordinary character if it appears at the beginning of the RE or the beginning of a parenthesized subexpression (after a possible leading `^`). Finally, single-digit back references are available, and `\<` and `\>` are synonyms for `[[:<:]]` and `[[:>:]]` respectively; no other escapes are available in BREs.
