[Home](../README.md) > [Optimization](README.md) > [Multithreading](Multithreading.md)
# Multithreading
Used for heavy computational tasks, data processing and background workers to offload the game thread for tasks that are not time sensitive.

## Identify Inputs and Outputs
Since multithreading is used to process data, identifying inputs and outputs is the first step. Threads should never modify actors, however in some exception you can attempt to read from and spawn actors. It's best to always use data that does not rely on the game thread.

## Identify which thread management to use
Avoid excessive async calls if the workload is too small, as thread management overhead can negate performance gains.

**AsyncTask** : Quick, fire-and-forget tasks that uses Unreal Engine's task graph system with a thread pool. Doesn't allow control over its execution.
```cpp
AsyncTask(ENamedThreads::AnyBackgroundThreadNormalTask, []()
{
});
```
**FQueueThreadPool** : When you need to run multiple threads. It uses unreal's thread queue, so it will wait if all threads are used. Allows full memory control with StackSize being the memory allocated per thread for function calls, local variables and recursion.
```cpp
FQueuedThreadPool* MyThreadPool = FQueuedThreadPool::Allocate();
MyThreadPool->Create(NumQueuedThreads, StackSize, TPri_Normal);

class FMyTask : public IQueuedWork
{
public:
    virtual void DoThreadedWork() override
    {
    }
    virtual void Abandon() override {}
};

MyThreadPool->AddQueuedWork(new FMyTask());
```

**FRunnable** : When you need a long-running background thread with full control. Avoid overusing FRunnable.
```cpp
class FMyRunnable : public FRunnable
{
private:
    FRunnableThread* Thread;
    bool bRunning = true;
public:
    FMyRunnable()
    {
        Thread = FRunnableThread::Create(this, TEXT("MyThread"));
    }
    virtual bool Init() override { return true; }
    virtual uint32 Run() override
    {
        while (bRunning)
        {
        }
        return 0;
    }
    virtual void Stop() override { bRunning = false; }
    ~FMyRunnable() { delete Thread; }
};
```

**Task Graph**: For multiple threads with dependencies and priorities. See [the section below](#organizing-threads-using-task-graph)

**ParallelFor**: Runs a loop in parallel across multiple threads for heavy computation over large data sets.

## Thread Synchronization
Having threads require thread-safe communication to retrieve outputs.

### Mutual exclusion
Two threads can read and write the same object, but they must wait until the other thread unlocks its usage.

**Mutex & Locks** : Using `FCriticalSection`, `FReadScopeLock`, `FWriteScopeLock` or `FScopeLock` to implement a mutex is necessary when reading or writing non-thread safe data. `TArray` and `TMap` are not thread safe.

Careful not to call a function that locks the mutex while needing a locked mutex. To avoid this:
- Limit the duration of the mutex by copying data then unlocking it.
- Separate public API et private API. Public API uses locks while private API assumes it's already locked. Private functions can call other private functions, but not public ones.
- RAII (Resource Acquisition Is Initialization) : Use FScopeLock to automatically release mutexes after it is released.

### Double buffer
Data is doubled where one version is being written to and the other one is read. They swap when the writing is entirely done. Can be done using an Atomic flag.

**Atomic** : TAtomic with primitive types and pointers to have thread-safe flags, counters and states. Avoid the need for a mutex and locking for simple types.

### Producer-Consumer messaging queues
Have the TQueue on the consumer thread (game thread for example) and the other thread(s) can enqueue data safely to an SPSC or MPSC queue while the consumer thread dequeues it anytime.

**TQueue** : TQueue is thread safe and doesn't need a mutex when there's a single consumer (game thread) and multiple providers (threads) using MPSC. You must use a FCriticalSection when there are multiple consumers.

### Other ways

**AsyncTask** : It's best to start a new AsyncTask from the thread using `ENamedThreads::GameThread` to return the result. Using TFuture is also possible to wait for completion in a non-blocking way.

**TFuture** : Multiple thread management return a TFuture when creating a thread. Using `IsReady()` to detect if it's completed and `Get()` to get the output.

**FEvent** : Make a thread sleep without using any CPU until another thread triggers the event. Useful when new data becomes available to process. `FEvent* WorkAvailable = FPlatformProcess::GetSynchEventFromPool(true);` to retrieve the event, `WorkAvailable->Wait();` to sleep and `WorkAvailable->Trigger();` to awake the thread.

Avoid Synchronization : Use Thread Graph, TQueue, Double Buffering, data copies.

## Organizing Threads using Task Graph
**Task Creating and Communication**:
```cpp
auto MyTask = TGraphTask<YourTaskClass>::CreateTask().ConstructAndDispatchWhenReady();
```
```cpp
FGraphEventRef MyGameThreadTask = FFunctionGraphTask::CreateAndDispatchWhenReady([]() 
    {
    }, TStatId(), ENamedThreads::GameThread);
```

**Task Dependencies** : 
```cpp
FGraphEventRef TaskA = TGraphTask<MyTask>::CreateTask().ConstructAndDispatchWhenReady();
FGraphEventRef TaskB = TGraphTask<MyTask>::CreateTask().ConstructAndDispatchWhenReady();
TaskB->DontCompleteUntil(TaskA);
```
**Task Separation** : Separate large tasks into smaller chunks to avoid blocking the game thread.

**Task Priority** : `ENamedThreads::LowThreadPriority` and `ENamedThreads::HighThreadPriority`

## Profiling and Debugging Threads
Use stats (`stat thread`, `stat taskgraph`, `stat clock`, custom stats), UnrealInsight for visualization of the Task Graph and typical breakpoints and logs.


© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).  