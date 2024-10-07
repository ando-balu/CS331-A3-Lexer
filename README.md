# CS 331 Assignment 3: Lexer for Programming Language "Nilgai"
## Description

This is a state machine lexer, written as a Lua module, to be used in the design of an interpreter for the programming language Nilgai, created by Glenn Chappell. Utilizing function `lexit.lex`, the lexer takes string input and categorizes the resulting lexemes as tokens for a parser to parse in order to be interpreted. The Lua module exports function `lexit.lex`, which takes a string parameter and allows for-in iteration through lexemes in the passed string, seven constants representing lexeme categories, and table `lexit.catnames`, which maps lexeme category numbers into printable strings. The *Lexeme Specification* for the lexing done is listed below.

The correspondence between lexeme category numbers and category names/strings should be as follows.
| Category Number | Named Constant | Printable Form |
| :---: | --- | --- |
| 1 | `lexit.KEY` | `Keyword` |
| 2 | `lexit.ID` | `Identifier` |
| 3 | `lexit.NUMLIT` | `NumericLiteral` |
| 4 | `lexit.STRLIT` | `StringLiteral` |
| 5 | `lexit.OP` | `Operator` |
| 6 | `lexit.PUNCT` | `Punctuation` |
| 7 | `lexit.MAL` | `Malformed` |

Thus, the following code should work.

**[Lua]**
```lua
lexit = require "lexit"

program = "x = 3; # Set a variable\n output(x+4, eol);\n"

for lexstr, cat in lexit.lex(program) do
    print(lexstr, lexit.catnames[cat])
end
```

### Lexit Flow Chart

![Lexit Flow Chart](https://github.com/user-attachments/assets/1d86291b-627c-4977-bdf5-29590c19a637)


## Lexeme Specification

This is a specification of the lexemes in the *Nilgai* programming language.

**Whitespace** characters are blank, tab, vertical-tab, new-line, carriage-return, form-feed. No lexeme, except for a **StringLiteral**, may contain a whitespace character. So a whitespace character, or any contiguous group of whitespace characters, is generally a separator between lexemes. However, pairs of lexemes are not *required* to be separated by whitespace.

A **comment** begins with a pound sign (#) occurring outside a **StringLiteral** lexeme or another comment, and ends at a newline character or the end of the input, whichever comes first. There are no other kinds of comments. Any character at all may occur in a comment.

Comments are treated by the lexer as whitespace: they are not part of lexemes and are not passed on to the caller.

**Legal** characters outside comments and **StringLiteral** lexemes are whitespace and **printable ASCII** characters (values 32 [blank] to 126 [tilde]). Any other characters outside comments and **StringLiteral** lexemes are **illegal**.

The maximal-munch rule is followed.

There are seven lexeme categories: **Keyword**, **Identifier**, **NumericLiteral**, **StringLiteral**, **Operator**, **Punctuation**, **Malformed**.

*Below, in a regular expression, a character preceded by a backslash means the literal character, with no special meaning.*

### Keyword

One of the following 16:

`and`  `char`  `def`  `else`  `elseif`  `eol`  `false`  `if`  `inputnum` `not`  `or`  `output`  `rand`  `return`  `true`  `while`

### Identifier

Any string matched by `/[a-zA-Z_][a-zA-Z_0-9]*/` that is not a **Keyword**.

Here are some Identifier lexemes.

`myvar`   `_`    `___x_37cr`   `HelloThere`   `RETURN`

*Note. The reserved words are the same as the **Keyword** lexemes.*

### NumericLiteral

Any string matched by `/[0-9]+([eE]\+?[0-9]+)?/`.

*Notes. A **NumericLiteral** must begin with a digit and cannot contain a dot (.). A minus sign is not legal in an exponent (the “e” or “E” and what comes after it). A plus sign is legal, and optional, in an exponent. An exponent must contain at least one digit.*

Here are some valid **NumericLiteral** lexemes.

`1234`    `00900`   `123e+7`   `00E00`   `3e888`

The following are *not* valid **NumericLiteral** lexemes.

`-42`   `3e`   `e`   `123E+`   `1.23`   `123e-7`

### StringLiteral

A single quote (') or double quote ("), followed by zero or more characters that are not newlines or the same as the opening quote mark, followed by a quote that matches the opening quote mark. There are no escape sequences. Any character, legal or illegal, other than a newline or a quote that matches the opening quote mark, may appear inside a **StringLiteral**. The beginning and ending quote marks are both part of the lexeme.

Here are some **StringLiteral** lexemes.

`"Hello there!"`   `''`   `'"'`   `"'--#!Ωé\"`

### Operator

One of the following fourteen:

`==`   `!=`   `<`   `<=`   `>`   `>=`   `+`   `-`   `*`   `/`   `%`   `[`   `]`   `=`

### Punctuation

Any single legal character that is not whitespace, not part of a comment, and not part of any valid lexeme in one of the other categories, including **Malformed**.

Here are some **Punctuation** lexemes.

`;`   `(`   `)`   `{`   `}`   `,`   `&`   `$`

### Malformed

There are two kinds of **Malformed** lexemes: *bad character* and *bad string*.

A **bad character** is any single character that is illegal, that is not part of a comment or a **StringLiteral** lexeme that began earlier.

A **bad string** is essentially a partial **StringLiteral** where the end of the line or the end of the input is reached before the ending quote mark. It begins with a double quote mark that is not part of a comment or **StringLiteral** that began earlier, and continues to the next newline or the end of the input, without a double quote appearing. Any character, legal or illegal, may appear in a **bad string**. If the lexeme ends at a newline, then this newline is not part of the lexeme.

Here are three **Malformed** lexemes that are bad strings.

`"a-b-c`    `'wx yz`    `"Ωé'`

In order to be counted as **Malformed**. each of the above must end at a newline (which would not be considered part of the lexeme) or at the end of the input.

*Note. The two kinds of **Malformed** lexemes are presented to the caller in the same way: they are both simply **Malformed**.*
