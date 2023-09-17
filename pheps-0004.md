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

A proposal to be able to write method selector with interrogation and exclamation marks.
It is good for building better DSL and in the future better testing methods.
This proposal contains one implementation of one method!

# Changelog

- 2023-09-17: initial document

# Motivation

Characters to express binary operators include ! and ?.
However none of exisiting binary methods use them. 


# Description

Here are some motivating examples.

### Examples

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




#### current Implementation of the example
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





## Implementation Details

The current proposal is accompanied by an implementation in form of pull requests.
This section describes some implementation details.

### Implementation Proposal

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
	self initializeChars: #($? $!) to: #alphabetic.
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
