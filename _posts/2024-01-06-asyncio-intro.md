---
title: Asyncio Introduction
author: Aya Elsayed
category: SQLAlchemy Workshop
date: 2024-01-06
layout: post
---

In this step, we're gonna learn about cooperative multitasking in Python through the popular library [`asyncio`](https://docs.python.org/3/library/asyncio.html).

To begin, checkout the branch [`step-5-asyncio-intro`](https://github.com/aelsayed95/sqlalchemy-sandbox/tree/step-5-asyncio-intro):

```sh
git checkout step-5-asyncio-intro
```

We'll start at the first file `01-sync_tests.py`.
Open up the file and take a look at the two synchronous python methods.
There is a `worker()` method, that simulates doing IO-bound work by calling `time.sleep()`.
The `main()` method calls a `worker()` instance for each of the jobs that need to be done.
Go ahead and run this file:

```sh
python 01-sync_tests.py
```

As you expect, of course, the workers execute the jobs synchronoushly.
How can we use coroutines to execute these jobs concurrently?

## Coroutines and `asyncio`

Coroutines in Python are methods that allow _cooperative multi-tasking_; a type of multi-tasking where the coroutines voluntarily yield control to other coroutines, as opposed to having a scheduler context-switch between them involuntarily, like it would with threads.
Coroutines can, therefore, be entered, exited and resumed at many different but deterministic points [1].

Coroutines can be defined using the keyword `async def` statement, and may contain synchronisation keywords like `await` and `yield` [1].

`asyncio` is a popular library for writing concurrent IO-bound code using the `async`/`await` syntax.
It allows you to schedule and run Python coroutines concurrently on an event loop, perform network IO, distribute tasks via queues and synchronise concurrent code [2].

## Using Coroutines

Let's turn our methods from `01-sync_tests.py` into coroutines and run them on `asyncio`'s event loop.

Open the file `02-async_tests.py`.
We have turned the `worker()` method into a coroutine using the `async def` keyword.
We also log the start time and end time of the job execution so we can better understand how long tasks take.

We have also turned our `main()` function into an `async` coroutine.
In `main()`, we `await` two tasks, "order milk" and "order bread", taking 1 and 2 seconds, respectively.

Finally, on L20, we start the execution by running `main()` on the event loop.
The program terminates when the event loop doesn't have any more tasks to run.

What do you expect to happen when we run this program?
Take a moment to think about the order of lines you expect to be printed to stdout.

Now, let's run the program:

```sh
python 02-async_tests.py
```

As you can see, the coroutines executed sequentially. And the tasks took 3s in total.
Why is that?

On L12, we schedule and run `worker(1, "order milk")`, let's call it `task1`.
Because of the `await` on L12, the `main()` coroutine is stopped there until `task1` is done executing.
Once the `await asyncio.sleep(delay)` on L8 is done, `task1` is resumed, prints its final message to stdout and signals that it's done.
This resumes the next ready coroutine on the event loop which is `main()` and advances the execution to L13.

Again, we schedule and run this task, which `await`s the sleep timer, and once done, `main()` is resumed again, and signals its end.
As there are no more tasks on the event loop, the program terminates.

So how do we make these coroutines run concurrently?

Comment out L12-13 and uncomment L18.
Now re-run this file.
You'll notice that now, the tasks take 2s instead of 3s.
How does `asyncio.gather()` achieve this?

## Concurrent Coroutines

Open the file `03-scheduling-tasks`.
To understand better how Tasks are scheduled and run on the event loop, we modify the example from `02-async_tasks.py` to schedule then `await` our tasks.

> ##### TIP
> 
> In `asyncio` terminology, a [Task](https://docs.python.org/3/library/asyncio-task.html#asyncio.Task) is a future-like object that runs a coroutine on an event loop [3].
{: .block-tip }

To do this, we use the `asyncio.create_task()` which creates a `Task` and schedules its execution.
So on L16-L17, we have scheduled both `task1` and `task2`.
Then on L21, we `await task1`.

So what order of execution do we expect now?
Run this file and observe the order of steps:

```sh
python 03-scheduling_tasks.py
```

How does this happen?
Here is a list of steps, showing the pseudo-state of the event-loop after each step.

T0: task1 scheduled, task2 scheduled, `main()` yields control at L21.
Event loop:

```txt
main: start          -- done
task 1: start        -- ready
task 2: start        -- ready
main: await task1.   -- busy
```

T1: task1 prints its start time, yields control as it awaits the sleep on L8.
Event loop:

```txt
task 1: start        -- done
task 2: start.       -- ready
main: await task1    -- busy
task 1: await sleep  -- busy
```

T2: task2 prints its start time, yields control as it awaits the sleep on L8.
Event loop:

```txt
task 2: start.       -- done
main: await task1    -- busy
task 1: await sleep  -- ready
task 2: await sleep  -- busy
```

T3: the next ready task is task1, it prints its end time and terminates, which makes main ready.
Event loop:

```txt
main: await task1    -- ready
task 1: await sleep  -- done
task 2: await sleep  -- busy
```

T4: the next ready task is main, its execution advances to the next await on L22.
Event loop:

```txt
main: await task1    -- done
task 2: await sleep  -- ready
main: await task2    -- busy
```

T5: task2 prints its end time and terminates, which makes main ready.
Event loop:

```txt
task 2: await sleep  -- done
main: await task2    -- ready
```

T6: finally, main resumes and terminates, the event loop is now empty.
execution resumes at L24 and the program terminates.

```txt
main: await task2    -- done
```

What happens if `task1` takes 2 minutes, and `task2` takes 1 minute?
How about if happens if `task1` takes 0 minutes, and `task2` takes 2 minutes?
What order do we expect?

So now, we can go back to `asyncio.gather()` in `02-async_tests.py`.
It has achieved concurrency by scheduling all the tasks passed to it, then awaiting them.

## Task Groups

Now that we understand how scheduling works, we can look at another useful alternative for scheduling and running tasks concurrently introduced in `Python3.11`, known as `asyncio.TaskGroup`.
It provides stronger safety guarantees than `gather()` for scheduling a nesting of subtasks by cancelling the remaining scheduled tasks if one task raises an exception [4].

Let's look at how it's used.
Open `04-task_groups.py`.
We have updated our `main()` coroutine to create a `TaskGroup` using an `async with` (async context manager).
We've named it `tg` and scheduled our tasks on it as usual.

Note that here, the `await` is implicit when the context manager exists [5].

Run the file and play around with the delay values:

```sh
python 04-task_groups.py
```

## async for

Open the file `05-async_for.py`.
This file introduces new syntax `async for`. How does it work?

It is used to loop over an `asynchronous iterable`.
An `asynchronous iterator` is an object that implements `__aiter__()` and `__anext__()`, the asynchronous counterparts to `__iter__()` and `__next__()` [6].

Go ahead and run the file:

```sh
python 05-async_for.py
```

you'll notice that each loop iteration executes when the async generator has produced a new value.
This can be useful so that you can do other work while waiting for values to be produced.

As an example, uncomment L25-L29 and re-run the file.
You can see that we can schedule other work to be done while waiting on the async generator.

## Using Queues

The last concept we will look at today is `asyncio.Queue`.
In our example `06-customers-shops`, we create two sets of tasks:

1. `customers`: who place orders on the queue

    the coroutine `place_order` places a random number of items `num_items`, 1 at a time, on the queue, with some browsing time in between.
    the coroutine terminates once all `num_items` have been placed.

2. `shops`: who process orders from the queue

    the coroutine `process_orders` runs indefnitely, `await`s an item to be available on the queue, processes the item, then marks the order (task) as done.

In main, we `await` `customers` to place all their orders, during which time some orders may be processed.
We then `await` the queue to drain, which waits for all orders to be processed.

Note that awaiting shops instead will cause our program to run indefinitely:

```py
await asyncio.gather(*shops)
```

This is because the coroutines `shops` run indefinitely.
Instead, we `await q.join()` then call `shop.cancel()` on each of the shop tasks to terminate them.

What happens if `main()` terminates while `shops` tasks are still running?

When `main()` terminates, `asyncio.run()` on L68 will be done, and it will cancel the remaining tasks on the event loop:
This raises the `asyncio.exceptions.CancelledError` exception, which we have chosen to handle on L51.

## References

1. [https://docs.python.org/3/glossary.html#term-coroutine](https://docs.python.org/3/glossary.html#term-coroutine)
2. [https://docs.python.org/3/library/asyncio.html](https://docs.python.org/3/library/asyncio.html)
3. [https://docs.python.org/3/library/asyncio-task.html#asyncio.Task](https://docs.python.org/3/library/asyncio-task.html#asyncio.Task)
4. [https://docs.python.org/3/library/asyncio-task.html#asyncio.gather](https://docs.python.org/3/library/asyncio-task.html#asyncio.gather)
5. [https://docs.python.org/3/library/asyncio-task.html#asyncio.TaskGroup](https://docs.python.org/3/library/asyncio-task.html#asyncio.TaskGroup)
6. [https://docs.python.org/3/glossary.html#term-asynchronous-iterator](https://docs.python.org/3/glossary.html#term-asynchronous-iterator)
