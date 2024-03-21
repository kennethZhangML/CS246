
**Correction**: `std::unique_ptr` only offers move semantics, copy operations are disabled

What is exception safety:

*It is not the case of*:
1. no exceptions raised, or
2. all exceptions handled
But that the program is in some valid state (not broken) 

There are 3 levels of exception safety:
1. **Basic**: program is in some valid but unspecified state; no memory leaks, no data corruption, class invariants are maintained
2. **Strong Guarantee**: the result must be as if the action never occurred (still no memory leaks or data corruption and class invariants are maintained)
3. **No Throw Guarantee**: operator will not fail, exception will not propagate 

```C++
class A { ... }
class B { ... }
class C { 
	A a;
	B b;

	public:
		void f() {
			a.g();
			b.h();
		}
};
```

Q: Can `C::F()` provide any safety guarantee if `A::g()` and `B::h()` dont'?
A: No, since we know nothing about the states of `C::a` and `C::b`.

Q: What if `A::g()` and `B::h()` both provide a strong exception safety guarantee but could raise an exception? What can we say about `C::f()`?
A: If `A::g()` and/or `B::h()` had side effects (e.g., changed static or global or reference variable state, performed I/O, it may be hard (or impossible) to reverse.
- `C::f()` can't provide a strong exception safety guarantee

If they don't have side effects, consider the cases:
1. Call to `a.g()` throws
	1. `a`'s state unchanged since `A::g()` it offers a strong guarantee
	2. `C::f()` has no handler, so stack unwinds, therefore as if `C::f()` wasn't called
2. if `b.h()` raises, need to undo changes to `C::a`
	1. How? what if we made a local copy and changed that? only copy over if call succeeded!
```C++
void C::f() {
	A tmpA = a;
	B tmpB = b;
	tmpA.g();
	tmpB.h();
	a = tmpA;
	b = tmpB;
}
```
*Copy-Swap Idiom*

Note: If copy constructors for A and/or B fail, this still meets strong exception safety guarantee 
- depending upon where copy assignment fails for `a = tmpA`, this may or may not meet strong safety exception guarantee 
- if `b = tmpB` fails, then we already changed `a`!

Solution: If we change the swap stage copy assignment into the exchange of 2 memory addresses, then it cannot fail!
- use the P.Impl Idiom "Pointer to Implementation"
- something used to hide implementation details

e.g., 
```C++
struct CImpl {
	A a;
	B b;
};

class C {
	std::unique_ptr<CImpl> ci;
	public:
		void f();
};

void C::f() {
	auto tmp = make_unique<CImpl>(*ci);
	tmp->a.g();
	tmp->b.h();
	std::swap(ci, tmp); // can't raise an exception!
}
```

**`std::vector` and Exception Safety**
Basic exception safety level guarantee requires no leaks
- `std::vector` uses RAII

e.g., 
```C++
class C { ... };

vector<C> v1; // destructor calls C's dtor
vector<unique_ptr<C>> v2; // destructor calls unique_ptr dtr to free heap memory
vector<C*> v3; // implies someone else is the owner of the heap memory and must free it, burden to free is not on the vector dtor; leaks memory if no one else frees 
```

Now consider `std::vector<T>::emplace_back` makes a ctor call for `T` to copy contents into the underlying array.
- if runs out of space, needs to allocate new array and copy contents over
	- if new fails, `emplace_back` offers strong exception safety guarantee so original array left unchanged
	- if copy assignment fails, same as before
- but, copy could be expensive .. so wouldn't move be better?
	- only if we can guarantee to the compiler that move operations won't raise an exception
	- declare move operations `noexcept`

e.g., 
```C++
class C {
	... 
	public:
		C(C &&o) noexcept;
		C& opertaor=(C&& o) noexcept;
	...
};
```
- if move operations aren't declared noexcept, compiler uses copy operations in `std::vector::emplace_back`

Style: If sure move ops won't throw exceptions, declare them `noexcept` so that the compiler can optimize things

**Casting**
C-style casting e.g.: `char c (char)255;`
- never use once learn C++ casting

1. `reinterpret_cast`: rarely ever used: strange conversions with undefined behaviour

```C++
class Student {
	Turtle *ptr = std::reinterpret_cast<Turtle*>(&s);
};
```

2. `const_cast`: add or remove `const` from something; dangerous since if we remove `const` if we weren't supposed to change it

```C++
void g(int *p);
void f(const int *p) {
	...
	g(std::const_cast<int*>(p));
	...
};
```

3. `static_cast`: convert between inheritance hierarchy types (pointer or reference) but you are guaranteeing to the compiler that this is legal
- undefined behaviour if not

```C++
Book *bp = new Text{ ... };
...
Text *tp = std::static_cast<Text*>(bp);
std::cout << tp->getTopic();
```
