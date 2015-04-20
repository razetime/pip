#Pip syntax

The syntax of Pip is generally C-like with strong influence from Python, Perl/PHP, and various golfing languages. A few features are reminiscent of Bash script or Lisp.

###Tokens

Any of the following is a token in Pip:

 - Number (consecutive digits, optionally followed by a period and more consecutive digits)
 - String literal (between "double quotes"--no escapes yet, though they are planned)
 - Single-character string literal (a ' single quote followed by any character--no escapes)
 - Operator (one or more symbols or uppercase letters)
 - Lowercase identifier (a single lowercase letter; runs of lowercase letters become multiple identifiers)
 - Uppercase identifier (any run of uppercase letters that is not an operator)
 - Delimiters (parens, square braces, curly braces, semicolon)

Comments come in two types: lines that start with a (possibly indented) semicolon, and anything at the end of a line if preceded by two or more spaces.

`; This is a comment.`
`x:42  This is a comment too.`

Whitespace is not significant except for indicating comments and separating tokens; for example, `(- -5)` returns 5, but `(--5)` returns 4 because `--` is the decrement operator.

### Types

There are currently 5 data types in Pip:

 - **Scalar** represents strings and numbers, similar to scalar variables in Perl. A string in numeric context is converted to a number by taking the leading run of digits, possibly with a minus sign or decimal point.
 - **List** is similar to lists in Python: dynamically resizeable, can hold items of any type including other lists, etc.
 - **Range** has a lower and upper bound, and represents all integers >= to the lower bound and < the upper bound. `5,10` is functionally equivalent to `[5 6 7 8 9]` in most contexts, but potentially more efficient since the numbers need not all be generated. Infinite ranges can be created by passing an upper bound of nil.
 - **Block** represents a code block or function.
 - **Nil** is a singleton type, similar to `null` or `None` from other languages. Note that many situations that would cause a runtime error in other languages (such as dividing by zero) simply return nil in Pip (unless warnings are turned on using the -w or -d flags). Most operators, when given nil as an operand, return nil.

A sixth type, Pattern, is planned (for regular expressions).

Boolean expressions return 0 and 1. The values 0 (and variants like 0.0), "", [], and nil are falsy; all others are truthy.

Many operations, including arithmetic and most string operators, function memberwise on ranges and lists, similar to array-programming languages like APL. For example, `[1 2 3]+[6 5 4]` is `[7 7 7]`, and `"Hello".1,3` is `["Hello1" "Hello2"]`.

### Syntax

Programs consist of a series of statements that are executed one by one. Bare expressions also count as statements. If the program ends with a bare expression, its value is automatically printed.

Expressions use infix operators; precedence/associativity may be coerced using parentheses. Basic operators are mostly chosen to coincide with familiar ones from C, Python, or Perl/PHP: `+-*/%!?` are as expected; comparison operators chain, as in Python; `.`, `X`, and string comparison operators are borrowed from Perl, `//` integer division and `**` exponentiation from Python. A few potential "gotchas":

 - `&` and `|` are logical, not bitwise operators
 - The assignment operator is `:`, freeing up `=` to be the (numeric) equality operator
 - Ternary operators do not have a symbol between their second and third arguments: `a?"Yes""No"`
 - Increment `++` and decrement `--` are pre- only (i.e. you can do `++x` but not `x++`)

Lists are constructed via square braces: `[1 2 3]`. No separator is necessary, except in cases of ambiguity in scanning or parsing: `[-1 2 3]` works as expected, but `[1 -2 3]` is actually `[-1 3]` because the `-` is treated as a binary operator if possible. Here, the expression terminator `;` can be used to eliminate the ambiguity: `[1;-2 3]`.

As in C, assignments are valid expressions that can be used anywhere an expression can be used: in a loop test, for example. Appending a colon to any operator turns it into a compound assignment operator: `x+:5` is equivalent to `x:x+5`; note that this also works with unary and ternary operators: `-:x` flips the sign of the variable.

Another meta-operator is `$`, fold. Prepended to a binary operator, it turns it into a unary fold operation on that operator: e.g., `$+[1 2 3 4]` gives 10. **Note**: the precedence of the compound operator is the same as that of the original binary operator, so `$.[1 2]+3` is interpreted as `$.([1 2]+3)` == `$.[4 5]` == 45. 

As in C, loops and if statements use curly braces to mark off their blocks; the curly braces can be dropped if the block is a single statement.

Curly braces are also used to construct functions, which are first-class objects and can be assigned to variables, passed to other functions, etc. Functions are called via a Lisp-like syntax: `(fxyz)` calls the function f with arguments x, y, and z. (This syntax makes parsing much, much easier!)

Within the function, the arguments are available through the local variables a-e. For example, the function `{a+b}` takes two arguments and adds them; it could be called like so: `({a+b}1 2)`, or assigned to a variable first. For functions with more than five arguments, the local variable g contains the entire list of arguments that were passed in (mnemonic: arGs). The local variable f contains a reference to the current function, allowing recursion even in anonymous functions. Functions do not have a set arity, although a function cannot be called with zero arguments. The last bare expression in a function is its return value. If a function does not end in an expression, it returns nil.

The main program is an implicitly declared anonymous function, which gets its arguments from the command-line args and prints its return value. Thus, a full factorial program can be written as `a?a*(fa-1)1`. Wrap it in curly braces, and it's a factorial function.

If the `(fxyz)` syntax is used with `s` being a list, scalar, or range rather than a function, it is an alternate syntax for subscripting: `(lij)` is equivalent to `l@i@j`, or `l[i][j]` in most languages.

The `M` operator is used to map a function to each element of a list, range, or scalar: `{a*a}M,5` gives `[0 1 4 9 16]`. Currently, the result is always a list.