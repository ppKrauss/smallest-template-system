# Introduction #
Didactical introduction: first the simplest algorithms.

## Simplest by zipper-join ##

![http://xmlfusion.org/images/ZipJoinFunction2.png](http://xmlfusion.org/images/ZipJoinFunction2.png)

```sql

CREATE FUNCTION array_zipper_string(  -- illustred as JoinEngine, of a LR-zipper
anyarray, -- L, the main array
anyarray  -- R, another array with same length or less
) RETURNS text AS $$
SELECT string_agg(t.val||coalesce($2[t.idx],''),'')
FROM  (SELECT generate_subscripts($1, 1) AS idx, unnest($1) AS val) AS t;
$$ LANGUAGE SQL IMMUTABLE;

CREATE FUNCTION tpl_positional(text,anyarray,varchar DEFAULT '%%') RETURNS text AS $$
SELECT array_zipper_string(string_to_array($1,$3),$2);
$$ LANGUAGE SQL IMMUTABLE;

CREATE FUNCTION tpl_positional(text, text, varchar DEFAULT '%%',varchar DEFAULT '|') RETURNS text AS $$
SELECT tpl_positional($1,string_to_array($2,$4),$3);
$$ LANGUAGE SQL IMMUTABLE;

-- -- -- --
-- TESTING:
SELECT tpl_positional('Hello %%!',array['John']);
SELECT tpl_positional('Hello %%!','Maria');
SELECT array_zipper_string(array['Hello ','!'],array['John']);

SELECT tpl_positional('Hello %%! Bye %%...', 'John|Maria');
SELECT tpl_positional('Hello %%! Bye %%...', array['John','Maria']);
SELECT array_zipper_string(array['Hello ','! Bye ', '...'], array['John','Maria']);

```

## Indexing ##

```sql

CREATE FUNCTION arrayidx_zipper_string(anyarray,anyarray,BOOLEAN DEFAULT false) RETURNS text AS $$
SELECT string_agg(COALESCE(
CASE WHEN part[1] IS NOT NULL THEN $2[part[1]::integer] ELSE NULL END,
''
) || part[2]  || COALESCE(CASE WHEN $3=true AND part[1] IS NULL THEN $2[idx]  ELSE NULL END,''), '')
FROM (
SELECT idx, regexp_matches( val, E'^(?:\{([0-9]+)\})?(.*)$' ) AS part
FROM  (SELECT generate_subscripts($1, 1) AS idx, unnest($1) AS val) AS t;
) AS t2;
$$ LANGUAGE SQL IMMUTABLE;
-- TESTING:
SELECT arrayidx_zipper_string(array['Hello ','{2}! Bye ', '{2}... And say hello to ','{3}'],array['','Jonh', 'Maria']);

CREATE FUNCTION tpl_indexed(text, anyarray, varchar DEFAULT '%%', BOOLEAN DEFAULT false) RETURNS text AS $$
SELECT arrayidx_zipper_string(string_to_array($1,$3),$2,$4);
$$ LANGUAGE SQL IMMUTABLE;
CREATE FUNCTION tpl_indexed(text, text, varchar DEFAULT '%%', varchar DEFAULT '|', BOOLEAN DEFAULT false)
RETURNS text AS $$
SELECT tpl_indexed($1, string_to_array($2,$4), $3, $5);
$$ LANGUAGE SQL IMMUTABLE;

-- -- -- --
-- TESTING:
SELECT tpl_indexed('Hello %%!',array['John'],'%%',true);
SELECT tpl_indexed('Hello %%{1}!',array['John']);
SELECT tpl_indexed('Hello %%{1}!','Maria');
SELECT tpl_indexed('Hello %%{1}! Bye %%{1}... Say hello to %%{2}!', 'John|Maria');
SELECT tpl_indexed('Hello %%{1}! Bye %%{2}...', array['John','Maria']);
```

## Caching ##
For big templates (a lot of _placeholders_ and/or big template content), and/or for managing a lot of templates, caching is a good solution.

Usually, the templates are edited by a designer, and then stored. This process is indifferent about  caching. Caching can always be done, and the cost this kind of template-cache (a table of text fragments) is very cheap.

# Complete implementation #

See [downloads](https://code.google.com/p/smallest-template-system/downloads/list).


---


TECHNICAL NOTE: [PostgreSQL 9.1 introduced the format function](http://www.postgresql.org/docs/9.1/static/functions-string.html), but with no  relevant conversion, only <tt>%s</tt>. So, unfortunately, this new 9.X feature can be ignored (the "%I" and "%L" format specifiers are useful only for [constructing dynamic SQL](http://www.postgresql.org/docs/9.1/static/plpgsql-statements.html#PLPGSQL-QUOTE-LITERAL-EXAMPLE)).