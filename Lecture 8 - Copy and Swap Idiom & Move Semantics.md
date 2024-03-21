- an "Idiom" is a program language level solution to a common problem
- "copy and swap" often used for copy assignment 

```C++
Node& Node::operator=(const Node& other) {

	// assume <utility> imported/included 

	Node tmp{other}; // copy constructor 
	swap(tmp); // tmp has the original info 
				// this has the "copy"
	return *this;
} // tmp is local, goes out of scope 
// destructor is called on tmp

void Node::swap(Node & other) {
	std::swap(other.data, data);
	std::swap(other.next, next); 
}
```

Question: Do we need a self-assignment check?
Answer: No, but the worst case is an unnecessary deep copy versus, cost of many tests 

Question: Can the local object (tmp) be removed?
Answer: Yes, pass "other" by value to make the parameter contain the deep copy 
Instead => replace "tmp" with "other"

**Move Semantics**
- "lvalues" have names/addresses/are pointed to
- "lvalues" reference is like a constant pointer (to an lvalue) that is automatically dereferenced 
- "rvalue" is not an "lvalue" (usually a temporary)
- "rvalue" reference is a reference bound to an "rvalue"

```C++
//lecture/10-rvalue/node.cc

Node oddsOrEvens() {
	// compiler can't optimize return value to perform elision 
	Node odds{1, new Node{3, new Node {5, nullptr}}};
	Node evens{2, new Node{4, new Node {6. nullptr}}};
	char e;
	cin >> e;

	if (e == '0') return evens;
	return odds;
}

int main() {
	// = is a copy constructor
	// basic constructor calls create 2x3 nodes 

	// copy constructor to copy returned Node on runtime stack 
	// copy constructor for each Node 
	// Copy constructo into $n$ 
	
	Node n = oddsOrEvens();	
}
```

- recognize that oddsOrEvens returns an "rvalue"
	- want to just copy the next fields address into n.next rather than do a deep copy since rvalue is about to be destroyed anyways 
	- We need to break shared memory addresses, so set other.next = nullptr 

```C++
Node(Node&& other)
	: data{other.data}, next{other.next}
	{ other.next = nullptr; }
```

- if no move constructor, but has copy constructor, use copy constructor 

Question: What about:
```C++
n = oddsOverEvens();
```
?

- define move assignment operator :
```C++
Node& operator = (Node&& other) {
	// other contains original data from "this" that now has other's info
	
	swap(other); // data for data, next for next 
	return *this;
}
```

Question: Should more assignment have self-assignment check?
Answer: What if we had "n = std::move(n);"?
	- makes $n$ be treated as a native 
	- if the object was complex and swaps could fail, probably a good idea 
	- if swaps fail (exception safety) and not complex, we could omit this 

- if we have both move and copy operators/constructors defined, the compiler uses move for rvalues 

**Copy/Move Elision**
- see lectures/11-elision/vec.cc
e.g., 
```C++
Vec makeVec() { return{0, 0}; } // basic constructor 
...
// compiler writes {0, 0} directly into n's bytes 
Vec n = makeVec(); 
```
- compiler optimizes away move or copy constructor calls if it can,
	- even if resulting control flow differs!

- can play around with code 
	1. make copy (vec2.cc) 
	2. change import <...> to \#include <>
	3. compile with :
		1. g++ -std=c++14 -fno-elide-constructors  vec2.cc

e.g., 
```C++
void doSomething(Vec v);
doSomething(makeVec()); 
// still performs elision on parameter 
```

**Rule of 5/Big 5** 
- if writing any one of :
	1. Destructor 
	2. copy constructor
	3. move constructor
	4. copy assignment
	5. move assignment 

Should consider whether or not all 5 need to be written. 
- criteria is that of "ownership" of resourcers
- e.g., dynamic memory, files, etc 
- should provide if need deep copy 

**Member Operations**
- some, such as assignment, **must** be part of the class 
- some don't have to be 
- e.g., Vec operator+(const Vec& v1), const Vec& v2);

```C++
struct Vec {
	...
	Vec operator+(const Vec& rhs);
};
```

- Vec operator*(const Vec& v, int s);
	- could be made method of class 
```C++
struct Vec{
	...
	Vec operator*(int s);
	...
};
```

Question: Vec operator*(int s, const Vec& v)? 
Answer: no. 

Note: If writing **operator+**, should write **operator+=** 
```C++
Vec Vec::operator+(const Vec& rhs) {
	Vec v;
	v.x = x + rhs.x;
	v.y = y + rhs.y;
	return y;
}

Vec& Vec::operator+=(const Vec& rhs) {
	Vec v{rhs}; 
}
```