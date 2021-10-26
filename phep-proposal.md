```
PhEP {
	#id: 0,
	#type: #process,
	#title: 'Pharo Contribution Guidelines',
	#authors: [ 'Esteban Lorenzano' ],
	#created: '2021-10-25'
}
```

# Abstract
This PhEP describes guidelines to follow to contribute changes to the Pharo environment and its ecosystem. 

# Motivation
The process of contributing with Pharo is simple, but sometimes is hard to get a right idea of what is needed to get a contribution done, and what is expected about the shape and content of the contribution. Moreover, not existing clear guidelines, some times it can happen that what is "good" for a reviewer is "bad" for another.  
The purpose of this PhEP is to list some guidelines contributors can follow to get their contributions acepted.  
Notice that this PhEP is destined to regular contributions (not the ones that are described in the PhEP-0001), even if many of their principles apply to feature ("big") contributions too.  

# Description 
We want to contribute to Pharo things we consider "done". But what done is is very subjective and changes depending on our humors and needs. 
We need a concrete definition of it, to stablish a common language about it.

## Definition of Done
There is a small list of requirements we have put in white and black for what we will consider "done":  

1. The functionality we are adding or fixing includes tests
2. It does not break other existing tests
3. If is more than a bugfix or is a complex fix, includes some documentation:
	- example method on the class when applicable
	- class comment. Useful (what is it? why?)
	- package comment. Description of the design of the tool
	- method comment for non-obvious methods
4. If this is a big project: installation instruction, CI

## PR acceptance
Once the contribution is made, it needs to reach the Pharo image through a pull request made against the development branch.  
The parameters for PR acceptance will be: 

1. All criteria from "Definition of done" is accomplished.
2. There is an explanation of intention: a comment about the intention of the change in the PR or in the code.
3. Sometimes this is ongoing work (a change that will be used in upcoming changes). In that case.
	- what is the next step?
	- link to an issue (or PhEP) describing the higher objective / goal
4. does not break release tests
5. code quality (if, cyclomatic complexity, etc.). 
6. respect modularity of the system

