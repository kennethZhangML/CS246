
```C++
Vec& Vec::operator+=(const Vec &v) {
	x += v.x;
	y += v.y;
	return *this;
}

Vec Vec::operator+(const Vec& v) {
	Vec tmp{*this};
	temp += v;
	return tmp;
}
```

Question: What about I/O operators?
Answer: Can't modify std::istream or std::ostream but could change vec 

```C++
struct Vec {
	// ...
	std::ostream& operator<<(std::ostream& out);
	// ...
}
```

```C++
std::ostream& Vec:::operator<<(std::ostream& out) {
	out << '(' << x << ", " << y << ')';
	return out;
}
```

Before, client would have written:
```C++
cout << v1 << '_' << v2 << endl;
```
But if operator<< is a method of Vec, has to be written as:
```C++
(v2 << (v1 << cout) << '_') << endl
```
which is counter-intuitive to normalize.
- Don't make I/O operators methods of the class 
- operators that must be methods of the class:
	- operator= (assignment)
	- operator[] (indexing e.g., std::string)
	- operator-> (dereferencing)
	- operator() (see function objects)
	- operatorT, where T is some type used for type coercion
		- e.g., coerce std::istream into a bool 

**Arrays of Objects**
Consider:
```C++
struct Vec {
	int x, y;
	// no default constructor
}

struct Vec2 {
	int x, yl

	// has default constructor 
	Vec2(int x = 0, int y = 0) 
		: x{x}, y{y} {}
}


int main() {

	Vec arr1[10]; // illegal (compile time error)
	Vec2 arr2[10]; // this is ok. hehe
	Vec* arr3[10]; // ok, 10 * (Vec *)

	for (int i = 0; i < 10; ++i) {
		arr3 = new Vec{i, i};
	}

	// ... 

	for (int i = 0; i < 10; ++i) delete arr3[i];
	Vec **arr4;
	arr4 = new Vec*[10];

	delete[] arr4;	
	return 0;

	Node** ptr;
	ptr = new(Node *)[10] {nullptr};
}
```

**Constant Objects** 
- usually seen in function/method parameter lists 
e.g., 
```C++
void foo(const Node &n);
// compiler checks that 'n' won't be changed by foo()
```
Consider a Student class with grade() method; 
e.g., 
```C++
void foo(const Student &s) {
	// ... 
	float g = s.grade();
	// ... 
}
```
- not a legal call since 
- Student::grade doesn't promise to not change (\*this)
- Can be fixed if we change both declaration and definition of grade() to be const
- i.e., promise not to change receiver object 

```C++
// .H File ---------------------
struct Student {
	// ... 
	float grade() const;
	// ...
}

// IMPL -------------------------
float Student::Grade() const {
	return ...;
}
```

Let's say we want to do some system profiling, i.e., count the number of methods called for each student :
```C++
struct Student { 
	int numMethodsCalled = 0;
	float grade() const {
		++numMethodsCalled; // illegal
		// ... 
	}
}
```

Divide "constness" into 2 versions: 
- "physical constness" implies modified object state (includes all data fields)
- "logical constness" only considers the data fields relevant to the concept of the class 
	- e.g., student grades, not numMethodsCalled counter 

- In C++, declare numMethodsCalled as "mutable" to indicate to the compiler that it isn't part of the logical constness and thus it's okay for grade() to change it

```C++
struct Student {
	mutable int numMethodsCalled = 0;
	// ...
}
```

Style Guideline: declare methods as const if appropriate 

Static in OOP 
- if we make a data field in a class "static" all instances/objects of the class share it 
- if make numMethodsCalled static, get a count of all methods for all objects 
```C++
// Header implementation
struct Student {
	static int numMethodsCalled  = 0;
	// ... 
	// int -> does not allocate space!
}


// .cc implementation 
int Student::numMethodsCalled = 0;

// Alternative:
struct Student {
	inline static int numMethodsCalled = 0;
	// ...
	static int getNMC() { return numMethodsCalled; }
}

int main() {
	cout << Student::getNMC();
	Student s;

	cout << s.getNMC();
	return 0;
}
```
- non-static methods can use/call static data fields/methods 
- static methods can only use static data fields/methods 

**Comparing Objects**
e.g., compare 2 strings
```C++
std::strng s1, s2;
// ...
if (s1 == s2) { ... }
else if (s1 < s2) { ... }
else { ... }
```
-> we had to do 2 comparisons
e.g., // in C 
```C++
int net = strlen(s1, s2);
if (ret == 0) { ... }
else if (ret < 0) { ... }
else { ... }
```
- C++20 introduced operator <=> ("space ship operator")
- import/include <compare>
e.g., std::strong_ordering res = s1 <=> s2;
i.e. res == 0 -> same 
	<= 0
	>= 0
	>0
	< 0
	!= 0

	   