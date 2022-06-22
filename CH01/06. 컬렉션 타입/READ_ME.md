 ## 컬렉션 타입이란?

 컬렉션 타입은 데이터들의 집합 묶음

 ## Array

 데이터 타입의 값들을 순서대로 저장하는 리스트

 ## Dictionary

순서없이 키(key)와 값(value) 한 쌍으로 데이터를 저장하는 컬렉션 타입

## Set

같은 데이터 타입의 값을 순서없이 저장하는 리스트

## Tuples

[Types](docs.swift.org/swift-book/ReferenceManual/Types.html)

```swift
let someTuple: (Double, Double) = (3.14159, 2.71828)
```

## Types

- *type* -> function-type
- *type* -> array-type
- *type* -> dictionary-type
- *type* -> type-identifier
- *type* -> tuple-type
- *type* -> optional-type
- *type* -> implicitly-unwrapped-optional-type
- *type* -> protocol-compostion-type
- *type* -> opaque-type
- *type* -> metatype-type
- *type* -> any-type
- *type* -> self-type
- *type* -> ( type )

## Grammar of Function Types

- *function-type* → attributes<sub>opt</sub> function-type-argument-clause **async**<sub>opt</sub> **throws**<sub>opt</sub> -> type
- *function-type-argument-clause* → ( )
- *function-type-argument-clause* → ( function-type-argument-list **...**<sub>opt</sub>)
- *function-type-argument-list* → function-type-argument | function-type-argument, function-type-argument-list
- *function-type-argument* → attributes<sub>opt</sub> **inout**<sub>opt</sub> type | argument-label type-annotation
- *argument-label* → identifier

## In-Out Paramters

<p>Function paramters are contants by default. Trying to change the value of a function<br>
paramter from within the body of that function results in a compile error. This means<br>
that you can't change the value of a parameter by mistake. If you want a function to modify a<br>
parameter's value, and you want those changes to persist after the funciton call has ended,<br>
define that parameter as an <strong><em>in-out parameter</em></strong> instead.</p>

<p>You can only pass a variable as the argument for in-out paramter. You can't pass a<br>
constant or a literal value as the argument, because constants and literals can't be modified.<br>
You place an ampersand(&) directly before a variable's name when you pass it as an<br>
argument to an in-out paramter, to indicate that it can be modified by the function</p>

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

## Throwing Functions and Methods

<p>Functions and methods that can throw an error must be marked with the <code>throws</code> keyword.<br>
These functions and methods are know as <strong><em>throwing functions</em></strong> and <strong><em>throwing methods</em></strong>. They<br>
have the following form:</p>

```swift
func ${function name}(${parameters}) throws -> ${return type} {
    ${statements}
}
```
<p>Calls to a throwing funciton or method must be wrapped in a <code>try</code> or <code>try!</code> expression (that is,<br>
in the scope of a <code>try</code> or <code>try!</code> operator).</p>

## Rethrowing Functions and Methods

<p>A function or method can be declared with the <code>rethrows</code> keyword to indicate that it throws an<br>
error only if one of its function paramters throws and error. These functions and methods are<br>
known as <em><strong>rethrowing functions</strong></em> and <em><strong>rethrowing methods</strong></em>. Rethrowing functions and methods<br>
must have at least one throwing function parameter.</p>

```swift
func someFunction(callback: () throws -> Void) rethrows {
    try callback()
}
```

<p>A rethrowing function or method can contain a <code>throw</code> statement only inside a <code>catch</code> clause.<br>
This lets you call the throwing function inside a <code>do-catch</code> statement and handle errors in the<br>
<code>catch</code> clause by throwing a different error. In addition, the <code>catch</code> clause must handle only<br>
errors thrown by one of the rethrowing function's throwing paremeters. For example, the<br>
following is invalid because the <code>catch</code> clause would handle the error thrown by<br>
<code>alwaysThrows()</code>.</p>

```swift
func alwaysThrows() throws {
    throw SomeError.error
}

func someFunction(callback: () throws -> Void) rethrows {
    do {
        try callback()
        try alwaysThrows() // Invalid, alwaysThrows isn't a throwing parameter
    } catch {
        throws AnotherError.error
    }
}
```

<p> A throwing method can't override a rethrowing method, and a throwing method can't satisfy<br>
a protocol requirement for a rethrrowing method. That said, a rethrowing method can override<br>
a throwing method, and a rethrowing method can safisfy a protocol requirement for a<br>
throwing method </p>

## Escaping Closures vs Non-Escaping Closure

<p>A closure is said to "escape" a function when it's called after after that funciton returns.<br>
Closures are non-escaping by default</p>

```swift
var closuresArray: [() -> Void] = []

func doClosures(completion: () -> Void) {
    completionHandlers.append(completion)
}

//ERROR!!!
passsing non-escaping parameter 'completion' to funciton expecting an @escaping closure
```

