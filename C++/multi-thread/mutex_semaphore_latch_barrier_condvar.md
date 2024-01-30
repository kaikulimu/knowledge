## Mutex
> [`std::mutex`](https://en.cppreference.com/w/cpp/thread/mutex) \
> [`bslmt::Mutex`](https://github.com/bloomberg/bde/blob/main/groups/bsl/bslmt/bslmt_mutex.h)
- *"A lock"*
- Multiple threads try to `lock()`.
- Thread `A` calls `lock()` successfully. Thread `B` calls `lock()` and is blocked until Thread `A` calls `unlock()`.
- `try_lock()` is the unblocking version of `lock()`. It returns false immediately if the mutex is already locked.

## Semaphore
> [`std::counting_semaphore`, `std::binary_semaphore`](https://en.cppreference.com/w/cpp/thread/counting_semaphore) \
> [`bslmt::Semaphore`](https://github.com/bloomberg/bde/blob/main/groups/bsl/bslmt/bslmt_semaphore.h)
- *"A restroom with `N` stalls"*
- Starts with an initial counter value `N`.
- Every time `acquire()` is called, decrement `N`. If `N` is already `0`, blocks.
- Every time `release()` is called, increment `N`, potentially unblocking an `acquire()` call.
- `std::binary_semaphore` is an alias for specialization of `std::counting_semaphore` with `N = 1`.

## Latch
> [`std::latch`](https://en.cppreference.com/w/cpp/thread/latch) \
> [`bslmt::Latch`](https://github.com/bloomberg/bde/blob/main/groups/bsl/bslmt/bslmt_latch.h)
- *"A gate which requires pressing a button `N` times to open"*
- Starts with an initial counter value `N`.
- Every time `count_down()` is called, decrement `N`.
- `wait()` is blocked until `N` reaches `0`.

## Barrier
> [`std::barrier`](https://en.cppreference.com/w/cpp/thread/barrier) \
> [`bslmt::Barrier`](https://github.com/bloomberg/bde/blob/main/groups/bsl/bslmt/bslmt_barrier.h)
- *"A gate which requires `N` simultaneous magic channeling to open"*
- Starts with an initial counter value `N`.
- `arrive_and_wait()` decrements `N` and blocks until `N` reaches `0`.

## Conditional Variable \[CV\]
> [`std::condition_variable`](https://en.cppreference.com/w/cpp/thread/condition_variable) \
> No BDE equivalent
- *"A notifiable light bulb checker with a timeout"*
- Works on top of a boolean predicate `P` protected by the mutex `M`.
- `bool wait_until(mutex M, time_point T, predicate P)` atomically releases `M`, and blocks the current thread.
- `notify_one()` and `notify_all()` notify one or all blocked threads respectively.
- When the CV is notified before time `T`, check `P`. If `P == true`, reacquire `M` and continue execution. If `P == false`, block again.
- If time `T` is reached, `wait_until()` returns `P`. Finally, continue execution.

