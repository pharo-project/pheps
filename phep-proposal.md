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
The process of contributing with Pharo is simple, but sometimes it is hard to get a right idea of what is needed to get a contribution done, and what is expected about the shape and content of the contribution. Moreover, without clear guidelines it can happen that what is "good" for a reviewer is "bad" for another.  
The purpose of this PhEP is to list some guidelines contributors can follow to get their contributions accepted.  
Notice that this PhEP applies to regular contributions (not the ones that are described in the PhEP-0001), even if many of their principles apply to feature ("big") contributions too.  

# Description 
We do not want to integrate contributions that are not "done". But what done is is very subjective and changes from a person to another.  
We must have a shared understanding of what "done" means for work to be completed and to ensure transparency.  

## Definition of Done
There is a small list of requirements we have put in white and black for what we will consider "done":  

1. Tested: the added functionality or the provided fix includes tests (common case and corner cases)
2. It does not break other existing tests
3. If is more than a bugfix or is a complex fix, includes some documentation:
	- example method on the class when applicable
	- class comment. Useful (what is it? why?)
	- package comment. Description of the design of the tool
	- method comment for informative comment, explanation of intent, clarification or warning. Do not use a comment when you can use a method or a variable to reveal the intent! You can read https://www.baeldung.com/cs/clean-code-comments for more information.
4. code quality (See anex *What we consider quality code*).
5. If this is a big project: installation instruction, CI

## PR acceptance
Once the contribution is made, it needs to reach the Pharo image through a pull request made against the current development branch.  
The parameters for PR acceptance will be: 

1. All criteria from "Definition of done" are met.
2. There is an explanation of intention: a comment about the intention of the change in the PR or in the code.
3. Sometimes this is ongoing work (a change that will be used in upcoming changes). In that case:
	- what is the next step?
	- link to an issue (or PhEP) describing the higher objective / goal
4. does not break release tests
5. respect modularity of the system (i.e. it does not introduces new dependencies on other parts of the system if not required).

# Anexes

## Annex 1: What we consider quality code
This is a complex matter since appreciation of what is good code contains always a certain level of subjectivity. Nevertheless, we try to follow certain guidelines (understanding that everything is subject to reviewer appreciation of the problem).  
This is our current quality review check list:  
- Am I able to understand the code easily (names reveal the intention, comments should explain *what is not obvious*)?
- Is the code written following the coding standards/guidelines?
	- Keep the design simple.
	- Follow Object Oriented Design
	    1. S - Single-responsiblity Principle : A class should have one and only one reason to change, meaning that a class should have only one job.
	    2. O - Open-closed Principle: Objects or entities should be open for extension but closed for modification.
	    3. L - Liskov Substitution Principle: Every subclass or derived class should be substitutable for their base or parent class.
	    4. I - Interface Segregation Principle: A client should never be forced to implement an interface that it doesn’t use, or clients shouldn’t be forced to depend on methods they do not use.
	    5. D - Dependency Inversion Principle: Entities must depend on abstractions, not on concretions. It states that the high-level module must not depend on the low-level module, but they should depend on abstractions.
	- Be consistent (naming, order, formatting)
	- Avoid references to globals
	- No magic numbers (use constants, configuration values)
	- Avoid multiple if blocks
	- Method should have at most 3 arguments (unless there are good reasons, like in FFI calls) 
	- Each method should work at a single level of abstraction
- Is the same code duplicated unnecesarily?
- Is this method too big (more than 10 lines)? If yes, is the method having too many responsibilities?
- Can I unit test/debug the code easily to find the root cause?
- Is this code documented when needed? Does it include examples of usage and unit tests?
