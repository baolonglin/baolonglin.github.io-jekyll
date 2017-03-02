---
layout: post
title: "Throw exception in C++ destructor"
date: 2016-12-07
---


We should `Prevent exceptions from leaving destructors` as **Effective C++** suggeust, and in that book, it also explains the reason: `Now there are two simulataneously active exceptions, and that's one too many for C++`. Is not that program interupt when throw one exception? Why two exceptions will be throw out? Let's use one example to make those clear.


```cpp
#include <iostream>
#include <vector>

class Exception
{
public:
	Exception(int i = 0) : _i(i) {
	}
	~Exception() throw(int)
	{
		std::cout << "Throw : " << _i << std::endl;
		throw _i;        // <-- 1
	}
private:
	int _i;
};

class OtherObject
{
public:
	~OtherObject() {
		std::cout << "~OtherObject" << std::endl;
	}
};

int main()
{
	try {
		OtherObject o; // <-- 2
		{
			Exception e(10);  // <-- 3
			//throw 3; // <-- 4
		} // <-- 4
		std::cout << "after throw" << std::endl; // <-- 5
	} catch (int i) {
		std::cout << "Catch : " << i << std::endl; // <-- 6
	}

	try {
		std::vector<Exception> v(4); // <-- 7
	} catch (...) {
		std::cout << "Catch something" << std::endl; // <-- 8
	}
}
```

In the example above, use g++ 4.3.4, output looks like:

```
Throw : 10
~OtherObject
Catch : 10
Throw : 0
Throw : 0
terminate called after throwing an instance of 'int'
Aborted (core dumped)
```

**NOTE:** in g++ 6.2.1, it only get one `Throw : 0`. Because the STL vector's destructor default is noexcept, it terminate the program met the first exception.

What happends? The program is interrupt when object `e` throws exception in destructor, that's why position 5 is not executed.
But C++ doing a little bit more, it tries to release the object inside the try block, that's why `~OtherObject` is printed out.
The exception throw by the destructor is caught successfully, position 6 is executed.
What happends at position 7, why position 8 is not executed?
Do another test, if we try to open the position 3, throw exception before object `e` to be destruct, what will happen? In that case two exceptions are going to be throw out. Now the console output looks like:

```
Throw : 10
terminate called after throwing an instance of 'int'
Aborted (core dumped)
```

Haaa, that's `too many exceptions for C++`, C++ could not handle more than one exception properly, call terminate directly.
Now go back to position 7, it creates 4 instance of `Exception` and store it in `vector`, from my understanding, `vector` just destroy the objects inside it, call destructor of each object.
Here C++ compiler don't interupt the program met the first throw, continue call the destructor of the second object.

It's really bad idea to throw exception from destructor incase C++ need to safely release the temporary object and don't support multiple exceptions could be thrown in one try catch block.
That's why noexcept is the default added in C++11. You'll see warnings like:

```
test.cc: In destructor ‘Exception::~Exception()’:
test.cc:12:9: warning: throw will always call terminate() [-Wterminate]
   throw _i;
         ^~
test.cc:12:9: note: in C++11 destructors default to noexcept
```
