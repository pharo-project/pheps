```
PhEP {
	#id: 0,
	#type: #image,
	#title: 'Underscores in Numeric Literals',
	#status: #discussion,
	#authors: [ 'Jean Privat' ],
	#created: '2023-02-13'
}
```

# Abstract 
This PhEP describes the extension of Pharo numeric literals to accepts (and ignore) underscore characters (`_` ASCII 95).

# Motivation

Many languages (including Python https://peps.python.org/pep-0515/ , Java https://docs.oracle.com/javase/7/docs/technotes/guides/language/underscores-literals.html or Ruby) accept some forms of numeric literal that ignore _.

The idea is to permit long literals that are still readable, eg. `1_000_000_000` is easier for a human than `100000000` especially since in the previous literal a zero is missing (I'm a tricky deceitful fellow).

## Related work

The PEP (Python) proposal includes a good history and motivation on the issue.


# Description

The main effect is to adapt the state machine of `NumberParser` to accept and ignore underscore.
A proof of concept is proposed as a distinct PR: https://github.com/pharo-project/pharo/pull/12479

Note that acceptance of underscore will not be optional since `NumberParser` is used to parse Pharo numerics and deals with Pharo syntax specificity that are not intended for generic number parsing.
E.g. `(NumberParser parse: '2r10.01e2') >>> 9.0`.
Note: in fact, `NumberParser` accepts a superset of Pharo language numeric as it accept special float strings. e.g. `(NumberParser parse: 'Infinity') >>> Float infinity`

## Backward compatibility

Existing valid numeric literals (without underscore) will still be parsed as the exact same numeric literals (including the combination of radix, decimal point and exponent parts).

The proposal will render valid literals that are currently. However, there is no expectation of problematic breakage.

In Pharo code, number literals starting with a `_` will still be parsed as identifiers (because the first character is used to decide the type of the token). It is an acceptable behavior.

The only issue might be in source code for selectors starting with an underscore and used on numeric literal without spacing. E.g. `10_foo` meaning `10 _foo` but a full search in the default image source code shows that (i) such call sites do not exist, (ii) there is only four selectors that start with `_` (all in the `ReferenceFinder` class that is not related to the numeric class hierarchy).

Note that old Pharo parser will not be able to parse new numeric literal with `_`.
No specific plan is proposed to address this issue except updating the image with a newer NumberParser.
Numeric objects and compiled methods are unaffected (since the internal object representation of numeric does not change).

## Constraints on the underscore

### In other languages

Languages like Java, Python or Ruby add some constraints on the placement of `_` (with subtle differences between them).
For instance, a common rule is no trailing `_`.

```
$ echo 'class X { int x = 1_; }' > X.java && javac X.java
X.java:1: error: illegal underscore
class X { int x = 1_; }
                   ^
1 error

$ python3 -c 'print(1_)'
  File "<string>", line 1
    print(1_)
           ^
SyntaxError: invalid decimal literal

$ ruby -e 'print(1_)'
-e:1: trailing `_' in number
print(1_)
```

In these languages, such errors are HARD errors and correspond to a invalid numeric literal (not just a random illegal character).

### Proposal for Pharo

Instead of a formal language definition or a list on complex rules, the following single rule is considered: LEGAL=*`_` is legal only between 2 digits*.

This means that leading, trailing and duplicated `_` are forbidden including around special characters `r`, `.` and `e`.

* Examples of valid literals: `1`. `1_1`. `1_1.1_1`. `1_1r1_1.1_1e1_1`.
* Examples of invalid literals: `_1`. `1_`. `1__1`. `1_.1`. `1_e1`.

Here a discussion on  behaviors for Pharo are:

1. Implements LEGAL in the NumberParser class but still follows the principle of the longest valid numeric token.

   `1_1_a` will be parsed as `1_1` (longest valid literal according to LEGAL) followed by `_a` ie `11` (integer) and `_a` (unary message). No error or warning message.

   This behavior is error prone since weird syntax will be unexpectedly and silently parsed as "something".
   However, this is the closest thing to the current behavior according to the current implementation. (e.g. can you guess how `16rDeADMAN1` is currently parsed?)
   Syntax highlighting (both number literal and unknown message) and fast runtime error is expected to help the programmer to deal with these cases.

2. Implements LEGAL in the NumberParser class but introduce an alternative illegal numeric literal as possible token.

  `1_1_a` will be parser as an error token `1_1_` ("numeric with trailing `_`") followed by the message send `a`.

   This behavior is the closest to the other considered languages (Java, Python, Ruby).

   Currently, `NumberParser`usually tries to parse until it can't. In most cases, the caller either accepts the longest valid numeric literal or complains if the whole input is not used.
   The equivalent of "a bad formatted numeric token" is handled trough the method `NumberParser>>#expected:` that signals a runtime error or callback the client or UI (the ugliness of this design is out of the scope of the discussion). E.g. `NumberParser parse: '0r'`.

3. Accept unrestricted use of `_` at the parsing level (no constraints at all) following the existing principle of the longest valid numeric token. LEGAL is applied at a second step at the Rule level (ie as a critique/warning of bad style).

   For instance, it means that `1_1_a` will be parsed as `1_1_` (longest unrestricted numeric literal) followed by `a`.
   ie. `11` (integer) and `a` (unary message send).
   And that the editor/commit/etc. will complain about how ugly `1_1_` is.

   The current PR advocates for this behavior since it (i) is the simplest thing to implement in `NumberParser`; (ii) move the responsibility of asserting ugliness and formatting to something more concerned about that (Rules, auto-formater, etc.); and (iii) this reduces the nitpicking and subtle differences between the specification of other languages. 


# Out of scope

Printing methods of numbers (e.g. `asString`) could be "improved" by adding `_` in string representations of numbers to ease the reader.
However, this PhEP does not address this (by default) as it might bring a lot of compatibility issues:

* String representations that might be read by another parser that does not accepts `_`.
* String representations that expect a precise number of characters or a precise position of some digits.
