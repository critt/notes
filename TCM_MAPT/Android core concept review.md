## Kotlin vs Java
- *Kotlin compiles down to Java bytecode and runs on the JVM.*
- *Kotlin has modern features like coroutines and extension functions.*
- *Kotlin has inbuilt null safety--Java does not.*
- *Java requires writing much more boilerplate and its Syntax is much more verbose than Kotlin's*


## Scope functions in Kotlin:
`let`, `run`, `with`, `apply`, `also`
- *A set of functions that allow you to execute a block of code within the context of an object.*
- *Each function has a context object: `this` or `it`*
- *They can return the context object itself or the result of the lambda expression:*
	- *`let`, `run`, and `with` return the lambda result*
	- *`apply`, and `also` return the context object*
- *They make for more concise and readable code.*
#### `let`
*Useful for operations on a nullable object and chaining operations on the same object*
*Returns lambda result*
```kotlin
val name: String? = "Kotlin"
name?.let {
    println("The length of the name is ${it.length}")
}
```
#### `run`
*Useful for operations on a nullable object*
*Also useful when you want to return a value*
*Returns lambda result*
```kotlin
val result = "Kotlin".run {
    length
}
println(result) // Outputs: 6
```
#### `with`
*Suitable for calling multiple methods on the same object without repeating the object's name*
*Returns lambda result*
```kotlin
val builder = StringBuilder()
with(builder) {
    append("Kotlin")
    append(" is")
    append(" fun!")
}
println(builder.toString()) // Outputs: Kotlin is fun!
```
#### `apply`
*Used to configure* an object and return it
*Often used for initializing objects*
*Returns context object*
```kotlin
val person = Person().apply {
    name = "Alice"
    age = 25
}
println(person) // Outputs: Person(name=Alice, age=25)
```
#### `also`
*Used for side effects, such as logging or performing additional operations on an object*
*Seems kinda like a grab bag--a mix of let and with*
*Returns context object*
```kotlin
val numbers = mutableListOf(1, 2, 3)
numbers.also { println("Current list: $it") }
        .add(4)
println(numbers) // Outputs: [1, 2, 3, 4]
```



## Kotlin generics
```kotlin
sealed class ApiResult <out T> {  
    data class Success<out R>(val data: R?): ApiResult<R>()  
    data class Error(val message: String): ApiResult<Nothing>()  
    object Loading: ApiResult<Nothing>()  
}
```
Generics in Kotlin allow you to write flexible and reusable code by defining classes, functions, and properties that can operate on various types while maintaining compile-time type safety. 
#### Generic class
```kotlin
class Box<T>(var item: T)

val intBox = Box(1) // Box<Int>
val stringBox = Box("Hello") // Box<String>
```

#### Generic function
```kotlin
fun <T> printItem(item: T) {
    println(item)
}

printItem(123) // Works with Int 
printItem("Kotlin") // Works with String
```

#### Generic with type constraint
*You can constrain a generic to ensure it matches a certain type or implements an interface*
```kotlin
fun <T : Number> add(a: T, b: T): Double {
    return a.toDouble() + b.toDouble()
}
```

#### Generic with covariance (`out` keyword)
*Allows a type to be assigned to a supertype. It is used for producers, meaning you can only return (produce) values of the specified type.*
```kotlin
class Producer<out T>(private val item: T) {
    fun produce(): T = item
}
```
*The general rule is this: when a type parameter `T` of a class `C` is declared `out`, it may occur only in the out-position in the members of `C`, but in return `C<Base>` can safely be a supertype of `C<Derived>`.*

*In other words, you can say that the class `C` is covariant in the parameter `T`, or that `T` is a covariant type parameter. You can think of `C` as being a producer of `T`'s, and NOT a consumer of `T`'s.*
```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```

#### Generic with contravariance (`in` keyword)
*Allows a type to be assigned to a subtype. It is used for consumers, meaning you can only accept (consume) values of the specified type.*
```kotlin
class Consumer<in T> {
    fun consume(item: T) { /*...*/ }
}
```
*In addition to `out`, Kotlin provides a complementary variance annotation: `in`. It makes a type parameter contravariant, meaning it can only be consumed and never produced. A good example of a contravariant type is `Comparable`:*
```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, you can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

#### Reified type parameters (`reified` keyword)
*The type arguments of generic function calls are also only checked at compile time. Inside the function bodies, the type parameters cannot be used for type checks, and type casts to type parameters (foo as T) are unchecked. The only exclusion is inline functions with `reified` type parameters, which have their actual type arguments inlined at each call site. This enables type checks and casts for the type parameters:*

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

```kotlin
inline fun <reified A, reified B> Pair<*, *>.asPairOf(): Pair<A, B>? {
    if (first !is A || second !is B) return null
    return first as A to second as B
}

val somePair: Pair<Any?, Any?> = "items" to listOf(1, 2, 3)

