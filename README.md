# Compiler Front End: Scanner and Recursive-Descent Parser

The front end of a compiler for a small custom language, written in C++. A **lexical scanner** turns
source text into tokens, and an **LL(1) recursive-descent parser** checks the program against the
language's grammar and builds a **parse tree**, which is then printed so its structure can be checked.

*Course project for CS 4280 (Program Translation), University of Missouri–St. Louis. Projects P1
(the scanner) and P2 (the parser) build on each other.*

## Example

Input program:

```
program
  var x : 5 ;
start
  read x ;
  if [ x > 3 ] then print x * 2 ;
trats
exit
```

The parser prints the tree in preorder, indented by depth, one node per line, with the tokens
stored at each node. The output looks like this:

```
<program>
  <vars> (KW_tk,'var') (IDENT_tk,'x') (NUM_tk,'5')
  <block>
    <stats>
      <stat>
        <in> (IDENT_tk,'x')
      ...
```

## The language

```
program → "program" vars block "exit"
vars    → "var" IDENT ":" NUMBER ";" vars | ε
block   → "start" stats "trats"
stats   → stat mStat
mStat   → stat mStat | ε
stat    → in | out | block | if | loop | assign
in      → "read" IDENT ";"
out     → "print" expr ";"
if      → "if" "[" expr RO expr "]" "then" stat
loop    → "while" "[" expr RO expr "]" stat
assign  → "set" IDENT ":" expr ";"
expr    → M expr'          expr' → ("+"|"-") M expr' | ε
M       → T M'             M'    → ("*"|"%") T M' | ε
T       → IDENT | NUMBER | "(" expr ")"
RO      → "<" | "<=" | ">" | ">=" | "eq" | "neq"
```

## How it works

- **Scanner** (`scanner.cpp`, `token.h`): reads characters and produces typed tokens (keywords,
  identifiers, numbers, operators), each tagged with its line number. Keywords such as `start`/`trats`
  and the word operators `eq`/`neq` are recognized from a lookup table.
- **Parser** (`parser.cpp`): one function per grammar rule. Each function uses the rule's FIRST set
  to pick a production with one token of lookahead. That's why the grammar is written with `expr'`
  and `M'`: it removes left recursion so a recursive-descent parser can handle it.
- **Error reporting:** on unexpected input the parser reports the expected token, the token it
  found, and the line number.
- **Parse tree** (`node.cpp`): each call builds a node labeled with its rule and keeps the meaningful
  tokens. `testTree.cpp` prints the tree in preorder, and the tree is freed when the program ends.

## Build and run

```bash
make                 # builds ./parser
./parser program.txt # parse a file
./parser < program.txt   # or read from standard input
make clean
```

## Files

| File | Purpose |
| --- | --- |
| `scanner.cpp`, `scanner.h`, `token.h` | Lexical analysis (P1) |
| `parser.cpp`, `parser.h` | Recursive-descent parser, one function per rule |
| `node.cpp`, `node.h` | Parse-tree nodes, creation and cleanup |
| `testTree.cpp`, `testTree.h` | Preorder tree printer |
| `main.cpp` | Command-line entry point |

The original assignment notes are in [`README.txt`](README.txt).
