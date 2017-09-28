### **Chapter 2. Program Structure**

This chapter provides details about the basic structural elements of a Go program.

### Names

In Go, the names of functions, variables, constants, types, statement labels, and packages follow a simple rule: a name begins with a letter (anything that Unicode deems a letter) or an underscore and may have any number of additional letters, digits, and underscores. The names are case-sensitive: `heapSort` and `Heapsort` are different names.

Go has 25 keywords like `if` and `switch` that may be used only where the syntax permits; they can't be used as names.

 | | |
-|-|-|-
`break`|`default`|`func`|`interface`|`select`
`case`|`defer`|`go`|`map`|`struct`
`chan`|`else`|`goto`|`package`|`switch`
`const`|`fallthrough`|`if`|`range`|`type`
`continue`|`for`|`import`|`return`|`var`

In addition, there are about three dozen *predeclared* names like `int` and `true` for built-in constants, types, and functions:

 |
-|-
Constants: | `true` `false` `iota` `nil`
Types: | `int` `int8` `int16` `int32` `int64` `uint` `uint8` `uint16` `uint32` `uint64` `uintptr` `float32` `float64` `complex128` `complex64` `bool` `byte` `rune` `string` `error`
Functions: | `make` `len` `cap` `new` `append` `copy` `close` `delete` `complex` `real` `imag` `panic` `recover`

These names are not reserved, so you may use them in declarations. Beware of the potential for confusion.

#### Local and exported names *

If an entity is:

* Declared within a function: it is *local* to that function.
* Declared outside of a function: it is visible in all files of the package to which it belongs.

The case of the first letter of a name determines its visibility across package boundaries:

* If the name begins with an upper-case letter, it is *exported* (visible and accessible outside of its own package and may be referred to by other parts of the program), as with `Printf` in the `fmt` package.
* Package names themselves are always in lower case.

#### Naming convention and style *

