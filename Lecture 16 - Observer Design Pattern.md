**Iterator design pattern revisited**
Now that we have abstract base classes and polymorphism, let's see how the Iterator design pattern is really structured. 

```C++
Abstract Iterator
+ operator*():Integers = 0
+ operator++():AbstractIterator = 0;
+ operator!=(AbstractIterator& pother): boolean = 0;
```

```C++
class AbstractIterator {

public:
	virtual ~AbstractIterator() {}
	virtual int operator*() const = 0;
	virtual AbstractIterator& operator++();
	virtual bool operator!=()(const AbstractIterator& other) const = 0;
};
```

```C++
class List {
	...
	public: 
		class Iterator: public AbstractIterator {

			public:
				int operator*() const override { ... }
				Iterator& operator++() { ... }
				bool operator!=(const AbstractIterator &o) { ... }
		}
		
}
```
- `this` now let's us write a foreach function that previews the STL \<algorithms\>
- `std::for_each`: all inherit from AbstractIterator (don't inherit from anything yet)

```C++
std:for_each:
void foreach(AbstractIterator& start, AbstractIterator& end, void (*f)(int&)){
	while (start != end) {
		f(*start);
		++start;
	}
} 
```

```C++
void inc(int &i) { ++i; }

List myList;
// ...
foreach(myList.begin(), myList.end(), inc);
```

**Observer Design Pattern**
- also known as "Publish-Subscribe"
- classic use case : spreadsheet 
	- has model that contains data and formula
	- spreadsheet is the view/visual representation of the model 
		- as well as how you interact with it 
	- can have charts associated with specific ranges of data
	- data model is the "subject" while the charts and spreadsheet are observed

e.g., Social Media Application 
```markdown
| Subject |
| --------: |
| + attach(Observer o) : void; |
| + detach(Observer o) |
| + modifyObservers() : void; |
```

```Markdown
| Observer |
| --------: |
| + modify(): void; |
```

```C++
class Subject {
	std::vector <Observer> observers;
	public:
		virtual ~Subject() {}
		virtual void attach(Observer* o) {
			observers.emplace_back(o);
		}
		
		virtual void detach(Observer* o) {
			for (auto it = observers.begin(); it != observers.end(); ++it) {
				if (*it == 0) { 
					observers.erase(it);
					break;
				}
			}
		}

		void notifyObservers() {
			for (auto o : observers) o->notify();
		}
}
```

```C++
class Observer {
	public:
		virtual ~Observer() {}
		virtual void notify() = 0;
};

class HorseRace: public Subject {
	ifstream in;
	string lastWinner;

	public:
		HorseRace(string source) : in{source} {}
		bool runRace();
		string winner() { return lastWinner; }
};

bool runRace() {
	bool result { in >> lastWinner };
	if (result) {
		std::cout << "Winner: " << lastWinner << std::endl;
	}

	return result;
}
```

```C++
class Bettor : public Observer {
	HorseRace *hr;
	string name, favHorse;

	public:
		Bettor(HorseRace *hr, string n, string fh) :
			hr{hr}, name{n}, favHorse{fh};
			{ hr->attach(this); }
		~Bettor() { hr->detach(this); }

		virtual void notify() override {
			cout << (hr->winner() == favHorse ? "Yay!" : "Boo!") << endl;
		}
};

int main() {
	string fname = races.tn;
	HorseRace hr{fname};
	Bettor strcmp{&hr, "stamp", "Runs like a Cow"};
	...
	while (hr.runRace()) {
		hr.notifyObservers();
	}
}
```
- A4Q12: Cell inherits from Observer and implements Subject 

**Decorator Design Pattern**
- want to change an objects attributes and/or behaviour dynamically at runtime 
- if only done statically, would need a class for every possible combination 
- classic example: windowing system 
- e.g., decorate window with things such as title bar, max/min/destroy buttons, horizontal vs vertical scroll bars etc...
- Component
	- action() = 0;
		- Concrete component
	- Decorator (composition) => subclass of component 