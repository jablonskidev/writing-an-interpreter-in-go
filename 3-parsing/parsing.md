# Parsing

Topics in this chapter:
- [Parsers](#Parsers)
- [Why Not a Parser Generator](#Why-Not-a-Parser-Generator)
- [Writing a Parser for the Monkey Programming Language](#Writing-a-Parser-for-the-Monkey-Programming-Language)
- Parser's First Steps: Parsing let Statements
- Parsing return Statements
- Parsing Expressions
    - Expressions in Monkey
    - Top-Down Operator Precedence (or: Pratt Parsing)
    - Terminology
    - Preparing the AST
    - Implementing the Pratt Parser
    - Identifiers
    - Integer Literals
    - Prefix Operators
    - Infix Operators
- How Pratt Parsing Works
- Extending the Parser
    - Boolean Literals
    - Grouped Expressions
    - If Expressions
    - Function Literals
    - Call Expressions
    - Removing TODOs
- Read-Parse-Print-Loop

## Parsers

A **parser** turns its **input** into a **data structure** that represents the input. 

For most interpreters, this data structure is called a **syntax tree** or an **abstract syntax tree** (**AST**). It's called "abstract" because some of the elements that are visible in the source code are not in the AST, such as semicolons, newlines, whitespace, comments, braces, bracket, and parentheses. These elements simply guide the parser in making the AST. The process of parsing is called **syntactic analysis** because parsers check if the input matches the expected structure. 

In this chapter, you'll write a parser for the Monkey programming language. The parser's input will be the tokens you defined and wrote a lexer for in [Lexing](https://github.com/jablonskidev/writing-an-interpreter-in-go/blob/main/2-lexing/lexing.md). You'll define an AST and construct instances of this AST while recursively parsing tokens.

## Why Not a Parser Generator

A **parser generator** is a tool that takes a formal description of a language and produces a parser.

This parser is code that can be compiled or interpreted as well as fed source code as input to produce a syntax tree. Most parsers take a **context-free grammar** (**CFG**) as their input. A CFG is a set of rules that describes how to make syntactically correct sentences in a language. The most common notational formats of CFGs are the **Backus-Naur Form** (**BNF**) or the **Extended Backus-Naur Form** (**EBNF**).

Parser generator are a good choice if you just want to get the job done, but if you want to learn how parsers work, then you can write one yourself.

## Writing a Parser for the Monkey Programming Language

There are two main approaches to parsing a programming language:
- **Top-down parsing** starts with constructing the root node of the AST and then descends.
- **Bottom-up parsing** does it the other way around.

The parser you're going to write is a **recursive descent parser**. It’ll be a **Pratt parser**, which is a **top down operator precedence** parser.

You'll start by parsing `let` and `return` statements. Then, you'll move on to parsing expressions. After that, you'll extend the parser so it can parse a large subset of the Monkey programming language. As you go, you'll build up the necessary structures for your AST.

## Parser's First Steps: Parsing let Statements

In Monkey, variable bindings look like this:

```
let x = 5;
let y = 10;
let foobar = add(5, 5);
let barfoo = 5 * 5 / 10 + 18 - add(5, 5) + multiply(124);
let anotherName = barfoo;
```

You'll start by parsing `let` statements. For now, you don't need to think about the expressions that produce the value being assigned. You need to  define the necessary parts of an AST that can represent `let` statements.

Look at how this Monkey program is structured:

```
let x = 10;
let y = 15;

let add = fn(a, b) {
    return a + b;
};
```

The `let` statements above fit the following pattern: `let <identifier> = <expression>;` In general, expressions produce values, but statements don’t. Your AST needs two different types of nodes:

- Expressions
- Statements

Now you can start making `ast.go`:

```go
// ast/ast.go

package ast

type Node interface {
    TokenLiteral() string
}

type Statement interface {
    Node
    statementNode()
}

type Expression interface {
    Node
    expressionNode()
}
```

You have three interfaces:
- **`Node`:** All nodes in your AST must implement the `Node` interface. They need to have a `TokenLiteral()` method that returns the literal value of the token it’s associated with. `TokenLiteral()` will be used only for debugging
and testing.
- **`Statement` and `Expression`:** Some of these nodes implement the `Statement` or `Expression` interfaces. These interfaces just have dummy methods called `statementNode()` and `expressionNode()`.

## Parsing return Statements
## Parsing Expressions
### Expressions in Monkey
### Top-Down Operator Precedence (or: Pratt Parsing)
### Terminology
### Preparing the AST
### Implementing the Pratt Parser
### Identifiers
### Integer Literals
### Prefix Operators
### Infix Operators
### How Pratt Parsing Works
## Extending the Parser
### Boolean Literals
### Grouped Expressions
### If Expressions
### Function Literals
### Call Expressions
### Removing TODOs
## Read-Parse-Print-Loop
