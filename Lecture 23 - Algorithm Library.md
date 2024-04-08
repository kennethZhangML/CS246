Remember `std::copy` from last lecture:
```C++
template<typename InIter, typename OutIter>
OutIter copy(inIter first, InIter last, OutIter result);

// copies from [first, last] into result, 
// returning iterator output next location
```

```C++
vector v{1, 2, 3, 4, 5, 6, 7};
vector<int> w(4);
copy(v.begin() + 1)

copy(v.begin() + 1, v.begin() + 5, w.begin());
// w = {2, 3, 4, 5};
```

```C++
template<typename InIter, typename OutIter, typename Fn>
OutIter transform(InIter first, InIter last, OutIter result, Fn f) {
	while (first != last) {
		*result = f(*first);
		++first;
		++result;
	}
	return result;
}
```

e.g., 
```C++
int add1(int i) { return ++i; }

transform(v.begin(), v.begin() + 3, w.begin(), add1);
// w = {2, 3, 4, 5}
```

Question: how can we further generalize transform?
i.e. what else can I use for `Fn` or for the iterators?

Let's start with `Fn` : two main, other choices:
1. Function objects
2. lambdas

**Function Objects**
- A class can override/overload the `operator()` function
e.g., 
```C++
class Plus1 {
public:
	int operator()(int i) { return i + 1; }
};

Plus1 p;
cout << p(4) << endl;

transform(v.begin(), v.begin() + 3, v.begin(), Plus1 {4});
```

```C++
class Plus {
	int n;
public:
	Plus(int n) : n{n} {}
	int operator()(int n) { return n + m; }
};

transform(v.begin(), v.begin() + 3, w.begin(), Plus{4});
```

The big advantage of function objects is the ability to remember 

e.g., 
```C++
class IncPlus {
	int m = 0;
public:
	int operator()(int n) { return n + (m++); }
	void reset() { m = 0; }
};
```

**Lambda**
Question: What if we want to find all even numbers in a vector but only ever use the function in one place? Do we really need a function?
Answer: No, write a lambda.

The usual definition:
```C++
bool even(int n) { return (n % 2 == 0); 
```

Can use count_if:
```C++
cout << count_if(v.begin(), v.end(), even);

... count_if(v.begin(), v.end(), []int(n) { return (n % 2 == 0); });
```

**Iterators**
Remember that an iterator is anything that supports : `* ++ !=`
- `<iterator>`: provides stream and inserter iterators

e.g., 
```C++
import <iterator>;
vector v{1, 2, 3, 4, 5};
ostream_iterator<int> out { cout, ", "};

copy(v.begin(), v.end(), out);
// outputs: 1, 2, 3, 4, 5, 
```

However, `copy(v.begin(), v.end(), w.begin());` doesn't work if `w` has insufficient space since the iterator only uses `operator=` to copy elements into `w` and won't know if/when/how to resize... 

Solution: use `std::back_inserter` iterator to iterate over output container: but only works for containers that provide method push_back e.g., `std::vector, std::deque` etc.

e.g., `copy(v.begin(), v.end(), back_inserter(w));`
Tip: familiarize yourself with `<algorithms>` and `<iterator>` to be effective 

Range Abstraction 
- many of the functions in `<algorithms>` take a pair of iterators
Q: What if we want to remove a set of consecutive items from a vector?
A: If not at the end of vector, we have to shuffle items down to fill the gap O(n) which is done multiple times (once for each item)

- if we can instead specify the range of items to remove, only shuffle O(n) once!
- suggests that we're not abstracting sufficiently, i.e. we want to specify a range
- many of these methods come with a range version

```C++
Iterator std::vector<T>::erase(iterator first, iterator last);
// removes [first last]
```

**Motivation**: can we compose functions
e.g., filter out all even numbers then square them?

e.g., 
```C++
auto even = [](int n) { return (n % 2 == 0); }
auto sqr = [](int n) { return n * n; }

vector v = { ... };
vector w, x;

copy_if(v.begin(), v.end(), back_inserter(w), even);
transform(w.begin(), w.end(), back_inserter(x), sqr);
```

Problems: 
1. Can't really chain copy_if and transform
2. needed intermediate storage (Vector w)

Really, we want a range not a pair of iterators.
This would let us write something like 
```C++
transform(copy_if(v, even), sqr);
```
- has a range specified by begin() end()

Now we can compose functions:
Q: can we get rid of the intermediate storage?
A: range only needs to look like it's sitting on a container, fetch data on demand 

```C++
auto x = std::ranges::views::transform(
	std::ranges::views::filter(v, even), sqr
);
```
Note: filter and transform are range adaptors and they have a second form that lets you treat them as predicates (functions)
e.g., `filter(pred), transform(pred)`
range not (directly) specified 
=> they become callable objects, parameterized by a range 
e.g., `filter(pred)(R)` where R is the range 
- `operator|` is defined such that `R | A = A(R)`
- `B(A(R)) => B(R|A) => R | A | B`
```C++
auto x = v | filter(even) | transform(sqr);
```

Just like a bash pipeline! 


