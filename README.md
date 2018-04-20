# Cpp-Taskflow
A fast C++ header-only library to develop complex and parallel task dependency graph algorithms.

# Get Started with Cpp-Taskflow

The following example (`simple.cpp`) contains the basic syntax you need to use Cpp-Taskflow.

```cpp
// A simple example to capture the following task dependencies.
//
// TaskA---->TaskB---->TaskD
// TaskA---->TaskC---->TaskD

#include "taskflow.hpp"

int main(){
  
  tf::Taskflow tf(std::thread::hardware_concurrency());

  auto [A, B, C, D] = tf.silent_emplace(
    [] () { std::cout << "TaskA\n"; },
    [] () { std::cout << "TaskB\n"; },
    [] () { std::cout << "TaskC\n"; },
    [] () { std::cout << "TaskD\n"; }
  );  

  A.precede(B);  // B runs after A
  A.precede(C);  // C runs after A
  B.precede(D);  // D runs after B
  C.precede(D);  // C runs after D

  tf.wait_for_all();  // block until all tasks finish

  return 0;
}

```
Compile and run the code with the following commands:
```bash
~$ g++ simple.cpp -std=c++1z -O2 -lpthread -o simple
~$ ./simple
TaskA
TaskC  <-- concurrent with TaskB
TaskB  <-- concurrent with TaskC
TaskD
```

# Easy-to-use API
Cpp-Taskflow has very expressive, easy-to-use, and robust API to implement complex dependency graphs.
Most applications are developed through the following three steps.

## Step 1: Create a Task
To start a task dependency graph, 
create a taskflow object and specify the number of working threads in a shared thread pool
to carry out tasks.
```cpp
tf::Taskflow tf(std::max(1u, std::thread::hardware_concurrency()));
```
Create a task via the method `emplace` and get a pair of `TaskBuilder` and `future`.
```cpp
auto [A, F] = tf.emplace([](){ std::cout << "Task A\n"; return 1; });
```
Or create a task via the method `silent_emplace`, if you don't need a `future` to retrieve the result.
```cpp
auto [A] = tf.silent_emplace([](){ std::cout << "Task A\n"; });
```
Both methods implement variadic templates and can take arbitrary numbers of arguments to create multiple tasks at one time.
```cpp
auto [A, B, C, D] = tf.silent_emplace(
  [] () { std::cout << "Task A\n"; },
  [] () { std::cout << "Task B\n"; },
  [] () { std::cout << "Task C\n"; },
  [] () { std::cout << "Task D\n"; }
);
```

## Step 2: Define Task Dependencies
Once tasks are created in the pool, you need to specify task dependencies in a 
[Directed Acyclic Graph (DAG)](https://en.wikipedia.org/wiki/Directed_acyclic_graph) fashion.
The `TaskBuilder` object returned upon task creation support methods for you to describe dependencies.

### Precede
Adding a preceding link forces one task to run ahead of one another.
```cpp
A.precede(B);  // A runs before B.
```

### Broadcast
Adding a broadcast link forces one task to run ahead of other(s).
```cpp
A.broadcast(B, C, D);  // A runs before B, C, and D.
```

### Gather
Adding a gathering link forces one task to run after other(s).
```cpp
A.gather(B, C, D);  // A runs after B, C, and D.
```

## Step 3: Execute the Tasks
There are three methods to carry out a task dependency graph, `dispatch`, `silent_dispatch`, and `wait_for_all`.
```cpp
auto future = tf.dispatch();  // non-blocking, returns with a future immediately.
tf.dispatch();                // non-blocking, no return
```
Only when all tasks are complete does the call to `wait_for_all` return.
```cpp
tf.wait_for_all();
```

# Debug a Taskflow Graph
Concurrent programs are notoriously difficult to debug. 
We suggest (1) naming tasks and dumping the graph, and (2) starting with single thread before going multiple.

```cpp
// debug.cpp
tf::Taskflow tf(0);  // force the master thread to execute all tasks
auto A = tf.silent_emplace([] () { std::cout << "Task A\n"; }).name("A");
auto B = tf.silent_emplace([] () { std::cout << "Task B\n"; }).name("B");
A.precede(B);
std::cout << tf.dump();
```
Run the program and inspect whether dependencies are expressed in the right way.
```bash
~$ ./debug
# dumped graph
Task "A" [dependents:0|successors:1]
  |--> task "B"
Task "B" [dependents:1|successors:0]
# runtime message
TaskA
TaskB
```

# Caveats
While Cpp-Taskflow enables the expression of very complex task dependency graph that might contain 
thousands of task nodes and links, there are a few amateur pitfalls and mistakes to be aware of.

+ Having a cycle in a graph may result in running forever.
+ Trying to modify a dispatched task can result in undefined behavior.
+ Touching a taskflow from multiple threads are not safe.

The current version is known to work on most Linux distributions and OSX.
We havn't found issues in a particular platform.
Please report to us if any.

# System Requirements
To use Cpp-Taskflow, you only need a C++17 compiler:
+ GNU C++ Compiler G++ v7.2 with -std=c++1z

# Compile Unit Tests and Examples
Cpp-Taskflow uses [CMake](https://cmake.org/) to build examples and unit tests.
We recommend using out-of-source build.
```bash
~$ cmake --version  # must be at least 3.9 or higher
~$ mkdir build
~$ cmake ../
~$ make 
```
## Unit Tests
Cpp-Taskflow uses doctest for unit tests.
```bash
~$ ./unittest/taskflow
```

## Examples
The folder `example/` contains a rich set of practices of how to use Cpp-Taskflow.

# Get Involved
+ Report bugs/issues by submitting a <a href="https://github.com/twhuang-uiuc/cpp-taskflow/issues">GitHub issue</a>.
+ Submit contributions using <a href="https://github.com/twhuang-uiuc/cpp-taskflow/pulls">pull requests<a>.

# Contributors
+ Tsung-Wei Huang
+ Chun-Xun Lin
+ You!

# License

Copyright © 2018, [Tsung-Wei Huang](http://web.engr.illinois.edu/~thuang19/), [Chun-Xun Lin](https://github.com/clin99), and Martin Wong.
Released under the [MIT license](https://github.com/twhuang-uiuc/cpp-taskflow/blob/master/LICENSE).

