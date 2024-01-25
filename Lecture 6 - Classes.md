
C++20 Compilation Shortcut:
Make not yet updated to include modules 

*Recall from last time...*
```
- must be compiled in C++ 20 modules in dependency order 
	1. System Files 
	2. Interfaces, in dependency order 
	3. Implementation Files 
- g++20h iostream 
- g++20m -c vec.cc -> creates vec.o
- g++20m -c vec-impl.cc -> creates vec-impl.o 
- g++20m -c main.c -> creates main.o 
- g++20m vec.o vec-impl.o main.o -o main 
- Otherwise, after compiling system headers:
	- g++20m -c \*.cc 
	- g++20m -c \*.cc -> compiles previous failures 
- g++20i -c \*.cc
- g++20i \*.o
```

Cheap Compilation trick:
	g++20m -c \*.cc
	g++20m -c \*.cc // should compile the ones that didn't successfully compile first time
	g++20m  \*.o [-o name] $\rightarrow$ links 

**Classes**
Benefit of OOP: Having Classes
- A "class" is a type definition (standalone class or part of a class)
	- has data fields plus methods (aka, member functions/operations)
- an instance of a class is called an:
	- "object"
- a struct in C++ *is* a class 

- e.g., // file:student.cc
```C++
export module student;
export struct Student {
	int assns, mt, final; // 3 data fields
	float grade(); // 1 method 
};
```

- file:student-impl.cc
```C++
module student;

// :: - scope resolution operator
float Student::grade() {
	return this->assns *. 0.4 + this->mt * 0.2 + this->final * 0.4;
}
```
Note: "::" is known as the scope resolution operator 
	- on the LHS of the operator is either the name of a namespace (e.g., std) or a class name

- file:main.cc
```C++
import <iostream>;
import Student;

int main() {
	Student s {60, 40, 70}; // aggregrate initialization 
	// 1st field goes to assignments 
	// 2nd field goes to mt 
	// 3rd field goes final 

	// Grade is a method of the class, so, we apply it to the Student s 
	// Student s is the receiver of the call of the method call 
	// and its data fields are used 
	std::cout << s.grade() << std::endl; 
}
```

- every method has an implicit first parameter "this"
```C++
Student s;
this = &s;
```

```C++
Student *p = new Student{100, 30, 99};
std::cout << p->grade() << std::endl;
p->mt = 95;
delete p;
```
- inline implementation -> DONT DO THIS! 

**Constructor Initialization**
- used to initialize objects 
	- can be overloaded, take default values for parameters
	- can perform more complicated code 
	- no return type on signature
	- must have class name as method name 

e.g., Student
```C++
struct Student {
	int assns, mt, final;

	// Default values go in interface 
	Student (int assns = 0, int mt = 0, int final = 0); 
};

// Implementation
Student::Student(int assns, int mt, int final) {

	// assns = assns; ? -> stores argument value back in argument 
	this->assns = assns;
	this->mt = mt;
	this->final = final;
}
```

```C++ 
Student s1; // s1{0,0,0}
// Student s1{}; does exactly the same thing 

Student s2{80};
Student s3{75, 67};
Student s4{100, 70, 99};
```

**Member Initialization List (MIL)**
Can only be used with constructor (ctor) definition
	- can be a more efficient form of initialization 
	- can't use anything more complex than an expression 
	- use as much as possible (no downsides!)

e.g., MIL for Student 
```C++
Student::Student(int assns, int mt, int final)
	// data field(s)
	: assns{assns}, mt{mt}, final{final} {// body of the constructor} 
```

Question: What does the following code do?
```C++
Student s1; 
Student s2 = s1;

// Before C++17, uses a copy constructor to initialize with s1.
// C++17 and up, doesnt do this. 
```

```C++
Student s3 = Student{10, 20, 30}; 
// RHS is a rvalue;
```
- prior to C++17, move the constructor call 
- C++17 and up, elision. 

**Default Constructor**
- compiler gives you a 0-parameter constructor for free 
	- lost as soon as you write any constructor 
	- for all data fields that are objects whose classes provides a default constructor, it calls the default constructor for that object 

e.g., 
```C++
Struct Vec {
	int x, y;
	Vec(int x, int y) : x{x}, y{y} {...};
};

struct Basis {
	Vec v1, v2;
};

// Calling
// illegal since Vec has no constructor 
Basis b; 
```

**Object Creation Steps**:
1. allocate object space 
2. initialize fields in declaration order 
3. run constructor body 
-> step 1 can't be changed
-> step 2 is too late for Vec constructor call 

- MIL however is run in Step 2!
- e.g., Basis Struct Example 
```C++
struct Basis {
	Vec v1, v2;
	Basis() : v1{0, 1}, v2{1, 0} {...}
	Basis (in tx1, int y1, int x2, int y2) 
		: v1{x1, y1}, v2{x2, y2} {}
}
```

**Question: What is the copy constructor form?**
Answer: e.g., Student 
```C++
Student::Student(const Student & other)
	: assns{other.assns}, mt{other.mt}, final{other.final} {
```

Unless you write your own, the compiler gives you:
	- default constructor(lost if you write any constructor)
	- move constructor 
	- copy constructor 
	- destructor 
	- copy assignment 
	- move assignment 

Question: When is the compiler provided copy constructor not good enough? 
Answer: Tuesday's lecture...