<p>An escaping closure grants the ability of the closure to outlive the function and can be<br>
stored elsewhere. By using escape closure, the closure will have existence in memory until<br>
all of it's content have been executed. To implement escaping closure, all we have to do is<br>
put <strong>@escaping</strong> in front of our closure.</p>

```swift
class ClosureTesting {
    var holder: ((String) -> Void)?

    func getUser(completion: @escaping (String) -> Void) {
        holder = completion
    }
}
```

## Restrictions for Nonescaping Closures

<p>A parameter that's a nonescaping function can't be stored in a property, variable, or constant<br>
of type <code>Any</code>, becuase that might allow the value to escape.<p>

<p>A parameter that's a nonescaping function can't be passed as an argument to another<br>
nonescaping funciton paramter. This restriction helps Swift perform more of its checks for<br>
conflicting access to memory at compile time instead of at runtime. For example:</p>

```swift
let external: (() -> Void) -> Void = { _ in 
    () 
}

func takesTwoFunctions(first: (() -> Void) -> Void, second: (() -> Void) -> Void) {
    first { first {} }      // Error
    second { second {} }    // Error

    first { second {} }     // Error
    second { first {} }     // Error

    first { external {} }   // OK
    external { first {} }   // OK
}
```

<p>In the code above, both of the paramters to <code>takesTwoFunctions(first:second:)</code> are<br>
functions. Neither parameter is marked <code>@escaping</code>, so they're both nonescaping as a result.</p>

<p>The four funcion calls marked "Error" in the example above cause compiler erros. Because<br>
the <code>first</code> and <code>second</code> parameters are nonescaping functions, they can't be passed as<br>
arguments to another nonescaping function parameter. In contrast, the two function calls<br>
marked "OK" don't cause a compiler error. These function calls don't violate the restriction<br>
because <code>external</code> sin't one of the parameters of <code>takesTwoFunctions(first:second:)</code>.</p>

<p>If you need to avoid this restriction, mark one of the parameters as escaping, or temporaily<br>
convert one of the nonescaping function parameters to an escaping function by using the<br>
<code>withoutActuallyEscaping(_:do:)</code> function.</p>

## Array Type

<p>The Swift language provides the following syntactic sugar for the Swift standard library<br>
<code>Array&lt;Element&gt;</code> type:</p>

```swift
[${type}]
```

<p>In other words, the following two declarations are equivalent:</p>

```swift
let someArray: Array<String> = ["Alex", "Brian", "Dave"]
let someArray: [String] = ["Alex", "Brain", "Dave"]
```

<p>In both cases, the constant <code>someArray</code> is declared as an array of strings. The elements of an<br>
array can be accessed through subscripting by specifying a valid index value in square<br>
brackets: <code>someArray[0]</code> refers to the element at index 0, <code>"Alex"</code></p>

<p>You can create multidimensional arrays by nesting paris of square brackets, where ther name<br>
of the base type of the elemetns is contained in the innermost pair of square brackets. For<br>
example, you can create a three-dimensional array of integers using three sets of square<br>
brackets:</p>

```swift
var array3D: [[[Int]]] = [[[1,2], [3,4]], [[5,6], [7,8]]]
```

<p>When accessing the elements in a multidimensional array, the left-most subscript index<br>
refers to the element at the index in the outermost array. The next subscript index to the<br>
right refers to the element at the index in the array that's nested one level in. And so on.<br>
This means that in the example above, <code>array3D[0]</code> refers to <code>[[1,2],[3,4]]</code>,<br>
<code>array3D[0][1]</code> refers to <code>[3,4]</code>, and <code>array3D[0][1][1]</code> refers to the value 4.</p>

## Dictionary Type

<p>The Swift language provides the following syntactic sugar for the Swift standard library<br>
<code>Dictionary&lt;Key, Value&gt;</code> type:</p>

```swift
[${key type}:${value type}]
```

In other words, the following two declarations are equivalent:

```swift
let someDictionary: [String: Int] = ["Alex": 31, "Paul": 39]
let someDictionary: Dictionary<String, Int> = ["Alex": 31, "Paul": 39]
```

<p>In both cases, the constant <code>someDictionary</code> is declared as a dictionary with strings as keys<br>
and integers as values.</p>

<p>The values of a dictionary can be accessed through subscripting by specifying the<br>
corresponding key in sqaure brackets: <code>someDictionary["Alex"]</code> refers to the value<br>
associated with the key <code>"Alex"</code>. The subscript returns an optional value of the dictionary's<br>
value type. If the specified key isn't contained in teh dictionary, the subscript returns <code>nil</code>.</p>

<p>The key type of a dictionary must conform to the Swift standard library <code>Hashable</code> protocol.</p>

## Sets

<p><strong>A set</strong> stores distinct values of the same type in a collection with no defined ordering. You can<br>
use a set instead of an array when the order of items isn't important, or when you need to<br>
ensure that an item only appears once.</p>

