# Lexing

Topics in this chapter:

- [Lexical Analysis](#Lexical-Analysis)
- [Defining our Tokens](#Defining-our-Tokens)
- The Lexer
- Extending our Token Set and Lexer

## Lexical Analysis

You'll change the representation of your source code twice before you evaluate it:

Source code → Tokens → Abstract Syntax Tree

Here's what happens in each of the two steps:

- **Source code → Tokens:** Turning **source code** into **tokens** is called **lexical analysis** or **lexing**. This transformation is done by a **lexer**, which is sometimes called a **tokenizer** or a **scanner**, even though there are subtle differences between the terms.

- **Tokens → Abstract Syntax Tree:** The **tokens** are small data structures that are easy to categorize. They're fed to a **parser**, which turns them into an **Abstract Syntax Tree**.
 
## Defining our Tokens

Start by defining just enough tokens to be able to lex the following sample of Monkey code:

```
let five = 5;
let ten = 10;

let add = fn(x, y) {
  x + y;
};
```

You need the following types of tokens:
- **Numbers** for `5` and `10`
- **Variable names** for `x`, `y`, `add`, and `result`
- **Keywords** for `let` and `fn`
- **Special characters** for `(`, `)`, `{`, `}`, `=`, `,`, `;`

You need to create a `Token` data structure that has:
- A type attribute so you can differentiate types of tokens
- A field that holds the literal value of the token

In a new `token` package, define the `Token` struct and the `TokenType` type:

```go
// token/token.go

package token

type TokenType string

type Token struct {
    Type TokenType
    Literal string
}
```

In the same file, define the options for `TokenType` as constants:

```go
// token/token.go

const (
    ILLEGAL = "ILLEGAL"
    EOF = "EOF"
    
    // Identifiers + literals
    IDENT = "IDENT" // add, foobar, x, y, ...
    INT = "INT" // 1343456

    // Operators
    ASSIGN = "="
    PLUS = "+"

    // Delimiters
    COMMA = ","
    SEMICOLON = ";"
    LPAREN = "("
    RPAREN = ")"
    LBRACE = "{"
    RBRACE = "}"

    // Keywords
    FUNCTION = "FUNCTION"
    LET = "LET"
)
```

## The Lexer
## Extending our Token Set and Lexer
