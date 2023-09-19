```
PhEP {
	#id: 0005,
	#type: #image,
	#title: 'Literal syntax for collections',
	#status: 'proposal',
	#authors: [ 'Stéphane Ducasse' ],
	#created: '2023-09-17'
}
```

# Abstract 

A proposal to be able to write literal collections in the same way we have DynamicArray.
The proposal is based on the idea of Dave Mason to support `{:Set 1 . 2 . 1}` to get a set with 2 elements. This approach works for any collection (assuming that they implement `withAll:`).

# Changelog

- 2023-09-17: initial document

# Motivation

The following illustrates the point

```
(Set new add: 1 ; add: (Set new add: 2; add: 2; yourself);  add: 1 ;yourself)
```

When scripting we would like to be able to write:

```
{:Set 1 . {:Set 2 . 2} . 1}
```

Similarly people could write

```
{:Dictionary #a -> 33 . #b -> 44 }
```

# Discussion

This design is nice since there is no look-ahead. 
In addition, we do not use another special character such as {{ or #{ or whatever.

# Implementation 

It is done and minimal: on extra tree node. We should just move the methods of the `RBParserLiteralCollection` to its superclass.

```
testNestedLiteralSet

	| compiler |
	compiler := OpalCompiler new.
	compiler compilationContext parserClass: RBParserLiteralCollection. 
	self assert: (compiler evaluate: 
		'{ :Set 1 . {  :Set 2 . 2 } . 1}' )
		
		equals:  (Set new add: 1 ; add: (Set new add: 2; add: 2; yourself);  add: 1 ;yourself).
```

See https://github.com/Ducasse/PharoPossibleExtensions/








