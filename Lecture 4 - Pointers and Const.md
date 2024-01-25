```C++
const int *p1 = ... ;
int * const p2 = ... ;
const int * const p3 = ... ;
const Node n2 {15, nullptr};
```
- Use const liberally, so we can let the computer help you catch mistakes

**Parameter Passing**
The left value (Lvalue) is something that can appear on the LHS of an assignment statement:
- can be accessed by name, by dereferencing the pointer to it, or through a reference
```C++
int i = 5;
int *ptr = &i;

int j = 6;
*ptr = 13;
i = j;
```

- A "value" is not a Lvalue, often, but not always temporary
```C++
i = 3 + 5;
```

Consider the following increment function
e.g, example3-function/ incrementBroken.cc
```C++

void inc(int n) {
	++n;
}

int i = 5;
inc(i);
std::cout << i << std::endl;
```
- inc() only changed the copy, then discarded it when the function execution ended 
- fixed via passing an address (pass by pointer) 
- works both for C and C++ 

Fixed increment function:
```C++ 
void inc(int *n) { // pointer passed by value
				   // change contents of dereferenced address
	++(*n);
}

int i = 5;
inc(&i);
std::cout << i << std::endl;
```

Question: Why did "cin >> i" successfully read into i without taking its address?
Answer: Because C++ passed them both by reference
```C++
std::istreamf operator >>(std::istream &in, int *i);
```

**References**
- A reference is a conceptually equivalent to a constant pointer that is automatically dereferenced
``` C++
int i = 5; // stored in memory i[5]
int &z = i; // z is equivalent to i
z = -1; // i[5] -> i[-1]
```

- if '&' is a part of the type for variable/parameter, it's a reference; otherwise, either taking the address of something or bit-wise

// Example3-Functions/incrementReference
```C++ 
void inc(int &n) {
	++n;
}

int i = 5;
inc(i);
std::cout << i << std::endl; // 6
```

**Things you cannot do with References**
1. Can't be left uninitialized 
```C++ 
int &g; // No
```

2. Reference to a reference i.e. &&
	1. Means something else, discussed with cover more semenatics
3. A pointer to a reference
```C++ 
int & *g; // Not legal
```
- but a reference to a pointer is legal
```C++ 
int * & g = ... // ok!
```
4. Array of references
```C++
int *arr[0];
```

- standard library containers (see later) can't contain reference either 

Question: Which of these 3 methods should be used for passing into a function parameter?
Answer: It depends on what you want and are willing to pay.

Consider a struct with a lot of information (a lot of fields)
```C++ 
struct Reallt Big{...}; 
void f1(ReallyBig rb){...} // by value -> making a copy 
void f2(ReallyBig *rb){...} // by pointer -> copy it's address, rb is mutable 
void f3(ReallyBig &rb){...} // reference (const address) copied; rb is mutable 
void f4(const ReallyBig &rb){...} // Reference copied, rb is immutable
```
- we prefer reference to pointer since fewer possible mistakes
- cheapest way is reference (constant reference if want it immutable)
- if size if int or smaller, pass by value is just as good 
- if function has to make a copy anyways, just pass by value

Question: What if we want to use references in combination with actual numbers?
```C++ 
void g(int &n) {...}
g(5); // Not legal -> value of 5 can't be changed, it's a constant
void h(const int &n) {...}
h(5); // Legal because it is a constant
```
**Question**: 5 is a Rvalue (right value), and references (left references) bound to Lvalue?
**Answer**: Compiler create temporary memory location, copies 5 into it, and n is bound to that location. 

Some things (e.g., streams) cannot be copied 
- must be passed or returned by reference since can't be copied 

**Dynamic Memory Allocation** 
- C has calloc/malloc/realloc and free, but not type safe since uses (void *)
- C++ has 2 keywords, new and delete 
	- type safe 
	- no realloc, so much allocate new memory and copy 
```C++ 
int *p1, *p2, *p3;
p1 = new int; *p1 = 3; // stack has p1, pointer to 3 in heap memory
delete p1; // Breaks connection between p1 and 3

p1 = new int[5];
p2 = new int[5]{0}; // {0, 0, 0, 0, 0}
delete []p2; // need [] to delete array

p2 = nullptr;
delete p2; // ok!

p3 = new int*[5];

for (int i = 0; i < 5; ++i) {
	p3[i] = new int[10]{1}; // 5 rows, each row with 10 elements initialized to 1
}

for (int i = 0; i < 5; ++i) {
	delete [] p3[i];
}

delete []p3;
```

- Do not mix the 2 forms of new/delete -> the behaviour is unspecified 
- A "memory leak" occurs when not all (dynamic) memory is freed 
	- program eventually has an error
	- use valgrind to check for memory leaks (-v flag)