val stringToSomething = somePair.asPairOf<String, Any>()
val stringToInt = somePair.asPairOf<String, Int>()
val stringToList = somePair.asPairOf<String, List<*>>()
val stringToStringList = somePair.asPairOf<String, List<String>>() // Compiles but breaks type safety!

/* Result:
stringToSomething = (items, [1, 2, 3])
stringToInt = null
stringToList = (items, [1, 2, 3])
stringToStringList = (items, [1, 2, 3])
*/
```



## Kotlin classes
#### data classes
*A class primarily used to hold data. Automatically generates useful methods, such as `equals()`, `hashCode()`, `toString()`, and `copy()`, based on the properties defined in the primary constructor.* 
```kotlin
data class User(val name: String, val age: Int)

fun main() {
    val user1 = User("Alice", 25)
    val user2 = user1.copy(age = 26)

    println(user1) // Outputs: User(name=Alice, age=25)
    println(user2) // Outputs: User(name=Alice, age=26)
}
```

*Also supports destructuring objects into multiple variables*
```kotlin
val (name, age) = user1
println("Name: $name, Age: $age") // Outputs: Name: Alice, Age: 25
```

#### sealed classes
*type safety, exhaustive `when`s, conciseness, readability for limited hierarchy use-cases like UI or network state*

*A **sealed class** in Kotlin is a special type of class that restricts the inheritance hierarchy, meaning all the possible subclasses of the sealed class are known at compile time. This makes sealed classes particularly useful for representing restricted hierarchies, such as states or types that can only have a specific set of values.*

*They also provide exhaustiveness in `when` statements*.
```kotlin
sealed class NetworkResult {
    data class Success(val data: String) : NetworkResult()
    data class Error(val error: Throwable) : NetworkResult()
    object Loading : NetworkResult()
}

fun handleNetworkResult(result: NetworkResult) {
    when (result) {
        is NetworkResult.Success -> println("Data: ${result.data}")
        is NetworkResult.Error -> println("Error: ${result.error.message}")
        NetworkResult.Loading -> println("Loading...")
    }
}
```

*Example using covariant generic type parameter*
```kotlin
sealed class ApiResult <out T> {
    data class Success<out R>(val data: R?): ApiResult<R>()
    data class Error(val message: String): ApiResult<Nothing>()
    object Loading: ApiResult<Nothing>()
}
```



## Kotlin Flows
*Kotlin Flow is a reactive streams library designed for handling streams of data asynchronously. It provides a type-safe way of working with asynchronous data streams and is built on top of coroutines.*

*Basic example: fetching data*
```kotlin
fun fetchData(): Flow<String> = flow {
    val data = // Fetch data from network
    emit(data)
}

fun main() = runBlocking {
    fetchData().collect { data ->
        println("Received data: $data")
    }
}
```

*Example of the producer end of a Flow. In this case, it is a `callbackFlow` that exposes data received by a socket with `trySend()`*
```kotlin
private fun initSocket(socket: Socket?, config: Map<String, Any>): Flow<SpeechData> = callbackFlow {
    socket?.connect()
    socket?.emit("startGoogleCloudStream", Gson().toJson(config))

    socket?.on("speechData") { args ->
        Gson().fromJson(args[0].toString(), SpeechData::class.java).let {
            println("Object speechData: $it")
            trySend(it)
        }
    }

    awaitClose {
        socket?.emit("endGoogleCloudStream")
        socket?.off("speechData")
        socket?.disconnect()
    }
}
```

*Example of the consumer end of a Flow. In this case, a `ViewModel` collecting data from a `callbackFlow` and posting all consumed values to `LiveData`*
```kotlin
viewModelScope.launch(context = Dispatchers.IO) {
    repository.connectSubject(langSubject.value!!.language, langObject.value!!.language).collect {
        Timber.d("speakerCurr: $speakerCurr")
        if (it.isFinal) {
            builderSubject.append(it.data)
            translationSubject.postValue(builderSubject.toString())
        } else {
            translationSubject.postValue(builderSubject.toString() + it.data)
        }
    }
}
```


#### SharedFlow
*A hot Flow that provides event cache and buffer features. Suitable for performance-relevant implementations involving many collectors*
- *New collectors can retrieve some of the latest events from the cache and collect new ones as they are emitted*
- *buffer behavior can be defined to provide events to slower consumers without suspending emitters*

#### StateFlow
*A simple Flow for managing state. Supports "current value" behavior*. 
- Extends`SharedFlow`
- hot
```kotlin
fun main() = runBlocking {
    val stateFlow = MutableStateFlow(0)

    launch {
        stateFlow.collect { value ->
            println("StateFlow value: $value")
        }
    }

    stateFlow.value = 1
    stateFlow.value = 2
    println("${stateFlow.value}")
}
```


## Passing data between Activities
*Activities can **share** data with a `ViewModel`, but passing data to one another implies a kind of independence, or implies something more akin to messaging or startup with configuration.

*For messaging, i'm not too familiar with any pattern that is commonly used. I'm not sure inter-Activity messaging really has many valid use-cases. I suppose you could use something like an event bus. But this all seems like an anti-Pattern.*

*For passing initialization data to a new Activity, the best way to do this is to put the data in  `Intent` extras, using `@Parcelize` for complex data, including the extras in your call to `startActivity()`, and grabbing the extras in the other Activity with `getExtras()`:*
```kotlin
// Create a Parcelable data class
@Parcelize
data class User(val name: String, val age: Int) : Parcelable

