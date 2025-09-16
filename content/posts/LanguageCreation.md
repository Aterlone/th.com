+++
title = "The Art and Science of Language Creation"
date = "2025-09-15"
author = "Tane Haines"
cover = ""
tags = ["programming", "damp"]
keywords = ["constructed languages", "damp"]
showFullContent = false
readingTime = true
hideComments = false
+++

I was on the plane creating a programming language. (Yes it's turing complete, and no I'm never going to touch it again.)

<!--more-->

# Overview

So yeah, I've been doing CTF's for 1-2 years now and have realised that lots of Reverse Engineering challenges are literally programs built on custom language architectures. So yeah I wanted to create some programming languages to warm myself up when creating one for reverse engineering >:).

Enter **Damp** a C-like programming language I built from scratch in Python(I know, you don't need to tell me). Why did I call it Damp? I don't know I was trying to come up with a name, I knew from the start it was going to be a speedrun anyway.

I wanted to create a turing complete language so I can say I have as well, so...

## Characteristics of a turing complete language

The language must have the ability to:

- Execute a series of computational steps in the order they are provided.
- Execute different computational paths conditionally based on a certain criteria. (e.g. if blocks)
- Repeat computational sub-processes over and over. (e.g. while loops)
- Store intermediate results for later use in memory. (e.g. variables)

## Components of an Interpreted Language (or well mine anyway.).

The Lexer:

- Tokenises components of the script passed, and categorizes them.
  The Parser:
- Run the code through a grammer checker to ensure that all of the code is in the correct format, and give the user warnings if it is not (I didn't bother writing this).
- Run the code through a organiser(I don't know the technical term) and make a syntax tree which groups up commands so it is easier to run the program
  The evaluator:
- Evaluates the components in the syntax tree and runs the program.

Yeah it's a lot more simple than I thought. I'm sure there will be much more as I dive deeper into the topic when creating my ctf challenges, but this is all you need for a minimal programming language.
I thought each step will be more complex but apparently not.

I don't really have time to write a proper introduction to my language, so here is an ai overview. I will write the conclusion myself though.

## What is Damp?

Damp is a minimal programming language that supports:

- Variable declarations (`int x;`)
- Variable assignments (`x = 5 + 7;`)
- While loops (`while (x < 20) { ... }`)
- If statements (`if (x == 10) { ... }`)
- Basic arithmetic (`+`, `-`)
- Comparison operators (`==`, `<`)

Here's a sample Damp program:

```damp
int x;
x = 5 + 7;
while ( x < 20 ) {
    x = x + 1;
    x = x + 1;
}
if (x == 20) {
    valid // strings are printed directly and no comments do not work.
}

```

## The Architecture

Damp follows the classic compiler/interpreter pipeline:

1. **Lexer** - Converts source code into tokens
2. **Parser** - Builds an Abstract Syntax Tree (AST)
3. **Evaluator** - Executes the program

### Tokenization

The lexer breaks down the source code into meaningful tokens:

```python
program_tokens = {
    "KEYWORD": ["if", "while"],
    "TYPE": ["int", "char"],
    "OPERATOR": ["+", "-", "="],
    "CONDITIONAL": ["==", "<"],
    "CASE": ["(", ")"]
}
```

The tokenizer handles:

- Keywords (`if`, `while`)
- Data types (`int`, `char`)
- Operators (`+`, `-`, `=`)
- Comparison operators (`==`, `<`)
- Parentheses for grouping
- Numeric literals
- Variable names

### Parsing

The parser builds an AST using a recursive descent approach. It handles:

- **Variable declarations**: `int x;` → `["VAR_DEC", [("TYPE", "int"), ("VAR_NAME", "x")]]`
- **Variable assignments**: `x = 5 + 7;` → `["VAR_ASSIGN", [("VAR_NAME", "x"), ["EXP", [...]]]]`
- **While loops**: `while (condition) { body }` → `["COND_BLOCK", [("KEYWORD", "while"), condition_exp, body_tree]]`
- **If statements**: `if (condition) { body }` → `["COND_BLOCK", [("KEYWORD", "if"), condition_exp, body_tree]]`

### Expression Evaluation

The evaluator handles arithmetic and comparison operations with proper precedence:

```python
def evaluate_expression(self, exp, variables):
    # Handles nested expressions, variable substitution,
    # arithmetic operations, and comparisons
```

## Key Features

### 1. Variable Management

Damp maintains a simple variable store as a list of `[type, name, value]` tuples:

```python
variables = [["int", "x", 12], ["int", "y", 5]]
```

### 2. Control Flow

The language supports both `if` statements and `while` loops with proper scoping and condition evaluation.

### 3. Expression Parsing

Expressions are parsed left-associatively, handling:

- Arithmetic: `5 + 7 - 3`
- Comparisons: `x < 20`, `y == 10`
- Variable references: `x + y`

## Current Challenges

As with any language implementation, Damp has its fair share of issues:

### 1. Variable Assignment Expression Creation

The expression creation for variable assignments sometimes fails, particularly with complex nested expressions. This is likely due to the recursive nature of the `create_exp` function not properly handling all token combinations.

### 2. If Condition Evaluation

The if condition evaluation isn't working correctly - variables in conditions aren't being properly resolved, leading to evaluation failures.

### 3. Limited Error Handling

The language lacks comprehensive error handling for syntax errors, type mismatches, and runtime errors.

### 4. No Function Support

Damp doesn't support functions, which limits its usefulness for larger programs.

## The Code

Here's the core of the Damp implementation:

```python
class Damp:
    def __init__(self):
        self.program_tokens = {
            "KEYWORD": ["if", "while"],
            "TYPE": ["int", "char"],
            "OPERATOR": ["+", "-", "="],
            "CONDITIONAL": ["==", "<"],
            "CASE": ["(", ")"]
        }

    def tokeniser(self, program):
        # Convert source code to tokens
        pass

    def parser(self, tokens):
        # Build AST from tokens
        pass

    def evaluator(self, tree):
        # Execute the program
        pass

    def run(self, program):
        tokens = self.lexer(program)
        tree = self.parser(tokens)
        self.evaluator(tree)
```

## Future Improvements

If I were to continue developing Damp, I'd focus on:

1. **Better Error Handling** - Add meaningful error messages and proper error recovery
2. **Function Support** - Add function definitions and calls
3. **More Data Types** - Support for strings, arrays, and booleans
4. **Better Testing** - Comprehensive test suite to catch regressions
5. **Optimization** - Basic optimizations like constant folding and dead code elimination
   (According to the chatbot anyway.)

## Conclusion

I think this was a good introduction to programming language building for me. I'll be hosting a ctf competition called VuwCTF in December, but I don't plan to create this sort of language, but be prepared.

I've made 2 languages over the last week, and plan to pop out more just for the experience of implementing them so I have more ease making quick microlanguages in the future for fun. I'm still debating adding this to my github, but I may in the future in a repo with all the dummy languages I've created.

My next blog might be on the constant attack on npm package maintainers. It is quite mainstream so I'm not sure but we will see. I need to find time to write it first lol. See ya.
