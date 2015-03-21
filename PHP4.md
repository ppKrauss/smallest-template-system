See [PHP download](http://code.google.com/p/smallest-template-system/downloads/detail?name=smallest-template-system.php) for more details. The code implements the TemplateSyntax conventions.

Here the issues for adapt PHP5 algorithm to PHP4.

# Simplest algorithms #
For didactic purposes, **here a PHP illustration** of the starting point used in the PHP implementation. See download scripts for serious use.

## Expand function ##

Implementation for PHP, form version 4.0.5 to version 5.2. For newer PHP versions (v5.3+) see [PHP implementation main article](PHP.md).
```php

/**
* string $T is a template, a HTML structure with $K and $X placeholders
* array  $K is a specific language constants for the template.
* array  $lang is the language, a standard 2-letter code. "en", "fr", etc.
* array  $X is a set of name-value (compatible with $T placeholders).
*
*/
function expandVars($T,$X, $prefix='@') { // template engine
global $_expMultTpl_X; // for callback communication
$_expMultTpl_X = $X;
return preg_replace_callback(
"/$prefix([a-z]+[a-z0-9_]*)/i",
create_function(
'$m',
'global $_expMultTpl_X;
return array_key_exists($m[1],$_expMultTpl_X)?
$_expMultTpl_X[$m[1]]:
"";
'
),
$T
);
}
```

## Multilingual ##
Same as [PHP v5.3 implementation](PHP.md).