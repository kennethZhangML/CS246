Consider the following code:
e.g., Example
```C++
Node *n = new Node {1, new
					Node {2, new}
					Node{3, nullptr}}};

// Copy constructor 
Node m = *n;
Node *p = new Node{*n};
p->next->data = -1;
```

REMINDER: Assignment 1 due this Friday, Feb 2, 2023 at 5PM~ 

Pointer is equivalent to an integer~ doesn't do anything special
	- does a straight copy 

- Created pointer $p$ to the node, initialized with dereferenced $n$ pointer
	- passing $n$ into the Node as a parameter 

- compiler provided copy constructor, performs a shallow copy 
- if you want "deep" copy, you need to write your own copy constructor

e.g., 
```C++
struct Node {
	...
	Node(const Node &n):
		data{no data},
		next{nullptr == n.next ?
				nullptr : new Node{*n.next}}
	{}
};
```
```C++
n.next = nullptr // -> do nothing
n.next != nullptr // new node{*n.next}
```

Question: What's wrong with this copy char declaration?
e.g, 
```C++
struct Node {
	...
	Node(Node n) { ... };
};
```
- if I'm passing by value, I', trying to write the copy constructor, i am using the definition i haven't completed, this is an infinite recursive declaration. 
- pass by value -> copy constructor 
	- infinite recursive definition 

There are 3 places a copy constructor is involved (if we ignore elision).
1. Declare an object and initialize it with an object of the same type
	2. e.g., Node n1{...}, n2{n1}, n3 = n1;
2. Pass by value 
3. Return by value 

Question of ownership
	- if you own it, make a deep copy 
	- write your own copy constructor 

Note: Single parameter constructor that take a parameter other than the class are used to perform type conversion/coercion 
e.g., 
```C++
struct Node {
	...
	Node(int v);
	...
};

Node n1{4};

// Silently converts 4 to a Node, 
// then uses the copy constructor 
Node n2 = 4; 

void f(Node n);
f(Node{4});
f(4); 
```

Main issue is that the compiler is performing this conversion silently, upon your behalf 
	- makes it hard to debug/track down errors since unexpected 
	- e.g., std::string s = "hello"; 
		- std::string has a constructor with a (const char \*)  parameter 
	- make constructor with explicit to force declaration of constructor use 
	- e.g., 
```C++
struct Node {
	...
	explicit Node(int d, Node *n = nullptr);
}

Node n1{...};
Node n2 = 4; // syntactically illegal 
Node n2 = Node{4};
f(4); // illegal as well
f(Node{4}); 
```

**Destructors**
- Compiler provided destructor calls destructor on data fields that are themselves objects 
- creation done in declaration order, destruction done in reverse order since it's on the stack 

**Destructor Steps:**
1. Run destructor body 
2. Call destructor on object data fields (reverse order)
3. Deallocate the object's space 

Consider previous linked list example:
e.g.,
```C++
Node *n = new Node{1, new NodeP{2, new Node{3}}};
Node m = *n;
Node *p = new Node{*m};
```
- if we don't delete $n$, this leaks 3 nodes;
- if delete $n$, frees first node, leaks the rest (2)
- $m$ is on the runtime stack, so it's deallocated when it goes out of scope 
	- leaks remaining nodes 
- $p$ is similar to $n$ 

- Add destructor to Node class 
- e.g., 
```C++
struct Node {
	...
	// if next is a nullptr, it does nothing 
	~Node(){ delete next; }
	...	
};
```
- this can't be overloaded 
- if nullptr recursively calls the destructor on the next node 

**Copy Assignment**
e.g., Copy assignment of Node 
```C++
Node n1{...}, n2{...};
...
n1 = n2; // copy assignment;, should be deep 
```
- compiler provided copy assignment operator calls copy assignment operator on all object data fields (copy assignment operator assignment)

// Version 1
So, can chain calls and doesn't copy what is returned.
```C++
Node & 

Node::operator = (const Node& other) {
	delete = other.data;
	if (other.next == nullptr) {
		next = nullptr
	} else {
		next = new Node(*other.next);
	}
	return *this;
}
```
if this->next wasn't a nullptr, then we just leaked memory!
- free next before we overwrite 
```C++
data = other.data;
delete next; 
next = other.next == nullptr ? 
		nullptr : new Node{* other.next};
		// deep copy via copy constructor!
return *this;
```
**Question**: what if we had "n = n;"?
Answer: Had already freed this->next when attempt deep copy 
	- segmentation fault! 

- Not very likely to write "n = n;", but if passing (Node \*) around "\*p = \*q" is more likely 
- need to add self-assignment check 
- compiler doesn't provide operator == (or operator !=)
- already have "this" (Node*)
- We can compare to $&other$ i.e. &(const Node&) -> (Node *)

// Version 3
```C++
if (this == &other) return *this;
data = other.data;
delete next;

next = other.next == nullptr ? nullptr : 
	new Node{*other.next};

return *this;
```
- if we run out of heap memory, already destroyed the original data
- only change original after safely made deep copy 
- e.g., if (this == &other) return \*this;
```C++
Node *tmp = next;
next = ...l; // deep copy 
data = other.data;
delete tmp;
return *this;
```