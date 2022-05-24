# Lexing

Topics in this chapter:

- [Lexical Analysis](#Lexical-Analysis)
- [Defining our Tokens](#Defining-our-Tokens)
- [The Lexer](#The-Lexer)
- Extending our Token Set and Lexer
- Start of a REPL

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
You now have a basic set of tokens that you can build on.

## The Lexer

Now that you've defined some tokens, you can write a lexer that will input Monkey source code and output tokens that represent the source code. The source code will be treated like a string. The lexer will go through it, character by character, and output the next token it recognizes. You'll write a method called `NextToken()` to make that happen.

Create a package and write a basic test so you can see if your lexer is working:

```go
// lexer/lexer_test.go

package lexer

import (
    "testing"
    
    "monkey/token"
)

func TestNextToken(t *testing.T) {
    input := `=+(){},;`
    
    tests := []struct {
        expectedType token.TokenType
        expectedLiteral string
    }{
        {token.ASSIGN, "="},
        {token.PLUS, "+"},
        {token.LPAREN, "("},
        {token.RPAREN, ")"},
        {token.LBRACE, "{"},
        {token.RBRACE, "}"},
        {token.COMMA, ","},
        {token.SEMICOLON, ";"},
        {token.EOF, ""},
    }
    
    l := New(input)

    for i, tt := range tests {
        tok := l.NextToken()
        if tok.Type != tt.expectedType {
            t.Fatalf("tests[%d] - tokentype wrong. expected=%q, got=%q",
                i, tt.expectedType, tok.Type)
        }
        
        if tok.Literal != tt.expectedLiteral {
            t.Fatalf("tests[%d] - literal wrong. expected=%q, got=%q",
                i, tt.expectedLiteral, tok.Literal)
        }
    }
}
```

This test checks a small subset of the tokens (`=+(){},;`).

Now that you have a test, you can start working on the lexer itself. Write a function called `New()` that will return `*Lexer`.

```go
// lexer/lexer.go

package lexer

type Lexer struct {
    input string
    position int // current position in input (points to current char)
    readPosition int // current reading position in input (after current char)
    ch byte // current char under examination
}

func New(input string) *Lexer {
    l := &Lexer{input: input}
    return l
}
```

Make a helper method called `readChar()` to move forward in the `input` string and get the next character:

```go
// lexer/lexer.go

func (l *Lexer) readChar() {
    if l.readPosition >= len(l.input) {
        l.ch = 0
    } else {
        l.ch = l.input[l.readPosition]
    }
    l.position = l.readPosition
    l.readPosition += 1
}
```

Use `readChar()` in `New()` so your `*Lexer` is working before `NextToken()` is called, with `l.ch`, `l.position`, and `l.readPosition` already initialized:

```go
// lexer/lexer.go

func New(input string) *Lexer {
    l := &Lexer{input: input}
    l.readChar()
    return l
}
```

Now you can write a first version of `NextToken()`:

```go
// lexer/lexer.go

package lexer

import "monkey/token"

func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    
    switch l.ch {
    case '=':
        tok = newToken(token.ASSIGN, l.ch)
    case ';':
        tok = newToken(token.SEMICOLON, l.ch)
    case '(':
        tok = newToken(token.LPAREN, l.ch)
    case ')':
        tok = newToken(token.RPAREN, l.ch)
    case ',':
        tok = newToken(token.COMMA, l.ch)
    case '+':
        tok = newToken(token.PLUS, l.ch)
    case '{':
        tok = newToken(token.LBRACE, l.ch)
    case '}':
        tok = newToken(token.RBRACE, l.ch)
    case 0:
        tok.Literal = ""
        tok.Type = token.EOF
    }

    l.readChar()
    return tok
}

func newToken(tokenType token.TokenType, ch byte) token.Token {
    return token.Token{Type: tokenType, Literal: string(ch)}
}
```

Extend the test case so it looks more like Monkey code:

```go
// lexer/lexer_test.go

func TestNextToken(t *testing.T) {
    input := `let five = 5;
let ten = 10;

let add = fn(x, y) {
  x + y;
};

let result = add(five, ten);
`
    tests := []struct {
        expectedType token.TokenType
        expectedLiteral string
    }{
        {token.LET, "let"},
        {token.IDENT, "five"},
        {token.ASSIGN, "="},
        {token.INT, "5"},
        {token.SEMICOLON, ";"},
        {token.LET, "let"},
        {token.IDENT, "ten"},
        {token.ASSIGN, "="},
        {token.INT, "10"},
        {token.SEMICOLON, ";"},
        {token.LET, "let"},
        {token.IDENT, "add"},
        {token.ASSIGN, "="},
        {token.FUNCTION, "fn"},
        {token.LPAREN, "("},
        {token.IDENT, "x"},
        {token.COMMA, ","},
        {token.IDENT, "y"},
        {token.RPAREN, ")"},
        {token.LBRACE, "{"},
        {token.IDENT, "x"},
        {token.PLUS, "+"},
        {token.IDENT, "y"},
        {token.SEMICOLON, ";"},
        {token.RBRACE, "}"},
        {token.SEMICOLON, ";"},
        {token.LET, "let"},
        {token.IDENT, "result"},
        {token.ASSIGN, "="},
        {token.IDENT, "add"},
        {token.LPAREN, "("},
        {token.IDENT, "five"},
        {token.COMMA, ","},
        {token.IDENT, "ten"},
        {token.RPAREN, ")"},
        {token.SEMICOLON, ";"},
        {token.EOF, ""},
    }
// [...]
}
```
For your lexer to work with this kind of input, it needs to be able to handle identifiers, keywords, and numbers.

Start with identifiers and keywords. Your lexer needs to:
- Know if the current character is a letter
- Keep reading until it reached a character that isn't a letter
- Use the correct `token.TokenType` for identifiers and keywords that it finds

Extend the `switch` statement by adding a default branch and `ILLEGAL` tokens:

```go
// lexer/lexer.go

import "monkey/token"

func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    
    switch l.ch {
// [...]
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            return tok
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
// [...]
}

func (l *Lexer) readIdentifier() string {
    position := l.position
    for isLetter(l.ch) {
        l.readChar()
    }
    return l.input[position:l.position]
}

func isLetter(ch byte) bool {
    return 'a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || ch == '_'
}
```
Your lexer needs to be able to differentiate between identifiers and keywords, such as `fn` or `let`, so update `token.go`:

```go
// token/token.go

var keywords = map[string]TokenType{
    "fn": FUNCTION,
    "let": LET,
}

func LookupIdent(ident string) TokenType {
    if tok, ok := keywords[ident]; ok {
        return tok
    }
    return IDENT
}
```

Now update the lexer:

```go
// lexer/lexer.go

func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    
    switch l.ch {
// [...]
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            tok.Type = token.LookupIdent(tok.Literal)
            return tok
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
// [...]
}
```

Your lexer needs to be able to ignore whitespace in order to work with Monkey code:

```go
// lexer/lexer.go

func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    
    l.skipWhitespace()

    switch l.ch {
// [...]
}

func (l *Lexer) skipWhitespace() {
    for l.ch == ' ' || l.ch == '\t' || l.ch == '\n' || l.ch == '\r' {
        l.readChar()
    }
}
```

Your lexer needs to be able to handle numbers, so add to the `default` branch of your switch statement, like you did for identifiers. This time, use `isDigit` instead of `isLetter`:

```go
// lexer/lexer.go

func (l *Lexer) NextToken() token.Token {
    var tok token.Token

    l.skipWhitespace()

    switch l.ch {
// [...]
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            tok.Type = token.LookupIdent(tok.Literal)
            return tok
        } else if isDigit(l.ch) {
            tok.Type = token.INT
            tok.Literal = l.readNumber()
            return tok
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
// [...]
}

func (l *Lexer) readNumber() string {
    position := l.position
    for isDigit(l.ch) {
        l.readChar()
    }
    return l.input[position:l.position]
}

func isDigit(ch byte) bool {
     return '0' <= ch && ch <= '9'
}
```
You now have a basic lexer that you can build on.

## Extending our Token Set and Lexer

## Start of a REPL
