# Parsing Exercises: Drawing ASTs

In these exercises, you will practice parsing by constructing abstract syntax trees (ASTs) by hand.

You are given a grammar describing a small subset of Pharo syntax. For each program, parse the program according to the grammar and **draw its AST on paper**.

Do not evaluate the expressions. The goal is to determine the structure imposed by the grammar.

## Reading the Grammar

The grammar is written in a notation similar to **BNF (Backus–Naur Form)**. Each rule defines how a syntactic construct can be formed.

For example:

```text
Assignment ::= Variable ':=' Expression
```

This means that an `Assignment` consists of a `Variable`, followed by the token `:=`, followed by an `Expression`.

The notation used in this grammar is:

```text
A ::= B C
```

means that `A` consists of `B` followed by `C`.

```text
A ::= B | C
```

means that `A` can be either `B` or `C`.

```text
A+
```

means **one or more** occurrences of `A`.

```text
A?
```

means that `A` is **optional**.

Parentheses group elements together. For example, `(KeywordPart Binary)+` means one or more pairs of a `KeywordPart` followed by a `Binary`.

Names such as `Expression` and `Binary` refer to other grammar rules. Symbols in quotes such as `':='`, `'('` and `'['` refer directly to those tokens in the source program.

Some rules refer to themselves. For example:

```text
Unary ::= Unary UnarySelector
        | Primary
```

means that a `Unary` is either a `Primary`, or a smaller `Unary` followed by one more selector. So `3 factorial factorial` is the unary `3 factorial`, followed by `factorial`.

## Grammar

The following grammar describes the subset of Pharo used in these exercises.
It has one rule per precedence level, from the lowest to the highest priority:
assignment, then keyword messages, then binary messages, then unary messages, then primaries.

```text
Expression ::= Assignment
             | Keyword

Assignment ::= Variable ':=' Expression

Keyword    ::= Binary (KeywordPart Binary)+
             | Binary

Binary     ::= Binary BinarySelector Unary
             | Unary

Unary      ::= Unary UnarySelector
             | Primary

Primary    ::= Variable
             | Literal
             | Block
             | '(' Expression ')'

Block      ::= '[' (BlockArgument+ '|')? Expression? ']'

BlockArgument ::= ':' Variable

Literal    ::= Integer | String | 'true' | 'false' | 'nil'
```

The tokens are:

```text
Variable       ::= identifier            e.g. answer, x
UnarySelector  ::= identifier            e.g. factorial, size
KeywordPart    ::= identifier ':'        e.g. at:, put:
BinarySelector ::= '+' | '-' | '*' | '/' | '=' | '<' | '>' | '<=' | '>='
```

Each rule says what each precedence level is made of:

* the receiver of a unary message is a primary or another unary message;
* the receiver and argument of a binary message are unary messages, and the receiver can also be another binary message;
* the receiver and arguments of a keyword message are binary messages;
* parentheses start again from `Expression`, which is the only way to put a lower-priority expression inside a higher-priority one.

## AST Nodes

Each grammar rule builds one AST node:

| Rule                                   | AST node                                     |
|----------------------------------------|----------------------------------------------|
| `Assignment`                           | `Assignment(variable, value)`                |
| `Keyword` with keyword parts           | `Message(receiver, selector, arguments)`     |
| `Binary` with a selector               | `Message(receiver, selector, [argument])`    |
| `Unary` with a selector                | `Message(receiver, selector, [])`            |
| `Block`                                | `Block(arguments, body)`                     |
| `Variable`                             | `Variable(name)`                             |
| `Integer`, `String`                    | `Integer(value)`, `String(value)`            |
| `true`, `false`, `nil`                 | `Boolean(value)`, `Nil`                      |

The alternatives `Keyword ::= Binary`, `Binary ::= Unary`, `Unary ::= Primary` and `Primary ::= '(' Expression ')'` build no node of their own: they return the node of the inner rule.

All three kinds of messages build the same `Message` node:

```text
Message
    receiver
    selector
    arguments
```

For example, `array at: 3 put: 42` builds one message with selector `at:put:` and two arguments.

## Exercise 1 — Literals

Draw the grammar derivation and its AST for:

```smalltalk
42
```

---

## Exercise 2 — Variables and Unary Messages

Draw the grammar derivation and its AST for:

```smalltalk
answer factorial
```

---

## Exercise 3 — Binary Messages

Draw the grammar derivation and its AST for:

```smalltalk
3 + 4
```

Pay attention to the fact that `+` is a **binary message**, not an arithmetic operation built into the grammar.

---

## Exercise 4 — Unary and Binary Messages

Draw the grammar derivation and its AST for:

```smalltalk
3 factorial + 2
```

Determine which expression is the receiver of `+`.

---

## Exercise 5 — Keyword Messages

Draw the grammar derivation and its AST for:

```smalltalk
array at: 3
```

Identify the receiver, selector, and argument of the message.

---

## Exercise 6 — Multiple Keyword Arguments

Draw the grammar derivation and its AST for:

```smalltalk
array at: 3 put: 42
```

The selector of the message is:

```text
at:put:
```

The message has two arguments.

---

## Exercise 7 — Message Precedence

Draw the grammar derivation and its AST for:

```smalltalk
object size + 1
```

Use the grammar to determine whether this is parsed as:

```text
object size
```

followed by `+ 1`, or as something else.

---

## Exercise 8 — Parentheses

Draw the grammar derivation and its AST for:

```smalltalk
(1 + 2) * 3
```

Compare the structure with what you would obtain from:

```smalltalk
1 + 2 * 3
```

The grammar, rather than conventional arithmetic precedence, determines the structure.

---

## Exercise 9 — Assignment

Draw the grammar derivation and its AST for:

```smalltalk
result := 3 + 4
```

The left-hand side is a variable, while the right-hand side is an expression.

---

## Exercise 10 — Blocks

Draw the grammar derivation and its AST for:

```smalltalk
[ 1 + 2 ]
```

The expression inside the brackets is the body of a block.

---

## Exercise 11 — Blocks with Arguments

Draw the grammar derivation and its AST for:

```smalltalk
[ :x | x + 1 ]
```

Your AST should represent:

* the block argument `x`;
* the body of the block;
* the binary message `+`.
