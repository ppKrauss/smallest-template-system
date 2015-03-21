See TemplateSyntax for "template syntax specification" of the _smallest-template-system project_. Here we explain formal concepts and present a rationale for the adopted syntax.

# Introduction #
A “placeholder mark” can indicate where an value can be expanded into the substitution. The string below is a template,
> “Hello %!”
where the character percent, “%”, is the placeholder mark.
When the template is processed by a _template engine_, the placeholder is replaced by a input value.

The syntax of this specific template is
> string placeholderMark string
or, symbolizing string by a letter _S_ and placeholder mark by _P_,
> T = S P S
That is: the template _T_ is the concatenation of a _S_, a _P_ and another _S_. A template with two placeholders, would something like
> > “Hello %, here I have % monkeys.”
having the syntax

> T = S P S P S
where we see that it requires two distinct input values.
Notice that syntax representation _S S_ is invalid, because the concatenation of two strings is a string. But _P P_ is valid, because it is a sequence of two neighbor placeholders.
The general case for _placeholder template syntax_, expressed as regular expression, is any sequence of _S_ and _P_ with these constraincts:

```
   T = P+ | S | P S | P* (S P+)+ S?
```

How to associate input itens with placeholders?  The order of the placeholders can be used as an information about input selection:
  * the first placeholder (P<sub>1</sub>) recives the first input value (V<sub>1</sub>),
  * the second placeholder (_P_<sub>2</sub>) recives the second input value (_V_<sub>2</sub>),
  * ... and so on ...

So, the natural datatype for input values, to this simplest template syntax, is a vector type (one dimensional array).

> V = (V<sub>1</sub>,V<sub>2</sub>,...)

There are a correspondence between each placeholder (P<sub>1</sub>,P<sub>2</sub>,...) and each vector element (V<sub>1</sub>,V<sub>2</sub>,...), that is, a placeholder with an index _i_ must be replaced by an input with the same index.

> P<sub>i</sub> associated with V<sub>i</sub>

# Labeled placeholders #

There are a practical problem with the simplest (positional) datatype above: when we need to repeat an input value, as in this example,

> “Hello %! Say hello to % (...) Bye %.”

> V = ('Mary','Jonh','Mary')

where the both placeholders, first and third, must be expanded by the same value (Mary). A simple solution, for to normalize `V`,  is,

> “Hello %1! Say hello to %2 (...) Bye %1.”

where “%1” is a placeholder for the first input value,  and “%2” the second input.

Generalizing, if we have a dictionary `D` for translate labels to indexes, we can use "%name" , that is, placeholder marks labeled by the `D` names.  Example:

> “Hello %x! Say hello to %y (...) Bye %x.”

Where `x` and `y` are dictionary entries, and the dictionary `D` maps `x` to 1 and `y` to 2, resulting in the above indexed example.