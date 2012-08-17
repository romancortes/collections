
# Collections

This package contains JavaScript implementations of common data
structures with idiomatic iterfaces.

-   `List(values, equals)`: an ordered collection of values with fast
    insertion and deletion and forward and backward traversal, backed by
    a cyclic doubly linked list with a head node.  Lists support most of
    the Array interface, except that they use and return nodes instead
    of integer indicies in analogous functions.
-   `Set(values, equals, hash)`: a collection of unique values stored like
    a hash table.  The underlying storage is a plain JavaScript object
    that maps hashes to lists of values that share the same hash.
    Values may be objects.  The `equals` and `hash` functions can be
    overridden to provide alternate definitions of "unique".  This
    collection is intended to be replaced by a native implementation
    that does not rely on `hash`.
-   `Map(map, equals, hash)`: a collection of key and value items with
    unique keys, backed by a set.  Keys may be objects.  This collection
    is intended to be replaced by a native implementation that does not
    rely on `hash`.
-   `SortedSet(values, equals, compare)`: a collection of unique values
    stored in stored order, backed by a splay tree.  The `equals` and
    `compare` functions can be overridden to provide alternate
    definitions of "unique".
-   `SortedMap(map, equals, compare)`: a collection of key value pairs
    stored in sorted order, backed by a sorted set.
-   `Iterator(iterable)`: a wrapper for any iterable that implements
    `iterate` or iterator the implements `next`, providing a rich lazy
    traversal interface.

For all of these constructors, the argument `values` is an optional
collection of initial values, and may be an array.  If the `values` are
in a map collection, the the values are taken, but the keys are ignored.

The `map` argument is an optional collection to copy shallowly into the
new mapping.  The `map` may be an object literal.  If `map` implements
`forEach`, the values for each key are copied.  So, `map` may be an
array, where each index is accepted as the key.

`equals(x, y)`, `compare(x, y)`, and `hash(value)` are all optional
arguments overriding the meaning of equality, comparability, and
consistent hashing for the purposes of the collection.  `equals` must
return a boolean.  `compare` must return an integer with the same
relationship to zero as x to y.  `hash` should consistently return the same
string for any given object.

The default `equals` operator is implemented in terms of `===`, but
treats `NaN` as equal to itself and `-0` as distinct from `+0`.  It also
delegates to an `equals` method of either the left or right argument if
one exists.  The default can be overridden by shimming `Object.equals`.

The default `compare` operator is implemented in terms of `<` and `>`.
It delegates to the `compare` method of either the left or right
argument if one exists.  It inverts the result if it uses the falls to
the right argument.  The default can be overridden by shimming
`Object.compare`.

The default `hash` operator is implemented in terms of `toString`, but
defers to the value's own `hash` member function if it provides one.  If
the hash changes, corresponding values will not be retrievable within
sets or maps that use it.  The default `hash` operator can be overridden
by shimming `Object.hash`.

Consistent hashing is tricky in JavaScript since the language
deliberately avoids providing unique values for each object.  However,
in conjunction with `WeakMap`, it is relatively easy to add a [Unique
Label][] to objects.

