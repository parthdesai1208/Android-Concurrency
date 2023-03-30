# [Coroutine](https://kotlinlang.org/docs/coroutines-overview.html)
[![Download](https://img.shields.io/maven-central/v/org.jetbrains.kotlinx/kotlinx-coroutines-core/1.7.0-Beta)](https://central.sonatype.com/artifact/org.jetbrains.kotlinx/kotlinx-coroutines-core/1.7.0-Beta)
Asynchronous programming made easy

## Why we use it?
Here are some of the reasons why you might want to use: 
- to easily write concurrent code without having to manage threads directly
- better readability and maintainability
- get rid of traditional callback-based approaches
- easily pause & resume operation

## Features

- Asynchronous programming: Kotlin coroutines allow you to write asynchronous code in a synchronous style, which makes it easier to reason about and less error-prone.
- Lightweight: Kotlin coroutines are lightweight threads that can be created and destroyed quickly, which makes them ideal for use in applications with a large number of concurrent tasks.
- Cancellation: Kotlin coroutines can be cancelled at any time, which allows for efficient resource management and helps prevent memory leaks.
- Structured concurrency: Kotlin coroutines provide a structured concurrency model, which ensures that all child coroutines are cancelled when a parent coroutine is cancelled.
- Suspending functions: Kotlin coroutines use suspending functions, which can be paused and resumed at any time, allowing for more efficient use of system resources.
- Coroutine builders: Kotlin provides several coroutine builders, including launch, async, and runBlocking, which make it easy to create and manage coroutines.
- Coroutine scopes: Kotlin coroutines can be created within a specific scope, which helps manage their lifecycle and ensures they are cancelled when they are no longer needed.

## Dependency
```sh
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:core_version"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:android_version"
```
[core_version](https://mvnrepository.com/artifact/org.jetbrains.kotlinx/kotlix-coroutines-core)
[android_version](https://mvnrepository.com/artifact/org.jetbrains.kotlinx/kotlinx-coroutines-android)

## Suspend 
Suspend function is a function that could be started, paused, and resume. means it doesn’t block the Thread, but release the Thread for the other coroutine to continue its work, and regain it back as the Thread is released.

![suspend-function-coroutines.png](https://amitshekhar.me/_next/image?url=%2Fstatic%2Fimages%2Fblog%2Fsuspend-function-coroutines.png&w=640&q=75)
>Only allowed to be called from a coroutine or another suspend function

## launch
- Used to start the coroutines
- returns a Job and does not carry any resulting value
>it's like fire and forget

#### UseCase - Just calling APIs & not worry about return result
```sh
 CoroutineScope(Dispatchers.IO).launch {
  //we will learn about Dispatchers.IO in coming section
            launch {
                Log.e(tag,API1())
            }
            launch {
                Log.e(tag,API2())
            }
            launch {
                Log.e(tag,API3())
            }
        }
    private suspend fun API1(): String {
        delay(2000)
        return "API1 Response"
    }

    private suspend fun API2(): String {
        delay(4000)
        return "API2 Response"
    }

    private suspend fun API3(): String {
        delay(1000)
        return "API3 Response"
    }   
//API3 Response    //after 1 second it will print
//API1 Response    //after 2 second it will print
//API2 Response    //after 4 second it will print
```



## async
- Used to start the coroutines
-  returns an instance of Deferred<T>, which has an await() function that returns the result of the coroutine
>perform a task and return a result

#### UseCase - Multiple API calls depends on each other
```sh
 CoroutineScope(Dispatchers.IO).launch {
            val APICall1 = async {
                API1()
            }
            val APICall2 = async {
                API2()
            }
            val APICall3 = async {
                API3()
            }
            val api1 = APICall1.await()
            val api2 = APICall2.await()
            val api3 = APICall3.await()
            Log.e(tag,"$api1 $api2 $api3")
        }
        
//API1 Response API2 Response API3 Response
//after all 3 API return the string it will print it's result
//all 3 jobs will wait for each other to finish
```

## runBlocking
- Runs a new coroutine and blocks the current thread until its completion.
- It is designed to actually write blocking code to libraries that are written in suspending style.
- To be used in main functions and in tests.

```sh
CoroutineScope(Dispatchers.IO).launch {
        Log.e(tag, "other thread is started working")
        delay(4000)
        Log.e(tag, "other thread is completed")
}
runBlocking {
        launch {
            Log.e(tag, API1())
        }
        launch {
            Log.e(tag, API2())
        }
        launch {
            Log.e(tag, API3())
        }
}
//other thread is started working
//API3 Response                     //after 1 second it will print
//API1 Response                     //after 2 second it will print
//API2 Response                     //after 4 second it will print
//other thread is completed         
//wait, why its printing after runBlocking{} block
//because runBlocking{} blocks other thread, 
//execute its child job & release that thread
```

## withContext
- Allows you to switch the context of a coroutine while preserving its job and continuation

```sh
CoroutineScope(Dispatchers.Main).launch {
    Log.e(tag, "Before withContext: Running on ${Thread.currentThread().name}")
    withContext(Dispatchers.IO) {
        Log.e(tag, "Inside withContext with Dispatchers.IO: Running on ${Thread.currentThread().name}")
    }
    Log.e(tag, "After withContext: Running on ${Thread.currentThread().name}")
}
//Before withContext: Running on main
//Inside withContext with Dispatchers.IO: Running on DefaultDispatcher-worker-1
//After withContext: Running on main
```
> As you can see, 
before withContext it was running on main thread, but with the use of withContext it was running in worker thread-1 

## Coroutine context
We have three types of context in Coroutines
#### 1.Dispatchers.Main 
This dispatcher is designed to be used with user interface elements in Android applications. It runs coroutines on the main thread.

#### 2.Dispatchers. io
This dispatcher is optimized for performing IO operations, such as reading from or writing to a file or a network socket

#### 3.Dispatchers.Default
This dispatcher is used by default if no other dispatcher is specified. It is backed by a shared pool of threads.

## Job
Every coroutine is associated with a job, which is created automatically when the coroutine is launched.
##### A coroutine job has several important properties:
- It can be cancelled using `cancel()` method
- It has a completion state: "active", "completed", or "cancelled". Queried using the `isActive`, `isCompleted`, and `isCancelled` properties.
- It can have children: A coroutine job can have child jobs, which are automatically cancelled when the parent job is cancelled or completed.
```sh
val exception = CoroutineExceptionHandler { coroutineContext, throwable ->
    Log.e(tag,throwable.message.toString())
}

val scope = CoroutineScope(Dispatchers.IO + Job() + exception)
scope.launch {
    scope.launch {
        Log.e(tag, API1())
    }
    scope.launch {
        Log.e(tag, API2())
    }
    scope.launch {
        Log.e(tag, API3())
    }
}
        
//API3 Response        
//API1 failed 
//now what about API2 Response? 
//because Job() not handled exception of child jobs very well & it will cancel the parent job too.
//to tackle this scenario we have SupervisorJob().
```

## SupervisorJob

if we use `SupervisorJob()` instead of `Job()` in above code,

```sh
val scope = CoroutineScope(Dispatchers.IO + SupervisorJob() + exception)
```

We have now output like this,
API3 Response
API1 failed
API2 Response //for this time we have API2 response

## Scope

A coroutine scope is an object that manages the lifecycle of a set of coroutines, that means coroutines launched in this scope will be cancelled when the scope is cancelled.

### Type of scopes

**1.GlobalScope**
This is a predefined coroutine scope that is available throughout the application.
```sh
GlobalScope.launch {
    //task
}
```
**2.CoroutineScope**
This is a custom coroutine scope that is created by the developer.
```sh
CoroutineScope(Dispatchers.IO).launch{
    //task
}
```
**3.lifecycleScope**
comes with [lifecycle ktx](https://mvnrepository.com/artifact/androidx.lifecycle/lifecycle-runtime-ktx?repo=google) & part of [Lifecycle-aware coroutine scopes](https://developer.android.com/topic/libraries/architecture/coroutines#lifecycle-aware)
used in activity to launch a coroutine
```sh
 lifecycleScope.launch {
    //task
 }
```
**4.viewModelScope**
comes with [lifecycle viewmodel ktx](https://mvnrepository.com/artifact/androidx.lifecycle/lifecycle-viewmodel-ktx) & part of [Lifecycle-aware coroutine scopes](https://developer.android.com/topic/libraries/architecture/coroutines#lifecycle-aware)
used in ViewModel to lanch a coroutine
```sh
viewModelScope.launch {
    // Coroutine that will be canceled when the ViewModel is cleared.
}
```

> Bonus tip => you can use [Restartable Lifecycle-aware coroutines](https://developer.android.com/topic/libraries/architecture/coroutines#restart)
> Even though the lifecycleScope provides a proper way to cancel long-running operations automatically when the Lifecycle is DESTROYED.
> You might want to collect a flow when the Lifecycle is STARTED and cancel the collection when it's STOPPED.

**5.coroutineScope**
When any child coroutine in this scope fails, this scope fails and all the rest of the children are cancelled.
```sh
runBlocking {
        try {
            coroutineScope {
                launch {
                    Log.e(tag, API1())
                }
                launch {
                    Log.e(tag, API2())
                }
                launch {
                    Log.e(tag, API3())
                }
            }
        } catch (e: Exception) {
            Log.e(tag, "catch block")
        }
}
//API3 Response
//catch block
```
> Note here API2 response isn't recorded. 

**6.supervisorScope**
Creates a CoroutineScope with SupervisorJob.
a failure of a child does not cause this scope to fail and does not affect its other children,
```sh
runBlocking {
    supervisorScope {
        launch {
            try {
                Log.e(tag, API1())
            } catch (e: Exception) {
                Log.e(tag, "catch block")
            }
        }
        launch {
            Log.e(tag, API2())
        }
        launch {
            Log.e(tag, API3())
        }
    }
}
//API3 Response
//catch block
//API2 Response     //for this time we have API2 response
```

## suspendCoroutine
- Used for converting legacy callback methods to suspend function

Suppose, we have legacy code like this,
```sh
fun getUser(id: String, callback: (User) -> Unit) {...}
//we had callback hell like this,
Api.getUser(id) { user ->
      Api.getProfile(user) { profile ->
          Api.downloadImage(profile.imageId) { image ->
              // ...
          }
      } 
}
//in order to use this method inside coroutine we have suspendCoroutine

suspend fun getUser(id: String): User  = suspendCoroutine { continuation ->
      Api.getUser(id) { user ->
          continuation.resume(user)
      }
}

//inside coroutine we can use it like this,
runBlocking {
    val user = getUser(id)    
}
```

## suspendCancellableCoroutine
Like `suspendCoroutine`, `suspendCancellableCoroutine` provides a bridge between the coroutine world and the callback world but also provide the ability to cancel the coroutine.

```sh
private suspend fun fetchUser(): User = suspendCancellableCoroutine { 
cancellableContinuation ->
  ...
  // We call "contiuation.cancel()" to cancel this suspend function.
  cancellableContinuation.cancel()
  ...
}
//we can use it like,

runBlocking {
        try {
           val user = fetchUser()
           updateUser(user)    
        } catch (exception: Exception) {
           //we get java.util.concurrent.CancellationException here
        }
}

```
