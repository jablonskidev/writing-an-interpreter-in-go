# Introduction

Topics in this chapter:
- [Interpreters in general](#Interpreters-in-General)
- [The interpreter you will write](#The-Interpreter-You-Will-Write)
- [The tools you will use](#The-Tools-You-Will-Use)

## Interpreters in General

Interpreters and compilers work differently:
- **Interpreters** take source code and evaluate it without producing a visible intermediate result that can be executed later. 
- **Compilers** take source code and produce output in another language that the underlying system can understand.

Interpreters vary in size and complexity:
- Some don't even have a parsing step and just interpret the input right away.
- Some parse the input, build an abstract syntax tree (AST), and then evaluate the AST.
- Some compile the input into and internal representation called bytecode and then evaluate that.
- Some compile input Just in Time (JIT) into native machine code that is then executed.

## The Interpreter You Will Write

You'll write an interpreter that will parse the input, build an abstract syntax tree (AST), and then evaluate the AST. This type of interpreter is sometimes called a tree-walking interpreter.

You'll implement a language called Monkey, which has the following features:
- C-like syntax
- Variable bindings
- Integers and booleans
- Arithmetic expressions
- Built-in functions
- First-class and higher-order functions
- Closures
- A string data structure
- An array data structure
- A hash data structure

Your interpreter will have the following major parts:
- A lexer
- A parser
- An Abstract Syntax Tree (AST)
- An internal object system
- An evaluator

## The Tools You Will Use

You'll need only:
- A text editor
- The Go programming language

As of March 2022, the latest version of the book uses Go 1.14.