* There is no limit on name length;
* Short names are preferred, especially for local variables with small scopes (`i` is better than `theLoopIndex`);
* The larger the scope of a name, the longer and more meaningful it should be.
* [Camel case](https://en.wikipedia.org/wiki/CamelCase) are used when forming names by combining words, e.g. `QuoteRuneToASCII` and `parseRequestLine` (instead of `quote_rune_to_ASCII` or `parse_request_line`).
* The letters of acronyms and initialisms like `ASCII` and `HTML` are always rendered in the same case, so a function might be called `htmlEscape`, `HTMLEscape`, or `escapeHTML`, but not `escapeHtml`.

### Declarations

A **declaration** names a program entity and specifies its properties. There are four major kinds of declarations:

1. `var` (variables)
2. `const` (constants)
3. `type` (types)
4. `func` (functions)

This chapter discusses variables and types. Constants are disucssed in [Chapter 3](ch3.md), and functions in [Chapter 5](ch5.md).

Each `.go` file has declarations in the following order:

1. A `package` declaration that says what package the file is part of.
2. Any `import` declarations
3. A sequence of package-level declarations of types, variables, constants, and functions, in any order.

For example, the following program declares a constant, a function, and a couple of variables:

<small>[gopl.io/ch2/boiling/main.go](https://github.com/shichao-an/gopl.io/blob/master/ch2/boiling/main.go)</small>

```go
// Boiling prints the boiling point of water.
package main

import "fmt"

const boilingF = 212.0

func main() {
	var f = boilingF
	var c = (f - 32) * 5 / 9
	fmt.Printf("boiling point = %g°F or %g°C\n", f, c)
	// Output:
	// boiling point = 212°F or 100°C
}
```

* Each package-level name is visible in all the files of the package.
* Local declarations are visible only within the function in which they are declared.

A function declaration has a name, a list of parameters an optional list of results (omitted if the function does not return anything), and the function body.

### Variables

A `var` declaration creates a variable of a particular type, attaches a name to it, and sets its initial value, with the general form:

```go
var name type = expression
```

<u>Either the type or the `= expression` part may be omitted, but not both:</u>

* If the type is omitted, it is determined by the initializer expression.
* If the expression is omitted, the initial value is the **zero value** for the type, which is:
    * 0 for numbers
    * `false` for booleans
    * `""` for strings
    * `nil` for interfaces and reference types (slice, pointer, map, channel, function).

The zero value of an aggregate type like an array or a struct has the zero value of all of its elements or fields.

The zero-value mechanism ensures that a variable always holds a well-defined value of its type; in Go there is no such thing as an uninitialized variable. This simplifies code and often ensures sensible behavior of boundary conditions without extra work. For example,

```go
var s string
fmt.Println(s) // ""
```

Go programmers often go to some effort to make the zero value of a more complicated type meaningful, so that variables begin life in a useful state.

It is possible to declare and optionally initialize a set of variables in a single declaration, with a matching list of expressions. Omitting the type allows declaration of multiple variables of different types:

```go
var i, j, k int // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```

* Initializers may be literal values or arbitrary expressions.
* Package-level variables are initialized before `main` begins ([Section 2.6.2](#package-initialization)).
* Local variables are initialized as their declarations are encountered during function execution.

A set of variables can also be initialized by calling a function that returns multiple values:

```go
var f, err = os.Open(name) // os.Open returns a file and an error
```

#### Short Variable Declarations

Within a function, an alternate form called a short variable declaration may be used to declare and initialize local variables. It takes the form `name := expression`, and the type of name is determined by the type of expression. For exmaple ([Animated GIFs](ch1.md#animated-gifs)),

##### **When to use short variable declaration and `var` declaration** *

* Short variable declarations are used to declare and initialize the majority of local variables, for brevity and flexibility.
* A `var` declaration tends to be reserved for:
    * Local variables that need an explicit type that differs from that of the initializer expression;
    * Local variables when they will be assigned a value later and its initial value is unimportant.

```go
i := 100 // an int
var boiling float64 = 100 // a float64
var names []string
var err error
var p Point
```

As with `var` declarations, multiple variables may be declared and initialized in the same short variable declaration:

```go
i, j := 0, 1
```

However, declarations with multiple initializer expressions should be used only when they help readability, such as for short and natural groupings like the initialization part of a for loop.

Keep in mind that `:=` is a declaration, whereas `=` is an assignment. A multi-variable declaration should not be confused with a [tuple assignment](#tuple-assignment), in which each variable on the left-hand side is assigned the corresponding value from the right-hand side:

```go
i, j = j, i // swap values of i and j
```

Like `var` declarations, short variable declarations may be used for calls to functions like `os.Open` that return two or more values:

```go
f, err := os.Open(name)
	if err != nil {
		return err
}
// ...use f...
f.Close()
```

##### **One subtle but important point** *

A short variable declaration does not necessarily declare all the variables on its left-hand side. If some of them were already declared in the same lexical block , then the short variable declaration acts like an assignment to those variables. For example,

```go
in, err := os.Open(infile)
// ...
out, err := os.Create(outfile)
```

In the above code:

* The first statement declares both `in` and `err`.
* The second declares `out` but only assigns a value to the existing `err` variable.

A short variable declaration must declare at least one new variable. Therefore, the following code will not compile:

```go
f, err := os.Open(infile)
// ...
f, err := os.Create(outfile) // compile error: no new variables
```

The fix is to use an ordinary assignment for the second statement.

A short variable declaration acts like an assignment only to variables that were already declared in the same lexical block; declarations in an outer block are ignored.

#### Pointers

A **variable** is a piece of storage containing a value.

* Variables created by declarations are identified by a name, such as `x`
* Many other variables are identified only by expressions like `x[i]` or `x.f`.

All these expressions read the value of a variable, except when they appear on the lefthand side of an assignment, in which case a new value is assigned to the variable.

A **pointer** value is the address of a variable. A pointer is thus the location at which a value is stored. Not every value has an address, but every variable does. With a pointer, we can read or update the value of a variable indirectly, without using or even knowing the name of the variable, if indeed it has a name.

##### **Pointer type (`*type`) and address-of (`&`) operators** *

If a variable is declared `var x int`, the expression `&x` ("address of `x`") yields a pointer to an integer variable (a value of type `*int`, which is pronounced "pointer to `int`"). If this value is called `p`, we say "`p` points to `x`", or equivalently "`p` contains the address of `x`". The variable to which `p` points is written `*p`. The expression `*p` yields the value of that variable, an `int`, but since `*p` denotes a variable, it may also appear on the left-hand side of an assignment, in which case the assignment updates the variable.

```go
x := 1
p := &x          // p, of type *int, points to x
fmt.Println(*p)  // "1"
*p = 2           // equivalent to x = 2
fmt.Println(x)   // "2"
```

Each component of a variable of aggregate type: a field of a struct or an element of an array, is also a variable and thus has an address too.

Variables are sometimes described as *addressable* values. Expressions that denote variables are the only expressions to which the *address-of* operator `&` may be applied.

##### **Comparing pointers** *

The zero value for a pointer of any type is `nil`. The test `p != nil` is true if `p` points to a variable. Pointers are comparable; two pointers are equal if and only if they point to the same variable or both are `nil`.

```go
var x, y int
fmt.Println(&x == &x, &x == &y, &x == nil) // "true false false"
```

<u>It is perfectly safe for a function to return the address of a local variable.</u> For example:

```go
var p = f()

func f() *int {
	v := 1
	return &v
}
```

<u>The local variable `v` created by this particular call to `f` will remain in existence even after the call has returned, and the pointer `p` will still refer to it.</u> Each call of `f` returns a distinct value:

```go
fmt.Println(f() == f()) // "false"
```

Passing a pointer argument to a function makes it possible for the function to update the variable that was indirectly passed. For example:

```go
func incr(p *int) int {
	*p++ // increments what p points to; does not change p
	return *p
}

v := 1
incr(&v)
// side effect: v is now 2
fmt.Println(incr(&v)) // "3" (and v is 3)
```

##### **Pointer aliasing** *

Each time we take the address of a variable or copy a pointer, we create new aliases or ways to identify the same variable. For example, `*p` is an alias for `v`. Pointer aliasing is useful because it allows us to access a variable without using its name, but this is a double-edged sword: to find all the statements that access a variable, we have to know all its aliases. <u>Aliasing also occurs when we copy values of other reference types like slices, maps, and channels, and even structs, arrays, and interfaces that contain these types.</u>

##### **Pointer and the `flag` package**

Pointers are key to the `flag` package, which uses a program's command-line arguments to set the values of certain variables for the entire program. To illustrate, this variation on the earlier `echo` command takes two optional flags:

* `-n` causes `echo` to omit the trailing newline that would normally be printed;
* `-s sep` causes it to separate the output arguments by the contents of the string `sep` instead of the default single space.

<small>[gopl.io/ch2/echo4/main.go](https://github.com/shichao-an/gopl.io/blob/master/ch2/echo4/main.go)</small>

```go
// Echo4 prints its command-line arguments.
package main

import (
	"flag"
	"fmt"
	"strings"
)

var n = flag.Bool("n", false, "omit trailing newline")
var sep = flag.String("s", " ", "separator")

func main() {
	flag.Parse()
	fmt.Print(strings.Join(flag.Args(), *sep))
	if !*n {
		fmt.Println()
	}
}
```

The function `flag.Bool` creates a new flag variable of type `bool`. It takes three arguments:

* The name of the flag (`"n"`),
* The variable's default value (`false`),
* A message that will be printed if the user provides an invalid argument, an invalid flag, or `-h` or `-help`.

This is similar to `flag.String`.

The variables `sep` and `n` are pointers to the flag variables, which must be accessed indirectly as `*sep` and `*n`.

When the program is run, it must call `flag.Parse` before the flags are used, to update the flag variables from their default values. The non-flag arguments are available from `flag.Args()` as a slice of strings. If `flag.Parse` encounters an error, it prints a usage message and calls `os.Exit(2)` to terminate the program.

The following are some test results:

```shell-session
$ go build gopl.io/ch2/echo4
$ ./echo4 a bc def
a bc def
$ ./echo4 -s / a bc def
a/bc/def
$ ./echo4 -n a bc def
a bc def$
$ ./echo4 -help
Usage of ./echo4:
  -n    omit trailing newline
  -s string
        separator (default " ")
```

#### The `new` Function

Another way to create a variable is to use the built-in function `new`. The expression `new(T)` creates an **unnamed variable** ([anonymous variable](https://golang.org/ref/spec#Variables)) of type `T`, initializes it to the zero value of `T`, and returns its address, which is a value of type `*T`.

```go
p := new(int)    // p, of type *int, points to an unnamed int variable
fmt.Println(*p)  // "0"
*p = 2           // sets the unnamed int to 2
fmt.Println(*p)  // "2"
```

A variable created with `new` is no different from an ordinary local variable whose address is taken, except that there's no need to invent (and declare) a dummy name, and we can use `new(T)` in an expression. <u>Thus `new` is only a syntactic convenience, not a fundamental notion.</u>

The two `newInt` functions below have identical behaviors:

```go
func newInt() *int {
	return new(int)
}
```

```go
func newInt() *int {
	var dummy int
	return &dummy
}
```

Each call to new returns a distinct variable with a unique address:

```go
p := new(int)
q := new(int)
fmt.Println(p == q) // "false"
```

There is one exception to this rule: two variables whose type carries no information and is therefore of size zero, such as `struct{}` or `[0]int`, may have the same address (depending on the implementation).

The `new` function is relatively rarely used because the most common unnamed variables are of struct types, for which the struct literal syntax ([Section 4.4.1](ch4.md#struct-literals)) is more flexible.

Since new is a predeclared function, not a keyword, it's possible to redefine the name for something else within a function, for example:

```go
func delta(old, new int) int { return new - old }
```

Within `delta`, the built-in `new` function is unavailable.

#### Lifetime of Variables

The lifetime of a variable is the interval of time during which it exists as the program executes.

* The lifetime of a package-level variable is the entire execution of the program.
* Local variables have dynamic lifetimes: a new instance is created each time the declaration statement is executed, and the variable lives on until it becomes *unreachable*, at which point its storage may be recycled.
* Function parameters and results are also local variables; they are created each time their enclosing function is called.

For example, in this excerpt from the Lissajous program of [Section 1.4](#animated-gifs):

```go
for t := 0.0; t < cycles*2*math.Pi; t += res {
	x := math.Sin(t)
	y := math.Sin(t*freq + phase)
	img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5),
		blackIndex)
}
```

* The variable `t` is created each time the for loop begins.
* New variables `x` and `y` are created on each iteration of the loop.

##### **How does the garbage collector know that a variable's storage can be reclaimed?** *

The basic idea is that every package-level variable, and every local variable of each currently active function, can potentially be the start or root of a path to the variable in question, following pointers and other kinds of references that ultimately lead to the variable. If no such path exists, the variable has become unreachable, so it can no longer affect the rest of the computation.

<u>Because the lifetime of a variable is determined only by whether or not it is reachable, a local variable may outlive a single iteration of the enclosing loop. It may continue to exist even after its enclosing function has returned.</u>

##### **Heap or stack?** *

A compiler may choose to allocate local variables on the heap or on the stack, but this choice is not determined by whether `var` or `new` was used to declare the variable.

```go
var global *int

func f() {
	var x int
	x = 1
	global = &x
}

func g() {
	y := new(int)
	*y = 1
}
```

In the above code:

* `x` must be heap-allocated because it is still reachable from the variable `global` after `f` has returned, despite being declared as a local variable; we say `x` *escapes* from `f`.
* Conversely, when `g` returns, the variable `*y` becomes unreachable and can be recycled. Since `*y` does not escape from `g`, it's safe for the compiler to allocate `*y` on the stack, even though it was allocated with `new`.

In any case, the notion of escaping is not something that you need to worry about in order to write correct code, though it's good to keep in mind during performance optimization, since each variable that escapes requires an extra memory allocation.

##### **Thoughts on garbage collection** *

Garbage collection is a tremendous help in writing correct programs, but it does not relieve you of the burden of thinking about memory. You don't need to explicitly allocate and free memory, but to write efficient programs you still need to be aware of the lifetime of variables.  For example, keeping unnecessary pointers to short-lived objects within long-lived objects, especially global variables, will prevent the garbage collector from reclaiming the short-lived objects.

### Assignments

The value held by a variable is updated by an assignment statement. In its simplest form, an assignment has a variable on the left of the `=` sign and an expression on the right:

```go
x = 1                       // named variable
*p = true                   // indirect variable
person.name = "bob"         // struct field
count[x] = count[x] * scale // array or slice or map element
```

Each of the arithmetic and bitwise binary operators has a corresponding *assignment operator*, which allows the last statement to be rewritten like `count[x] *= scale`. This saves us from having to repeat (and re-evaluate) the expression for the variable.

Numeric variables can also be incremented and decremented by `++` and `--` statements:

```go
v := 1
v++ // same as v = v + 1; v becomes 2
v-- // same as v = v - 1; v becomes 1 again
```

#### Tuple Assignment

**Tuple assignment** allows several variables to be assigned at once. <u>All of the right-hand side expressions are evaluated before any of the variables are updated, making this form most useful when some of the variables appear on both sides of the assignment.</u>

Tple assignment can be used in the following scenarios and examples:

Swapping the values of two variables:

```go
x, y = y, x
a[i], a[j] = a[j], a[i]
```

Computing the [greatest common divisor](https://en.wikipedia.org/wiki/Greatest_common_divisor) (GCD) of two integers:

```go
func gcd(x, y int) int {
	for y != 0 {
		x, y = y, x%y
}
	return x
}
```

Computing the *n*-th Fibonacci number iteratively:

```go
func fib(n int) int {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		x, y = y, x+y
}
	return x
}
```

Tuple assignment can also make a sequence of trivial assignments more compact:

```go
i, j, k = 2, 3, 5
```

However, as a matter of style, avoid the tuple form if the expressions are complex, since a sequence of separate statements is easier to read.

##### **Expressions that produce multiple results** *

Certain expressions produce several values.  When such a call is used in an assignment statement, the left-hand side must have as many variables as the function has results.

For example, a function call with multiple results:

```go
f, err = os.Open("foo.txt") // function call returns two values
```

Often, functions use these additional results to indicate some kind of error by returning either of the following:

* An `error` (as in the call to `os.Open`)
* A `bool`, usually called `ok`.

There are three operators that sometimes also behave this way:

* Map lookup (§4.3)
* Type assertion (§7.10)
* Channel receive (§8.4.2)

When any of the above three appears in an assignment in which two results are expected, each produces an additional boolean result:

```go
v, ok = m[key] // map lookup
v, ok = x.(T)  // type assertion
v, ok = <-ch   // channel receive
```

As with variable declarations, we can assign unwanted values to the blank identifier:

```go
_, err = io.Copy(dst, src) // discard byte count
_, ok = x.(T)              // check type but discard result
```

#### Assignability

Assignment statements are an explicit form of assignment. There are many places in a
program where an assignment occurs implicitly:

* A function call implicitly assigns the argument values to the corresponding parameter variables;
* A `return` statement implicitly assigns the `return` operands to the corresponding result variables;
* A literal expression for a [composite type](ch4.md) such as this slice:

```go
medals := []string{"gold", "silver", "bronze"}
```

This implicitly assign each element, as if written like this:

```go
medals[0] = "gold"
medals[1] = "silver"
medals[2] = "bronze"
```

This implicit assignment also applies to the elements of maps and channels.

##### **Assignability rule** *

<u>An assignment, explicit or implicit, is always legal if the left-hand side (the variable) and the right-hand side (the value) have the same type. More generally, the assignment is legal only if the value is *assignable* to the type of the variable.</u>

The rule for assignability has cases for various types. Relevant cases will be explained when each new type is introduced.

For the types discussed so far, the rules are simple:

* The types must exactly match.
* `nil` may be assigned to any variable of interface or reference type.

[Constants](ch3.md#constants) have more flexible rules for assignability that avoid the need for most explicit conversions.

##### **Assignability and comparability** *

Whether two values may be compared with `==` and `!=` is related to assignability: in any comparison, the first operand must be assignable to the type of the second operand, or vice versa. As with assignability, relevant cases for comparability will be explained when each new type is introduced.

### Type Declarations

The type of a variable or expression defines the characteristics of the values it may take on, such as:

* Size.
* How they are represented internally.
* Intrinsic operations that can be performed on them.
* Methods associated with them.

Variables can share the same representation but signify very different concepts. [p39]

A `type` declaration defines a new *named type* that has the same *underlying type* as an existing type. <u>The named type provides a way to separate different and perhaps incompatible uses of the underlying type so that they can't be mixed unintentionally.</u>

```go
type name underlying-type
```

Type declarations most often appear at package level, where the named type is visible throughout the package; if the name is exported (it starts with an upper-case letter), it's accessible from other packages as well.

#### Example of type declarations: temperature scales *

The following example turns different temperature scales into different types:

<small>[gopl.io/ch2/tempconv0/celsius.go](https://github.com/shichao-an/gopl.io/blob/master/ch2/tempconv0/celsius.go)</small>

```go
// Package tempconv performs Celsius and Fahrenheit temperature computations.
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC     Celsius = 0
	BoilingC      Celsius = 100
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

This package defines two types, `Celsius` and `Fahrenheit` for the two units of temperature. <u>Even though both have the same underlying type, `float64`, they are not the same type, so they cannot be compared or combined in arithmetic expressions.</u> Defining two types avoids errors like inadvertently combining temperatures in the two different scales; an explicit type conversion like `Celsius(t)` or `Fahrenheit(t)` is required to convert from a `float64`.

* `Celsius(t)` and `Fahrenheit(t)` are conversions, not function calls. <u>They don't change the value or representation in any way, but they make the change of meaning explicit.</u>
* The functions `CToF` and `FToC` convert between the two scales; they do return different values.

#### Type conversion *

For every type `T`, there is a corresponding conversion operation `T(x)` that converts the value `x`
to type `T`. A conversion from one type to another is allowed if any of the following holds:

* Both have the same underlying type.
* Both are unnamed pointer types that point to variables of the same underlying type.

These conversions change the type but not the representation of the value.

If `x` is assignable to `T`, a conversion is permitted but is usually redundant.

Conversions are also allowed in the following cases:

* Between numeric types
* Between string and some slice types (discussed in [Chapter 3](ch3.md)).

These conversions may change the representation of the value. For instance, converting a floating-point number to an integer discards any fractional part, and converting a string to a `[]byte` slice allocates a copy of the string data. In any case, a conversion never fails at run time.

The underlying type of a named type determines its structure and representation, and also the set of intrinsic operations it supports, which are the same as if the underlying type had been used directly. That means that arithmetic operators work the same for `Celsius` and `Fahrenheit` as they do for `float64`.

```go
fmt.Printf("%g\n", BoilingC-FreezingC) // "100" °C
boilingF := CToF(BoilingC)
fmt.Printf("%g\n", boilingF-CToF(FreezingC))  // "180" °F
fmt.Printf("%g\n", boilingF-FreezingC)        // compile error: type mismatch
```

Comparison operators like `==` and `<` can also be used to compare a value of a named type to
another of the same type, or to a value of the underlying type. But two values of different
named types cannot be compared directly:

```go
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)          // "true"
fmt.Println(f >= 0)          // "true"
fmt.Println(c == f)          // compile error: type mismatch
fmt.Println(c == Celsius(f)) // "true"!
```

Note that in the last case, the type conversion `Celsius(f)` does not change the value of its argument, just its type. The test is true because `c` and `f` are both zero.

#### Advantages of named types *

A named type provides notational convenience if it helps avoid writing out complex types over and over again. The advantage is small when the underlying type is simple like `float64`, but big for complicated types (detailed when discussing [structs](ch4.md#structs)).

Named types also make it possible to define new behaviors for values of the type. These behaviors are expressed as a set of functions associated with the type, called the type's **methods**. Methods are discussed in [Chapter 6](ch6.md). The following example of the `String` method gives a taste the mechanism.

The declaration below, in which the `Celsius` parameter `c` appears before the function name, associates with the `Celsius` type a method named `String` that returns `c`'s numeric value followed by `°C`:

```go
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }
```

#### The `String` method *

Many types declare a `String` method of this form because it controls how values of the type appear when printed as a string by the `fmt` package, discussed in [Section 7.1](ch7.md#interfaces-as-contracts).

```go
c := FToC(212.0)
fmt.Println(c.String()) // "100°C"
fmt.Printf("%v\n", c)   // "100°C"; no need to call String explicitly
fmt.Printf("%s\n", c)   // "100°C"
fmt.Println(c)          // "100°C"
fmt.Printf("%g\n", c)   // "100"; does not call String
fmt.Println(float64(c)) // "100"; does not call String
```

### Packages and Files

Similar to libraries or modules in other languages, packages in Go supports modularity, encapsulation, separate compilation, and reuse. The source code for a package resides in one or more `.go` files, usually in a directory whose name ends with the import path; for instance, the files of the `gopl.io/ch1/helloworld` package are stored in directory `$GOPATH/src/gopl.io/ch1/helloworld`.

* Each package serves as a separate *name space* for its declarations.
    For example, within the `image` package the identifier `Decode` refers to a different function than does the same identifier in the `unicode/utf16` package. To refer to a function from outside its package, we must *qualify* the identifier to make explicit whether we mean `image.Decode` or `utf16.Decode`.
* Packages hides information by controlling which names are visible outside the package, or *exported*. In Go, exported identifiers start with an upper-case letter.

Suppose we want to make our temperature conversion software available to the Go community as a new package.

Create a package called [`gopl.io/ch2/tempconv`](https://github.com/shichao-an/gopl.io/tree/master/ch2/tempconv), a variation on the previous example. The package itself is stored in two files to show how declarations in separate files of a package are accessed.

The declarations of the types, their constants, and their methods are in `tempconv.go`:

<small>[gopl.io/ch2/tempconv/tempconv.go](https://github.com/shichao-an/gopl.io/blob/master/ch2/tempconv/tempconv.go)</small>

```go
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC     Celsius = 0
	BoilingC      Celsius = 100
)

func (c Celsius) String() string    { return fmt.Sprintf("%g°C", c) }
func (f Fahrenheit) String() string { return fmt.Sprintf("%g°F", f) }
```

The conversion functions are in `conv.go`:

<small>[gopl.io/ch2/tempconv/conv.go](https://github.com/shichao-an/gopl.io/blob/master/ch2/tempconv/conv.go)</small>

```go
// Package tempconv performs Celsius and Fahrenheit conversions.
package tempconv

// CToF converts a Celsius temperature to Fahrenheit.
func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

// FToC converts a Fahrenheit temperature to Celsius.
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

Each file starts with a package declaration that defines the package name. When the package is imported, its members are referred to as `tempconv.CToF` and so on. <u>Package-level names declared in one file of a package are visible to all the other files of the package, as if the source code were all in a single file.</u>

[p42]

To convert a Celsius temperature to Fahrenheit in a package that imports `gopl.io/ch2/tempconv`, we can write the following code:

```go
fmt.Println(tempconv.CToF(tempconv.BoilingC)) // "212°F"
```

#### Doc comment of a package *

The *doc comment* ([Section 10.7.4](#ch10.md#documenting-packages)) immediately preceding the package declaration documents the package as a whole.

* Conventionally, it should start with a summary sentence in the style illustrated.
* Only one file in each package should have a package doc comment.
* Extensive doc comments are often placed in a file of their own, conventionally called `doc.go`.

#### Imports

##### **Import path** *

Every package within a Go program is identified by a unique string called its **import path**, which appears in an `import` declaration like `gopl.io/ch2/tempconv`. The language specification doesn't define where these strings come from or what they mean; it's up to the tools to interpret them. When using the `go` tool ([Chapter 10](ch10.md)), <u>an import path denotes a directory containing Go source files that make up the package.</u>

##### **Package name** *

In addition to its import path, each package has a **package name**, which is the short buy not necessarily unique name that appears in its package declaration. By convention, a package's name matches the last segment of its import path. For example, the package name of `gopl.io/ch2/tempconv` is `tempconv`.

To use `gopl.io/ch2/tempconv`, we must import it:

<small>[gopl.io/ch2/cf/main.go](https://github.com/shichao-an/gopl.io/blob/master/ch2/cf/main.go)</small>

```go
// Cf converts its numeric argument to Celsius and Fahrenheit.
package main

import (
	"fmt"
	"os"
	"strconv"

	"gopl.io/ch2/tempconv"
)

func main() {
	for _, arg := range os.Args[1:] {
		t, err := strconv.ParseFloat(arg, 64)
		if err != nil {
			fmt.Fprintf(os.Stderr, "cf: %v\n", err)
			os.Exit(1)
		}
		f := tempconv.Fahrenheit(t)
		c := tempconv.Celsius(t)
		fmt.Printf("%s = %s, %s = %s\n",
			f, tempconv.FToC(f), c, tempconv.CToF(c))
	}
}
```

The import declaration binds a short name to the imported package that may be used to refer to its contents throughout the file. The `import` the above code enables us refer to names within `gopl.io/ch2/tempconv` by using a qualified identifier like `tempconv.CToF`. By default, the short name is the package name (`tempconv` in this case), but an import declaration may specify an alternative name to avoid a conflict ([Section 10.3](ch10.md#the-package-declaration)).

The `cf` program converts a single numeric command-line argument to its value in both Celsius and Fahrenheit:

```shell-session
$ go build gopl.io/ch2/cf
$ ./cf 32
32°F = 0°C, 32°C = 89.6°F
$ ./cf 212
212°F = 100°C, 212°C = 413.6°F
$ ./cf -40
-40°F = -40°C, -40°C = -40°F
```

It is an error to import a package and then not refer to it. This check helps eliminate dependencies that become unnecessary as the code evolves, although it can be a nuisance during debugging. [p44]

##### **`goimports`** *

The [golang.org/x/tools/cmd/goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) tool automatically inserts and removes packages from the import declaration as necessary; most editors can be configured to run `goimports` each time you save a file. Like the `gofmt` tool, it also pretty-prints Go source files in the canonical format.

#### Package Initialization

Package initialization begins by initializing package-level variables in the order in which they are declared, except that dependencies are resolved first:

```go
var a = b + c  // a initialized third, to 3
var b = f()    // b initialized second, to 2, by calling f
var c = 1      // c initialized first, to 1

func f() int { return c + 1 }
```

If the package has multiple `.go` files, they are sorted by the `go` tool, given to the compiler and initialized in this order.

An variable declared at package level starts life with the value of its optional initializer expression. However, for some variables, like tables of data, it's not easy to set their initial values using initializer expressions, in which case the `init` function mechanism may be simpler. A file may contain any number of `init` functions like the following:

```go
func init() { /* ... */ }
```

Except that `init` functions can't be called or referenced, they are normal functions. Within each file, `init` functions are automatically executed when the program starts, in the order in which they are declared.

* One package is initialized at a time, in the order of imports in the program, dependencies first.
    * For example, if package `p` imports `q`, then `q` is fully initialized before `p`'s initialization begins.
* Initialization proceeds from the bottom up; the `main` package is the last to be initialized. All packages are fully initialized before the application's `main` function begins.

The package below defines a function `PopCount` that returns the number of set bits (bits whose value is 1) in a `uint64` value, which is called its [*population count*](https://en.wikipedia.org/wiki/Hamming_weight). It uses an `init` function to precompute a table of results, `pc`, for each possible 8-bit value so that the `PopCount` function needn't take 64 steps but can just return the sum of eight table lookups. (This is definitely not the fastest algorithm for counting bits, but it's convenient for illustrating `init` functions, and for showing how to precompute a table of values, which is often a useful programming technique.)

<small>[gopl.io/ch2/popcount](https://github.com/shichao-an/gopl.io/blob/master/ch2/popcount/main.go)</small>

```go
package popcount

// pc[i] is the population count of i.
var pc [256]byte

func init() {
	for i := range pc {
		pc[i] = pc[i/2] + byte(i&1)
	}
}

// PopCount returns the population count (number of set bits) of x.
func PopCount(x uint64) int {
	return int(pc[byte(x>>(0*8))] +
		pc[byte(x>>(1*8))] +
		pc[byte(x>>(2*8))] +
		pc[byte(x>>(3*8))] +
		pc[byte(x>>(4*8))] +
		pc[byte(x>>(5*8))] +
		pc[byte(x>>(6*8))] +
		pc[byte(x>>(7*8))])
}
```

Note that in the above code the `range` loop in `init` uses only the index, since the value is not need. The loop could also have been written as:

```go
for i, _ := range pc {
```

### Scope

A declaration associates a name with a program entity, such as a function or a variable. The scope of a declaration is the part of the source code where a use of the declared name refers to that declaration.

#### Scope vs. Lifetime *

Don't confuse scope with lifetime.

* The scope of a declaration is a region of the program text; it is a compile-time property.
* The lifetime of a variable is the range of time during execution when the variable can be referred to by other parts of the program; it is a run-time property.

#### Lexical Blocks *

##### **Syntactic Block and Lexical Block** *

* A **syntactic block** (or [block](https://en.wikipedia.org/wiki/Block_(programming))) is a sequence of statements enclosed in braces (e.g. function, loop). A name declared inside a syntactic block is not visible outside that block. The block encloses its declarations and determines their scope.
* A **lexical block** does not surround declarations with braces. It is generalized from the notion of syntactic blocks. Each of the following elements has a lexical block:
    * The entire source code, called **universe block**
    * Each package
    * Each file
    * Each `for`, `if`, and `switch` statement
    * Each case in a `switch` or `select` statement
    * Each explicit syntactic block

##### **Determining Scopes** *

The lexical block of a declaration determines its scope.

* Entire source code. The declarations of built-in types, functions, and constants (e.g. `int`, `len` and `true`) are in the universe block. They can be referred to throughout the entire program.
* Package. Declarations outside any function (i.e. at package level) can be referred to from any file in the same package.
* File. Imported packages (e.g. `fmt` in the `tempconv` example) are declared at the file level, so they can be referred to from the same file, but not from another file in the same package without another import.
* Local. Many declarations (e.g. the variable `c` in the `tempconv.CToF function`) are local, so they can be referred to only from within the same function or a part of it.

The scope of a control-flow label, as used by `break`, `continue`, and `goto` statements, is the entire enclosing function.

##### **Multiple declarations of the same name** *

A program may contain multiple declarations of the same name as long as each declaration is in a different lexical block.

For example:

* You can declare a local variable with the same name as a package-level variable.
* As shown in [Section 2.3.3](#the-new-function), you can declare a function parameter called `new`, even though a function of this name is predeclared in the universe block.

However, this should not be overdone. The larger the scope of the redeclaration, the more likely you are to surprise the reader.

When the compiler encounters a reference to a name, it looks for a declaration, starting with the innermost enclosing lexical block, up to the universe block:

* If the compiler finds no declaration, it reports an "undeclared name" error.
* If a name is declared in both an outer block and an inner block, the inner declaration will be found first. In that case, the inner declaration is said to *shadow* (or hide) the outer one, making it inaccessible.

For example:

```go
func f() {}

var g = "g"

func main() {
	f := "f"
	fmt.Println(f) // "f"; local var f shadows package-level func f
	fmt.Println(g) // "g"; package-level var
	fmt.Println(h) // compile error: undefined: h
}
```

Within a function, lexical blocks may be nested to arbitrary depth, so one local declaration can shadow another. Most blocks are created by control-flow constructs like `if` statements and `for` loops.

The program below has three different variables called `x` because each declaration appears in a different lexical block.

```go
func main() {
	x := "hello!"
	for i := 0; i < len(x); i++ {
		x := x[i]
		if x != '!' {
			x := x + 'A' - 'a'
			fmt.Printf("%c", x) // "HELLO" (one letter per iteration)
		}
	}
}
```

The expressions `x[i]` and `x + 'A' - 'a'` each refer to a declaration of `x` from an outer block. (Note that the latter expression is not equivalent to `unicode.ToUpper`). This is explained in detail below.

##### **Explicit and Implicit Blocks** *

Not all lexical blocks correspond to explicit brace-delimited sequences of statements; some of them also correspond to implicit blocks. In the previous example, the `for` loop above creates two lexical blocks:

* The explicit block for the loop body.
* An implicit block that additionally encloses the variables (e.g. `i`) declared by the initialization clause. The scope of a variable declared in the implicit block is the condition, post-statement (`i++`), and body of the `for` statement.

The example below also has three variables named `x`, each declared in a different block: one in the function body, one in the `for` statement's block, and one in the loop body; but only two of the blocks are explicit:

```go
func main() {
	x := "hello"
	for _, x := range x {
		x := x + 'A' - 'a'
		fmt.Printf("%c", x) // "HELLO" (one letter per iteration)
	}
}
```

Like `for` loops, `if` statements and `switch` statements also create implicit blocks in addition to their body blocks. The code in the following `if-else` chain shows the scope of `x` and `y`:

```go
if x := f(); x == 0 {
	fmt.Println(x)
} else if y := g(x); x == y {
	fmt.Println(x, y)
} else {
	fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```

<u>The second `if` statement is nested within the first, so variables declared within the first `if` statement's initializer are visible within the second.</u> Similar rules apply to each case of a `switch` statement: there is a block for the condition and a block for each case body.

At the package level, the order in which declarations appear has no effect on their scope, so a declaration may refer to itself or to another that follows it. This enables us to declare recursive or mutually recursive types and functions. However, the compiler will report an error if a constant or variable declaration refers to itself.

##### **Examples: `if` statement** *

In the following program:

```go
if f, err := os.Open(fname); err != nil { // compile error: unused: f
	return err
}
f.ReadByte() // compile error: undefined f
f.Close()    // compile error: undefined f
```

The scope of `f` is just the `if` statement, so `f` is not accessible to the statements that follow, resulting in compiler errors: "undefined f". Depending on the compiler, you may get an additional error reporting that the local variable `f` was never used.

Thus it is often necessary to declare `f` before the condition so that it is accessible after:

```go
f, err := os.Open(fname)
if err != nil {
	return err
}
f.ReadByte()
f.Close()
```

You may be tempted to avoid declaring `f` and `err` in the outer block by moving the calls to `ReadByte` and `Close` inside an else block:

```go
if f, err := os.Open(fname); err != nil {
	return err
} else {
	// f and err are visible here too
	f.ReadByte()
	f.Close()
}
```

However, <u>the normal practice in Go is to deal with the error in the `if` block and then return, so that the successful execution path is not indented.</u>

##### **Short variable declarations and scope** *

Short variable declarations require awareness of scope. The following program starts by obtaining its current working directory and saving it in a package-level variable. This could be done by calling `os.Getwd` in function main, but it might be better to separate this concern from the primary logic, especially if failing to get the directory is a fatal error.  The function [`log.Fatalf`](https://golang.org/pkg/log/#Fatalf) prints a message and calls [`os.Exit(1)`](https://golang.org/pkg/os/#Exit).

```go
var cwd string
func init() {
	cwd, err := os.Getwd() // compile error: unused: cwd
	if err != nil {
		log.Fatalf("os.Getwd failed: %v", err)
	}
}
```

The `:=` statement declares `cwd` and `err` as local variables. The inner declaration of `cwd` makes the outer one inaccessible, so the statement does not update the package-level `cwd` variable as intended. Current Go compilers detect that the local `cwd` variable is never used and report this as an error, but they are not strictly required to perform this check. A minor change (as shown below), such as the addition of a logging statement that refers to the local `cwd` as shown below, would defeat the check.

```go
var cwd string

func init() {
	cwd, err := os.Getwd() // NOTE: wrong!
	if err != nil {
		log.Fatalf("os.Getwd failed: %v", err)
	}
	log.Printf("Working directory = %s", cwd)
}
```

The global `cwd` variable remains uninitialized, and the apparently normal `log` output obfuscates the bug.

There are a number of ways to deal with this potential problem. The most direct is to avoid `:=` by declaring `err` in a separate `var` declaration:

```go
var cwd string

func init() {
	var err error
	cwd, err = os.Getwd()
		if err != nil {
	log.Fatalf("os.Getwd failed: %v", err)
	}
}
```

### Doubts and Solutions

#### Verbatim

##### **p32 on short variable declaration**

> A short variable declaration acts like an assignment only to variables that were already declared in the same lexical block; declarations in an outer block are ignored.

<span class="text-danger">Question</span>: What does "declarations in an outer block are ignored" mean?

##### **p45 on `init` function**

<span class="text-danger">Question</span>: What exact algorithm does `PopCount` use? It seems different from the Hamming weight algorithm on Wikipedia.
