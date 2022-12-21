```
PhEP {
        #id: 0,
        #type: #usability,
        #title: 'Introduce a non misleading not equals selector',
        #authors: [ 'Christophe Demarey' ],
        #created: '2022-12-21'
}
```

# Abstract
A proposal to introduce a non misleading not equals selector in the default Pharo image.

# Motivation
In Smalltalk 80, `~=` was chosen as selector to answer whether the receiver and argument do not represent the same object.
`~=` is misleading since `~` character is used in mathematics to describe proportionality or approximation (see https://en.wikipedia.org/wiki/Approximation).
I propose to introduce `!=`, in addition to `~=`, and use it everywhere in Pharo standard image. This selector is already used in well known languages for this purpose and is more meaningful, especially for people learning Smalltalk.
For coherence, I propose to introduce `!==` as an alias for `~~` (not identical to) and use it in the standard Pharo image.
It will ease a lot Pharo learning as well as improving code readability.

# Description
The implementation could be divided into different steps:
1. Introduce `!=` and `!==` selectors calling `!=` and `!==` respectively
2. Make `!=` and `!==` as fast as `~=` and `~~`. Indeed, `~=` and `~~` are part of the special selectors array that is used by the bytecode generator of Opal Compiler. These specials selectors have a dedicated bytecode in Sista v1 encoder (see `EncoderForSistaV1>>#genSendSpecial:numArgs:`).
3. Detect packages that are meant to run on quite old Pharo versions (all the Zinc packages, Fuel by example)
4. Update all uses of `!=` and `!==` in the Pharo standard image but the packages detected in 3. to use `!=` and `!==`

Note: `~=` and `~~` will be kept into Pharo image for Smalltalk and backward compatibility but will be discouraged to be used.
