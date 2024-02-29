
Another more subtle problem occurs when we combine polymorphism and object arrays. 

```C++
Comic c[2] = {{"a1", "t1", 10, "h1"}, {"a2", "t2", 15, "h2"}};
f(c);
```

where:
```C++
void f(Book books[]) {
	books[1] = Book{"ba", "bt", 100};
}
```

**What does c[] now contain?**
```C++
{{"a1", "t1", 10, "ba"},
{"bt", ???}} // who knows what goes on here?
```
- objects are different sizes
- unexpectedly modifying that we don't expect in adjacent memory locations
- Information > or < than expected, you will get unexpected memory changes 
- might get strange values if the types were different 


**Abstract Classes**
In some scenarios, a base class provides our common core and interface, but isn't something that we want to be able to create.

e.g., 
```C++
class Student {
	...
	public: 
		double calcFees();
	...
};

class CoopStudent : public Student { 
	...
	public:
		double calcFees();
	...
};

class RegStudent : public Student {
	...
	public:
		double calcFees();
	...
};
```
- we don't know how to calculate tuition feeds for the base class Student, and shouldn't be able to instantiate it 
- done in C++ by adding at least 1 "pure virtual" method 

e.g., 
```C++
c;ass Student {
	...
	public:
		// pure virtual 
		virtual double calcFees() = 0; 
}
```
- a pure virtual method may, or may not have an implementation 
	- if a subclass doesn't implement the pure virtual method, it is also abstract 
		- can't be instantiated 
	- if it implements it, it is a concrete class and can be instantiated 

**UML Notation**
Italicize the class name of abstract base class (ABC) and pure virtual methods 

***Student***
\+ *calcFees():Double = 0*
$\leftarrow$ 
**Reg Student** 
\+ *calcFees():Double*

Q: How do we prevent a class from being used as a base class?
A: declare it to be "**final**";

e.g., **class Comic final : public Book {...}**


**Inheritance and the Big 5**

Destruction
1. run destructor body 
2. run parent constructor 
3. run destructors for object data fields 
4. deallocate space 

e.g., //lecture/19-inheritance/example5
```C++
class X {
	int *x;
	public:
		X(int x) : x{new int[n]} {}
		~x() { delete []x; }
}

class Y {
	int *y;
	public:
		Y(int m, int n) X{n}, y{ new int[m] } {}
		~Y() { delete[] y; }
}

int main() {
	X *myX = new Y{10, 100};
	delete myX; // memory leak since calls ~X instead of ~Y 
}
```

**Solution**: make the destructors virtual 
**Best practice**: always declare your destructor virtual! 

If need/want the class to be an Abstract Base Class (ABC) but all methods have reasonable (default) implementations, make the destructor pure virtual 
- still must be implemented!

e.g., Example of AbstractBook class 
```C++
class AbstractBook {
	std::string author, title;
	int numPages;

	public:
		AbstractBook(...);
		// accesors and mutators are reasonable
		// thus pick the pure virtual destructor 
		virtual ~AbstractBook() = 0;
}

// somewhere in implementation file...
AbstractBook::~AbstractBook() {}

```C++
class Book {
	...
	public:
		Book(const Book &other) :
			author{other.author}, title{other.title}, numPages{other.numPages} {}
	...
	
}

class Comic : public Book {
	...
	public:
		Comic(const Comic &other) :
			Book{other}, hero{other.hero} {}
	...
}
```
**More Constructors**
e.g., 
```C++
Book::Book(Book && other) :
	// forces string move constructor to be used 
	author{std::move(other.author)}, 
	title{std::move(other.title)},
	numPages{other.numPages} {}

Comic::Comic(Comic && other):
	Book{std::move(other)}, hero{std::move(other.hero)} {}
```
Note: if parent class provides own move/copy constructors but subclasses don't, then calls parent constructor and performs field-by-field copy and move if objects 

**Issues with Copy Assignment** also exist with move assignment, so we'll focus on copy assignment 

**Version 1** of copy assignment 
```C++
Book & Book::operator=(const Book& other) {
	if (this == &other) return *this;
	author = other.author;
	title = other.title;
	numPages = other.numPages;
	return *this;
}

Comic Comic::operator=(const Comic& other) {
	if (this == &other) return *this;
	Book::operator(other);
	hero = other.hero;
	return *this;
}

e.g., 
Book b1{...}, b2{...};
Comic c1{...}, c2{...};
Comic c3 = c1;
Comic c4{c2};

c3 = c2;
b1 = b2;

```




