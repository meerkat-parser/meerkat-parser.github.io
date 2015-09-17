
## Layout

Traditionally, programming languages define whitespace and comment. Whitespaces and comments, also referred to as _layout_, can appear anywhere in a program, and in many programming languages, these parts of the program have no effect on the program execution, and therefore, insignificant. For example, in Java one can type:

    1 + 2 * 3 // equal to 7!
    
instead of ```1+2*3```.

In traditional, two phase parsing, a lexer (the lexing phase) produces a stream of tokens throwing out the layout, and the grammar (parser) is written if no layout exists. In single-phase parsing, there is no separate lexing phase, and a parser has to explicitly deal with layout. For example, a nonterminal defining layout, say ```LAYOUT```, can be inserted between each two symbols in the grammar rules:

    E ::= E LAYOUT '*' LAYOUT E
        | E LAYOUT '+' LAYOUT E
        | Num

Such layout insertion can also be done automatically. Our library supports automatic layout insertion, which can be conveniently used for the most parts of a grammar, manual layout insertion, and no layout insertion.

To support automatic layout insertion, our binary sequence combinator ```~``` declares an implicit parameter of type ```Layout```, so that a parser for layout, defined in the scope of (a part of) the grammar, can be implicitly passed to ```~```. Sequence combinator ```~``` uses this parser and a more basic sequence combinator ```~~``` to compose two argument parsers inserting layout between them.

To define a parser for layout, function ```layout``` (which is similar to ```syn```) can be used as follows:

    implicit val L = layout { """\s?""".r }

    val E: Nonterminal = syn ( E ~ "*" ~ E
                         | E ~ "+" ~ E
                         | Num )

In this case, we use scala regular expression to define layout as optional whitespace. The result of ```layout``` is of type ```Layout```. Also note that ```L``` is declared as an implicit value, and nonterminal parser ```E``` can be defined as if no layout exists. 

Our library provides a default layout definition (```LAYOUT```), which uses the following scala regular expression


    """((/\*(.|[\r\n])*?\*/|//[^\r\n]*)|\s)*""".r


That is, the default layout can recognize whitespace, single line comment and C-style multiline comment.

If no layout is defined in the scope of the grammar, the default layout will be passed to ```~```. Note that the default layout will also be used if ```syn``` is accidentally used in place of ```layout```, or ```implicit``` keyword is missing.