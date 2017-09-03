# Bixlabs Kotlin Styleguide and rules

This page contains the current coding style for the Kotlin language and some rules we use to encourage a healthy development environment.

## Naming Style

If in doubt, default to the Java Coding Conventions such as:

* Hungarian notation is forbidden and I'll break your arms if I see you using it.
* Use of camelCase for names (and avoid underscore in names unless for constants). 
* Types start with upper case
* Methods and properties start with lower case
* Use 4 space indentation
* Public functions should have documentation such that it appears in Kotlin Doc
* Constants use uppercase letters with each word separated by an underscore: eg TXT_FILE_EXT
* Classes/enums/objects use the title case naming scheme (well known abbreviations are permitted) and use singular nouns: eg StringFormatter, NOT StringFormatters
* Avoid using reserved keywords for names

## Colon

There is a space before colon where colon separates type and supertype and there's no space where colon separates instance and type:

```kotlin
interface Foo<out T : Any> : Bar {
    fun foo(a: Int): T
}
```

## Lambdas

In lambda expressions, spaces should be used around the curly braces, as well as around the arrow which separates the parameters from the body. Whenever possible, a lambda should be passed outside of parentheses.

```kotlin
list.filter { it > 10 }.map { element -> element * 2 }
```

In lambdas which are short and not nested, use the `it` convention instead of declaring the parameter explicitly. In nested lambdas with parameters, parameters should be always declared explicitly.

## Class header formatting

Classes with a few arguments can be written in a single line:

```kotlin
class Person(id: Int, name: String)
```

Classes with longer headers should be formatted so that each primary constructor argument is in a separate line with indentation. Also, the closing parenthesis should be on a new line. If we use inheritance, then the superclass constructor call or list of implemented interfaces should be located on the same line as the parenthesis:

```kotlin
class Person(
    id: Int, 
    name: String,
    surname: String
) : Human(id, name) {
    // ...
}
```

For multiple interfaces, the superclass constructor call should be located first and then each interface should be located in a different line:

```kotlin
class Person(
    id: Int, 
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker {
    // ...
}
```

Constructor parameters can use either the regular indent or the continuation indent (double the regular indent).

Keep defined classes closed (omit the use of the **open** keyword) unless there is a good reason to open up a class and allow inheritance. Avoid API polution please.

## Unit

If a function returns Unit, the return type should be omitted:

```kotlin
fun foo() { // ": Unit" is omitted here

}
```

## Functions vs Properties

In some cases functions with no arguments might be interchangeable with read-only properties. Although the semantics are similar, there are some stylistic conventions on when to prefer one to another.

Prefer a property over a function when the underlying algorithm:

* does not throw any Exceptions
* has a `O(1)` complexity
* is cheap to calculate (or caсhed on the first run)
* returns the same result over invocations

## Annotations

Since v1.0.6, Kotlin compiler does insert `@Nullable` and `@NotNull` from `org.jetbrains.annotations` in your code, so there's no need to manually add them to your methods. Nullability in Kotlin is inferred through the usage of `?` and optionals.


## apply

Use `apply` for initialization instead or repeating the same variable name over and over again.

```kotlin
val foo = createBar().apply {
    property = value
    init()
}
```

## also

Use `also` over `apply` if the receiver is used for anything other than setting properties or function calls on it.

```kotlin
class Baz {
    var currentBar: Bar?
    val observable: Observable

    val foo = createBar().also {
        currentBar = it
        observable.registerCallback(it)
    }
}
```

Prefer `also` over `apply` if there already are multiple receivers in scope, especially if you make calls on any outer receivers.

```kotlin
class Foo {
    fun Bar.baz() {
        val stuff = callSomething().also {
            it.init()
            this@baz.registerCallback(it)
        }
    }
}
```

example: Using also for a "singleton with parameter" (from Google's architecture components sample code):

```kotlin
class UsersDatabase : RoomDatabase() {

    companion object {

        @Volatile private var INSTANCE: UsersDatabase? = null

        fun getInstance(context: Context): UsersDatabase =
                INSTANCE ?: synchronized(this) {
                    INSTANCE ?: buildDatabase(context).also { INSTANCE = it }
                }

        private fun buildDatabase(context: Context) =
                Room.databaseBuilder(context.applicationContext,
                        UsersDatabase::class.java, "Sample.db")
                        .build()
    }
}
```

## apply/with/run

Prefer `apply/run` over `with` if the receiver is nullable.

```kotlin
getNullable()?.run {
    init()
}

getNullable()?.apply {
    init()
}
```

Prefer `run/with` over apply if the returned value is not used.

```koltin
view.run {
    textView.text = "Hello World"
    progressBar.init()
}

with(view) {
    textView.text = "Hello World"
    progressBar.init()
}
```

Choose one of the above and **use it consistently**.

## let

Prefer `let` over `run` in method chains that transform the receiver

```kotlin
val baz: Baz = foo.let { createBar(it) }.convertBarToBaz()
// or with function references
val baz: Baz = foo.let(::createBar).convertBarToBaz()
```

## Array Initialization 
Use the `arrayOf()` initializer without specifying the `<>` if the array type is already defined.

```kotlin
val foo: List<String> = arrayOf("a", "b", "c")
```

Prefer inmmutable `collection`s over mutable ones whenever you know the `collection` will not be mutated.

## When
Use `when` in case there are two or more branches of if-else

```kotlin
when {
    foo > 10 -> print("10")
    foo > 5 -> print("0")
    foo > 0 -> print("0")
    else -> print("else")
}
```

Use `is` in case you are comparing a class type

```kotlin
val hoge: Hoge = Hoge()
when (hoge) {
    is Hoge -> {

    }
    else -> {
       
    }
}
```

Use `range` in case you are comparing `Int` values

```kotlin
val hoge = 10
when (hoge) {
    in 0..4 -> print("0..4")
    in 5..10 -> print("5..10")
}
```

## Don't use `!!`

Don't use `!!`, it will simply erase the benefits of Kotlin (Null-safety wise).
You use it only when you want to explicitly raise a Null Pointer Exception or when you are **ABSOLUTELY** sure there's no chance of nullity (that means never).

Use a scope function in case of checking a `null` value

```kotlin
class Hoge {
    fun fun1() {}
    fun fun2() {}
    fun fun3() = true
}

var hoge: Hoge? = null

hoge?.run { 
   fun1()
} ?: run {
    hoge = Hoge().apply {
        fun1()
        fun2()
    }
}
```
