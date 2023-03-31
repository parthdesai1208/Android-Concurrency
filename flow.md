# [Flow](https://kotlinlang.org/docs/flow.html)
Asynchronous and reactive programming in Kotlin made easy

## Flow Basics
- Flow is a stream that produces values asynchronously
- Flow uses coroutines internally. And because of this, it enjoys all the perks of structured concurrency
- When you cancel the scope, you also release any running coroutines. The same rules apply to Kotlin Flow as well. When you cancel the scope, you also dispose of the Flow. You don’t have to free up memory manually

## [Flow Builder](https://kotlinlang.org/docs/flow.html#flow-builders)
In simple words, we can say that it helps in doing a task and emitting items.
Here are a few examples of creating flow using `flow builder`
**1.flow{}** 
- Using the `flow` builder function with a lambda expression to emit values to the flow. This is the most common way to create a flow
```sh
val myFlow = flow {
    delay(1000)
    emit(1) // Emit values to the flow
}

myflow.collect{it -> 
    print(it)
}
//1
```
**2.flowOf**
- Using the `flowOf` function to create a flow from a fixed set of values.
```sh
val myFlow = flowOf(1, 2, 3).collect{
    print(it)
}
//1 2 3
```

**3.asFlow**
- Using the `asFlow` extension function to convert other types of collections or sequences to flows
```sh
(1..5).asFlow().collect{ print(it) } 
//1 2 3 4 5
```
**4.channelFlow**
- Using the channelFlow function to create a flow that can emit values from multiple coroutines concurrently
```sh
val myFlow = channelFlow {
    send(1) // Emit values to the flow
}
//1
```
## [Operator](https://kotlinlang.org/docs/flow.html#intermediate-flow-operators)
- The operator helps in transforming the data from one format to another.

Here are some of the commonly used flow operators:
**1.map**
Transforms each emitted value of the flow using a provided function.
```sh
flow{
    (1..3).forEach{
        emit(it)
    }
}.map{
    it * it
}.collect{
    print(it)
}
//1 4 9
```
**2.filter**
 Filters the values emitted by the flow based on a provided predicate function.
 ```sh
 flow{
     (1..3).forEach{
         emit(it)
     }
 }.filter{
     it > 1
 }.collect{
     print(it)
 }
 //2 3
 ```
 **3.take**
 Limits the number of values emitted by the flow to a specified number.
 myFlow.take(5)
```sh
 flow{
     (1..3).forEach{
         emit(it)
     }
 }.take(1)
 .collect{
     print(it)
 }
 //1
```
**4.zip**
Combines two flows into a new flow of pairs.
```sh
val flow1 = flowOf(1, 2, 3)
val flow2 = flowOf("one", "two", "three")
flow1.zip(flow2) { a, b -> "$a -> $b" }.collect{
    print(it)
}
// 1 -> one  2 -> two  3 -> three 
```
**5.flowOn**
To controlling the thread on which the task will be done
```sh
val flow = flow {
    // Run on Background Thread (Dispatchers.Default)
    (0..10).forEach {
        // emit items with 500 milliseconds delay
        delay(500)
        emit(it)
    }
}
.flowOn(Dispatchers.Default)    <--------------controlling thread
```
**6.flatMapConcat**
Transforms elements emitted by the original flow by applying transform, that returns another flow, and then concatenating and flattening these flows.
```sh
private val firstNameFlow = flow {
    emit("Ahmad")
}
firstNameFlow.flatMapConcat { firstName ->
    flow {
        emit(firstName)
        emit("Kazimi")
    }
}.collect {
    println("My name is : $it ")
}
//My name is Ahmad Kazimi    
```
**7.retry**
Used to retry an upstream flow collection in case of failure.
```sh
doLongRunningTask().
        .retry(retries = 3) { cause ->   //we declare 3 retries here
            if (cause is IOException) {
                return@retry true
            }else{
                 return@retry false
            }
        }.catch {
            // error
        }.collect {
            // success
        }
```
**8.debounce**
```sh
searchView.getQueryTextChangeStateFlow()
    .debounce(300) 
```
- Mainly used in search functionality
- when the user types “a”, “ab”, “abc”, in a very short time. So, there will be so many network calls. But the user is finally interested in the result of the search “abc”. So, we must discard the results of “a” and “ab”. Ideally, there should be no network calls for “a” and “ab” as the user typed those in a very short time.
- So, the `debounce` operator comes to the rescue. It will wait for the provided time for doing anything

**9. distinctUntilChanged**
Used to avoid duplicate network calls.
- Let say the last on-going search query was “abc” and the user deleted “c” and again typed “c”. So again it’s “abc”. So if the network call is already going on with the search query “abc”, it will not make the duplicate call again with the search query “abc”.

```sh
searchView.getQueryTextChangeStateFlow()
          .distinctUntilChanged()
```

**10. flatMapLatest**
Used to avoid the network call results which are not needed more for displaying to the user.
-  Let say the last search query was “ab” and there is an ongoing network call for “ab” and the user typed “abc”. Then, we are no more interested in the result of “ab”. We are only interested in the result of “abc”. So, the `flatMapLatest` comes to the rescue. It only provides the result for the last search query(most recent) and ignores the rest.

```sh
searchView.getQueryTextChangeStateFlow()
          .flatMapLatest { query ->
                    dataFromNetwork(query)
                    .catch {
                        emitAll(flowOf(""))
                    }
          }
```

**11. reduce**
This operator applies a given operation to all the emitted items and returns the final result.
```sh
val sum = flowOf(1, 2, 3).reduce { a, b -> a + b }
print(sum)
//6
```

**12. toList()**
collects all the items emitted by the flow and returns them as a list.
```sh
val list = flowOf(1, 2, 3).toList()
print(list)
//[1, 2, 3]
```

**13. fold**
This operator is similar to `reduce`, but it also takes an initial value and applies the given operation to the initial value and all the emitted items.
```sh
val sum = flowOf(1, 2, 3).fold(1) { acc, value -> acc + value }
print(sum)
//7
```


## Collector
- The collector collects the items emitted using the Flow Builder which are transformed by the operators.
```sh
flowOf(4, 2, 5, 1, 7)
.collect {
    //here we collect items which are emitted by flow
}
```










