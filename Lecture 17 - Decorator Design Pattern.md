Component: ABC Provides UI Framework
Concrete Component: Subclass being decorated 
Decorator: ABC may provide a default implementation of "operation" i.e. `next->operator()`
Concrete Decorator: Implements operation 

Application Example: piazza with choice of toppings, stuffed crust, dipping sauce
\[see lectures repository\]

Pizza: $\leftarrow$ Decorator $\leftarrow$ Topping, Dipping Sauce, Stuffed Crust 
\+ cost() : Float = 0
\+ dose() : String = 0
$\leftarrow$ Crust and Sauce

```C++
class Pizza { 
	public:
		virtual ~Pizza() {}
		virtual float cost() const = 0;
		virtual string desc() const = 0;
};

class Decorator : public Pizza { 
	protected:
		Pizza *next;
	public:
		Decorator(Pizza *p) : next{p} {}
		virtual ~Decorator() { delete next; }
};

// Has no next pointer => nullptr
class CrustAndSauce : public Pizza {
	public:
		float cost() const override { return 5.99; }
		string desc() const override { return "crust & sauce pizza"; }
};

class Topping : public Decorator {
	float *p.cost;
	float *p.desc;

	public:
		Topping(Pizza *p, float cost, string desc) : 
			Decorator{p}, top.cost{cost}, top.desc{desc} {}
		float cost() const { return top.cost + next->cost() }
		string desc() const { return top.desc + " and " + next->desc() }
};

int main() {
	Pizza *p = new CrustAndSauce{};
	p = new StuffedCrust{p};
	p = new Topping{p, 2.50, 'bacon'};
	cout << p->desc() << " $" << p->cost << endl;

	return 0;
}
```

**Note on Assignment:** 
TextProcess
\+ getWord() : String = 0;
\+ outstream(istream \*in) = 0;
$\leftarrow$
Echo

1. Returns a string with not length 0
2. The EOF exception will get raised 
3. get word must return a non-empty string
4. Test harness => has pointer to decorated chain
	1. After built => has not idea what's there
	2. keep asking for getWords till EOF exception get's raised
5. Double word => complicated
	1. harness asks returns double word to double word
	2. ask echo for word
	3. if echo returns word
	4. ah this is the first time i received the word
		1. lemme remember the word
		2. returns cat to test harness
		3. here's ur first word
		4. test harness says 
		5. where am i?
		6. return cat for first time
		7. return it for a second time => doesnt talk to echo
		8. change info (and im done)
		9. third time, double word looks at state info
		10. ask echo for the next word
		11. makes sense?
6. Not allowed to return empty string for get word
	1. where did you get input from: echo!
	2. Ask echo for the next word
	3. comes back with the word
	4. ask echo again
	5. here's the next word
	6. Drop first has to ask echo as many times as necessary until EOF is raised as an exception
	7. that's the trick 
7. Independent of each other 

**Exceptions**
Motivation: `std::vector::at(int i)`  checks that 
`0 <= i <= std::vector::size()`  

Question: What happens if `i < 0` or `i >= v.size()`?
Answer: Raises `std::out_of_range` exception

When an exception is raised, unless you properly handle it, your program will end with an unhandled exception.
- if not caught, program terminates with an unhandled exception 

'C' way of checking for errors requires either modifying global errno variable or return values 
- passive i.e. can't force programmer to check 
C++ uses exceptions - hard to ignore since the program terminates
- lets you separate error-handling code from "normal" code

Lets error go to whoever is best able to deal with it: 
`std::vector` can determine the index out of range but won't know what to do
- client knows what to do but can't easily determine the error 

e.g.,
```C++
#include <stdexcept>
#include <vector>

... 
vector<int> v;
...
v.at(100000); // a source out of range 
...

// Called a try catch block 
vector<int> b;
...
try {
	v.at(100000);
} catch (std::out_of_range) {
	cerr << ...;
}
// continues after...

void h(vector<int> &v) {
	v.at(100000);
}

void g(vector<int> &v) {
	...
	h(v);
	...
}

void f(vector<int> &v) {
	...
	g(v);
	...
}

int main() {
	vector<int> v;
	try {
		f(v);
	} catch (std::out_of_range) {
		...
	}
}
```

By definition, performing "stack unwinding" to find the matching handler 
=> `h` has no handlers, so not a candidate => remote activation frame and destroy any local objects
=> `g` same
=> `f` same
=> main has matching handler so run that

C++ allows exception hierarchies 
e.g., 
```C++
throw derivedExn{}; // Matching handler 

// get's object sliced 

...

try {
	f();
} catch (BaseExn e) {
	...
}
```

Best Practice: Catch by reference to prevent slicing 

Note: if we have exception hierarchy, order of catch classes matter
executed even if raised a DerivedExn etc... 
```C++
try {
	...
} catch (BaseExn &e) {
	...
} catch (DerivedExn &e) {
	...
}
```

Best practice: order classes from most specific to least

Raising by value:
- DerivedExn c;
- BaseExn &b = d;
- throw by;

Question: what is actually raised?
A: statically bound to type of what was raised i.e.

```C++
BaseExn
```
=> so raise by value: 
`throw d;`
or throw DerivedExn {};

Question: What happens:
```C++
catch (baseExn &e) {
	...
	throw e; // raises BaseExn!
	throw; // Raises Derivation! 
	...
}
```

**Exceptions and Destructors**
Best practice: never raise an exception in your destructor.
Reason: could have been unwinding the stack => already have an exception raised so which takes precedence? **Forbid it instead!**

C++ lets you throw anything as an exception 
```C++
try {
	...
} catch (...) { // literal '...' (should be the last clause) 
	// it always is last!
	// do something 
 	...
}
```

// enuf to finish question 1 and 3 (add handler for EOF to the main)