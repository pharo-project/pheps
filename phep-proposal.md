```
PhEP {
	#id: 0,
	#type: #image,
	#status: #discussion,
	#title: 'String interpolation for Pharo',
	#authors: [ 'Esteban Lorenzano' ],
	#created: '2022-05-19'
}
```

# Abstract
A proposal to include [String interpolation](https://en.wikipedia.org/wiki/String_interpolation) as part of the Pharo language.

# Motivation
Many of our work today includes some kind of string manipulation (to show messages, write logs, whatever), and the code becomes easily illegible, you can find examples of this all around the image and projects of the ecosystem. This is specially important in projects for the web (producing pages or exchanging data in the form of JSON), since they are usually very "string" dependent.  
This PhEP has as objective to add a simpler way to format strings in Pharo, other than concatenating (which becomes verbose and confusing) and formatting (which puts the variables to insert far from their insertion point).  

# Description 
As the wikipedia states, "String interpolation is an alternative to building string via concatenation, which requires repeated quoting and unquoting; or substituting into a printf format string, where the variable is far from where it is used".  
Pharo supports several ways to format text strings. It implements string concatenation (with the #, message) and string "printf" style formatting (with the #format: message), along with other more or less derivated from this two (macro replacing, stream contents, etc. all are variations in this style).
Each of this mechanism have advantages, but all have as fundamental disadvantage the fact they become cumbersome to use in practice.  
Take into account this different implementations of the same string concatenation, compared with the one proposed:  
```Smalltalk
'Hello ', name, ', what can I do for you?'.
'Hello {1}, what can I do for you?' format: { name }.
'Hello {name}, what can I do for you?' format: { #name -> name } asDictionary.
'Hello [name], what can I do for you?'.
```
This will show clearly the limitations of the first two mechanisms:  
1. string concatenation adds closing string, concatenation, opening string each time we want to insert a variable, making the string difficult to read and the expression difficult to understand. This explodes when we need to do it with several variables (or even expressions).  
2. string "printf" formatting preserves the string readability, but puts the variables or expressions very far away making difficult to match one and the other (even when using "named" insertion points).  

This PhEP does not removes or deprecates the mechanism already existing, it adds a new one.  

## Prototype implementation

About implementation details, there is already a prototype made by @guillep: https://github.com/guillep/pharo-string-interpolation

## Design

The overal design is very simple, thanks to the infrastructure we have built before: We implement it as a **compiler plugin** that will pre-process.  
The basic code is: 

- we create a plugin (from `OCCompilerASTPlugin`)
- in `transform` method, we find the string literals
- we ask if they have an interpolation
- if there are no interpolation, we return and compilation continues and we are the happy owners of a string literal.
- if there are interpolation, we replace the literal with a message send that will actually execute the interpolation
	
in the compiled method, this code:
```Smalltalk
greet := 'Hello [name], what can I do for you?'.
```
will become something like: 
```Smalltalk
greet := StringInterpolator 
	interpolate: 'Hello [name], what can I do for you?'
	withAssociations: { #name -> name }.
```

Implementation details are still to be defined, this could even be a simple call to `#format:` (already existent in String).  

NOTE: This is how the prototype is working now, we still need to solve some minor issues. 


### Backwards compatibility
There may be cases where the string interpolation mechanism is incompatible with already existing packages. To allow this packages to be loaded we will add the capability to disable string interpolation as per package or class basis.

### Tooling
The main restriction for the adoption of string interpolation is the functionning of the tools. For this, we need to make special attention for it to work properly and in a non disruptive way.  
A particular case of it is the debugger. Ideally the stepping on it will not be disturbed by the fact the bytecodes will be affected in presence of interpolated strings. As this objective is hard to achieve, the minimum acceptable interaction level is "to behave as optimized code does now", which means it steps reasonable well, but it does not deoptimize the code.  

### About the syntax to use
The proposal is taking @dionisiydk idea of using block syntax for interpolable strings instead of curly braces. This approach has some obvious advantages: 
- being "as a block", entry point for new users is a lot easier than learning a new syntax.
- interpolation strings become compatible "by design" with other ways of construct strings: #format:, #tokens, etc.

