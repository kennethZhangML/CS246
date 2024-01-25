
```C++
Xwindow wx; // will see it on A5
ostringstream oss;
oss << "Score; " << score;
xwindowString(10, 10, oss.str());
extracts(const char*) by std::string:c_str(); 
```

```C++
// Example A:
// Exit Loop on either EOF or integer

string n;
int i;
while (true) {
	std::cout << "Enter a number: ";
	std::cin >> n;

	// Coursed into a boolean (did it succeed or not)
	if (istringstream iss{n}; iss >> i) break; 
	std::cout << "Not a number" << n << std::endl;
}

std::cout << i << std::endl;
```

```C++

// Example B:
// Loop until EOF, outputingg ints 
// Ignores non-integers 

// istringstream iss{s} -> uniform initialization syntax 

string s; // string s{};
while (std::cin >> s) {
	int n;
	if (istringstream iss{s}; iss >> n) { 
		std::cout << n << std::endl;
	}
}
```

**Question**: What is the difference in control flow between examples A and B?

**Question**: Why isn't istringstream::clear() called in either example when reading an integer fails? 
**Answer**: istringstream -> is being used for a single read, then being discarded 
used for the single if-statement (local to the if-statement) then is discarded 
iss is local to if and desetroyed when it ends (clear not needed)

**Command-Line Arguments**:
- works just like in C

**Argv**:
	- 0th => program name
	- ...
	- argc => initialized to nullptr 
	- argc >= 1 
	- 

```C++
int main(int argc, char* argv[]) {

	// # of command line arguments (program name isn't an argument)
	./pgm 123 abc 4 // argc = 4
}
```

- Convert argv elements to std::string 
```C++

int main(int argc, char* argv[]) {

	for (int i = 1; i < argc; ++i) {
		string s = argv[i];
		// ...
	}
}
```

```C++
// 02-io/ args-sum.cc
int main(int argc, char* argv[]) {

	int sum = 0;
	for (int i = 1; i < argc; ++i) {
		string s = argv[i];
		int n;
		if (istringstream iss{s}; iss >> n) {
			sum += n;
		}
	}
	cout << sum << endl;
}
```

**Default Function Parameters**
```C++
//03-functions/default.cc

void printSuiteFile(); // reads from "suite.txt" (Default file)
void printSuite(string fname); // reads from fname

// Invoke it without any value -> suite.txt is used (default)
-> void printSuiteFile(string fname = "suite.txt");
// ...
printSuiteFile(); // suite.txt used
printSuiteFile("suite2.txt"); // suite2.txt used 
// Cannot have a mixture of default and non-default 
```

**Note: all default parameters MUST be at the end of the parameter list** 
- The caller has to push all function parameters onto the run-time stack 
- Called functions does know how it was invoked so pops all parameters including defaults , from runtime stack. 
	- would be an error if had insufficient values 
	- compiler pushes all default values for the call on the caller's behalf 
	- default values tied to function declaration not implementation; in separate compilation place in header/interface file, not implementation

**Overloading**
- remember that C doesn't overload 
	- int readInt(int i)
	- bool regBool(bool b);

- unlike C++ (03-functions/overload.cc)
```C++
int negate(int i);
bool negate(bool b);
```
- resolved statically (compile time) by function name and number/types of parameters
- NOT, return values.

**Structs**
- backwards-compatible with C struct, but can do a lot more
```C++
struct Node {
	int value;
	Node* next; // no need to say struct again
};

// Field initialization 
Node n{13, nullptr};
```

- in C++, don't use 0 or NULL, use nullptr. 

- Question what is wrong with this definition?
```C++
struct Node {
	int value;
	Node next;
};

// uses incomplete definition
// make it a pointer 
// endless recursive definition
```

**Constants**
- don't use preprocessor # define 
	- not type safe
- must be initialized upon declaration
- e.g., const int MAX_SIZE = 10;
- ...
- int array[MAX_SIZE]; 
- make as many things constant as possible (parameters, return types, etc etc)
	- compiler will warn of attempts to modify 