<p>A type must be <strong><emp>hashable</emp></strong> in order to be stored in a set -- that is, the type must provide a way<br>
to compute a <strong><emp>hash value</emp></strong> for itself. A hash value is an <code>Int</code> value that's the same for all objects<br>
that compare equally, such that if <code>a == b</code>, the hash value of <code>a</code> is equal to hash value of <code>b</code></p>

<p>All of Swift's basic types (such as <code>String</code>, <code>Int</code>, <code>Double</code>, and <code>Bool</code>) are hashable by default,<br>
and can be used as aset value types or dictionary key types. Enumeration case values<br>
without associated values (as described in Enumeration) are also hashable by default.</p>

<p>The type of a Swift set is written as <code>Set&lt;Element&gt;</code>, where <code>Element</code> is the type that the set is<br>
allowed to store. Unlike arrays, sets don't have an equivalent shorthand form.</p>

<p>You can create an empty set of a certain type using initializer syntax:</p>

```swift
var letters = Set<Character>()
print("letters is of type Set<Character> with \(letters.counter) items.")
// Prints "letters is of type Set<Character> with 0 items."
```

<p>Alternatively, if the context already provides type information, such as a function argument or<br>
an already typed variable or constant, you can create an empty set with an empty array<br>
literal:</p>

```swift
letters.insert("a")
// letters now contains 1 value of type Character
letters = []
// letters is now an empty set, but is still of type Set<Character>
```

<p>You can also initialize a set with an array literal, as a shorthand way to write one or more<br>
values as a set collection.</p>

<p>The example below creates a set called <code>favoriteGenres</code> to store <code>String</code> values:</p>

```swift
var favoriteGenres: Set<string> = ["Rocker", "Classical", "Hip Hop"]
```

<p>The <code>favorite Genres</code> variable is declared as "a set of <code>String</code> values", written as <code>Set&lt;string&gt;</code>.<br>
Because this particular set has specified a value type of <code>String</code>, it's <strong><emp>only</emp><strong> allowed to store<br>
<code>String</code> values. Here, the <code>favoriteGeneres</code> set is initialized with thre <code>String</code> values (<code>"Rock"</code>,<br>
<code>"Classical"</code>, and <code>"Hip hop"</code>), written within an array literal.</p>

<p>A set type can't be inferred from an array literal alone, so the type <code>Set</code> must be explicitly<br>
declared. However, because of Swift's type inference, you don't have to write the type of the<br>
set's elements if you're initializing it with an array literal that contains values of just one type.<br>
The initialization of <code>favoriteGenres</code> could have been written in a shorter form instead:</p>

```swift
var favoriteGenres: Set = ["Rocker", "Classical", "Hip Hop"]
```

Because all values in the array literal are of the same type, Swift can infer that <code>Set&lt;String&gt;</code><br>
is the correct type to use for the <code>favoriteGenres</code> variables

<p>You access and modify a set through its method and properties.</p>

<p>To find out the number of items in a set, check its read-only <code>count</code> property:

```swift
print("I have \(favoriteGenres.count) favorite music genres.")
// Prints "I have 3 favorite music genres."
```

<p>Use the Boolean <code>isEmpty</code> property as a shortcut for checking whther the <code>count</code> property is<br>
equal to <code>0</code>:

```swift
if favoriteGenres.isEmpty {
    print("As far as music goes, I'm not picky.")
} else {
    print("I have particular music preferences.")
}
// Prints "I have particular music preferences."
```

<p>You can add a new item into a set by calling the set's <code>insert(_:)</code> method:</p>

```swift
favoriteGenres.insert("Jazz")
// favoriteGenres now contains 4 items
```

<p>You can remove an item from a set by calling the set's <code>remove(_:)</code> methods, which removes<br>
the item if it's a member of the set, and returns the removed value, or return <code>nil</code> if the set<br>
didn't contain it. Alternatively, all items in a set can be removed with its <code>removeAll()</code> method.</p>

```swift
if let removedGenre = favoriteGenres.remove("Rock") {
    print("\(removedGenre)? I'm over it.")
} else {
    print("I never much cared for that.")
}
// Prints "Rock? I'm over it."
```

<p>To check whether a set contains a particular item, use the <code>contains(_:)</code> method.</p>

```swift
if favoriteGenres.contains("Funk") {
    print("I get up on the good foot.")
} else {
    print("It's too funky in here.")
}
// Prints "It's too funky in here."
```

<p>You can iterate over the values in a set with a <code>for-in</code> loop.</p>

```swift
for genre in favoriteGenres {
    print("\(genre)")
}
// Classical
// Jazz
// Hip hop
```

<p>Swift's <code>Set</code> type doesn't have a defined ordering. To iterate over the values of a set in a<br>
specific order, use the <code>sorted()</cdoe> method, which returns the set's elements as a array sorted<br>
using the < operator.</p>

```swift
for genre in favoriteGenres.sorted() {
    print("\(genre)")
}
// Classical
// Hip hop
// Jazz
```