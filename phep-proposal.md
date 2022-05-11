```
PhEP {
	#id: 0,
	#type: #image,
	#title: 'Compile-Time Literal Expression Syntax',
	#authors: [ 'Daniel Slomovits', 'Marcus Denker' ],
	#created: '2022-05-11'
}
```

# Abstract 
This PhEP describes an extension to the compiler that allows arbitrary expressions to be evaluated at compile time, with the results stored in the method's literal frame. This is inspired by and will be compatible with the implementation in Dolphin Smalltalk. I refer to such expressions interchangeably as "compile-time literals", "compile-time expressions", and "optimized expressions".

# Motivation
To provide a generic way of embedding large or expensive-to-construct, but ultimately static, objects within a method. This can result in more readable code by keeping e.g. lookup tables closer to their usage rather than storing them in class variables and lumping the values together in #initialize. More generally, it acts as a fully-generic (if not always the most concise) "object literal" which is automatically compatible with any conceivable custom data structure etc, and does not risk violating invariants by manipulating instance variables directly.

# Description

## Syntax

The proposed syntax is `##(<sequence>)`, where `<sequence>` is an `RBSequenceNode`, that is, zero-or-more temps (almost always none) followed by zero-or-more (almost always exactly one) statement(s). This syntax can appear anywhere a literal value such as `1` or `#foo` is valid, including within an array.

### Note on ambiguity

Right now, the parser accepts any number of leading `#` characters before symbol and array literals: `#(#foo)` and `##(#foo)` are treated as equivalent. This proposal would change that, since the latter would now be a compile-time expression evaluating trivially to `#foo`. I believe the only true ambiguity is with either single-element arrays, or the use of Squeak hash-less symbol-array notation, e.g. `#(foo bar)` to mean `#(#foo #bar)`. `##('foo' size)` currently evaluates to `#('foo' #size)`, where under this proposal it would evaluate to `3`. Other cases will either not parse as a compile-time expression (e.g. `##(#foo #bar)`, `##(1 2 3)`), or will encounter an error evaluating (`##('foo' notAMessage)`).

I propose deprecating both multiple hashes and hash-less symbols in arrays, with appropriate warnings. When `##()` is encountered, we should attempt to parse it as a compile-time expression. If the parse fails, we can mention "if you meant this to be an array, remove the extra `#`". In the rare case where the parse succeeds but evaluation fails, we may again want to modify the error message.

## Semantics

`<sequence>` is evaluated similarly to a DoIt, with the instance class of the method as the receiver--that is, the receiver is `Object` whether the method is `Object>>foo` or `Object class>>foo`. The value of the last statement is stored in the method's literal frame, and the expression is treated as a literal with that value.

CONSIDER: Should the value be made read-only (#isReadOnlyObject: true)? Mutating the value of a compile-time expression is obviously a really bad idea, but Dolphin does not do this. I don't see it as a major gotcha either way.

## Examples

The most common use-case in my experience is to construct a literal Dictionary for methods whose primary purpose is to look things up *in* said dictionary:

```
someLookupMethod: arg
	^##(Dictionary new
		at: #foo put: #bar;
		...;
		yourself) at: arg
```

There is also a clever trick that Dolphin uses all over the place to eliminate hard-coding the class name of the receiver:

```
isAbstract
	^self == ##(self)
```

Also applicable to classes that *can* be subclassed, but have certain behavior that only applies to that *exact* class, e.g.

```
Array>>isLiteral
	^self class == ##(self) and: [self allSatisfy: [:each | each isLiteral]]
```

We can also create truly-literal Arrays of classes, e.g. `#(##(Object) ##(Array))`, though in practice brace-array notation is probably superior. And, we can choose to reveal the intent of magic constants that have a direct literal representation, e.g. `seconds := days * ##(60 * 60 * 24)`.

## Implementation

Much of the implementation *aside* from the actual code-generation and decompilation components can be ported directly from Dolphin Smalltalk.

### Parsing

The parser should be a straight port from Dolphin--in summary:

* New parse node class, `RBOptimizedNode`, under `RBValueNode`--sibling to `RBLiteralNode`.
* New lexer token class, `RBOptimizedToken`, with additional case in `RBCanner>>scanLiteral`.
* New `RBParser>>parseOptimizedExpression`, called when an optimized token is encountered as a primitive value or within another literal.

### Code Generation

We must essentially re-invoke the compiler for each optimized expression, applying the same transformation that a Workspace DoIt does (wrapping in a method and converting the last statement to a return). The value is stored as a literal and bytecode generation proceeds as for an ordinary literal node.

We should almost certainly also store the CompiledCode for the optimized expression in the literal frame--albeit after those literals which are actually used--in order for literals within it to be found by "Browse References". This may also assist with decompilation.

### Decompilation

Decompiling an optimized expression is somewhat fraught--the first challenge is simply *recognizing* one, and this is not always possible without attaching additional debug information.

The simplest approach would be to recognize when a literal is not one of the types with a primitive literal representation, and output a compile-time expression containing its #storeString (which we'd probably need to parse and re-format). This would not recognize the `##(self)` and `##(60 * 60 * 25)` cases above, but would produce something that should at least compile back to produce the same output, so long as the objects involved properly implement #storeString.

A more complete approach would be to maintain a mapping, or a scheme by which one can be derived, from the literal results themselves to the CompiledCode which produced them (also stored in the literal frame per above). Then when we encounter a "push literal <n>" bytecode, check if literal <n> exists in the mapping, and if so, re-invoke the decompiler on the corresponding CompiledCode and insert that tree as the body of a new RBOptimizedNode in the output.