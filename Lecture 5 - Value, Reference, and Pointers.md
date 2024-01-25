
**Returning By Value/Reference/Pointer**

Question: We want to modularize information creation
e.g., Node ??? n = getNode();

- Returning By Value
```C++
Node n = getNode(); // copies the result 
Node getNode() {
	node loc {13, nullptr};
	return loc;
}

// Copies loc onto run-time stack
// Then copies into n 
```

Alternative:
```C++
Node getNode() {return Node {13, nullptr}; } 
// 13 is a return value, anonymous object 

// C++20 : The compiler deduces the type to return 
Node getNode() {return {13, nullptr}; }
```
-> Copying the value is expensive 

- Return By Pointer 
```C++
Node *ptr = getNode();

// Creates a dangling pointer 
// NOT GOOD
Node *getNode() {
	Node *getNode() {
		Node loc {13, nullptr};
		return &loc;
	}
}
```

Better:
```C++
Node *getNode() {
	Node *np = new Node {13, nullptr};
	return np;
}

// ALTERNATIVELY 
Node *getNode() { return new Node{13, nullptr}; };
```

Returning By Reference:
```C++
Node *nref = getNode();
Node getNode() {

	// Binds the local variable that no longer exists 
	Node n{13, nullptr};
	return n;
}
```
- Ask what is the difference between reference pointer 

- Could do this instead:
```C++
Node &nref getNode() {
	return *(new Node{13, nullptr}); }

	// Recipient has to remember to eventually free it 
	delete &nref;
}
```

Question: But operator >> returns by reference?
Answer: Because the (istream&) was defined outside the function call, it still exists when the function ends, so it can be safely returned by reference 

Question: Which one do we use?
Answer: If it can be copied, return by value
	- Not as expensive as you may think
	- if it cannot be copied, return by reference 

**Operator Overloading**
- Search for overloading on cppreference, and it will provide a list of what can and/or cannot be overloaded in C++:
```C++
struct vector {int x, y; };

vector v1 {0, 1}, v2 {1, 0};
vector v3 = v1 + v2;

vector operator+(const vector &v1, const vector &v2) {
	vector v {v1.x + v2.x, v1.y + v2.y};
	return v;
}

// Scalar Multiplication 
vector operator*(const int &s, const vector &v) {
	return vector{v.x * s, v.y * s};
}

// now we do s * v
vector operator*(const vector &v, const int &s) {
	return (s * v); // calls the previous definition 
}
```

Question: Consider a Grade Struct, if we read in, how do we ensure that the value is in range from 0 - 100?
Answer: struct Grade { int value; };
```C++
istream& operator >> (istream& in, Grade& g) {
	in >> g.value;
	if (g.value < 0) { 
		g.value = 0;
	} else if (g.value > 100) { 
		g.value = 100;
	}
	return in;
}

ostream& operator << (ostream& out, const Grade& g) {
	out << g.value << '%';
	return out;
}
```

**Separate Compilation**
- Need to separate "interface" from "implementation"
	- Interface contains constant and function declarations plus the type definition
	- Put implementations in implementation files since we can declare them more than once but only define them once 
- Implementation files contain implementation code 

```C++
// Modules 
// file : vec.cc 
export module vec;
export struct Vec {
	int x, y;
};

export Vec operator*(const int& s, const Vec& v) {
	...
}
```

```C++
// include 
// file : vec_h

#ifdef VEC_H 
#define VEC_H

struct Vec { ... }; 
Vec operator*(...);
...
#endif
```

```C++ 
// file : vec-impl.cc 
module vec;

// implement functions 
```

```C++
// file : vec..
#include "vec.h"
// implement functions 
```

- Main file where we will use our modules 
```C++
// file : main.cc 
import vec;
import <iostream>
int main() {
	Vec v;
	...
}
```

```C++
// file : main.cc 
#include "vec.h"
#include <iostream>

int main() {
	...
}
```

- must be compiled in C++ 20 modules in dependency order 
	1. System Files 
	2. Interfaces, in dependency order 
	3. Implementation Files 
- g++20h iostream 
- g++20m -c vec.cc -> creates vec.o
- g++20m -c vec-impl.cc -> creates vec-impl.o 
- g++20m -c main.c -> creates main.o 
- g++20m vec.o vec-impl.o main.o -o main 
- Otherwise, after compiling system headers:
	- g++20m -c \*.cc 
	- g++20m -c \*.cc -> compiles previous failures 
- g++20i -c \*.cc
- g++20i \*.o