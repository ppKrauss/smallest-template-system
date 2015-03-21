This project is a didactic investigation about the simplest ways to express **template systems** and "template engine" algorithms, and about improved versions of these algorithms for practical use, that we named the "smallest" ones.

The simplest and well-known template is that: "_Hello **X**!_" where **X** is a placeholder, that you can substitute by "world" or by "Mary" to produce "_Hello world!_" or "_Hello Mary!_".

See **how the smallest is inspired by the simplest**:
  * [simplest template syntax](SimplestSyntax.md) and
  * [Simplest template engine algorithms](SimplestAlgorithm.md)

For see the **smallest algorithms**, choose your preferred programming language:
  * [PHP](PHP.md) (and download available)
  * [Javascript](Javascript.md)
  * [PL/pgSQL](PLpgSql.md) (PostgreSQL stored procedures language)

See [habemus-papam project](https://code.google.com/p/habemus-papam/) for a good test-kit and bentchmark reference.

NOTE: there are two main scales of use, under construction at downloads,
  * _string scale_: functions with a `sprintf` functionality (more similar to [Python string templates](http://docs.python.org/library/string.html#formatspec))
  * _document scale_: simple MVC frameworks.