# New Set methods

See [formal spec WIP](https://ginden.github.io/set-methods/).

# (Semi)relevant previous discussions

* [Map#map and Map#filter](https://github.com/tc39/ecma262/pull/13)
* [Map.prototype.map and Map.prototype.filter (spec) + Set](https://esdiscuss.org/notes/2014-11-19)
* [Map: filter/map and more](https://esdiscuss.org/topic/map-filter-map-and-more)
* [Original topic regarding this proposal](https://esdiscuss.org/topic/new-set-prototype-methods)
* [Newer topic regarding this proposal](https://esdiscuss.org/topic/new-set-methods-again)
 

# Motivations

* it's consistent with already familiar and *widely used* `Array` API (reduced cognitive overhead)
  * easier refactoring
  * certain function become generic
* reduces need to depend on [Immutable.js `Set<T>`](https://facebook.github.io/immutable-js/docs/#/Set)
* reduces boilerplate code when dealing with common use cases of `Set`
* no new syntax
* allow developers coming from other languages to use familiar APIs

# Adoption

* No npm package duplicating this proposal was found
* Very similar API was found in popular [Collections.js](https://www.npmjs.com/package/collections) (205k downloads per month)
* This proposal is inspired by [Set<T> API from Immutable.js](https://facebook.github.io/immutable-js/docs/#/Set) (3M downloads per month)

## Comparision with Immutable.js

* No static `of` and `fromKeys` methods in this proposal
* No static `intersect`, `union` methods in this proposal
* `union`, `intersect`, `difference` takes single argument
* `map`, `filter`, `some`, `every`,   methods are identical and follow Array API
* no `reverse`, `sort` etc. APIs inherited from `Immutable.Collection` (no sense in unordered collection) in this proposal
* no `flatMap`, `filterNot` etc. APIs not found in Array API in this proposal
* No `Set.isSet`

## Comparison with Collection.js

* Naming (`addEach` vs. `addAll`, `union` vs. `concat` etc.)
* `some`, `filter`, `every`, `map` etc. are identical and follow Array API
* many methods from Collection.js are not present in this proposal (some of them are related to observing collection feature)

## Comparison with other languages

### Java

Java `Set` interface is rather limited and doesn't include APIs found in this proposal.

Though, Java 8 introduces [stream API](http://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html) that allow to deal with arbitrary collections.

Stream API has methods from this proposal like `map`, `filter`, `union` (as `concat`), `find` (as `findFirst`).

Stream API nor `Set` does not include `intersect`, `xor`, `some`, `every`.


### C# HashSet

Most of APIs included in this proposal are present in [C# `HashSet`](https://msdn.microsoft.com/en-us/library/bb359438.aspx)

* `intersect`, `union` (as `Concat`), `difference `,  `map`, `filter` (as `Where`), `some`, `every`, `filter`, `xor` (as `SymmetricExceptWith`) are present
* Other methods outside of scope of this proposal are present (coming from `System.Linq.Enumerable` mostly)

### Other languages

* [F# - Collections.Set](https://msdn.microsoft.com/en-au/vstudio/ee340244(v=vs.89))
* [Haskell - Data.Set](http://hackage.haskell.org/package/containers-0.5.10.2/docs/Data-Set.html)
* [Python - set/frozenset](https://docs.python.org/3.6/library/stdtypes.html#set)
* [Hack - HH\Set](https://docs.hhvm.com/hack/reference/class/HH.Set/)
* [Ruby - Set](https://ruby-doc.org/stdlib-2.5.0/libdoc/set/rdoc/Set.html)

# `.union`, `.intersect` etc. desired signature

Signature of these functions isn't obvious. They can accept:

* single Set
  * best possible performance
  * would slowdown adoption
* multiple Sets
  * looks like rare use case
* Single iterable
  * Consistent with other languages (eg. [LINQ `.intersect` method](https://msdn.microsoft.com/en-us/library/bb460136(v=vs.100).aspx))
* Multiple iterables
  * Used by Immutable.js
  * Certain algorithms can be ineffective without converting arguments to `Set` instances (`intersect` without at least `log(n)` random access seems to be too slow for real world)


# Proposal

This proposal does not change grammar of language. 

New methods are added to `Set.prototype`.

* Array-like methods. These methods replicates functionality of `Array.prototype` methods:
  * `Set.prototype.filter(predicate, thisArg)`
  * `Set.prototype.map(fn, thisArg)`
  * `Set.prototype.find(fn, thisArg)`
  * `Set.prototype.reduce(fn, initialValue)`
  * `Set.prototype.join(separator)`
  * `Set.prototype.some(predicate, thisArg)`
  * `Set.prototype.every(predicate, thisArg)`
* Set theory methods:
  * `Set.prototype.intersect(iterable)` - method creates new `Set` instance by set intersect operation.
  * `Set.prototype.union(iterable)` - method creates new `Set` instance by set union operation.
    * Alternative name ([1](https://github.com/Ginden/set-methods/issues/12#issuecomment-357887331)): `.concat`
  * `Set.prototype.difference(iterable)` - method creates new `Set` without elements present in `iterable`.
  * `Set.prototype.symmetricDifference(iterable)` - returns `Set` of elements found only in either `this` or in `iterable`.
    * Alternative name: `.xor`
* New methods:
  * `Set.prototype.addAll(...elements)` - similar to `Array.prototype.push`. Adds all of arguments to existing `Set`.
    * Alternative name: `.addEach`
  * `Set.prototype.deleteAll(...elements)` - reverse of `addAll`. Remove every `element` in `elements` from existing `Set`.
    * Alternative names: `.deleteEach`


## Rejected Array methods

* `fill` - idea of "filling" set doesn't make sense
* `findIndex`, `indexOf` - sets aren't indexed
* `includes` - already covered by native `.has`
* `shift`, `unshift`, `pop`, `push` - sets don't have idea of "last element" or "first element"
* `forEachRight`, `reduceRight` methods - iteration order for Set should be thought as implementation detail. `reduceRight` explicitly starts from "last element" - but there is no last element in set.
* `sort` - sets are unordered
 
## Not included in this proposal but worth considering

* `Set.prototype.isSubsetOf(otherSet)`
* `Set.prototype.isSupersetOf(iterable)`
* `Set.prototype.flatMap`, `Set.prototype.flatten` - should be added if [`Array.prototype.flatMap`](https://github.com/tc39/proposal-flatMap) is added to language
* Static `Set.union(...iterables)`, `Set.intersect(...iterables)`


# Why not `%IteratorPrototype%` methods

* Explicit usage of iterators is verbose
  * Compare `new Set(set.entries().filter(fn))` to `set.filter(fn)`. 19 characters of boilerplate.
    * Even small codebase can have hundreds occurrences of this pattern - and hundreds places to make a mistake.

## Alternative

* Add methods to `%SetIteratorPrototype%` or even more generic `%IteratorPrototype%` and make `Set` methods to use them internally.
    * Allows for better optimization for many cases (no intermediate collections)
    * **Can be delayed** - `Set` methods can be changed in future to internally use `%SetIteratorPrototype%` and it's unlikely to break the web
        * Code that subclass `Set` **and** redefines `@@iterator` to not use `%SetIteratorPrototype%`.
* [Protocols proposal](https://github.com/michaelficarra/proposal-first-class-protocols)
    
    
```javascript
Set.prototype.map = function map(fn) {
    const Ctor = SpeciesConstructor(this, Set);
    const iterator = this.entries().map(fn);
    return new Ctor(iterator);
}
```
