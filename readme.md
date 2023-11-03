# *YaLinqo: Yet Another LINQ to Objects for PHP*

Features
========

* The most complete port of .NET LINQ to PHP, with [many additional methods](#implemented-methods).
* Lazy evaluation, error messages and other behavior of original LINQ.
* [Detailed PHPDoc and online reference](http://athari.github.io/YaLinqo) based on PHPDoc for all methods. Articles are adapted from original LINQ documentation from MSDN.
* 100% unit test coverage.
* Best performance among full-featured LINQ ports (YaLinqo, Ginq, Pinq), at least 2x faster than the closest competitor, see [performance tests](https://github.com/Athari/YaLinqoPerf).
* Callback functions can be specified as closures (like `function ($v) { return $v; }`), PHP "function pointers" (either strings like `'strnatcmp'` or arrays like `array($object, 'methodName')`), string "lambdas" using various syntaxes (`'"$k = $v"'`, `'$v ==> $v+1'`, `'($v, $k) ==> $v + $k'`, `'($v, $k) ==> { return $v + $k; }'`).
* Keys are as important as values. Most callback functions receive both values and the keys; transformations can be applied to both values and the keys; keys are never lost during transformations, if possible.
* SPL interfaces `Iterator`, `IteratorAggregate` etc. are used throughout the code and can be used interchangeably with Enumerable.
* Redundant collection classes are avoided, native PHP arrays are used everywhere.
* Composer support ([package](https://packagist.org/packages/athari/yalinqo) on Packagist).
* No external dependencies.

Implemented methods
===================

Some methods had to be renamed, because their names are reserved keywords. Original methods names are given in parenthesis.

* **Generation**: cycle, emptyEnum (empty), from, generate, toInfinity, toNegativeInfinity, matches, returnEnum (return), range, rangeDown, rangeTo, repeat, split;
* **Projection and filtering**: cast, ofType, select, selectMany, where;
* **Ordering**: orderBy, orderByDescending, orderByDir, thenBy, thenByDescending, thenByDir;
* **Joining and grouping**: groupJoin, join, groupBy;
* **Aggregation**: aggregate, aggregateOrDefault, average, count, max, maxBy, min, minBy, sum;
* **Set**: all, any, append, concat, contains, distinct, except, intersect, prepend, union;
* **Pagination**: elementAt, elementAtOrDefault, first, firstOrDefault, firstOrFallback, last, lastOrDefault, lastOrFallback, single, singleOrDefault, singleOrFallback, indexOf, lastIndexOf, findIndex, findLastIndex, skip, skipWhile, take, takeWhile;
* **Conversion**: toArray, toArrayDeep, toList, toListDeep, toDictionary, toJSON, toLookup, toKeys, toValues, toObject, toString;
* **Actions**: call (do), each (forEach), write, writeLine.

In total, more than 80 methods.

Example
=======

*Process sample data:*

```php
// Data
$products = array(
    array('name' => 'Keyboard',    'catId' => 'hw', 'quantity' =>  10, 'id' => 1),
    array('name' => 'Mouse',       'catId' => 'hw', 'quantity' =>  20, 'id' => 2),
    array('name' => 'Monitor',     'catId' => 'hw', 'quantity' =>   0, 'id' => 3),
    array('name' => 'Joystick',    'catId' => 'hw', 'quantity' =>  15, 'id' => 4),
    array('name' => 'CPU',         'catId' => 'hw', 'quantity' =>  15, 'id' => 5),
    array('name' => 'Motherboard', 'catId' => 'hw', 'quantity' =>  11, 'id' => 6),
    array('name' => 'Windows',     'catId' => 'os', 'quantity' => 666, 'id' => 7),
    array('name' => 'Linux',       'catId' => 'os', 'quantity' => 666, 'id' => 8),
    array('name' => 'Mac',         'catId' => 'os', 'quantity' => 666, 'id' => 9),
);
$categories = array(
    array('name' => 'Hardware',          'id' => 'hw'),
    array('name' => 'Operating systems', 'id' => 'os'),
);

// Put products with non-zero quantity into matching categories;
// sort categories by name;
// sort products within categories by quantity descending, then by name.
$result = from($categories)
    ->orderBy('$cat ==> $cat["name"]')
    ->groupJoin(
        from($products)
            ->where('$prod ==> $prod["quantity"] > 0')
            ->orderByDescending('$prod ==> $prod["quantity"]')
            ->thenBy('$prod ==> $prod["name"]'),
        '$cat ==> $cat["id"]', '$prod ==> $prod["catId"]',
        '($cat, $prods) ==> array(
            "name" => $cat["name"],
            "products" => $prods
        )'
    );

// Alternative shorter syntax using default variable names
$result2 = from($categories)
    ->orderBy('$v["name"]')
    ->groupJoin(
        from($products)
            ->where('$v["quantity"] > 0')
            ->orderByDescending('$v["quantity"]')
            ->thenBy('$v["name"]'),
        '$v["id"]', '$v["catId"]',
        'array(
            "name" => $v["name"],
            "products" => $e
        )'
    );

// Closure syntax, maximum support in IDEs, but verbose and hard to read
$result3 = from($categories)
    ->orderBy(function ($cat) { return $cat['name']; })
    ->groupJoin(
        from($products)
            ->where(function ($prod) { return $prod["quantity"] > 0; })
            ->orderByDescending(function ($prod) { return $prod["quantity"]; })
            ->thenBy(function ($prod) { return $prod["name"]; }),
        function ($cat) { return $cat["id"]; },
        function ($prod) { return $prod["catId"]; },
        function ($cat, $prods) {
            return array(
                "name" => $cat["name"],
                "products" => $prods
            );
        }
    );

print_r($result->toArrayDeep());
```

*Output (compacted):*

```
Array (
    [hw] => Array (
        [name] => Hardware
        [products] => Array (
            [0] => Array ( [name] => Mouse       [catId] => hw [quantity] =>  20 [id] => 2 )
            [1] => Array ( [name] => CPU         [catId] => hw [quantity] =>  15 [id] => 5 )
            [2] => Array ( [name] => Joystick    [catId] => hw [quantity] =>  15 [id] => 4 )
            [3] => Array ( [name] => Motherboard [catId] => hw [quantity] =>  11 [id] => 6 )
            [4] => Array ( [name] => Keyboard    [catId] => hw [quantity] =>  10 [id] => 1 )
        )
    )
    [os] => Array (
        [name] => Operating systems
        [products] => Array (
            [0] => Array ( [name] => Linux       [catId] => os [quantity] => 666 [id] => 8 )
            [1] => Array ( [name] => Mac         [catId] => os [quantity] => 666 [id] => 9 )
            [2] => Array ( [name] => Windows     [catId] => os [quantity] => 666 [id] => 7 )
        )
    )
)
```

Requirements
============
PHP 7.4 or higher.

Usage
=====

Add to `composer.json`:

```json
{
    "require": {
        "serviceinfoadn/yalinqo": "^1.0.0"
    }
}
```

Add to your PHP script:

```php
require_once 'vendor/autoloader.php';
use \YaLinqo\Enumerable;

// 'from' can be called as a static method or via a global function shortcut
Enumerable::from(array(1, 2, 3));
from(array(1, 2, 3));
```
