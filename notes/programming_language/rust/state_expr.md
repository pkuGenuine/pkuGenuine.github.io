# Statement & Expression

We talked earlier that function bodies are made up of a series of statements optionally ending in an expression. Actually, statements and expression are components of a block, which is in turn a component of an outer expression or function.

For example, you can write the code below:

~~~rust
{
    let x = 1; // statement
    let y = 0; // statement
    x + 1      // expression
}
~~~

which is a block and also an expression that evaluates to `2`. Besides, it follows the "a series of statements optionally ending in an expression" rule.

I won't dive into the details of the language construct. For detailed information, exam [The Rust Reference](https://doc.rust-lang.org/reference/introduction.html).

## Expression

An expression may have two roles: it always produces a value, and it may have effects ( otherwise known as "side effects" ).

As we mentioned before, calling a function is an expression. `if` block is also an expression.


## Statement 

~~~
Syntax
Statement :
      ;
   | Item
   | LetStatement
   | ExpressionStatement
   | MacroInvocationSemi
~~~

> Aside: It is interesting that `;` itself is an statement. You can add as many as you want inside a function.

Rust has two kinds of statement: declaration statements and expression statements.

### Declaration Statement

A declaration statement is one that introduces one or more names into the enclosing statement block. The declared names may denote new variables or new items ( for example, a function ).

**Item declarations**:

~~~rust
fn outer() {
  let outer_var = true;

  fn inner() { /* outer_var is not in scope here */ }

  inner();
}
~~~

**`let` statements**:

~~~rust
let x = 3;
~~~


### Expression Statement

An expression statement is one that evaluates an expression and ignores its result. As a rule, an expression statement's purpose is to trigger the effects of evaluating its expression.

~~~rust
v.pop();          // Ignore the element returned from pop
~~~

An expression that consists of only a block expression or control flow expression, if used in a context where a statement is permitted, can omit the trailing semicolon.

~~~rust
if v.is_empty() {
    v.push(5);
} else {
    v.remove(0);
}    // Semicolon can be omitted.
~~~

When the trailing semicolon is omitted, the result must be type `()`.


~~~rust

#![allow(unused)]
fn main() {
    // bad: the block's type is i32, not ()
    // Error: expected `()` because of default return type
    if true {
        1
    }

    // good: the block's type is i32
    if true {
        1
    } else {
        2
    };
}
~~~


BTW, if a block doesn't end with an expression, its return type will be `()`.

One more note: the return type of a control flow expression should be consistent. For example:

~~~rust
// bad
// two block have different return type.
if true {
    0
} else {
    String::from("0")
}
~~~

The code above can not be compiled.

