title: De-recursing the Tower of Hanoi Solver
description: Some thoughts about solving the Tower of Hanoi puzzle in Python, and how to convert the standard recursive solution into a non-recursive one.

# De-recursing the Tower of Hanoi Solver

In the first chapter of David Kopec's *Classic Computer Science Problems in Python*, we are presented with a very standard recursive solver for the [Tower of Hanoi](https://en.wikipedia.org/wiki/Tower_of_Hanoi) puzzle.

The puzzle involves 3 towers (let's call them $A$, $B$ and $C$) and a collection of $n$ disks of distinct sizes. 
The puzzle starts with all $n$ disks stacked on tower $A$ in size order, with the largest disk at the bottom and the stack and the smallest disk at the top. 
The goal is to perform a series of moves to move the entire stack over to tower $C$ (with the order preserved). 
A move consists of removing a disk from the top of the stack on one of the towers and placing it on top of a stack on another tower, with the restriction that a disk can only be placed on top of a disk larger than itself.

The standard recursive solution is to realise that, if we have a way to move the top $n-1$ disks from one tower to another, we can do this from tower $A$ to tower $B$, and then we can simply move the largest disk from tower $A$ to the (currently empty) tower $C$, before moving the $n-1$ disks from tower $B$ over to tower $C$ and the puzzle is solved.
This amounts to solving an $n-1-disk puzzle (from $A$ to $B$), moving one disk (from $A$ to $C$) and then solving the an $n-1-disk puzzle (from $B$ to $C$).
With a $1$-disk base case (simply moving the disk from its start to its finisn), the puzzle is now solved recursively.

After setting up an explicit `Stack` data-structure
```py linenums="1"
class Stack():
    def __init__(self):
        self._container = []

    def push(self, item):
        self._container.append(item)

    def pop(self):
        return self._container.pop()

    def __repr__(self):
        return repr(self._container)
```
which is to be filled in ascending order with
```py linenums="1"
tower_a = Stack()
tower_b = Stack()
tower_c = Stack()
for i in range(1, num_discs + 1):
    tower_a.push(i)
```
(where $1$ represents the largest disk, $2$ the next largest, and so on) Kopec presents a simple implementation of the recursive solution as
```py linenums="1"
def hanoi(begin, end, temp, n) -> None:
    if n == 1:
        end.push(begin.pop())
    else:
        hanoi(begin, temp, end, n - 1)
        hanoi(begin, end, temp, 1)
        hanoi(temp, end, begin, n - 1)

#solve with:
hanoi(tower_a, tower_c, tower_b, num_discs)
```
This is a very intuitive solution, and it works well. 
However, Python is not particularly well-suited to recursion.
Unlike many other languages, Python doesn't perform any kind of recursion optimization.
Indeed, if you go *too* deep with recursion, Python will raise a `RecursionError` and the program will terminate.
So, I decided to figure out how to write a Hanoi solver that avoids recursion altogether.

While there would be a number of ways to solve the Tower of Hanoi puzzle without recursion, one interesting approach is to literally implement the recursive version... but without recursion.
To do this, it is necessary to explicitly maintain your own function stack, instead of relying on Python's function stakc through recursive calls.
I did this by simply reusing the `Stack` data-structure already defined.

I present my solution below, with comments to explain what each part does in relation to the standard recursive version.
```py linenums="1"
def hanoi_nonrecursive(begin, end, temp, n):
    """
    3-tower hanoi solver with explicit stack to avoid recursion
    """
    if n == 1: # solve the base case in the same way as before
        end.push(begin.pop())
        return

    recursion_stack = Stack() # use own function stack

    #  add the original call to the stack:
    recursion_stack.push((begin, end, temp, n, 0)) 
    # the final element of the tuple is a 'position' integer that records 
    # whether we are before or after the first recursive call:
    # 
    # positions:
    #  - 0: before first recursive call
    #  - 1: after first recursive call

    # while loop iterates while there are still calls on the stack
    while len(recursion_stack._container) > 0: 

        # get the latest call from the stack:
        begin_curr, end_curr, temp_curr, n_curr, position = \
            recursion_stack.pop()

        if position == 0: # before the first recursive call? If so, then...

            #...add that call back to the stack with updated position:
            recursion_stack.push(\
                (begin_curr, end_curr, temp_curr, n_curr, 1)
            )
            # and then either...
            if n_curr == 2:
                #...solve the first n_curr-1 call directly if n_curr-1 == 1:
                temp_curr.push(begin_curr.pop())
            else:
                #...or otherwise add the n_curr-1 call to the stack
                recursion_stack.push(\
                    (begin_curr, temp_curr, end_curr, n_curr-1, 0)
                )

        # after the first recursive call? If so, then... 
        elif position == 1:

            # no need to put current call back on stack, we just...
            # ...move the 1 remaining disk from the start to the end
            end_curr.push(begin_curr.pop()) 
            # and then either...

            if n_curr == 2:
                #...solve the first n_curr-1 call directly if n_curr-1 == 1:
                end_curr.push(temp_curr.pop())
            else:
                #....or otherwise add the n_curr-1 call to the stack
                recursion_stack.push((\
                    temp_curr, end_curr, begin_curr, n_curr-1, 0)
                )
    # once recursion_stack is empty, we are done!
    return
```