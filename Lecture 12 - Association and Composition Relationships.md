
Extra Resources: UML Distilled Chapter 3 and 4 
- Good extra resource provided on the CS246 course website
- will be helpful for the final project

```C++
List

theList::Node
ctr : Integer

+add(i : Integer) : void
-ith(i : Integer) : Integer
+size() : Integer
```

Generally, we don't add constructors, destructors, accessors and mutators since their presence is expected $\implies$ this is useful when a third person or the customer wants to implement a program using the List
- - means private
- + means public 

**Association**
- Simplest form of a relationship is "association"
- We can decorate the ends of association lines with multiplicity (constraints)
- The idea is that if we have a class model, and add specific multiplicity on the ends, we know which constraints should not be violated on the runtime and what is the relationship between the two. 

```C++
// & => 0 or more
// n ... m => between n and m instances
// n => exactly n instances
```

$List \rightarrow Node$
![[Screenshot 2024-02-15 at 12.18.14 PM.png]]
- The list knows about the Node
- Each List has either 0 or 1 Node
- The Node also has 0 or 1 self-referential Nodes
	-  Imagine an arrow on the Node pointing to itself 
- decorate an association end with a role name (replaces shown data fields in class)
- Note: some objects do not get stored/referenced/pointed to
- That is, they are just short-term relationships
![[Screenshot 2024-02-15 at 12.23.43 PM.png]]
- Comments are written like sticky notes, as shown above 


**Composition Relationships**
- Normally described as $\text{A owns B}$ or $\text{Class A is composed of Class B}$
- E.g., the basis and vec program, a Basis class is composed of two vec classes
- Relationship is shown as a solid diamond 
![[Screenshot 2024-02-15 at 12.29.34 PM.png]]

- A owns B, that forces a multiplicity of 1 on the solid diamond, we don't need to show that
- No sharing, we must destroy when destroyed
	- This means that we will always have to make a deep copy 

**Question: Can B exist independently of A?**
Answer: 
- Until B (Vec) is not assigned to A (Basis), it can exist independently, image all the other linear algebra that requires Vectors
- However, the moment it is assigned to Basis, it is bound to it, since the Basis must have 2 vectors
- I can have other Vec objects floating around since they are not Bound to the Basis
	- However, once you bind it, then it is as composite relationshi
- We could replace the Engine of a Car, though if a Car gets destroyed, so is the engine


**Aggregation Relationships**
- A "has-a" B sharing is allows
- Since sharing is allowed, shallow copying is good enough, You are probably not the own of this object, so not responsible for the cleanup
- It is showed using an unfilled or clear diamond
- For example, multiple Scrablle games that share a single english dictionary
- if a pond dries out, all ducks are not killed, they just change ponds 
![[Screenshot 2024-02-15 at 12.36.49 PM.png]]
- Given we have * (0 or more) ducks in the above example, we must place a physical limitation on what the upper limit is 
- A duck can be in a Pond object, or independent of it, in which case there is no Pond object that "has-a" particular duck object
- Note: static data fields and methods are underlined

Question: Consider
![[Screenshot 2024-02-15 at 12.38.09 PM.png]]
Answer: 
- Having a pointer (Node \*) next doesn't tell us if Node is composed of (owns) next
- We must look at the class and see if Node had a destructor, or Node must be made such that it recursively deletes all the owned nodes
- presence of a destructor points towards a composition relationship


Question: Consider the following...
![[Screenshot 2024-02-15 at 12.44.34 PM.png]]Answer:
- Tells us that the Node is the owner of all subsequent Nodes
- When we implement this Node, we must remember to add a Destructor object
- Recursive, destruction, starts with the List.

Question: What if the class model is:
![[Screenshot 2024-02-15 at 12.53.29 PM.png]]
Answer: 
- The list owns the Node object, but the Node has an aggregation relationship with next
- List is responsible for destroying
- Probably means that the List will loop to destroy all the Node objects

**Inheritance/Generalization/Specialization**
- Generalization works like
	- "I have an awful lot of classes that have things in common, I should make a common class"
- Specialization works like 
	- " I have a class that gets changed for an awful lot of situations, I should make special classes of this general class"
e.g., Math Student is a Student
- Shown by Unified Triangle Arrowhead, don't use V head unless you hate your marks
![[Screenshot 2024-02-15 at 1.17.13 PM.png]]
- Inheritance doesn't roll with multiplicities and roll names
- Here, student can be called the "superclass" or the "parent class" or "base class"
	- MathStudent and EngStudent will be called "subclasses" and children class or derived class respectively
	- We should not be duplicating the information we are inheriting, that will cuase problems and will be a very very not smart thing to do
	- Unless you want every class to have different implementatons of a function you inherited from the parent class
- Motivating Example, Catalogue of Book, Comic and text objects
![[Screenshot 2024-02-15 at 1.24.52 PM.png]]
- Note that isHeavy() is different for different objects and hence the function is overloaded and modified


```C++
class Book {
	std::string author, title;
	int numPages;

	public: 
		Book(std::string author, std::string title, int n) 
			: author{author}, title{title}, numPages{n}
			{}
		bool isHevavy() { return numPages > 200; }
		// ... 
}
```

```C++
class Comic : public Book{
	std::string hero;

	public: 
		Comic(std::string a, std::string t, int n, std::string h)
			: author{a}, title {t}, numPages{n}, hero{h} {}
		bool isHeavy() { return numPages > 30; }
		// ... 
}
```

- This is problematic, as private thingies of the parentClass are also private from childrenClasses
- Note: public inheritance doesn't allow children classes to see the private parent fields then private and protected inheritance are even more stringent 

- so either use accessors/mutators (but still have a problem in regards to parent constructor class, that is book constructor isn't a default constructor)
- Or we could make data fields/method protected (but that doesn't fix the problem either, also weakens encapsulation)

**Object creation steps are now.** 
1. Allocate space for both parent and child (If Book takes X amount of space, and Comic takes Y more bytes, allocate X+Y amount of space). 
2. Initialize parent. 
3. Run the child’s Member Initialization List.  
4. Run Child ctor body. 
- It turns out, running the child’s MIL gives us the opportunity to run the Parent’s ctor. Sweat. 

```C++
Comic::Comic(std::string a, std::string t, int n, std::string h) :
	Book{a, t, n},
	hero{h} 
	{}
```