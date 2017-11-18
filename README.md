# README

> This is the new version of `lemondb`.

## Multi Thread Process Strategy

The multi threading is mostly done in `main.cpp` and `int main()` function.

We choose the strategy to read in all the queries, then output all the results, so it is not same as previous `lemondb`'s immediate response.

### Separating Different Queries

We change the input string into different `query` classes. By return `commandName` and `tableName`, we can start separating the queries.

Also we insert the output string to the output vector (global variable) to keep the order of output.

The threads are designed in the structure of

```
+------------+   ...   +------------+
| t1         |         | t_n        |
+------------+         +------------+
| write task |         | write task |
+------------+         +------------+
| read tasks |         | read tasks |
+------------+         +------------+
| write task |         | write task |
...                    ...
```

with `unordered_map` as hash table.

### Encountering Cost of Too Much Thread

In the real testing environment, the machine may complain `Temporary resource unavailable` if the thread number is not controlled. Our resolution is to add a global `counter` for the running thread. If the thread number is greater then specified, we let all the incoming new `query` wait until `counter` get smaller.

### Cleanup After All Inputs

Go through `thread_vector` to join all the threads, go through `outut_list` to finish output. By checking the `thread_vector`'s threads' job, we go pass or join them in case the output is still unavailable.

##Make Method

Please compile the whole program with `make` or `make re`. The test should be better performed under `Linux` machine.

## Some Naive Performance Test

On a 2 core 4 thread CPU, platform Arch Linux, the time usage is about $50\%$.

## Some Common Problems in Implementing

- The multi threading output ordering problem:

  If the problem is implemented with multi thread method, then we have to face the fact that different job will be finished in different time length. Thus the order of output should be preserved.

  We derive 2 solutions:

  - The output is temporarily stored in a list or a vector, if it is filled with no "hole" inside, then it is cleared. The output list is then updated.
  - The output in a vector is fixed, then we just wait until all the threads finish its job, output the vector content.

  We choose the later one since it may cost less in time and compute resources.

- If too much thread is called, the problem may not happen on windows, but `linux` may occur annoying `Temporary resource unavailable` complain, thus the program will not perform in the way we want.

  We derive some kind of solution by fitting with hardware. We control the running thread number by adding a global counter. If the running counter is larger then some hardware specified value.