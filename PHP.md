See [PHP download](http://code.google.com/p/smallest-template-system/downloads/detail?name=smallest-template-system.php) for more details. The code implements the TemplateSyntax conventions.

# Simplest algorithms #
To achieve the _smallest_, we start with the _simplest_.

Here the source-codes are **illustrations for didactic purposes**. It was the starting point used in the PHP implementation. See download for serious use.

P.S. for colaboration: it is expected to show here the  "more elegant programs", in the  Chaitin's aception of elegant, that is, the shortest program that produces a given output.

## Expand function ##
It is really small: the whole template engine have 2 command lines.

Implementation for PHP v5.3+. For older versions see [PHP4](PHP4.md) implementation.
```php
function expandVars($T,$X, $prefix='\$', $lazy=0) { // it is a Template Processor!
	return preg_replace_callback(
		"/$prefix([a-z][a-z0-9_]*)/i",
		function($m,$X=NULL) use ($X) {
			return array_key_exists($m[1],$X)? $X[$m[1]]: ($lazy? $m[0] :'');
		},
		$T
	);
}
```

PS: to add an "escape syntax" use a prefix like <tt>'(?<!\\\\)\$'</tt> that adds a "not preceded by '/'" assertion in the [PCRE expression](http://www.php.net/manual/en/regexp.reference.assertions.php).

In this algorithm the parsing and expantion are coupled. When parser is isolated, it can generates a "prepared template", see [zipJoin function](#zipJoin_function.md).

### Use ###
```php
$T1 = "\n<p>Hello $name!
  Unknown end tag for </p>
";
$T2 = "\n<p>... Byebye $name $surname.
  Unknown end tag for </p>
";
$X1 = array('name'=>"Maria", 'surname'=>'Joana');
$X2 = array('name'=>"Jonh", 'surname'=>'Smith');
print expandVars($T1,$X1); // 1
print expandVars($T2,$X1); // 2
print expandVars($T2,$X2); // 3
```

|print # | Rendered HTML outputs|
|:-------|:---------------------|
|1 |<p>Hello Maria!</p>|
|2  | <p>... Byebye Maria Joana.</p>|
|3  | <p>... Byebye Jonh Smith.</p>|

See [zipJoin function](#zipJoin_function.md) for caching template.

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
  2. by "lazy-expand": same placeholder syntax.

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
Unknown end tag for </p>

<p>¡Hola Maria Joana!<br/>... Adiós.
Unknown end tag for </p>
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

# zipJoin function #

The [Zipper\_join function](https://code.google.com/p/smallest-template-system/wiki/SimplestAlgorithm#Zipper_join_function) is an alternative to _Expand_ function when the template can be prepared (cached) as an array, by a parser.

> ![http://xmlfusion.org/images/ZipJoinFunction2.png](http://xmlfusion.org/images/ZipJoinFunction2.png)

Two string vectors, the "prepared template" `$t` and the input data `$x`, are merged and converted into a simple string.

```php
function zipJoin(&$t,&$x) {
return join(
array_map(  // concatenating array elements
function($te,$xe){ return $te.$xe; },
$t,
$x
)
);
}
```

## Use ##
As `$t` is the "prepared template", only "cached mode" can be used.

```php
$t = array("Hello ",  "!\n"); // parsed from the "Hello $X!" template
$x0 = array();
$x1 = array("world");
$x2 = array("Mary");
print zipJoin($t,$x0);
print zipJoin($t,$x1);
print zipJoin($t,$x2);
```
Results:
```html

Hello !
Hello world!
Hello Mary!```


## zipJoin\_idx() ##
For practical use of zipJoin() function, input data must be normalized (not repeated) and need  option to use associative array.
There are no PHP-build-in function, like array\_reduce, for optimize this algorithm.

```php

function zipJoin_idx(&$t,&$x,&$Idx) {
$s = '';
for($i=0; $i<count($t); $i++)
$s.= $t[$i] . ((isset($Idx[$i]) && isset($x[$Idx[$i]]))? $x[$Idx[$i]]: '');
return $s;
}
```
Using:

```php

$T = array(" Hello ",  ", Goodbye ", "...\n"); // parsed from "Hello $X, Goodbye $Y" template
$x1 = array("world", "peole");
print "\nUsing idx for repeat element or for change the element order (...):\n";
$idx = array(0=>0, 1=>0); // repeat
print zipJoin_idx($T,$x1,$idx);
$idx = array(0=>1, 1=>0); // selection
print zipJoin_idx($T,$x1,$idx);

print "\n(...) and/or for restrict elements number:\n";
$x2 = array("Mary", "Peter","Jonh"); // more elements than template needs
print zipJoin_idx($T,$x2,$idx);
$idx = array(0=>0, 1=>2);  // selection and restriction
print zipJoin_idx($T,$x2,$idx);

print "\nUsing idx for map an associative array:\n";
$x3 = array('prefix'=>"Captain", 'given'=>"Peter", 'surname'=>"Lachlan");  // label=>value
$rel = array(0=>'prefix', 1=>'surname'); // idx=>label
print zipJoin_idx($T,$x3,$rel);
```

Results:
```html
Using idx for repeat element or for change the element order (...):
Hello world, Goodbye world...
Hello peole, Goodbye world...

(...) and/or for select requested elements:
Hello Peter, Goodbye Mary...
Hello Mary, Goodbye Jonh...

Using idx for map an associative array:
Hello Captain, Goodbye Lachlan...
```

## prepareTemplate() ##
Transform the template into an array, and generates the array of indexes, when required.

```php

function prepareTemplate($T, $prefix='\$') { // Template Pre-Processor
...
}
```
