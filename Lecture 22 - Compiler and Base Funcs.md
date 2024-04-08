
Compiler turns non-virtual methods into regular functions. Not stored with object and address of call "hard-coded". 

- Dealing with virtual methods isn't part of the language standard but pretty much all compilers use `vtables`.
- Every object of a class with at least 1 virtual method contains a `vptr` (virtual pointer) to the *shared* `vtable` (virtual table)
- This implies that at runtime we use the objects' `vptr` to look up the location of the method in the `vtable` then go to that location 

This is why:
```C++
Book *pb = new Text { ... };
pb->isHeavy();
```
- invokes the correct virtual method at runtime.
![[IMG_4468.jpg]]
- space cost for `vptr` in every object of a class with a virtual method
- efficiency cost: dereference `vptr` then pointer in `vtable` to get to code 

Question: Why have we put the `vptr` first rather than last?
Answer: Putting the `vptr` first allows the derived object (pointer to) to look similar to the base object.

Question: What about multiple inheritance?

e.g., A4Q1
![[IMG_4469.jpg]]

e.g.,
```C++
class A {
protected: 
	int a;
};

class B {
protected:
	int b;
};

class C : public A, public B {
	int c;
public:
	void f() { cout << a << ' ' << b << ' ' << c << endl; }
};
```

Question: what happens when our hierarchy looks like 
Answer: Now we have 2 problems:
1. How many copies of A's data fields should we have? 1 or 2?
2. Need to disambiguate any `A` access as coming through either `B` or `C` if there are 2 copies 

If collapse copies of into 1 copy, creates a "deadly diamond" 
![[Screenshot 2024-03-28 at 9.07.56 AM.png]]

e.g., 
- Can be done by using virtual inheritance which turns A into a virtual base class. 
```C++
class A { ... };
class B : virtual public A { ... }; 
class C : virtual public A { ... };
class D : public B, public C { ... };
```
e.g., I/O library 

So what does the layout for the diagram above `D d` look like?
we might think it looks like
1. `vptr`
2. A's fields
3. B's fields 
4. C's fields
5. D's fields

Let's consider `B b;` (`C` will be similar ) 
1. `vptr_b` (B*) 
2. B fields (B*)
3. `vptr_a` (A*)
4. A's fields (A*)

Distance between "end" of B and "start" of A is $\emptyset$. 

Actual `D` object layout:
Note that the distance between the end of B and the start of A is now not zero!

Since the distance isn't a fixed constant value, but depends upon the object structure
- compiler needs to store the distance to the base class portion in the `vtable`
	- same type conversions will move the address for that object to be pointing to another `vptr` e.g., convert to C or A
	- `static_cast` or `dynamic_cast` under multiple inheritance 

**Template Functions**
e.g., 
```C++
int min(int a, int b) { return (a < b ? a : b); }
```
Can be re-written to use any type that provides operator< 

e.g., 
```C++
template<typename T> 
	T min(T a, T b)  { return (a < b ? a : b); }
```

Compiler may be able to deduce the type from the parameter types 
e.g., 
```C++
cout << min(1, 3);
```

From previous lecture:
```C++
void foreach(AbstractIterator& start, AbstractIterator& stop,
			void (f*)(int)) {
	while (start != stop) {
		f(*start);
		++start;
	}
} 
```

STL \<algorithm\> has a for_each function 
e.g., 
```C++
template <typename Iter, typename Fn>
	Iter for_each(Iter first, Iter last, Fn f) {

	while (first != last) {
		f(*first);
		++first;
	}
	return first;
}
```
- Works even with pointers since they provide: `*, ++, !=`

e.g., 
```C++
template <typename Iter, typename T>
Iter find(Iter first, Iter last, const T& val);
```
- returns Iterator to first element in 
- (First, Last) that matches val or last, `T` must define `operator==`
- Count returns the number of occurrences of val 

e.g., 
```C++
template<typename InIter, typename, OutIter>
OutIter copy(InIter first, InIter last, OutIter res);
```
- copies from (First, Last) into container staring at res, returns final value of res
- Problem: how does `OutIter` know to resize?
- For now, would have to reserve sufficient space.
- 