```
PhEP {
	#id: 0004,
	#type: #image,
	#title: 'Reserving !? in selector for DSL',
	#status: #accepted
	#authors: [ 'Stéphane Ducasse' ],
	#created: '2023-09-17'
}
```

# Abstract 

A proposal to be able to write method selector with interrogation mark.
It is good for building better DSL and in the future better testing methods.
This proposal proposes a simple implementation to play with the idea.

# Changelog

- 2023-09-17: initial document

# Motivation

Characters to express binary operators include ?.

# Description

Here are some motivating examples in favor of being able to use $? in plain selectors (and not only binary message)

## Examples

The following example shows that with the proposed solution we can make sure that returning value methods are always identifiable.

Reading this expression it is not possible to know if the method is a predicate or if is returning an interesing value.

```smalltalk
BasicCommandLineHandler new handleExist: x
```

With the proposal the developer could write

```smalltalk
BasicCommandLineHandler new handleExist?: x
```
And it would avoid that he is forced to read the implementation below to understand that the method is returning a boolean.

Note that another implementation could be

```smalltalk
BasicCommandLineHandler new canHandleExist?: x
```



#### Current implementation of an example
```
BasicCommandLineHandler >> handleExit: exit
	^ self handleExit: exit  for: self
```
```
BasicCommandLineHandler >> handleExit: exit for: aCommandLinehandler

	(Smalltalk isInteractive or: [ Smalltalk isInteractiveGraphic ])
		ifFalse: [ ^ exit pass ].

	exit isSuccess
		ifFalse: [ ^ Error signal: exit messageText ].

	self inform: aCommandLinehandler name, ' successfully finished'.

	"for failing subcommands return self which is used to check if the subcommand failed"
	exit isSuccess
		ifTrue: [ ^ aCommandLinehandler ]
```

### Other cases

The following example is striking.

```
CharacterScanner new handleIndentation
MorphicEventDispatcher new handleStep: anEvent
```

There is not simple way to know if such  messages are returning an interesting value or not. 

With this proposal the author would have the freedom to write

```
CharacterScanner new handleIndentation?

MorphicEventDispatcher new handleStep: anEvent
```

Finally sometimes developers rely on using third person form to convey that the message is a predicate
as in 

```
Object new handles: anException
```

with the proposal this expression can be expressed as 

```
Object new handles?: anException
```

# Analysis of the current situation

In Pharo 12 image, only one implementor is using it. 

```
> (ByteSymbol allInstances select: [ :each | each isBinary ]) select: [ :each | each includes: $? ]
```

```
? association
	^ self withQuery: association
```

Note that all the senders 

```
next

	nextPage ifNil: [ nextPage := 1 ].
	^ [ 
		| aUrl |
		aUrl := self request asUrl ? (#page -> nextPage asString).
		result := self api getRaw: aUrl asString.
		STON fromString: result contents

		] ensure: [ nextPage := nextPage + 1 ]
```

### About !

For exclamation mark the situation is different

```
(ByteSymbol allInstances select: [ :each | each isBinary ]) select: [ :each | each includes: $! ] #(#'!=' #'!==')
```

The value to use ! in selector that are not binary can be questionable because while in Scheme the pattern is to use ! to denote mutation, Pharo is an imperative language so it would not make sense to have method with exclamation mark in a normal situation. 

### About ? for binary selector AND in selector

An alternate approach that would not remove $? from binary characters would be to consider the role of space.

```
x ? foo
```

would be a binary message with an argument foo. And 

```
x ?foo
```

would be a selector '?foo'

This approach has a benefit to make code more readable but at the cost of the possibility to introduce missbehaviors. 

As the author of the proposal at the time of writing I would favor this approach because it is more general and could be applied to other characters.

This alternate solution is backward compatible assuming that the author did format their method nicely which is the case in Pharo 12.

## Implementation Details

The current proposal is accompanied by an implementation.

### Implementation Proposal (1): not using $? in binary selector

```
RBScanner class >> initializeClassificationTable

	PatternVariableCharacter := $`.
	KeywordPatternCharacter := $@.
	CascadePatternCharacter := $;.
	classificationTable := Array new: 255.
	self
		initializeChars: (Character allByteCharacters
			select: [ :each | each isLetter ])
		to: #alphabetic.
	self initializeUnderscore.
	self initializeChars: '01234567890' to: #digit.
	self initializeChars: Character specialCharacters  to: #binary.
	self initializeChars: '().:;[]{}^' to: #special.
	self initializeChars: #($?) to: #alphabetic.
	self
		initializeChars: (Character allByteCharacters
			select: [ :each | each isSeparator ])
		to: #separator
```

```
Character class >> specialCharacters
	^ '+-/\*~<>=@,%|&' , (String
		   with: self centeredDot
		   with: self divide
		   with: self plusOrMinus
		   with: self times)
```

Note that the current implementation is a bit strange since `Character class >> #specialCharacters` returns not the special characters for the parser but the characters for binary selectors. 
Renaming `Character class >> #specialCharacters` to `Character class >> #binaryCharacters` could be a separated task.

### Implementation Proposal (2): supporting both $? in binary selector and plain selector

tobe done.
