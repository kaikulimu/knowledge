## Problem Statement
We would like to implement a thread-safe multi-producer single-consumer queue. We can use a linked list. Consider the following scenario: The queue has three messages and both threads `T1` and `T2` want to push a message to the queue.

```
queue: head_ptr -> (M3) -> (M2) -> (M1)
T1: thread_ptr_1 -> (M4)
T2: thread_ptr_2 -> (M5)
```

In below examples, `T1` always attempts to push a message first, but `T2` always succeeds first.

## Approach 1 - Locks
1. `T1` calls to acquire a mutex on the queue.
2. `T2` calls to acquire a mutex on the queue.
3. `T2` acquires the mutex. `T1` is blocked.
4. `T2` pushes `(M5)` to the queue.
5. `T2` releases the mutex.
6. `T1` acquires the mutex.
7. `T1` pushes `(M4)` to the queue.
8. `T1` releases the mutex.

- *Issue 1*: The operation is blocking for `T1`
- *Issue 2*: Using mutexes requires additional memory and runtime

## Approach 2 - Atomic Load + Atomic Compare-Exchange
```C++
    void push(const T& val)
    {
        node<T>* thread_ptr = new node<T>(val);
        thread_ptr->next = head_ptr.load();
        while(!head_ptr.compare_exchange_weak(thread_ptr->next, thread_ptr));
    }
```

1. `T1` atomic loads the address of `head_ptr` into `thread_ptr_1->next`.
2. `T2` atomic loads the address of `head_ptr` into `thread_ptr_2->next`.
3. `T2` atomic compare-exchanges `*head_ptr` and `*thread_ptr_2`. It succeeds, resulting in `head_ptr -> (M5) -> (M3) -> (M2) -> (M1)`.
4. `T1` atomic compare-exchanges `*head_ptr` and `*thread_ptr_1`. :warning: It fails the comparison since `head_ptr` has changed, so it reloads `head_ptr` into `thread_ptr_1->next`.
5. `T1` atomic compare-exchanges `*head_ptr` and `*thread_ptr_1` again. It succeeds, resulting in `head_ptr -> (M4) -> (M5) -> (M3) -> (M2) -> (M1)`.

- *Issue*: If multiple threads atomic compare-exchange at the same time, one will succeed and rest will fail. Retrying wastes time.

## (optimal) Approach 3 - Atomic Exchange
```C++
    void push(const T& val)
    {
        node<T>* thread_ptr = new node<T>(val);
        node<T>* old_ptr = thread_ptr;
        thread_ptr.exchange(head_ptr);
        old_ptr->next = *thread_ptr;
    }
```

1. `T1` stores `old_ptr = thread_ptr_1`.
2. `T2` stores `old_ptr = thread_ptr_2`.
3. `T1` atomic exchanges `thread_ptr_1` and `head_ptr`. State:
```
queue: thread_ptr_1 -> (M3) -> (M2) -> (M1)
T1: head_ptr, old_ptr_1 -> (M4)
T2: thread_ptr_2, old_ptr_2 -> (M5)
```
4. `T2` atomic exchanges `thread_ptr_2` and `head_ptr`. State:
```
queue: thread_ptr_1 -> (M3) -> (M2) -> (M1)
T1: thread_ptr_2, old_ptr_1 -> (M4)
T2: head_ptr, old_ptr_2 -> (M5)
```
5. `T2` links `old_ptr_2->next` to `*thread_ptr_2`. State:
```
queue: thread_ptr_1 -> (M3) -> (M2) -> (M1)
T1: old_ptr_1 -> (M4)
T2: head_ptr -> (M5) -> (M4)
```
6. `T1` links `old_ptr_1->next` to `*thread_ptr_1`. State:
```
queue: head_ptr -> (M5) -> (M4) -> (M3) -> (M2) -> (M1)
```

- *Key Insight*: In Approach 2, we link new head to existing head **before exchange**, which might fail. In Approach 3, we link new head to existing head **after exchange**, which *always* succeeds.
