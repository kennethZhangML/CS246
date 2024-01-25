Operators >> (cin, x)
We can say:
```C++
cin >> x >> y >> g;

// Above is equivalent to below:
operator >> (operator >> (operator >> (cin, x), y));
```
``` C++
while (cin >> i) // no integer at all (return whether or not it fails

if (!cin) break; // ! operator cin.fail()
if (!(cin >> i)) // same as the above
```

In CS 136L, >> and << were used to shift bits 
e.g., 8 << 3
int operator << (int n, int pos);

In CS 246, now see << and >> for IO! 
``` C++
cout << "Hello"; 

// overloading 
// above is technically equivalent to: 
std::ostream& operator << (std::ostream& out, const char * s); 
```

We can "overload" functions in C++ .
Function name is the same, but the parameter values are different.
Therefore, it must be unambiguous (cannot be the slightest bit confused as to which one you wish to use). This means that:
- function name is the same
- number and/or types of parameters differ 
- must be unambiguous 
- need to know which one to call
- compiler does not look at return types!
	$\rightarrow$ must be determined "statically", i.e. at compile time 
- Overloaded for all the different types 

Let's change our program to read in and output integers:
	- stop on EOF 
	- throw away non-integers

```C++
import <iostream>
using namespace std;

int main() {

	int i;
	while (true) {
		cin >> i;

		if (!cin) {
			if (cin.eof()) {
				break;
			}
			// failed on attempted
			// state bits are auto set to fail
			// reset to clean slate 
			cin.reset(); // reset state bits 
			cin.ignore();
			// must reset first since ignore() reacts 
		} else {
			cout << i << endl;
		}
	}

}
```


*I/O Manipulators*
- used to format output (can affect input)
- ```import <iomanip>;```
- state change is "sticky" 
- i.e. persists until reset
- most are sticky so "cout << dec;" to reset 
- considered best practice to reset stream state when done 

*Other Manipulators*: std::endl, std::setw, setfill, std::skips, noskipcs

e.g., 
``` C++
cout << 95 ; // 95 in base 10
cout << hex << 95; // 5f, base 16
```

*Strings*
- in C, we use (char *) or (char []) 
	- must be terminated with "\0"; (with the NULL terminator)
	- must manually allocate new space and copy if resize/append 
	- easily to accidentally over-write "\0" and the overwrite adjacent memory 

- in C++, we have std::string which MUST be used 
	- ```import <string>```
		- does all memory allocation for you 
		- handles all memory allocation 
		- include resizing when appending strings together 
			- e.g., cat + fish = catfish (dynamically allocated) -> takes care of resizing for you 
		- handles all memory allocation include resizes and frees
		- concatenated, search, substring, etc (large set of things)
e.g., 
``` C++
import <string>

string s; // empty
string t = "Hello"; // t = "hello"
string w{"world"}; // w = "world"

// Equality
== !=, e.g., if (s == t) ...
<, <=, >, >=
size(), length(), e.g. if (s.size() == 0) ...

// concatentation
s = t + w; // s = "helloworld";
s += w; // "Helloworldworld";
```

- lexicographical comparison: <, <=, >, >=
- concatenation 

e.g., 
``` C++
import <iostream>
import <string>; 
using namespace std; 

int main() {
	string s;
	// skips leading whitespace 
	// appends read chars until whitespace or EOF 
	cin >> s;

	cout << s << cendl;
	
}
```

*Question*: What if we want to read in an entire line's worth of input into a string, include whitespace? 
*Answer*: ```std::getline```
e.g., getline(cin, s); given s is a string 
// reads from current input cursor position to '\n' which it throws away 

- streams are an "abstraction", 
	- a way of manipulating byte sequences for input and output 
- input and output 
	- file streams work exactly like input/output streams

file access in C 
``` C
#include <stdio.h>
int main() {
	char s[256];
	FILE *f = fopen("file.txt", "r");
	while (1) {
		fscan(f, "%256s", s);
		if (feof()) break;
		printf("%s\n");
	}
}
```

file access in C++
```C++
import <iostream>;
import <fstream>;
import <string>;
using namespace std;

int main() {
	string s;
	ifstream f{"file.txt"}; // fail bit will be set if fails 

	while (f >> s) {
		coust << s << endl;
	}
} // closes file when goes out of scope 
```

- output files must be closed to flush buffer 

*Question*: is there anything else that works like streams?
*Answer*: string streams (sstream)
- ostringstream may be useful for (x11 output)
- istringstream (until learn about exceptions)
	- can be used to convert a string to our int 

e.g., 
```C++
int stoint(string s) {
	istringstream oss{s};
	int i;
	oss >> i;
	return i;
}

int main() {
	string s;
	while (cin >> s) {
		cout << stoint(s) << endl;
	}
}
```