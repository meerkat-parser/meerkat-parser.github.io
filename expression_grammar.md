
## An Expression Grammar

As a first step in building a parser for an expression grammar, we define an Algebraic Data Type (ADT), represented by the following Scala case classes:

{% highlight scala %}
sealed trait E

case class Add(l: E, r: E) extends E
case class Mul(l: E, r: E) extends E
case class Sub(l: E, r: E) extends E
case class Div(l: E, r: E) extends E
case class Neg(l: E)       extends E
case class Pow(l: E, r: E) extends E
case class Num(n: Int)     extends E
{% endhighlight %}

As can be seen, the trait `E` defines an expression and each case class defines a constructor, representing an
operation on the expressions. For example `Add` represents addition. 

A central philosophy in Meerkat parsers is that the user thinks in terms of the semantic model, rather than a
specific parsing technology when writing a parser. This means a natural grammar which is close to the underlying
ADT the user has in mind. In case of our expression grammar, we would ideally want a grammar as follows:

{% highlight ebnf %}
E ::= E '+' E
    | E '*' E
    | E '-' E
    | E '/' E
    | '-' E
    | E '^' E
    | Int
{% endhighlight %}

However, this grammar is ambiguous. For example `1+2*3` can be recognized as either `(1+2)*3` or `1+(2*3)`.
As we remember from the basic arithmetics, we want to specify the *right* derivation using the precedence of operators as follows:

| Operator      | Associativity |
| ------------- |:-------------:|
| ^             | right         |
| - (unary)     | -             |
| *  /          | left          |
| + -           | left          |

In this table, `^` has the highest priority and is right associative, meaning that `1^2^3` should be grouped
as `1^(2^3)`, and not as `(1^2)^3`. The operator precedence decreases from top to down, i.e., `^` has the highest and `+` and `-` have the lowest precedence. Operators with the same precedence level are shown in the same row, e.g., `*` and `/`, which are left associative and left associative with regard to each other, meaning that 
`1*3/4` should be grouped as `(1*3)/4` and not `1*(3/4)`.

Meerkat parsers enable you to directly encode the natural grammar of expressions and operator precedence in Scala.
Let's first start by writing the grammar in combinator style in Scala:

{% highlight scala %}
val E: Nonterminal
= syn ( E ~ "^" ~ E
      | "-" ~ E
      | E ~ "*" ~ E 
      | E ~ "/" ~ E 
      | E ~ "+" ~ E 
      | E ~ "-" ~ E
      | "(" ~ E ~ ")"
      | "[0-9]".r 
      )
{% endhighlight %}

Here, we defined a nonterminal parser `E`. As in Scala, recursive definitions should have the return type,
we have specified the return type as `val E: Nonterminal`. The `|` and `~` combinators as usual represent alternative and sequence. The `syn` function defines a syntactic construct and is essential in dealing with
left recursion and memoization. In Meerkat parsers, terminals can be represented by String, as in `"^"` or
regular expressions, as in `"[0-9]".r`.

Now we use the `|>`, `left`, and `right` to specify priority, left and right associativity, respectively.
Note that all these combinators operate on the whole alternative, rather than just operators, as is the case
in Yacc for example. This way we don't need to distinguish between unary and binary minus, say, as it is clear
from the rule they appear in. To add operator precedence information, we need to first change the parser type
to `OperatorNonterminal`, and then replace the alternative combinator, `|`, with `|>`, and specify left and right associativity. 

{% highlight scala %}
val E: OperatorNonterminal
= syn ( right ( E ~ "^" ~ E )
      |> "-" ~ E 
      |> left ( E ~ "*" ~ E 
      | E ~ "/" ~ E )
      |> left ( E ~ "+" ~ E 
      | E ~ "-" ~ E )
      | "(" ~ E ~ ")"
      | "[0-9]".r
      )
{% endhighlight %}

As can be seen from this example, operators that have the same priority are put in the same associativity group, e.g, `left ( E ~ "*" ~ E | E ~ "/" ~ E )`. More information about operator precedence in Meerkat parsers can be found [here](https://github.com/Anastassija/Meerkat/wiki/operator-precedence).
