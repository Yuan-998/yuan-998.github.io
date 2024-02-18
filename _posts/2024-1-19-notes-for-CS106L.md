---
layout: post
title: Study Notes for CS106L at Standford University
tags: [C++, Parallel Programming]
readtime: true
---
## Initialization & References
#### Initialization
- **Direct initialization**: `int num = 12` or `int num(12.0) (narrowing conversion)`
- **Uniform initialization** (Safe and Ubiquitous): `int num{12}`. Uniform initialization **does** care about types.
- **Structured Binding** (C++ 17): e.g. `return {className, buildingName, language}`
```c++
#include <iostream>
#include <string>

using namespace std;

struct Student
{
    string name;
    string state;
    int age;
};


int main() {
    Student s{"Haven", "AR", 12};
    // Student s{"Haven", "AR", 12.0}; fail compilation 
}
```

#### References
```c++
int num = 5;
int& ref = num;
```
ref is a variable of type `int&`, that is an ***alias*** to `num` (Pointing to the same memory address).

##### Passing by reference
`void squareN(int& N)`: **take in the actual piece of memory, don't make a copy**

##### Passing by value
`void squareN(int N)`: **make a copy, don't take in the actual variable**

##### A classic reference-copy bug
```c++
#include <iostream>
#include <math.h>
#include <vector>

void shift(std::vector<std::pair<int, int>> &nums) {
    for (auto [num1, num2]: nums) { // passing by value here. should be auto& [num1, num2]
        num1++;
        num2++;
    }
}
```

#### l-value and r-value
##### Is `int& num` an l-value?
1. r-values are temporary
2. we cannot pass in an r-value by reference because they're temporary. (No memory address) 

#### const
**A qualifier for objects that declares they cannot be modified.**

**You can't declare a non-const reference to a const variable**

```c++
const std::vector<int> const_vec{1, 2, 3};
const std::vector<int>& const_ref_vec{const_vec};
```

## Streams
**A general input/output(IO) abstraction for C++**, providing interface for reading and writting data.

![](../assets/img/cs106l/streams.png)

#### standard iostreams -- basic_ostream and basic_istream


#### std::stringstream -- a way to treat strings as streams

`>>` only reads to the next whitespace. 

`istream& getline(istream& is, string& str, char delim)`: reads up until the `delim` char and stores it in some buffer, `str`. `delim` is by default `\n`. And `getline()` consumes the delim character.


#### Output Streams
`std::cout` stream is **line-buffered**. Contents in buffer not shown on external source until an explicit flush occurs.

`std::endl` tells the stream to end the line and also to **flush**. **Flushing** is an expensive operation.

> In many implementations, **standard output** is line-buffered, and writing '\n' causes a flush anyway, **unless std::ios::sync_with_stdio(false) was executed**. In those situations, unnecessary endl only degrades the performance of file output, not standard output.
>
>   https://en.cppreference.com/w/cpp/io/manip/endl


#### Input/Output File Streams
`std::ifstream` and `std::ofstream`

##### std::cin
- buffered
- buffer stops at a whitespace --> `std::cin` reads only until the next whitespace

## Containers

#### Two types of containers:
- Sequence:
  - Containers that can be accessed sequentially
  - Anything with an inherent order goes here
- Associative:
  - Containers that don't necessarily have a sequential order
  - More easily searched
  - Maps and sets go here

#### Maps
**Maps are implemented with pairs** (std::pair<const key, value>)

- **const key**: Keys are immutable
- Indexing into the map (myMap[key]) searches through the underlying collection of pairs first attribute for the key and will return its second attribute

##### Ordered/Unordered maps and sets
- **Ordered** maps/sets require a **comparison operator** to be defined
- **Unordered** maps/sets require a **hash function** to be defined

Unordered maps/sets are usually faster than ordered ones!

#### Container Adaptors
Container adaptors are "wrappers" to existing containers
- Wrappers **modify the interface** to sequence containers and change what the client is allowed to do/how they can interact with the containers

For example, how to make a wrapper to implement a queue from a deque?

