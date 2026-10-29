<!---
{
  "id": "f5737440-0818-442e-8330-396eb85df159",
  "teaches": "Cpp: Iterators and Standard Algorithms",
  "depends_on": [
    "6a5c3351-73bd-4cff-8119-01db07dbb801",
    "e2ab3230-2ec2-4a65-a2e6-57db1908770f",
    "ce9d3ddc-a17a-4f76-b03a-3c85c14a154f"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2026-10-29",
  "keywords": [
    "C++",
    "iterator",
    "begin",
    "end",
    "std::sort",
    "std::find",
    "std::count",
    "operator overloading",
    "range-based for",
    "g++"
  ]
}
--->

# Cpp: Iterators and Standard Algorithms

> In this exercise you walk through containers with iterators, run `std::find`, `std::sort` and `std::count` on them, and then write a small iterator class yourself. This shows that an iterator is an ordinary class object with overloaded operators, and that the range-based `for` loop is built on top of it. No `sudo` is required.

## Introduction

The standard library has containers on one side and algorithms on the other. The algorithms do not know which container they work on. What connects the two are **iterators**: objects that point to an element, can be moved to the next one and can be dereferenced. An iterator generalises the pointer: for an array it is a pointer, for a list it has to follow the `next` link of the node.

Because an iterator is an object of a class, it gets its behaviour from overloaded operators: `*it` returns the element, `++it` moves on, `it != end` compares two positions. Every container offers `begin()`, which points to the first element, and `end()`, which points **behind** the last one. The pair describes the half-open range `[begin, end)`.

### 1.1) Further Readings and Other Sources

* [cppreference: Iterator library](https://en.cppreference.com/w/cpp/iterator)
* [cppreference: Algorithms library](https://en.cppreference.com/w/cpp/algorithm)
* [cppreference: Range-based for loop](https://en.cppreference.com/w/cpp/language/range-for)

## Tasks

### Task 1: Walk with an iterator

```bash
mkdir -p ~/cpp-iter && cd ~/cpp-iter
```

```c++
// file: walk.cpp
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> v{"tesla", "benz", "vw"};

    for (auto it = v.begin(); it != v.end(); ++it)
        std::cout << *it << " has " << it->size() << " letters" << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra walk.cpp -o walk && ./walk
```

`*it` is the element, `it->` reaches its members, `++it` moves on. The condition is `!=` and not `<`: not every iterator can be compared with `<`.

### Task 2: The half-open range

```c++
// file: range.cpp
#include <iostream>
#include <iterator>
#include <list>
#include <vector>

int main() {
    std::vector<int> v{4, 5, 6};
    std::vector<int> empty;

    std::cout << "elements:          " << std::distance(v.begin(), v.end()) << std::endl;
    std::cout << "empty: begin==end: " << (empty.begin() == empty.end()) << std::endl;
    std::cout << "last element:      " << *(v.end() - 1) << std::endl;

    std::list<int> l{4, 5, 6};
    std::cout << "list elements:     " << std::distance(l.begin(), l.end()) << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra range.cpp -o range && ./range
```

For an empty container `begin()` equals `end()`, so a loop that runs `while (it != end)` needs no special case. Never dereference `end()`. Change `v.end() - 1` to `l.end() - 1` for the list and read the error.

### Task 3: One function, every container

```c++
// file: generic.cpp
#include <array>
#include <iostream>
#include <list>
#include <vector>

template <typename It>
void print(It first, It last) {
    for (; first != last; ++first)
        std::cout << *first << " ";
    std::cout << std::endl;
}

int main() {
    std::vector<int> v{1, 2, 3};
    std::list<int>   l{4, 5, 6};
    std::array<int, 3> a{7, 8, 9};
    int c[3] = {10, 11, 12};

    print(v.begin(), v.end());
    print(l.begin(), l.end());
    print(a.begin(), a.end());
    print(c, c + 3);                    // a plain C pointer is an iterator, too
}
```

```bash
g++ -std=c++17 -Wall -Wextra generic.cpp -o generic && ./generic
```

The function is a template over the iterator type. It works for four different containers, including a C array. This is what the standard algorithms do.

### Task 4: Find returns an iterator

```c++
// file: find.cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> v{3, 1, 4, 1, 5, 9};

    auto it = std::find(v.begin(), v.end(), 4);
    if (it != v.end())
        std::cout << "found " << *it << " at index " << std::distance(v.begin(), it) << std::endl;

    auto missing = std::find(v.begin(), v.end(), 7);
    std::cout << "7 found: " << (missing != v.end()) << std::endl;

    std::cout << "count of 1: " << std::count(v.begin(), v.end(), 1) << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra find.cpp -o find && ./find
```

`std::find` returns `end()` if nothing was found. You must always test for that before you dereference the result.

### Task 5: Sort, reverse and sum

```c++
// file: algos.cpp
#include <algorithm>
#include <iostream>
#include <numeric>
#include <vector>

int main() {
    std::vector<int> v{5, 2, 9, 1, 7};

    std::sort(v.begin(), v.end());
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    for (auto it = v.rbegin(); it != v.rend(); ++it)
        std::cout << *it << " ";
    std::cout << std::endl;

    std::cout << "sum: " << std::accumulate(v.begin(), v.end(), 0) << std::endl;
    std::cout << "min: " << *std::min_element(v.begin(), v.end()) << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra algos.cpp -o algos && ./algos
```

`rbegin()` and `rend()` are *reverse iterators*: the same class of object, but `++` moves backwards.

### Task 6: Sort your own type

```c++
// file: cars.cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

class car_t {
private:
    std::string name;
public:
    car_t(std::string n) : name{n} {}
    std::string get_name() const { return name; }
    bool operator<(const car_t &other) const { return name < other.name; }
};

int main() {
    std::vector<car_t> cars{car_t{"tesla"}, car_t{"vw"}, car_t{"benz"}};
    std::sort(cars.begin(), cars.end());
    for (const auto &c : cars) std::cout << c.get_name() << " ";
    std::cout << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra cars.cpp -o cars && ./cars
```

`std::sort` compares elements with `a < b`. Remove `operator<` and read the first line of the error: the algorithm requires an operation of your type, just like the template in the templates exercise.

### Task 7: An iterator is a class object

```c++
// file: type.cpp
#include <iostream>
#include <list>
#include <typeinfo>
#include <vector>

int main() {
    std::vector<int>::iterator vi;
    std::list<int>::iterator   li;
    std::cout << "vector iterator: " << typeid(vi).name() << ", " << sizeof(vi) << " bytes" << std::endl;
    std::cout << "list iterator:   " << typeid(li).name() << ", " << sizeof(li) << " bytes" << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra type.cpp -o type && ./type | c++filt -t
```

`iterator` is a **nested type** of every container, declared inside the class like a method. The names show that these are class (template) types, not plain pointers. The size equals one pointer: the object only stores a position.

Now write the class yourself:

```c++
// file: myiter.cpp
#include <iostream>

class Counter {
public:
    class Iter {
    private:
        int value;
    public:
        Iter(int v) : value(v) {}
        int  operator*() const { return value; }
        Iter &operator++()     { ++value; return *this; }
        bool operator!=(const Iter &o) const { return value != o.value; }
    };

    Counter(int n) : n(n) {}
    Iter begin() const { return Iter(0); }
    Iter end()   const { return Iter(n); }
private:
    int n;
};

int main() {
    Counter c(5);
    for (auto it = c.begin(); it != c.end(); ++it)
        std::cout << *it << " ";
    std::cout << std::endl;

    for (int x : c)                     // the same loop, written short
        std::cout << x << " ";
    std::cout << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra myiter.cpp -o myiter && ./myiter
```

The class `Counter` has no memory for its numbers, `Iter` only counts. Remove `operator!=` and read the error. Then remove `begin()` and watch the range-based `for` fail too: the compiler rewrites it to exactly the loop above.

### Task 8: Copy into another container

```c++
// file: copy.cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <list>
#include <vector>

int main() {
    std::vector<int> v{3, 1, 2};
    std::list<int>   l;

    std::copy(v.begin(), v.end(), std::back_inserter(l));
    l.sort();
    for (int x : l) std::cout << x << " ";
    std::cout << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra copy.cpp -o copy && ./copy
```

`std::back_inserter` is an iterator, too: assigning to it calls `push_back`. Write the destination as `l.begin()` instead, and describe what goes wrong (hint: the list is empty).

## Questions

- What do `begin()` and `end()` of a container return, and why does `end()` point behind the last element?
- Why can the same function `print` handle `std::vector`, `std::list` and a C array?
- What does `std::find` return if the value does not exist, and what happens if you dereference that?
- Which operations must `car_t` offer so that `std::sort` accepts it?
- Why is `std::vector<int>::iterator` a class and not simply `int*`, and which operators make it behave like a pointer?
- Which three member functions or operators does the range-based `for` loop need from `Counter`?

## Advice

When you meet a new container or algorithm, ask two questions: which iterators does it give me, and which operations does it need from my type. For the next chapter, keep the loop with `begin()` and `end()` in mind: it is the shape of the algorithms, and anonymous functions plug into exactly this shape.
