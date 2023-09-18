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
The proposal is based on the idea of Dave Mason to support `{:Set 1 . 2 . 1}` to get a set with 2 elements.

# Changelog

- 2023-09-17: initial document

# Motivation

It is done

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








