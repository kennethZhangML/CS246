- Looks at material from 3 perspectives 
	- programmer who wants to write quality code and avoid errors
	- compiler support for various constructions
	- software design i.e. organize system of code using OOP (object oriented programming)

We'll use the C++20 standard. Assume basic knowledge of C. If a C++ way exists as an alternative, we'll require you to use it. 

**Notes on Aliases**
**g++11** $\rightarrow$ specific compiler version 
**-std=c++20** $\rightarrow$ use C++20 standard 
**-fmodules-ts** $\rightarrow$ turns on modules support 
**-Wall** $\rightarrow$ turn on warnings 
**-g** $\rightarrow$ saves symbol table info so can use GDB and ValGrind
**-x c++system-header** $\rightarrow$ compiling system header(s)

Hello world in C versus C++ 

```C
// C
#include <stdio.h>
int main() {
	printf("Hello, World!\n");
	return 0;
}
```

``` C++
import <iostream>; // actually a cmd, so ends with a semi-color

int main() {
	// << std::cout -> output 
	// >> std::cin -> input 
	std::cout << "Hello, World!" << std::endl;

	// default return value 
	return 0; // main must return an int and not a void
}
```

- C++ main() must return an int
- C++ has global objects 
	- ```std::cout ``` is standard output 
	- ```std::cerr``` is standard error
	- ```std::cin``` is standard input 
	- ```std::endl``` is a newline plus output buffer flush request 
- can dispense with "std::" if put ':' using namespace std

```g++20h``` $\rightarrow$ iostream 
```g++20m``` $\rightarrow$ hello.cc

``` C++
// This is our plus.cc

import <iostream>;
using namespace std;

int main() {
	// declare 2 ints 
	// Don't assume 0 initialization 
	int x, y;
	cin >> x >> y;
	cout << x + y << endl; // end with a newline 
	return 0; // optional in default 
}
```
```g++20m plus.cc```

- C++ allows spaces, tabs, newlines, to separate integers 
- A period **cannot** be considered a part of the integer or non-integer characters
- e^10 works as an integers 

- ```std::cin``` has "state bits": good, bad, fail and EOF, with corresponding methods that return them ```cin >> x >> y```; 

``` C++
if (cin.good()); // read 2 ints
else if (cin.eof()); 
else if (cin.fail()); // not successful read includes EOF 
```
- first attempt to read then check
- must read first 

```C++
// Readints.cc

import <iostream>;
using namespace std;

// reads and outputs int until either EOF or fails 

int main() {
	int i;
	while (true) {
		cin >> i;
		if (cin.fail()) break;
		cout << i << endl;
	}
}
```

``` C++
import <iostream>;
using namespace std;

int main() {
  int i;
  while (true) {
    cin >> i;

	// this is changed
	// calls the "!" operator on cin 
	// "!" is equivalent to cin.fail()
    if (!cin) break;
    
    cout << i << endl;
  }
}

```C++
import <iostream>;
using namespace std;

int main() {
  int i;
  while (true) {
    if (!(cin >> i)) break;
    cout << i << endl;
  }
}
```
Question: Why does "while (cin >> i)" work?
i) "cin >> i" returns cin
ii) can "type coerce" cin (std::istream) -> input stream object 
iii) turn it into a boolean value 
So the boolean value returns "!cin.fail()" (not EOF or fail) 