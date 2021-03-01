# <p style="text-align: center;"> Scanner </p>

Unlike crafting interpreters which uses classes to write different aspects of a scanner we are going to using the `Module` feature in OCaml to write the different parts of Scanner.


## <p style="text-align: center;"> Defining Types </p>

The first step of scanning involves breaking the source code as tokens (series of characters or chunks of characters which make sense to the compiler). This is done by comparing the string (input source code also known as lexeme).

We can do this by comparing the strings, I know in the crafting interpreters this is considered slow and ugly but let's see how to first implement it badly in a new language.

So first we need to take the input string and convert it to a list of characters. The type of the function is going to look like <br>
`[INPUT] : [Type: String]` <br>
 `[OUTPUT] : [Type: List of Tokens]`

In OCaml when we want to represent just the function signature (also known as an interface in OCaml) it is represented as <br>

```
val <function name> : 
    <type of first input> -> <type of second input> -> <type of third input> -> ... -> <type of nth input> -> <type of the output>
```

Looking at the definition above and also looking at how OCaml functions look like I can roughly create a function interface for converting source to tokens in the following way : <br>

`val scan_tokens : string -> token list`

I have taken the liberty to copy the name from [Crafting Interpreters]() also you might think what is a `token list` and you might be correct to point out that it's not intutive. But think of this as foreshadowing (a coverup for my bad writing skills). The only think that you gotta know is `token list` is a way to `List<token>` in Java or `A List of token` in English.

Before we define what a token is we need to define a `token_type` which is the enum mentioned in [Scanning](https://craftinginterpreters.com/scanning.html) chapter in [Crafting Interpreters](). Also the I've copied most of the scanner code from https://github.com/ludwigpacifici/saumon.

Just to make things clear there are some things that I have changed to make sense for me but I wanted to mention the great work done by the good samaritan mentioned above.

## <p style="text-align: center;"> The token type </p>

This is the first thing that is mentioned in [Crafting Interpreters]() is an enum called `TokenType`. The best way to do it in OCaml is by converting it into a [`Sum Type`](https://en.wikipedia.org/wiki/Tagged_union).

```
type token_type =
  | LEFT_PAREN
  | RIGHT_PAREN
  | LEFT_BRACE
  | RIGHT_BRACE
  | COMMA
  | DOT
  | MINUS
  | PLUS
  | SEMICOLON
  | SLASH
  | STAR
  | BANG
  | BANG_EQUAL
  | EQUAL
  | EQUAL_EQUAL
  | GREATER
  | GREATER_EQUAL
  | LESS_EQUAL
  | LESS
  | IDENTIFIER of string
  | STRING of string
  | NUMBER of int
  | AND
  | CLASS
  | ELSE
  | FALSE
  | FUN
  | FOR
  | IF
  | NIL
  | OR
  | PRINT
  | RETURN
  | SUPER
  | THIS
  | TRUE
  | VAR
  | WHILE
  | EOF
[@@deriving show { with_path = false }, eq, ord]
```

The only thing that I am not going to explain is the `[@@@deriving show { with_path = false }, eq, ord]` part because the only thing I know is that it's a `ppx` and this specific ppx is [`ppx_deriving`](https://github.com/ocaml-ppx/ppx_deriving). And the way I think about how it works is similar to [`procedural macros`](https://doc.rust-lang.org/reference/procedural-macros.html#:~:text=Procedural%20macros%20allow%20you%20to,crate%20type%20of%20proc%2Dmacro%20.) in rust. While the idea in rust might have been inspired from OCaml it makes more sense to explain things in terms of the language which is in the current zeitgeist (for now :P). So it takes the type and generates some functions for the `token_type`.

One of those functions is `show_token_type` which converts the type to string and it's something that I have used in my source code. The following code displays that :

`let to_string s = Printf.sprintf "%s\n" (show_token_type s)`

In rust this can be done by deriving a `Debug` proc macro and using the `println!("{:#?}", ..)` declarative macros with `#?` string. This comment was added just to give a one-to-one correlation between using `derving` ppx and `Debug` macro, it has nothing to do with the actual implementation but it makes things easier for a programmer.

Now coming back from the tangent we have defined what the `token_type` is, now we need to define tokens. So unlike the Java implementation which can have `mutable variable` inside a `class` with `functions and constructors` <br>
```
class Token {
  final TokenType type;
  final String lexeme;
  final Object literal;

 Token(TokenType type, String lexeme, Object literal, int line) {
    this.type = type;
    this.lexeme = lexeme;
    this.literal = literal;
    this.line = line;
  }

  public String toString() {
    return type + " " + lexeme + " " + literal;
  } final int line; 
}
```

To convert the above into OCaml I tried to do something like this

```
Module Token = struct
    type token = {
        _type  : TokenType.token_type;
        location : Location.location
    }

    ...
    ...
end
```

Now there might be valid questions circling in your head. Let's start by answering them one by one :

-  Q : "What are these braces syntax? I thought the `type` keyword is associated with `Sum types`.<br>
   A : This is called a [record syntax](https://www.cs.cornell.edu/courses/cs3110/2019sp/textbook/data/records.html) and the `type` keyword can be used to define new types and this is another variant of types that are used called `Product types`

- Q:"What do `TokenType` and `Location` mean?<br>
  A: So to answer that one needs to understand what are OCaml [`Modules`](https://www.cs.cornell.edu/courses/cs3110/2020sp/textbook/modules/ocaml_modules.html) and it might take some time to wrap your head around it but they are extremenly powerful (it's so powerful that I don't have yet to see it's final form while I'm writing this post). Also I did mention in the first paragraph this is what I'm going to do.

  There is a rough conversion table in my mind<br>

  | Java | OCaml |
  | ------ | ----------- |
  | Class  | Module |
  | functions | functions |
  | mutable variables | records |
  | enums | sum types

  Let me be clear - *<b>The table above is wrong</b>*, there are many holes which programmers from each camp will point out. But up until now this seems to be working fine. If this doesn't work I'll happily take any suggestion that is helpful.

  So coming back to the answer we have made 2 different modules `TokenType` and `Location` which have `token_type` and `location` types respectively. Although we have defined `token_type` we haven't gotten into what `location` is, it looks something like this
  ```
  Module Location = struct 
    type location = {
        line : int;
        column : int;
    }
    ...
    ...
  ```
  which again goes back to the idea of using a line and column way to locate characters which was rejected in Crafting Interpreters because it is slow. And since I'm a total noob I think this can be forgiven (or not :D).

  ## <p style="text-align: center;"> Re-colleting stuff </p>

  So if we sum up we have talked about 3 modules which define how a token is formed - `TokenType`, `Location`, `Token`. 

  But we haven't discussed this uptill this point. We will go at it in the next chapter where we discuss how we use OCaml's pattern matching to tokenize the raw lexemes.