---
layout: post
title: Study Notes for CS106L at Standford University
tags: [C++, Parallel Programming]
readtime: true
---
## Initialization & References
#### Initialization
- Direct initialization: `int num = 12` or `int num(12.0) (narrowing conversion)`
- Uniform initialization (Safe and Ubiquitous): `int num{12}`. Uniform initialization **does** care about types.
- Structured Binding (C++ 17): e.g. `return {className, buildingName, language}`
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