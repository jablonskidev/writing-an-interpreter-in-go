# Parsing

Topics in this chapter:
- [Parsers](#Parsers)
- Why Not a Parser Generator?
- Writing a Parser for the Monkey Programming Language
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

## Why Not a Parser Generator?
## Writing a Parser for the Monkey Programming Language
## Parser's First Steps: Parsing let Statements
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
