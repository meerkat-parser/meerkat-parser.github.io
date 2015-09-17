---
---

{% include_relative header.md %}
{% include_relative navigation.md %}

<div markdown="1">

## Getting Started

We use some small examples to show how you can start using Meerkat parsers. Check the 
[Download and Install](https://github.com/Anastassija/Meerkat/wiki/download-and-install) page to get the necessary libraries. You can use a text editor such as [Sublime Text](http://www.sublimetext.com/3) or an IDE such as
[Scala IDE](http://scala-ide.org/) or [IntelliJ IDEA](https://www.jetbrains.com/idea/features/scala.html) to run these examples.

### Imports
The first step is to create a Scala file and add the necessary imports:

{% highlight scala %}
import org.meerkat.Syntax._
import org.meerkat.parsers._
import Parsers._
{% endhighlight %}

### Balanced parenthesis
As the first example, let's write a parser that recognizes balanced parenthesis. We can directly
write the grammar in Scala using the Meerkat library:

{% highlight scala %}
val S: Nonterminal = 
  syn ( "(" ~ S ~ ")"
      | epsilon
      )
{% endhighlight %}

The `syn` function defines a nonterminal and assigns it to `S`. Since the definition of `S` is recursive, we
need to define the return type explicitly, as required by Scala. `S` has two alternatives: the first alternative defines the balanced parentheses and the second alternative produces an empty string, which effectively terminates 
the recursion in the first alternative. The `~` combinator defines the sequence and `|` defines alternation.
Note that the `~` combinator also allows arbitrary whitespace and comments between the symbols. The default whitespace and comment (which we refer to as **Layout**) can be redefined by the user. More information on
Layout insertion in the Meerkat library can be found [here](https://github.com/Anastassija/Meerkat/wiki/layout).

We can parse an input string, say `(())`, using the `parse` method. The parse function returns an instance of
`Either[ParseError, ParseSuccess]` which encapsulates either a parse success or a parse error. We can use Scala pattern matching to process the result of parsing:

{% highlight scala %}
parse(S, "(())") match {
    case Right(success) => println(success.root)
    case Left(error)    => println(error)
    }
{% endhighlight %}

A `ParseSuccess` contains the root of the created parse tree, accessible via `root`, and some statistics about parsing, such as the number of nodes in the parse tree and ambiguities. 
The format of the parse trees Meerkat Parsers produce is explained [here](https://github.com/Anastassija/Meerkat/wiki/parse-tree) in detail. In the next section section, we show how you
can traverse the parse tree and pretty print the brackets.

### Pretty printing balanced parentheses

{% highlight scala %}
import org.meerkat.tree._

val Nt = NonterminalSymbol
val Term = TerminalSymbol
{% endhighlight %}

{% highlight scala %}
def prettyPrint(t: Tree, i: Int = 0): String = t match {
    // S ::= "(" S ")"
    case RuleNode(Rule(Nt("S"), Sequence(Term("("), Nt("S"), Term(")"))), 
                                List(_, s, _)) => s"""|${" " * i}(
                                                      |${prettyPrint(s, i + 2)}
                                                      |${" " * i})
                                                      """.stripMargin
    // S ::= epsilon                                                   
    case RuleNode(Rule(Nt("S"), Sequence(Term("epsilon"))), _)    => ""
}
{% endhighlight %}

</div>

{% include_relative footer.md %}