// In the sending Activity
val user = User("Alice", 25)
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("user_key", user)
startActivity(intent)

// In the receiving Activity
val user = intent.getParcelableExtra<User>("user_key")
```


## `lazy`, `lateinit` delegates

##### `lazy`
- *Used to initialize a property only when it is accessed for the first time. Useful for expensive initializations that you don't want to perform unless necessary, and if so only want to perform once.*
- *Thread safe*
```kotlin
val property: Type by lazy {
    // Initialization code
    Type()
}
```

##### `lateinit`
- *Used only with non-null `var` types.*
- *Used when you can't initialize your property when it is declared and don't want it to be nullable.*
- *Examples:*
	- ***Dependency Injection**: Commonly used in Dagger where dependencies are injected after an object's creation.*
	- ***UI Components**: Useful in Android for initializing views after the view hierarchy is created.*
```kotlin
lateinit var message: String

fun setup() {
    message = "Hello, Kotlin!"
}

fun main() {
    setup()
    println(message) // Prints: Hello, Kotlin!
}
```


## Kotlin Multiplatform
- *What it is is self-evident. Its a way to share core/common code (often business logic)*
- *It compiles down to the appropriate bytecode for the platforms you are supporting*
	- *Javas*
	- *JavaScript*
	- *Native (iOS, macOS, Linux, Windows)*
- *Uses something kinda like an abstract/concrete pattern to define common properties and functions and provide their platform-specific implementations:*

##### Example: project structure
```kotlin
|-- my-multiplatform-project
    |-- commonMain
        |-- kotlin
            |-- commonCode.kt
    |-- androidMain
        |-- kotlin
            |-- androidCode.kt
    |-- iosMain
        |-- kotlin
            |-- iosCode.kt
```

##### Example: common code
```kotlin
// commonMain/kotlin/Greeting.kt
expect fun platformName(): String

class Greeting {
    fun greet(): String = "Hello, ${platformName()}!"
}
```

##### Example: Android code
```kotlin
// androidMain/kotlin/Greeting.kt
actual fun platformName(): String = "Android"
```

##### Example: iOS code
```kotlin
// iosMain/kotlin/Greeting.kt
actual fun platformName(): String = "iOS"
```


## Services
*A component for performing (often long-running) background operations independently from the main application itself. They don't intrinsically have a UI, but they can sometimes be associated with custom notification-based UIs (like playback controls, or background download progress indicators).*

*If you need some kind of processing to persist through app closure, or ensure processing occurs when the app is garbage collected or backgrounded, you almost always use some kind of `Service`*

##### Foreground Service
*Runs in the foreground and is noticeable to the user because it always displays a notification. Its used for tasks that the user is actively aware of and can benefit from updates on.*

*Example use cases*
- *Playback controls*
- *Progress indicators*

*Example implementation*
```kotlin
val notification = createNotification()
startForegroundService(Intent(this, MyService::class.java).apply {
    putExtra("inputExtra", "Foreground Service Example")
})
startForeground(1, notification)
```

##### Background Service
*Runs in the background without any user interaction.*

*Example use cases*
- *Syncing data (where the user doesn't need to know about it)* 
- *Other network operations*
- *Database operations*

*Example implementation*
```kotlin
override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
    // Perform long-running task in background thread
    return START_NOT_STICKY
}
```

##### Bound Service
*Provides an interface that allows components like Activities to bind to and interact with it.*

*Example: long running, high volume file syncing operation that supports Pause and Resume functionality*
```kotlin
private val binder = LocalBinder()
inner class LocalBinder : Binder() {
    fun getService(): MyService = this@MyService
}

override fun onBind(intent: Intent): IBinder {
    return binder
}
```







### Jetpack Compose
What are side effects in jetpack compose?
What is disposable effect?
Suppose if I want to set 3 elements in a row in a grid in compose, what would you use?
App crashes when we use lazyColumn in column how would you solve this issue?
How to improve the performance of recyclerView, lazyColumn?
What is LaunchedEffect? (Gave a scenario and i had to tell him the solution for it. The answer was via a launchedeffect)


### Coroutines
Coroutines in Kotlin.
What is difference between kotlin coroutines and threads in Java.
Different types of coroutine scopes.
How to switch context in coroutines?
What is Dispatcher in coroutine and types of Dispatcher.
Difference between async and launch?


What is dagger hilt?
What is @provides, @module?
What is singleton?



What is service & types of service.

How to handle circular dependency in multi module app and how can we avoid it.
How to manage versions in multi module apps - version catalogs.

Some questions on my app projects and how i implemented them.
