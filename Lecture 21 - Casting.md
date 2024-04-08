`std::dynamic_cast` 
- only works for classes that have at least 1 virtual method 

Type of Cast: 
Reference -> Raises std::bad_cast
pointer -> nullptr

```C++
Book *b = new Text {...};
Text *tp = dynamic_cast<Text*>(b);
if (tp != nullptr) tp->getTopic();

Comic c{...};
Book &b = c;

try {
	Text& ref = dynamic_cast<Text&> (b);
	tref.getTopic();
} catch (std::bad_cast) {
	// ... 
}
```

Question: Does casting work with smart pointers?
Answer: only works with `std::shared_ptr` but there is a way to do all 4 cast types

e.g., 
```C++
std::reinterpret_pointer_cast
std::const_pointer_cast
std::static_pointer_cast
std::dynamic_pointer_cast
```

Question: can we use std::dynamic_cast to fix our move and copy assignment operator in an inheritance hierarchy? 
Answer: yes - we'll only look at copy assignment, but same approach works for assignment we should alway check Book!

```C++
Text& Text::operator=(const Book& other) {
	Text& tref = dynamic_cast<Text&>(other);
	if (this == &tref) { return *this; }

	Book::operator=(other);
	topic = tref.topic;
	return *this;
}
```

Question: Is use of `std::dynamic_cast` good style?
Answer: It depends, it's fine for concrete subclass, big 5, as previously seen since adding new classes doesn't cause existing code to change. Consider it's use for runtime

Type Information (RIIII)
e.g., 
```C++
void whatIs(shared_ptr<Book> b) {
	if (dynamic_pointer_cast<Comic>(b)) { cout << "Comic"; }
	else if (dynamic_pointer_cast<Text>(b)) { cout << "Text"; }
	else if (b) { cout << "Book"; }
	else { cout << "Nothing"; }
}
```
=> code in whatIs() has to change as add subclass (all casts of this form in fact)
=> time consuming error-prone
- could instead have the hierarchy provide a virtual "identity" method

e.g., 
```C++
class Book { 
	// ...
	virtual void identify() { cout << "Book"; }
	// ... 
}

void whatIs(Book *b) {
	if (b) b->identify();
	else cout << "Nothing";
}
```
=> create uniform interface across the hierarchy 

Question: When does inheritance make sense in your design?
Answer: if 
	1. potentially unlimited number of subclasses and,
	2. each has the same interface 
This tells us if have only a few classes that never change and interfaces aren't same maybe inheritance isn't the best approach.

```C++
class Turtle {
	public:
		void stealShell();
};

class Bullet {
	public:
		void deflect();
};
```
- classes are hardly ever added to and if they are then I expect to have to do the updates 
- C++20 introduces `std::variant` where we `import/include <variant>`
- Instead of using "typedef";
	- using `Enemy = std::variant<Turtle, Bullet>;`
	- strongly type-safe 

```C++
Enemy e1{Turtle{}}, e2{Bullet{}}, e3; // e3 defaults to Turtle{}

if (std::holds_alternative<Turtle>(e2)) {
	cout << "Turtle";
} else {
	cout << "Bullet";
}

try {
	Turtle = std::get<Turtle>(e1);
	// ...
} catch (std::bad_variant_access&) {
	cerr << ... ;
}
```

Note: that the declaration of e3 requires the first type in the variant to have a default constructor else compile-time error:

Solution: Pick 1
1. Ensure the first type has a default constructor
2. specify the constructor call values
3. use `std::monostate` as the first type

e.g., 
```C++
variant<monostate, T> // T or nothing
```
- a similar idea is `<optional>`'s `std::optional<T>`;
- often used for functions that either fail or return something 

**Virtual Method Implementation**
Consider the following 2 classes:
```C++
struct Vec1 { 
	int x, y;
	void f();
};

struct Vec2 {
	int x, y;
	virtual void g();
};

Vec1 v1;
Vec2 v2;

cout << sizeof(int) << ' ' // 7
	<< sizeof(v1) << ' ' // 8
	<< sizeof(v2) << ' ' // 16; 4->x, 4->y, 8->"vptr"

v1.f(); // f's location defined at compile time 
v2.g(); // acces v2 is vptr to get the vtable look up 
```





