**From last lecture:**
```cpp
class Book {
	...
	public:
		Book& operator=(const Book& other);
		...
};

class Comic : public Book {
	...
	public:
		Comic & operator=(const Comic & other);
		...
};

Book b1{...},b2{..};
Comic c1{..},c2{..};
Text t1{..},t2{..};

b1 = c1; //legal; c1 is object-sliced
c2 = b2; // **won't compile since Book is not a Comic**
c2 = t2; // **a Text is not a Comic; doesn't compile**

Book *p1 = &c1;
Book *p2 = &b2;
*p1 = *p2; // **calls Book::operator= due to static binding
					// => performs *partial assignment* since Book doesn't have a "hero" data field**
```

If want `**Comic**::operator=` to be called, need to make the `**Book**::operator=` `virtual`and `override` it in child class

```cpp
class Book {
	...
	public:
		virtual Book & operator=(const Book & other);
		...
};

class Comic : public Book {
	...
	public:
		virtual Comic & operator=(const Book & other) override;
	...
};

*p1 = *p2 // **now calls Comic::operator=
					// => still partial assignment!**

b1 = c1; // **legal, object is sliced**

t1 = c1; // now legal; "***mixed assignment***"
// **=>** will require exceptions and dynamic casting to fix so that we can **deep copy**
// **polymorphically**
```

The solution we’ll see next **prevents mixed** and **partial assignment**, but **lose polymorphic deep copy.**

⇒ introduce abstract base class, `AbstractBook`, that `Book`, `Text`, and `Comic` inherit from.

⇒ make `AbstractBook`’s copy assignment `protected` and not `virtual`

eg)

```cpp
class AbstractBook {
	string title, author;
	int numPages;
	protected:
		AbstractBook & operator=(const AbstractBook & other);
	public:
		...
		virtual ~AbstractBook() = 0;
};

virtual AbstractBook::~AbstractBook() {}

class Book : public AbstractBook {
	...
	public:
		Book & operator=(const Book & other){
			if (this == &other) return *this;
			AbstractBook::operator=(other);
			return *this;
		}
};

assume Text and Comic are defined here as well, similarly...
```

```cpp
Book b{...};
Comic c{...};
Text t{...};
Abstract Book *p1 = &b, *p2 = &c;
b = c; // **doesn't compile**
*p1 = *p2; // protected, doesn't compile
```

## Templates

Consider our previous `List` class

> What if we want to have `List` of char? string? etc.

⇒ would need a new class where type of data field in `Node` changes

An **alternate** approach is to use a _**template class**_ using `template`

eg)

```cpp
**template**<typename T> class Stack{ 
		T arr[10];
		int size = 0;
	public:
		void push(T data);
		T pop();
		bool empty();		
};

**OR

template**<class T> class ...

```

Since the full class definition must be known by the time we instantiate the template class, must put implementation in header file.

eg)

```cpp
Stack<int> s1; s1.push(3);
Stack<std::string> s2; string s3 = "hello" ;  s2.push(s3);
Stack<Stack<int>> s4;
s4.push(s1);
```

eg)

```cpp
template<class T> class List {
	struct Node {
		T data;
		Node * next;
	};
	
	Node* theList = nullptr;
	
	public:
		~List() { delete theList; }
		void add(T data){
			theList = new Node {data, theList};
		}
		T ith(int index){ 
			for(int ctr = 0; Node * ptr = theList;
					ctr <= index && ptr != nullptr; ++ctr, ptr=ptr->next){
						if (ctr == index) return ptr->data;
					}
	// The only change in **iterator** code is **return type** of **operator***
};

List<int> myList1; myList1.add(5);
List<List<int>> myList2; myList2.add(myList1);

for (auto elem : myList1) cout << elem << endl; //using iterator(assumed its defined)

List<int>::Iterator it = myList.begin();
```

## STL containers

- Standard Template Libraries (STL) has a variety of containers
    
- We’re going to start with `std::vector`
    
    ⇒ `import/include <vector>`
    

Note: if **add/remove** from vector, any existing iterators are invalidated.

⇒ default vector ctor creates an “empty” vector

eg)

```cpp
std::vector<int> v1; // v1.size() == 0

std::vector<int> v2(5,0); // v2 = {0,0,0,0,0}

vector<int> v3{4,5}; // v3 = {4,5}

v3.push_back(6); // v3 = {4,5,6}

cout << v3.pop_front();

for (auto i : v3) cout << i << endl; // using vector iterator

//assume **Vec** without default ctor
vector<Vec> vecs;
vecs.emplace_back(1,2); // Vec ctor call with parameters (1,2) **<- HINT A4 Q1!!!**
```

```cpp
vector<int>::reverse_iterator rit = v3.rbegin();
for (; rit != v3.rend() ; ++rit) {...}

if(v3[0] == 2) ...

```

`std::vector::erase()` takes an iterator

Consider a vector `v` that contains `{3,6,5,5,-1,5}` .

→ We’d like to **remove all 5’**s from `v`

```cpp
for( auto it = v.begin(); it != v.end(); ++it){
	if(*it == 5) v.erase(it);
} //wrong; misses the second 5.
```

```cpp
for (auto it = v.begin(); it != v.end(); ){
	if( *it == 5 ) it = v.erase(it);
	else ++it;
} // correct
```