Note: A nested class can see/access anything declared "static" in the outer class even if private 
- if has a pointer/reference /instance of the outer class, can access non-static field/ method even if private; nested class can see anything that is in the outer class as long as it is static (even if private)

**Design Patterns**
Historical aside: Idea comes from architects Christopher Alexander and his co-authors
- design patterns provide a common solution to a common problem 
- focus on class level solutions 
- let you program the interface not the implementation 
- should be able to adapt to re-use 

**Iterator Design Pattern**
- abstraction of a pointer "into" an abstract data type
- not full formal pattern since not using inheritance 
- will rely on "duck typing"
	- C++ does not care about the underlying type, so long as you provide the correct operations, C++ will work i.e. type not directly important

e.g., 
```C++
class List {
	struct Node {...};
	Node* theList = nullptr;

	public: 
		class Iterator {
			Node *p;
			public :
				explicit Iterator(Node *p) : p{p} {}
		}
}

bool Iterator::operator!=(const Iterator& other) const {
	return p != other.p; 
}

int& Iterator::operator*() {
	return p->data; 
}

Iterator& operator**() { 
	p = p->next;
	return *this;
}

Iterator begin() const {
	return Iterator { theList };
}

Iterator end() const {
	return Iterator { nullptr };
}

}; 
```


```C++
List mylist;

for (List::Iterator it = myList.begin(); it != myList.end(); ++it) {
	std::cout << *it << std::endl;
} 
```
- C++ provides a "range-based" for loop

e.g., 
```C++
// elem -> dereferenced iterator 
// myList -> iterating over 

// elem is a copy of the returned info 
for (auto elem : myList) {
	std::cout << elem << std::endl;
}
```
- if you want to be able to change elem:
e.g., 
```C++ 
for (auto &elem : myList) ++elem; 
```

**Question**: But client can create a List::Iterator directly and not use List::begin/end 
- how to fix this?
**Answer**: Make Iterator constructor private 

**Question**: but now List can't create Iterator objects either! What's the fix?
**Answer**: make List as a "friend" to the Iterator 
- friendship breaks encapsulation and use rarely and with caution 
- friend class declaration can be placed anywhere in the Iterator class definition 

```C++
class List {
	...

	public: 
		class Iterator {
			friend class List;
			Node *p;
			explicit Iterator(Node *p);

			public:
				// ...
		};
		// ...
};
```

- if don't use friendship, class could provide accessors (getters) and mutators (setters)
e.g., 
```C++
class Vec {
	int x, y;

	public :
		// ...
		int getX() const { return x; } // accessors 
		int getY() const { return y; } 
		void setX(int nx) { x = nx; }
		void setY(int ny) { y = ny; }
};
```
- if class shouldn't/doesn't have accessors? and mutators, but want to overload << and >>, could declare those specific functions to be friends

e.g., 
```C++
class Vec {
	// ... 

	friend ostream& operator<<(ostream& out, const Vec& v);
	friend istream& operator>>(istream& in, const Vec& v);
}
```

```C++
ostream& operator<<(ostream& out, const Vec& v) {
	out << '(' << v.x << ', ' << v.y << ')';
	return out;
}
```

**Equality Revisited**
- if we know the size of a list, may be able to determine if two lists aren't equal by comparing their sizes 
	- could have a size method that counts, but O(n)
	- could instead have a counter, incremented for every add, decremented for every remove. size() becomes (amortized) O(1)

e.g., 
```C++

class List {
	int ctr = 0;

	public :
		int size() const { return ctr; }
		bool operator!=(const List& o) const {
			if (size()! = o.size()) return true; 
			return (*this<=>o) != 0;
		}
}

auto operator<=>(const List& other) const {
	using std::strong_ordering; 
	
	if (!theList && !other.theList) return equal;
	if (!theList && other.theList) return less;
	if (theList && !other.theList) return greater;
	
	return (*theList <=> *other.theList);
	// <=> -> Node::operator<=>
} 
```

**Unified Modelling Language (UML)**
- we will use class models to depict classes and their relationships 
- we will only be using it informally so want to always adhere to the standard 
- minimum info is class name e.g., List 
- can add Data Fields 
	- e.g., List 
		- theList : Node 
		- ctr's Integer 
	- '+' public 
	- language adnostic so implementation choice as to type of Node, e.g, pointer/reference/object 