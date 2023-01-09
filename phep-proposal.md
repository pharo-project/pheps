```
PhEP {
	#id: 000x,
	#type: #image,
	#title: 'New Finalization',
	#authors: [ 'Guillermo Polito' ],
	#created: '2022-01-09'
}
```

# Abstract 

A proposal describing a new finalization mechanism for Pharo with the following properties:

 - API independent from its implementation
 - Avoids memory leaks

This document describes both a finalization API as a Pharo library and an efficient virtual machine implementation through Ephemerons.

# Motivation

Some applications require tracking when objects are *about to* be garbage collected to perform some treatment. For example, file descriptors and sockets must be closed when garbage-collected.
Such a mechanism is implemented through automatic finalization.

The finalization mechanism present in Pharo up to version 10 (inclusive) is suboptimal.
First, it requires a full scan of many data structures to detect finalization candidates.
Second, the bad usage of the finalization mechanism leads to memory leaks.
Third, the current API is tied to its implementation (`WeakRegistry`).

# Description

## Terminology

- **Finalizable**: an object of interest regarding garbage collection. When this object is about to be garbage-collected we want to notify a **finalizer**.
- **Finalizer**: an object that gets notified when a **finalizable** is about to be garbage-collected to perform some action.
- **Finalization Registry**: a registry of *(finalizable, finalizer)* pairs containing finalizable objects and their corresponding finalizer.

## Finalization Overview and Protocol

Finalization is managed at the user level by a finalization registry.
The finalization registry contains (finalizable, finalizer) pairs.
The registry has two main methods: `#add:` and `#add:finalizer:`.
Finalizers need to implement `#finalize` with their corresponding finalization

- `FinalizationRegistry>>#add:` adds an object as both finalizable and finalizer. This means the object is its own finalizer.
- `FinalizationRegistry>>#add:finalizer:` adds an object as finalizable with another object as finalizer.
- `[User Object]>>#finalize` invokes the finalization mechanism of the given finalizer. The default `#finalize` method in `Object` is defined to do nothing. 

### Examples

The following example shows how to set a pair (finalizable, finalizer).

```smalltalk
"Create a finalizable object implementing #finalize"
MyFinalizer << Object
	package: 'MyPackage'.
MyFinalizer >> #finalize [
	'Gone!' traceCr.
]

"Add it to a registry"
objectOfInterest := Object new.
registry := FinalizationRegistry new.

"Object is different from the finalizer"
registry add: objectOfInterest finalizer: MyFinalizer new.

"Make the object collectable and see the transcript"
objectOfInterest := nil.
"Transcript:
  Gone!"
```


The following example shows how an object can be its own finalizer.

```smalltalk
"Create a finalizable object implementing #finalize"
MyObject << Object
	package: 'MyPackage'.
MyObject >> #finalize [
	'Gone!' traceCr.
]

"Add it to a registry"
objectOfInterest := MyObject new.
registry := FinalizationRegistry new.

"Object is its own finalizer"
registry add: objectOfInterest.

"Make the object collectable and see the transcript"
objectOfInterest := nil.
"Transcript:
  Gone!"
```

### Strong and Weak references in the Registry

Two things must be noticed from the finalization registry: first, it does not prevent the garbage collection of its finalizables; and second, the registry must be kept strongly for finalization to work.

As shown in the examples above, the finalization registry holds a reference to the finalizable object.
However, this does not prevent the GC to identify such objects as collectable.
In practice, a registry's finalizables are referenced weakly although, instead of putting a tombstone for the reference, the corresponding finalizer is invoked.

Moreover, it's the user's responsibility to hold their own finalization registry strongly by its application.
Otherwise, if a finalization registry is collected, the relation between (finalizable, finalizer) pairs is lost, and finalization cannot take place.
The proposed implementation provides a globally visible registry as a singleton

```smalltalk
FinalizationRegistry class >> default [

	^ Default ifNil: [ Default := self new ]
]
```

## Ephemerons

The proposed implementation makes use of ephemerons.
This section briefly explains what are ephemerons and how they are used for finalization.
For more details, check out the paper.

> Barry Hayes. 1997. Ephemerons: a new finalization mechanism. SIGPLAN Not. 32, 10 (Oct. 1997), 176–183. https://doi.org/10.1145/263700.263733

Ephemerons are key-value pairs that are treated specially by the garbage collector, where

 - the key is the finalizable object
 - the value is any other object whose life-cycle is associated to the key

The garbage collector detects when an ephemeron key is a candidate for collection, in which case it puts the ephemerons in a queue for later finalization.
The ephemeron implementation guarantees that strong references from the ephemeron's value do not prevent the key's collection, avoiding memory leaks.

The finalization library consumes the objects from the queue of ephemerons and sends the message *#mourn* to each of them. `#mourn` will remove the subscription from the corresponding finalization registry and send the message `#finalize` to its value -- the finalizer.

## Implementation Details

The current proposal is accompanied by an implementation in form of pull requests.
This section presents the proposal and describes some implementation details.

### Implementation Proposal

- **Library Pharo PR:** The following PR introduces in Pharo 11 the API described above.

[https://github.com/pharo-project/pharo/pull/12042/files](https://github.com/pharo-project/pharo/pull/12042/files)

- **VM support:** The support for Ephemerons was started a long time ago by Eliot Miranda et al. A great effort has been done in the previous months to improve its robustness. The latest version is already available in branch `pharo-10` of https://github.com/pharo-project/pharo-vm, with latest integrated PR being [https://github.com/pharo-project/pharo-vm/pull/511](https://github.com/pharo-project/pharo-vm/pull/511)

### Threading and Error Management

The library side of finalization will consume ephemerons from the queue using a separate green thread. This process will consume one by one the ephemerons and send them the `#mourn` message.
If an error happens while finalizing an object, a new process will be open consuming the rest of the ephemerons.

### Other Concerns
It is possible, although not recommended, to *revive* a finalizable object during finalization.
When a finalizer is being notified, it means its corresponding finalizable is a candidate for collection.
In such case, it is possible from the `#finalize` method to take the finalizable object and subscribe it some a global reference.


### Removals from Pharo >= 10 and Backwards Compatibility

We propose:

 - **Backwards compatibility:** to deprecate `WeakRegistry` and replace its usages with `FinalizationRegistry`. This will keep backwards compatibility for users of finalization.
 - **Cleanups:** To replace the internals of `WeakRegistry` and use a `FinalizationRegistry` instead. This will allow to remove the old finalization implementation without affecting its users.

 