[Unique Label]: (http://wiki.ecmascript.org/doku.php?id=harmony:weak_maps#unique_labeler)

## Collection Methods

Where these methods coincide with the specification of an existing
method of Array, Array is noted as an implementation.  Array* refers to
shimmed arrays, as installed with the `array` module.

-   `has(key)`: (Map, SortedMap, WeakMap) whether a value for the given
    key exists.
-   `has(value, opt_equals)`: (List, Set, SortedSet, Array*) whether a
    value exists.  collection.  This is slow for list (linear), but
    fast (logarithmic) for Set and SortedSet.
-   `get(key)`: (Map, SortedMap, WeakMap, Array*) the value for a key.
    If a Map or SortedMap lacks a key, returns `getDefault(key)`.
-   `getDefault(value)`: (Map, SortedMap) returns undefined.
-   `get(value)`: (List, Set, SortedSet) gets the equivalent value, or
    falls back to `getDefault(value).
-   `getDefault(key)`: (List, Set, SortedSet) returns undefined.
-   `set(key, value)`: (Map, SortedMap, WeakMap, Array*) sets the value
    for a key.
-   `add(value)`: (List, Set, SortedSet) adds a value.  Sets silently
    drop the value if an equivalent value already exists.
-   `add(value, key)`: (Map, SortedMap, Array*) sets the value for a
    key, convenient in conjunction with `forEach` due to the callback
    argument order.
-   `addEach(values)`: (List, Set, Map, SortedSet, SortedMap, Array*)
    adds all values or key value pairs to this collection.  Works for
    arrays and objects as well as any other collection.
-   `delete(key)`: (Map, SortedMap, WeakMap, Array*) deletes the value
    for a given key.  Returns whether the key was found.
-   `delete(value)`: (List, Set, SortedSet) deletes a value.  Returns
    whether the value was found.
-   `find(value, opt_equals)`: (List, SortedSet, Array*) finds a value.
    For List and SortedSet, returns the node at which the value was
    found.  For SortedSet, the optional `equals` argument is ignored.
-   `findLast(value, opt_equals)`: (List, Array*) finds the last
    equivalent value, returning the node at which the value was found.
-   `findLeast()`: (SortedSet) finds the smallest value, returning the
    node at which it was found, or undefined.  This is fast
    (logarithmic) and performs no rotations.
-   `findLeastGreaterThan(value)`: (SortedSet) finds the smallest value
    greater than the given value.  This is fast (logarithic) but does
    cause rotations.
-   `findLeastGreaterThanOrEqual(value)`: (SortedSet) finds the smallest
    value greater than or equal to the given value.  This is fast
    (logarithmic) but does cause rotations.
-   `findGreatest()`: (SortedSet)
-   `findGreatestLessThan(value)`: (SortedSet)
-   `findGreatestLessThanOrEqual(value)`: (SortedSet)
-   `push(...values)`: (Array, List)
-   `pop()`: (Array, List)
-   `shift()`: (Array, List)
-   `unshift(...values)`: (Array, List)
-   `slice(start, end)`: (Array, List)
-   `splice(start, length, ...values)`: (Array, List)
-   `swap(start, length, values)`: (List, Array*) performs a splice
    without variadic arguments.
-   `wipe()`: (List, Set, Map, SortedSet, SortedMap, Array*) Deletes the
    all values.
-   `concat(...iterables)`: (Array, Iterator, List, Set, Map, SortedSet,
    SortedMap) Produces a new collection of the same type containing all
    the values of itself and the values of any number of other
    collections.  Favors the last of duplicate values.  For map-like
    objects, the given iterables are treated as map-like objects and
    each successively updates the result.  Array is like a map from
    index to value.  List, Set, and SortedSet are like maps from nodes
    to values.
-   `keys()`: (Map, SortedMap, Object) returns an array of the keys
-   `values()`: (Map, SortedMap) returns an array of the values
-   `items()`: (`items`, SortedMap) returns an array of `[key, value]`
    pairs for each item
-   `reduce(callback(result, value, key, object, depth), basis, thisp)`:
    (Array, Iterator, List, Set, Map, SortedSet, SortedMap)
-   `reduceRight(callback(result, value, key, object, depth), basis,
    thisp)`: (Array, List, Map, SortedSet, SortedMap)
-   `forEach(callback(value, key, object, depth), thisp)`: (Array,
    Iterator, List, Set, Map, SortedSet, SortedMap)
-   `map(callback(value, key, object, depth), thisp)`: (Array, Iterator,
    List, Set, Map, SortedSet, SortedMap)
-   `toArray()`: (Iterator, List, Set, Map, SortedSet, SortedMap,
    Array*)
-   `toObject()`: (Iterator, Map, SortedMap, Array*) converts any
    collection to an object, treating this collection as a map-like
    object.  Array is like a map from index to value.
-   `filter(callback(value, key, object, depth), thisp)`: (Array, List,
    Set, Map, SortedSet, SortedMap)
-   `every(callback(value, key, object, depth), thisp)`: (Array,
    Iterator, List, Set, Map, SortedSet, SortedMap) whether every value
    passes a given guard.  Stops evaluating the guard after the first
    failure.  Iterators stop consuming after the the first failure.
-   `some(callback(value, key, object, depth), thisp)`: (Array, List,
    Set, Map, SortedSet, SortedMap) whether there is a value that passes
    a given guard.  Stops evaluating the guard after the first success.
    Iterators stop consuming after the first success.
-   `any()`: (Iterator, List, Set, Map, SortedSet, SortedMap, Array*)
    whether any value is truthy
-   `all()`: (Iterator, List, Set, Map, SortedSet, SortedMap, Array*)
    whether all values are truthy
-   `min()`: (Iterator, List, Set, Map, SortedSet, SortedMap, Array*)
    the smallest value.  This is fast for sorted collections
    (logarithic), but slow for everything else (linear).
-   `max()`: (Iterator, List, Set, Map, SortedSet, SortedMap, Array*)
    the largest value.  This is fast for sorted collections
    (logarithic), but slow for everything else (linear).
-   `one()`: (List, SortedSet, Array*) any single value, or throws an
    exception if there are no values.  This is very fast (constant) for
    all collections.  For a sorted set, the value is not deterministic.
-   `only()`: (List, SortedSet, Array*) the one and only value, or
    throws an exception if there are no values or more than one value.
-   `count()`: (List, Set, Map, SortedSet, SortedMap, Array*)
-   `sum()`: (Iterator, List, Set, Map, SortedSet, SortedMap, Array*)
-   `average()`: (Iterator, List, Set, Map, SortedSet, SortedMap,
    Array*)
-   `flatten()`: (Iterator, List, Set, Map, SortedSet, SortedMap,
    Array*)
-   `zip(...collections)`: (List, Set, Map, SortedSet, SortedMap,
    Array*)
-   `enuemrate(zero)`: (Iterator, TODO List, Set, Map, SortedSet,
    SortedMap, Array*)
-   `sorted(compare)`: (List, Set, Map, Array*)
-   `clone(depth, memo)`: (List, Set, Map, SortedSet, SortedMap, Array*)
    replicates the collection.  If `Object.clone` is shimmed, clones the
    values deeply, to the specified depth, using the given memo to
    resolve reference cycles (which must the `has` and `set` parts of
    the Map interface, allowing objects for keys)
-   `constructClone(values)`: (Iterator, List, Set, Map, SortedSet,
    SortedMap, Array*) replicates a collection shallowly.  This is used
    by each `clone` implementation to create a new collection of the
    same type, with the same options (`equals`, `compare`, `hash`
    options), but it leaves the job of deeply cloning the values to the
    more general `clone` method.
-   `equals(that)`: (List, Set, Array*, TODO SortedSet, Map, SortedMap)
-   `compare(that)`: (TODO)
-   `iterate()`: (List, Set, SortedSet, SortedMap, TODO Array*)
-   `iterate(start, end)`: (SortedSet, TODO List) returns an iterator
    for all values in the half-open interval [start, end), that is,
    greater than start, and less than end.  The iterator is resilient
    against changes to the data.
-   `log(charmap, stringify)`: (Set, Map, SortedSet) writes a tree
    describing the internal state of the data structure to the console.
-   `splay(value)`: (SortedSet) rotates the internal splay tree such
    that the root node is less than or equal to the given value.

### Iterator

-   `dropWhile(callback(value, index, iterator), thisp)`
-   `takeWhile(callback(value, index, iterator), thisp)`
-   `mapIterator(callback(value, index, iterator))`: (Iterator) returns
    an iterator for a mapping on the source values.  Values are consumed
    on demand.
-   `filterIterator(callback(value, index, iterator))`: (Iterator) returns
    an iterator for those values from the source that pass the given
    guard.  Values are consumed on demand.

### Iterator utilities

-   `cycle(iterable, times)`
-   `concat(iterables)`
-   `transpose(iterables)`
-   `zip(...iterables)`: variadic transpose
-   `chain(...iterables)`: variadic concat
-   `range(start, stop, step)`: iterates from start to stop by step
-   `count(start, step)`: iterates from start by step, indefinitely
-   `repeat(value, times)`: repeats the given value either finite times
    or indefinitely

## List

Lists are backed by a cyclic doubly-linked list with a head node.  The
nodes are returned by "find" methods and accepted by "slice" and
"splice" as representatives of positions within the list.  Their
properties and methods are part of the interface of the structure.

-   `prev`: the previous node, or the `head` of the list if this is the
    first node
-   `next`: the next node, or the `head` of the list if this is the last
    node

## Set and Map

Set and map are like hash tables, but not implemented with a block of
memory as they would be in a lower-level language.  Most of the work of
providing fast insertion and lookup based on a hash is performed by the
underlying plain JavaScript object.  Each key of the object is a hash
string and each value is a List of values with that hash.  The inner
list resolves collisions.  With a good `hash` method, the use of the
list can be avoided.

Sets and maps both have a `log` function that displays the internal
structure of the bucket list in an NPM-style.

```
┣━┳ 1
┃ ┗━━ {"key":1,"value":"a"}
┣━┳ 2
┃ ┣━━ {"key":2,"value":"c"}
┃ ┗━━ {"key":2,"value":"d"}
┗━┳ 3
  ┗━━ {"key":3,"value":"b"}
```

## Sorted Set and Sorted Map

A binary splay tree is a balanced binary tree that rotates the most
frequently used items toward the root such that they can be accessed the
most quickly.  `sorted-set` and `sorted-map` are backed by a splay tree.

All map implementations use an underlying set implementation.  Any map
can be implemented trivially atop a set by wrapping `compare`, `equals`,
or `hash` to operate on the key of an item.

The sorted set has a `root` node.  Each node has a `left` and `right`
property, which may be null.  Nodes are returned by all of the "find"
functions, and provided as the `key` argument to callbacks.

Both `sorted-set` and `sorted-map` implement a `log` function which can
produce NPM-style visualizations of the internal state of the sorted
tree.

```
> set.log(SortedSet.ascii)
  .-+ -3
  | '-- -2
.-+ -1
+ 0
| .-- 1
'-+ 2
  '-- 3
```

```
> set.log(SortedSet.unicodeRound)
  ╭━┳ -3
  ┃ ╰━━ -2
╭━┻ -1
╋ 0
┃ ╭━┳ 1
┃ ┃ ╰━━ 2
╰━┻ 3
```

## Map and SortedMap

Maps share most of their implementation through `abstract-map`,
delegating to an `itemSet` property and overriding their operators to
follow the `key` property of each item in the set.  The set does most of
the work.


## Coupling

These collections strive to maximize overlapping implementations where
possible, but also be as loosely coupled as possible so developers only
pay for the features they need in the cost of download or execution
time.

For example, the default operators are simple, but much more powerful
operators can be shimmed, enhancing all of the collections.

Also, collections supply a `clone` method, but it can only do shallow
clones unless you shim `Object.clone`.  `Object.clone` works fine by
itself, but can only resolve reference cycles if you either explicitly
provide a map for its `memo` argument, or shim `WeakMap`.

Another example, every collection provides an `iterate` implementation,
but each is only obligated to return an iterator that implements `next`.
For a much richer iterator, you can buy the `iterator` module and use
`Iterate(collection)` to get a much richer interface.


## References

- a SplayTree impementation buried in Fedor Indutny’s super-secret
  [Callgrind](https://github.com/indutny/callgrind.js). This
  implementation uses parent references.
- a SplayTree implementation adapted by [Paolo
  Fragomeni](https://github.com/hij1nx/forest) from the V8 project and
  based on the top-down splaying algorithm from "Self-adjusting Binary
  Search Trees" by Sleator and Tarjan. This does not use or require
  parent references, so I favored it over Fedor Indutny’s style.
- the interface of ECMAScript harmony [simple maps and
  sets](http://wiki.ecmascript.org/doku.php?id=harmony:simple_maps_and_sets)
- a SplayTree implementation from [JavaScript data
  structures](derrickburns/Javascript-Data-Structures) mainted by
  Derrick Burns that supports change-resilient iterators and a
  comprehensive set of introspection functions.

## Future work

Goals

- tests
- docs
- shallow change dispatch and listeners
- alternative module systems song and dance
- optional new on constructors
- all functions named

More methods

- equals
- compare
- fast list splicing

More collections

- sorted-list (sorted, can contain duplicates, perhaps backed by splay
  tree with relaxation on the uniqueness invariant)
- sorted-multi-map (sorted, can contain duplicate entries, perhaps
  backed by sorted-list)
- multi-map (unordered, can contain duplicates)
- ordered-set (preserves traversal order based on insertion, unique
  values)
- ordered-map (preserves traversal order based on insertion, unique
  keys)
- ordered-multi-map (preserves traversal order based on insertion, may
  contain duplicate keys)
- string-set (set of strings, backed by a trie)
- dict (string-map, map of strings to values, backed by a string set)
- immutable-* (mutation functions return new objects that largely share
  the previous version's internal state, some perhaps backed by a hash
  trie)
- lru-set (least recently used cache)
- lru-map
- arc-set (adaptive replacement cache)
- arc-map

Consolidate shims from ES5-Shim and elsewhere

- weak-map-shim
- object-shim
- object-sham
- array-shim
- array-sham
- date-shim

- array heap implementation
- binary heap implementation

