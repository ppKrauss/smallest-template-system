The source-codes (of different programming languages) implements the TemplateSyntax conventions, and use the algorithm below, here it is an ilustrative one (not a formal specification).


# Simplest algorithms #
To achieve the _smallest_, we start with the _simplest_.

This algorithm is an **illustration for didactic purposes**. It was the starting point used in the implementations (see also download for serious use).

## Expand function ##
It is really small: the whole template engine have [2 command lines in PHP](PHP.md). Here an step-by-step description of the `expandVars()` function:


  1. _Get inputs_: the template `T`, and the templating-input values `X`.
  1. _Check `T` by a regular expression_, that search valid placeholders:  `/\$([a-z][a-z0-9_]*)/i`.
  1. _For each expression-found placeholder_, returns the corresponding `X` value, that is, a value in the vector `X` associated with the placeholder label. If no value in `X`, returns empty string.


### Use ###

> `T`<sub>1</sub> = "\n`<`p`>`Hello $name!`<`/p`>`";

> `T`<sub>2</sub> = "\n`<`p`>`... Byebye $name $surname.`<`/p`>`";

> `X`<sub>1</sub> = {('name',"Maria"), ('surname','Joana')};

> `X`<sub>2</sub> = {('name',"Jonh"), ('surname','Smith')};

|call | Rendered HTML outputs|
|:----|:---------------------|
|`expandVars(T`<sub>1</sub>,`X`<sub>1</sub>`)` |<p>Hello Maria!</p>|
|`expandVars(T`<sub>2</sub>,`X`<sub>1</sub>`)`  | <p>... Byebye Maria Joana.</p>|
|`expandVars(T`<sub>2</sub>,`X`<sub>2</sub>`)`  | <p>... Byebye Jonh Smith.</p>|

## Zipper\_join function ##

The _Zipper\_join_ function is an alternative to _Expand_ function when the expansion was made _a priori_, p. ex. by a parser. The "parsed template `T`" can be expressed as a vector, where each pair (`T`<sub>i</sub>, `T`<sub>i+1</sub>) encloses a (indexed) placeholder. Because `X` is also a vector, and have same number of elements (indexed placeholder values), the two vectors, `T` and `X`, are compatible.

See use in the [PLpgSql](PLpgSql.md) or [PHP](PHP#zipJoin_function.md)  implementation.

> ![http://xmlfusion.org/images/ZipJoinFunction2.png](http://xmlfusion.org/images/ZipJoinFunction2.png)

Now, describing the "generic Zip\_join", independent of use or not  with templates. Two string vectors are mixed and converted into a simple string.

Taking two vectors, `L` and `R`, for left and right portions of a "zipper" that we want to join:
`L` is the trailer, and `R` follows. Algorithm step-by-step:

  1. take inputs `L` and `R` as string vectors;
  1. for each `L`<sub>i</sub> item, returns into `Z`<sub>i</sub> the concatenation of  `L`<sub>i</sub> and the correspondent `X`<sub>i</sub>.
  1. return the concatenation (vector join) of all itens `Z`<sub>i</sub>.

## Multilingual/Multistyle helper-functions ##
Using templates in a context where the same content is translated to other languages.

It is usual that a boilerplate text, like a letter of a marketing campaign, need to be translated to another languages.
Example: a letter "<i>Hello @name! ... Goodbye.</i>", in english (en) to be translated to spanish (es) and frensh (fr).

The multilingual algorithm is a two-level templating task. Some static content of the template string, now is dynamic, varying by the language choice.

### Direct ###
```php

function expandVarsDual($T,$K,$X) {
$T = expandVars($T,$K,'#');
return expandVars($T,$X,'@');
}
```

### Cached ###
There are two ways to implement the cache:

  1. by syntax, like expandVarsDual();
  1. by "lazy-expand": same placeholder syntax.

```php

// implementation option 1.
function expandLang($T,$K) { // by syntax default($) + '#'.
$T2 = array();
foreach($K as $lang=>$vals)
$T2[$lang] = expandVars($T,$K[$lang],'#');
return $T2;
}

// implementation option 2.
function expandLazy($T,$K,$prefix='\$') { // expand only available $K
$T2 = array();
foreach($K as $lang=>$vals)
$T2[$lang] = expandVars($T,$K[$lang],$prefix,true);
return $T2;
}
```

### Direct use ###
Use example.
```php

// CONFIGURE:
$T = "<p>#K0 @name @surname!<br/>... #K1.

Unknown end tag for &lt;/p&gt;

"; //structure
$K = array( // static content, for each language
'en'=>array('K0'=>'Hello','K1'=>'Goodbye'),
'es'=>array('K0'=>'¡Hola','K1'=>'Adiós')
);

// USE (with dynamic content):

print expandVarsDual($T, $K['en'], array('name'=>'Jonh', 'surname'=>' Smith')  );
print expandVarsDual($T, $K['es'], array('name'=>'Maria','surname'=>'Joana')   );
```
Results:
```html

<p>Hello Jonh Smith!<br/>... Goodbye.

Unknown end tag for &lt;/p&gt;



<p>¡Hola Maria Joana!<br/>... Adiós.

Unknown end tag for &lt;/p&gt;


```

### Cached use ###
For the same results,
```php

// Same configs: $Kskins and $T
// Same $X
$T2 = expandLang($T,$K); // cache it
print expandVars($T2['en'],$X);
// more uses with $T2['es']...
print expandVars($T2['es'],$X);
// more uses with $T2['es']...
```

And with the expandLazy,
```php

$T = "<p>@K0 @name @surname!<br/>... @K1.

Unknown end tag for &lt;/p&gt;

"; //structure, uniform syntax
$T2 = expandLazy($T,$K['en']); // cache it

print expandVars($T2['en'],$X);
print expandVars($T2['es'],$X);
```

## Skin ##
Another usual kind of two-folding templating process is the
"skin template": when not only CSS changes, each new skin have a different "frame structure". After process _skin template_, we have to process the _content template_.

### Direct use ###
```php

// CONFIGURE:
$Kskins = array( // style-specific structures
'style1'=>'<h1>|

Unknown end tag for &lt;/h1&gt;

', 'style2'=>'<hr/><i>|

Unknown end tag for &lt;/i&gt;

'
);
$T = '#0 Hello @name! #1'; // general structure and static content

// USE (with dynamic content):
$X = array('name'=>'Maria','surname'=>'Joana');
print expandVarsDual($T,$Kskins['style1'],$X );
print expandVarsDual($T,$Kskins['style2'],$X );
```

Results:
```html

<h1> Hello Maria! 

Unknown end tag for &lt;/h1&gt;



<hr/><i> Hello Maria! 

Unknown end tag for &lt;/i&gt;


```

### Cached use ###
For the same results,
```php

// Same configs: $Kskins and $T
// Same $X
$T2 = expandLang($T,$Kskins); // cache it

print expandVars($T2['style1'],$X);
print expandVars($T2['style2'],$X);

// ... for expandLazy,
$T = '@0 Hello @name! @1'; // uniform syntax, and using named placeholders
$T2 = expandLazy($T,$Kskins,'@'); // cache it

print expandVars($T2['style1'],$X,'@');
print expandVars($T2['style2'],$X,'@');
```