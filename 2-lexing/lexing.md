# Lexing

> **_NOTE:_** To skip to the code for this chapter, go to [writing-an-interpreter-in-go/2-lexing/src/monkey/](https://github.com/jablonskidev/writing-an-interpreter-in-go/tree/main/2-lexing/src/monkey).

Topics in this chapter:

- [Lexical Analysis](#Lexical-Analysis)
- [Defining our Tokens](#Defining-our-Tokens)
- [The Lexer](#The-Lexer)
- [Extending our Token Set and Lexer](#Extending-our-Token-Set-and-Lexer)
- [Start of a REPL](#Start-of-a-REPL)

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

Now that you have a test, you can start working on the lexer itself. Write a function called `New()` that will return `*Lexer`:

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

Now that you have a basic set of tokens as well as a lexer for those tokens, you can define more tokens and extend the lexer accordingly. 

In this section, you'll add support for:
- **One-character tokens:** 
    - `!`
    - `-`
    - `/`
    - `*`
    - `<`
    - `>`
- **Two-character tokens:**
    - `==`
    - `!=`
- **Keywords:**
    - `true`
    - `false`
    - `if`
    - `else`
    - `return`

Start with `-`, `/`, `*`, `<`, and `>`. Add these characters to your test case:

```go
// lexer/lexer_test.go

func TestNextToken(t *testing.T) {
    input := `let five = 5;
let ten = 10;

let add = fn(x, y) {
    x + y;
};

let result = add(five, ten);
!-/*5;
5 < 10 > 5;
`
// [...]
}
```

Although `!-/*5` doesn't make sense as Monkey code, that won't stop the lexer from turning the `input` into tokens. For lexers, write test cases that cover:
- All tokens
- Off-by-one errors
- Edge cases at end-of-file
- Newline handling
- Multi-digit number parsing

Now define tokens for the characters you added to the test:

```go
// token/token.go

const (
// [...]

    // Operators
    ASSIGN = "="
    PLUS = "+"
    MINUS = "-"
    BANG = "!"
    ASTERISK = "*"
    SLASH = "/"
    
    LT = "<"
    GT = ">"

// [...]
)
```

Now extend your switch statement in the `NextToken()` method of `Lexer` so the lexer can return the tokens with the expected `TokenType`:

```go
// lexer/lexer.go

func (l *Lexer) NextToken() token.Token {
// [...]
    switch l.ch {
    case '=':
        tok = newToken(token.ASSIGN, l.ch)
    case '+':
        tok = newToken(token.PLUS, l.ch)
    case '-':
        tok = newToken(token.MINUS, l.ch)
    case '!':
        tok = newToken(token.BANG, l.ch)
    case '/':
        tok = newToken(token.SLASH, l.ch)
    case '*':
        tok = newToken(token.ASTERISK, l.ch)
    case '<':
        tok = newToken(token.LT, l.ch)
    case '>':
        tok = newToken(token.GT, l.ch)
    case ';':
        tok = newToken(token.SEMICOLON, l.ch)
    case ',':
        tok = newToken(token.COMMA, l.ch)
// [...]
}
```
Now that you've added tokens and changed the cases in the `switch` statement, your test should pass.

You can now work on adding support for the keywords `true`, `false`, `if`, `else`, and `return`. Extend the `input` in your test to include the new keywords:

```go
// lexer/lexer_test.go

func TestNextToken(t *testing.T) {
    input := `let five = 5;
let ten = 10;

let add = fn(x, y) {
    x + y;
};

let result = add(five, ten);
!-/*5;
5 < 10 > 5;

if (5 < 10) {
    return true;
} else {
    return false;
}`
// [...]
}
```

Add the keywords to the lookup table for `LookupIdent()`:

```go
// token/token.go

const (
// [...]

    // Keywords
    FUNCTION = "FUNCTION"
    LET = "LET"
    TRUE = "TRUE"
    FALSE = "FALSE"
    IF = "IF"
    ELSE = "ELSE"
    RETURN = "RETURN"
)

var keywords = map[string]TokenType{
    "fn": FUNCTION,
    "let": LET,
    "true": TRUE,
    "false": FALSE,
    "if": IF,
    "else": ELSE,
    "return": RETURN,
}
```
The test should now pass. Now, you need to extend your lexer so it'll recognize tokens made of two characters:
- `==`
- `!=`

Since your switch statement takes the current character `l.ch` as the expression to compare against the cases, you can’t just add new cases like `case '=='`. The compiler won’t
let you. You can’t compare your `l.ch` byte with strings like `'=='`.
Instead, you can reuse the existing branches for `'='` and `'!'` and extend them. You're going to look ahead in the input and then determine whether to return a
token for `=` or `==`:

```go
// lexer/lexer_test.go
func TestNextToken(t *testing.T) {

    input := `let five = 5;

let ten = 10;

let add = fn(x, y) {
    x + y;
};

let result = add(five, ten);
!-/*5;
5 < 10 > 5;

if (5 < 10) {
    return true;
} else {
    return false;
}

10 == 10;
10 != 9;
`
// [...]
}
```
Add a helper method defined on `*Lexer` called `peekChar()`:

```go
// lexer/lexer.go

func (l *Lexer) peekChar() byte {
    if l.readPosition >= len(l.input) {
        return 0
    } else {
        return l.input[l.readPosition]
    }
}
```

`peekChar()` is similar to `readChar()`, but it doesn’t increment `l.position` and `l.readPosition`. You just want to “peek” ahead in the input rather than move around in it so
that you'll know what a call to `readChar()` would return.

Define the new tokens:

```go
// token/token.go

const (
    // [...]
    
    EQ = "=="
    NOT_EQ = "!="

// [...]
)
```
Now that the references to `token.EQ` and `token.NOT_EQ` in the tests for the lexer are fixed, you'll get the correct failure message when you run `go test`:

```
$ go test ./lexer
--- FAIL: TestNextToken (0.00s)
  lexer_test.go:118: tests[66] - tokentype wrong. expected="==", got="="
FAIL
FAIL monkey/lexer 0.007s
```

When the lexer finds a `==` in the `input`, it sees it as two `token.ASSIGN` tokens rather than one `token.EQ` token. You can fix that with `peekChar()`:

```go
// lexer/lexer.go

func (l *Lexer) NextToken() token.Token {
// [...]
    switch l.ch {
    case '=':
        if l.peekChar() == '=' {
            ch := l.ch
            l.readChar()
            literal := string(ch) + string(l.ch)
            tok = token.Token{Type: token.EQ, Literal: literal}
        } else {
            tok = newToken(token.ASSIGN, l.ch)
        }
// [...]
    case '!':
        if l.peekChar() == '=' {
            ch := l.ch
            l.readChar()
            literal := string(ch) + string(l.ch)
            tok = token.Token{Type: token.NOT_EQ, Literal: literal}
        } else {
            tok = newToken(token.BANG, l.ch)
        }
// [...]
}
```
Since you save `l.ch` in a local variable before calling `l.readChar()` again, you don’t lose the current character and can move the lexer forward so it leaves the `NextToken()` with `l.position` and `l.readPosition` in the correct state. 

Your test should now pass. You now have a lexer that produces all of the tokens that you've defined.

## Start of a REPL

REPL stands for Read Eval Print Loop. It is sometimes referred to as a console or interactive mode.

A REPL:
- Reads input
- Sends the input to the interpreter for evaluation
- Prints the output of the
interpreter
- Starts the process again

Although you can't eval Monkey source code yet, you can tokenize it. Parsing and evaluation will come later. Start by making a REPL that tokenizes Monkey source code and prints the tokens:

```go
// repl/repl.go

package repl
    
import (
    "bufio"
    "fmt"
    "io"
    "monkey/lexer"
    "monkey/token"
)

const PROMPT = ">> "

func Start(in io.Reader, out io.Writer) {
    scanner := bufio.NewScanner(in)

    for {
        fmt.Fprintf(out, PROMPT)
        scanned := scanner.Scan()
        if !scanned {
            return
        }

        line := scanner.Text()
        l := lexer.New(line)

        for tok := l.NextToken(); tok.Type != token.EOF; tok = l.NextToken() {
            fmt.Fprintf(out, "%+v\n", tok)
        }
    }
}
```

Here are the steps:
- Read the input until you find a newline
- Take the line that was just read and pass it to an instance of your lexer
- Print all the tokens the lexer gives us until  reaching EOF

Make a `main.go` file to start the REPL and greet the user:

```go
// main.go

package main

import (
    "fmt"
    "os"
    "os/user"
    "monkey/repl"
)

func main() {
    user, err := user.Current()
    if err != nil {
        panic(err)
    }
    fmt.Printf("Hello %s! This is the Monkey programming language!\n", user.Username)
    fmt.Printf("Feel free to type in commands\n")
    repl.Start(os.Stdin, os.Stdout)
}
```

You can now produce tokens interactively:

```
$ go run main.go
Hello mrnugget! This is the Monkey programming language!
Feel free to type in commands
>> let add = fn(x, y) { x + y; };
{Type:LET Literal:let}
{Type:IDENT Literal:add}
{Type:= Literal:=}
{Type:FUNCTION Literal:fn}
{Type:( Literal:(}
{Type:IDENT Literal:x}
{Type:, Literal:,}
{Type:IDENT Literal:y}
{Type:) Literal:)}
{Type:{ Literal:{}
{Type:IDENT Literal:x}
{Type:+ Literal:+}
{Type:IDENT Literal:y}
{Type:; Literal:;}
{Type:} Literal:}}
{Type:; Literal:;}
>>
```

Your next step is to parse tokens in the [Parsing chapter](https://github.com/jablonskidev/writing-an-interpreter-in-go/blob/main/3-parsing/parsing.md).
