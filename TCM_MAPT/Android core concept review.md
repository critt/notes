### Kotlin vs Java
- *Kotlin compiles down to Java bytecode and runs on the JVM.*
- *Kotlin has modern features like coroutines and extension functions.*
- *Kotlin has inbuilt null safety--Java does not.*
- *Java requires writing much more boilerplate and its Syntax is much more verbose than Kotlin's*


### Scope functions in Kotlin:
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



### Kotlin generics
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



### Kotlin classes
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







Coroutines in Kotlin.
What is difference between kotlin coroutines and threads in Java.
Different types of coroutine scopes.
How to switch context in coroutines?
What is Dispatcher in coroutine and types of Dispatcher.
Difference between async and launch?

What is MVVM ?
MVVM vs MVC vs MVI. Which is better for what case and why?
What is a ViewModel?
Do we write business logic in the ViewModel?

What is Flow?
Difference between StateFlow and SharedFlow.
How to collect the latest value in the StateFlow in activity?

How to pass data between activities?

What is dagger hilt?
What is @provides, @module?
What is singleton?

What is lazy, lateinit?

What are side effects in jetpack compose?

App crashes when we use lazyColumn in column how would you solve this issue?

What is disposable effect?

Suppose if I want to set 3 elements in a row in a grid in compose, what would you use?

What is intent? Why do we use it?

What is service & types of service.

Why does android app lag?


How to improve the performance of recyclerView, lazyColumn?

Explain kotlin multiplatform

What is application context? Difference between it and getContext



What is LaunchedEffect? (Gave a scenario and i had to tell him the solution for it. The answer was via a launchedeffect)

How to handle circular dependency in multi module app and how can we avoid it.

Difference between Bluetooth and Bluetooth low energy.

How to send data from one app to another. - intents



How to manage versions in multi module apps - version catalogs.

Some questions on my app projects and how i implemented them.
