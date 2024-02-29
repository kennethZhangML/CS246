**From last lecture**: Class Book doesn't have a default constructor and its data fields are private. So how does Comic call the parent class constructor?  

1. Allocate space 
2. Initializer parent part 
	1. If we have a default constructor, this is where it would be run...
	2. but we don't.
3. Initialize (child) data fields
4. Run (child) constructor body 
5. Use step 2, leveraging MIL, to call the parent constructor 

**e.g., Example with Comic Child class**
```C++
Comic::Comic(std::string, a, std::string b, int n, std::string h):
	Book{a, t, n}, hero{h} {}
```
------------------------------------------------------------------------
**Question: Why not make inherited data fields protects so comic can initialize them directly?** 
**Answer: Book still doesn't have a default constructor ($\implies$ compile-time error)** 
$\implies$ what default values are reasonable?
$\implies$ lose ability to force subclasses to maintain class invariants 
$\implies$ potential duplication of work
	- now I have to redo all the tests that the Book class had to do to validate the data, we have to do something that makes it sensible.
	- would it be nice to delegate a section of the code for purposeful functionality? 
$\implies$ have to duplicate work done by the parent
$\implies$ better: have protected mutators that enforce invariants 

------------------------------------------------------------------------

Goal: Single container to hold Book/Comic/Text objects 
1. C-style tagged unions or union 
```C++
BookElem {
	Book *b, Comic *c, Text *t;
};

BookElem arr[10];
```
- have to keep track in a parallel data structure which field of the union was used 

```C++
arr[0].b = new Book{...};
arrType[0] = 0;
```
- error prone so not the most type safe 
- C-style (void *) array 
	- still need parallel data structure even less type safe than previous approach 

Let's pretend for a moment that our Book inheritance hierarchy has default constructors. 
- Can we make this work since a Comic (or Text) "is-a" Book?
```C++
Comic {"Author", "Title", 32, "hero"};
Book arr[10];
```

What happens if we write:
```C++
arr[0] = c;
// arr[0] is now a Book{"author", "title", 32};
// the hero part has been sliced off
// we have "object sliced" , i.e., it is now a book object
```
Remember, Book::isHeavy() returns true if Book::numPages > 200, and Comic::isHeavy() returns if Book::numPages > 30 e.g.,:
```C++
std::cout << boolalph << c.isHeavy();
// true

std::cout << arr[0] << isHeavy();
```
- outputs false since compiler statically binds the function call to be to 
```C++
Book::isHeavy
```
- can't have arrays of references 

Question: what about pointers?
A: Almost
```C++
Book *bptr = &c; // no object slicing 
```
```C++
Comic *cptr = &c;
std::cout << cptr->isHeavy();
// Statically bound to Comic::isHeavy true!
```
**Question**: What we want is a way to avoid object slicing and dynamically (at run-time) determine which method to call based upon what object the pointer (or reference) is actually pointing (bound) to
- make the method in the parent class virtual, e.g.,
```C++
class Book {
	... 
	public:
		virtual bool isHeavy() const;
	...
}

bool Book::isHeavy() const {
	return numPages > 200;
}
```
- UML notation is non-standard, i.e., in this course only, we initialize virtual methods or put "{abstract}" in front

```C++
Book
+ isHeavy():
	Boolean 
```
Note: Virtual methods of parents are inherited as virtual whether or not we explicitly declare them as virtual 
```C++
class Comic:public Book {
	...
	public: 
		bool isHeavy() const;
		// virtual since signature match
};
```
e.g. 
```C++
class Comic: ... {
	...
	public:
		virtual bool isHeavy() const;
}
```

**Best Practice: declare method in child class to "override" that of the parent.**

$\implies$ causes compiler to check that parent class has a virtual method with the exact same signature. 

```C++
class Comic: ... {
	... 
	public: 
		// last only in the interface
		virtual bool isHeavy() const override; 
	...
}
```
e.g., 
```C++
bool Comic::isHeavy() const {
	return getNumPages() > 30;
}

std::cout << bptr->isHeavy(); 
// outputs true since called 
Comic::isHeavy()
```
- determined method to call dynamically 
- i.e. at runtime

By having pointers + virtual methods (inheritance hierarchy) we've acquired polymorphism (many forms)
- uses an abstract type to hold multiple other types 

e.g., 
```C++
Book *arr[10];
...
arr[i] = new Comic { ... };
arr[i + 1] = new Text { ... };
arr[i + 2] = new Book { ... };

for (int i = 0; i < 10; ++i) {
	std::cout << boolalpha << arr[i]->isHeavy() std::endl;
}

void f(istream &in) f { ... }
ufstream infile{"test.txt"};
f(infile);
```