## Iterators and Pointers
#### Iterators
- Iterators let you access **all** data in containers programmatically
- An iterator has a certain **order**; it knows what element will come next
  - Not necessarily the same each time you iterate

In the STL, all containers implement iterators, but they are not all the same.
- Each container has its own iterator, which can have different **behaviour**.
- All iterators implement a few shared operations:
  - Initializing: s.begin()
  - Incrementing: ++iter
  - Dereferencing: *iter
  - Comparing: iter != s.end()
  - Copying: new_iter = iter

![](../assets/img/cs106l/iterators.png)
- **Input** iterators can appear on the RHS of an = operator (`auto elem = *it;`)
- **Output** iterators can appear on the LHS of an = operator
- **Bidirectional** iterators can go forward as well as backward (`--iter; ++iter;`)
- **Random-access** iterators allow you to direcly access values without visiting all elements sequentially (`iter += 5;`)

**Iteration with iterators is const**: When you iterate over a container using iterators, the elements accessed through the iterators are treated as const. This means you can read the values but cannot modify them through the iterator.

#### Pointers
Iterators are a particular type of pointer
- Iterators "point" at particluar elements in a **container**
- Pointers can "point" at **any objects** in your code


## Class
- Containers are classes defined in the STL

#### Comparing 'stuct' and 'class'
***structures which are classes without access restrictions;***

#### Class design
1. A constructor
2. Private member functions/variables
3. Public member functions (interface for a user)
4. Destructor

### Container adapters
- Container adapters provide the interface for serveral classes and act as a template parameter

### Inheritance
- A virtual function, meaning that it is instantiated in the base class but overwritten in the subclass.

## Template Classes
Template Class: A class that is parametrized over some number of types; it is comprised of memober variables of a general type/types

A {template} container.h
```c++
template <typename T>
// or template <typename T, typename U>, A template declaration list
class Container {
  public:
    Container (T val);
    T getValue();
  
  private:
    T value;
}
```

### Template functions
Spot the difference:
```c++
template <class T>
Container<T>::container(T val) {
  this->value = val;
}

template <typename T>
T container<T>::getValue() {
  return value;
}
```

```c++
template <class T> // class and typename are interchangeable in basic cases
Container::Container(T val) {
  this->value = val;
}

template <typename T>
T Container::getValue() {
  return value;
}
```
In the second code snippet, we don't pass in the template parameter in the class namespace

C++ wants us to specify our template parameters in our namespace because, based on the parameters our class may behave differently. **There is no "one" Container, there is one for an int, char, etc.**

##### Another quirk of templates
When making template classes you need to #include the `.cpp` implementation in the `.h` file. A compiler quirk.
```c++
template <typename T>
class Container {
  public:
    Container (T val);
    T getValue();
  
  private:
    T value;
}

//must include this
#include "Container.cpp"
```

### Const correctness
```c++
std::string stringify(const Student& s) {
  return s.getName() + " is " + std::to_string(s.getAge()) + " years old";
}
```
**Compile error** for the above code.
- By passing in `s` as `const` we made a promise to *not* modify `s`
- The compiler doesn't know wheter or not **getName()** and **getAge()** modify `s`
- Remember, member functions *can* access and modify member variables

Solution:
```c++
...
  private:
    using String = std::string;
    ...
  public:
    ...
    String getName() const;
    int getAge() const;
    ...
...
```

### Const Interface
Definition: Objects that are const can only interact with the const-interface

The const interface is simply the functions taht are const/do not modify the object/instance of the class

```C++
...
int& at(size_t index) {
  return _array[index];
}

int& at(size_t index) const { // a const version of our .at()
  return _array[index];
}
...
```

But this process can be very painful when the size of the function grows.

Solution: **Casting**
```c++
int& findItem(int value) {
  for (auto& elem: arr) {
    if (elem == value) return elem;
  }

  throw std::out_of_range("value not found");
}

const int& findItem(int value) const { 
  return const_case<IntArray&> (*this).findItem(value);
}
```
Breakdown:
- `(*this)`: cast so that it is pointing to a non-const object
- Call the non-const version of the function
- Then cast the non-const return from the function call to a const version

