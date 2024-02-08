
- std::strong_ordering has 4 constant values: less, greater, equal, equivalent(last 2 are the same)
    - operands must be comparable; otherwise use std::partial ordering or std::weak_ordering
- Since std::strong_ordering is a lot to type, use type deduction:
	- - auto x = \<expr>;
	- //type of x is expr is
	- example: auto res = s1 < = > s2;

Q: what if we want to add < = > to Vec?
A: equality is just 
```C++
v1.x == v2.x && v1.y==v2.y
"less" v1.x < v2.x || (v1.x==v2.x && v1.y < v2.y)
```

```C++
struct Vec{ 
	auto operator <=>(const Vec& rhs){ auto n = x <=> rhs; 
	if(n!=0) return n; n = y <=> rhs.y; return n; } 
}

//Example: 
Vec v1{1,2}, v2{1,3}; auto res = v1 <=>v2; 
//remember <=> gives us: <, <=, >, >=, ==, != since can rewrite to compare result to 0 ...(v1 <=> v2) < 0
```

Note: compiler doesn’t provide ++ or ≠ for free
Compiler does have default < = > operator
→ compress fields lexicographically just as Vec::operator < = > does
Could instead write Vec and tell compiler to use the default < = >

Q: when is the default < = > behavior not good enough?
A: Consider:
```C++
struct Node{int data; Node *next};
```

```C++
struct Vec{ auto operator<=>(const Vec &v) const = default; };
```

Comparing data does the right thing, but there’s no point comparing heap addresses

Assertion: must be comparing 2 non-empty lists since comparing Node objects and there’s no equivalent to a null pointer

- We can do smth like if 2 list have the same length, compare data
- If 2 list has the same length and same data, ==
- If 2 list has different length, < or >

```C++
struct Node{ 
	auto operator <=>(const Node& other){ auto n = data <=> other.data; 
	if(n!=0) return n; 
	if(next == nullptr && other.next == nullptr){ return n } 
	if(next == nullptr) return std::strong_ordering::less; 
	if(other.next == nullptr) return std::strong_ordering::greater; 
	
	return next <=> other.next; //recursively calling <=> on 2 nodes } }
```

Question: When is the default <=> behaviour not good enough?
Answer: Consider:
```C++
Struct Node { int data; Node *next; };
```
Comparing data does the right thing but not comparing heap addresses 
- Assertion: must be comparing 2 non-empty lists since comparing Node objects and there's no equivalent to a null object 
- n1 <=> n2;

e.g.,
```C++
struct Node {
	auto operator<=>(const Node& other) {
		auto n = data<=>other.data;
		if (n != 0) return n;
		if (next == nullptr && other.next == nullptr) return n;
		if (next == nullptr) return std::strong_ordering::less;
	}
}
```

```C++
if (other.next == nullptr) {
	return std::strong_ordering::greater;
}
return next <=> other.next;
```

**Invariants and Encapsulation**
A class invariant is a statement that must be true for the lifetime of that class' instance. 

Consider:
```C++
struct Node {
	// ...
	~Node() { delete next; }
	// ...
};

Node n1{1, new Node{2, nullptr}};
Node n2{3, nullptr};
Node n3{4, &n2};
```
- crashes on n3 going out of scope since next had address of runtime stack allocated object 
- unstated Node class invariantis that either next is a nullptr or it's a heap address (valid)

e.g., Stack invariant: item popped is most recently pushed item 
We care about invariants since we can't reason about code if they don't hold 
- wrap classes in "capsules", black/opaque boxes so that client code can't access directly, only through the 

(public) interface/methods of the class
- may require the addition of a wrapper class 
- in C++, adds 3 more keywords 
	- private: anything  outside of the class can't access whatever follows
	- public: anybody can access whatever follows
	- protected: will talk about when we get to inheritance 
		- can be repeated multiple times in a class definition

e.g., 
```C++
struct List {
	private: 
		struct Node; // forward definition
		Node* theList = nullptr;

	public:
		~List() { delete theList; }
		void add(int i); // adds i to the beginning of the list
		int ith(int i) const;

	private: 
		
}
```

```C++
// Nested Class -> can access anything that is static 
// if has a pointer/reference object of type List, we can access anything 
// including private info 
struct Node {
	int data;
	Node *next;
}
```
- by default, everything in a struct is public so could forget to make things private 
	- use class keyword instead, where default visibility/accessibility is private 
```C++
export module list;

export class List {
	struct Node;
	Node& theList = nullptr;

	public:
		~List();
		void add(int i);
		int ith(int ith) const;
}
```

```C++
module list;
struct List::Node {
	int data;
	List::Node *next;
		~Node() { delete next; }
};

List::~List() { delete theList; }
```

```C++
void List::add(int i) {
	theList = new Node{i, theList};
}

int List::ith(int i) const {
	Node *ptr = theList;
	for (int ctr = 0; ctr < i; ++ctr) {
		
	}
	return ptr->data;
}
```

```C++
import list;
import <iostream>;

int main() {
	List myList;
	myList.add(1);
	myList.add(3);

	for (int i = 0; i < size; ++i) {
		cout << myList.ith(i) << ' ';
	}
	cout << endl;
}
```

But now list traversal is O(n^2), not O(n)!
- motivation for iterators (next class!)