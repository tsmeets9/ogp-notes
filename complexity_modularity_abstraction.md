# Managing Complexity through Modularity and Abstraction

The question addressed by this course is: how can we manage the complexity of the task of building large software systems?

The approach we teach is an instance of a general approach for any complex task: _divide and conquer_. This means that we try to split the problem up into subproblems, such that 1) each subprogram is easier to solve than the overall problem, and 2) solving all of the subprograms yields a solution for the overall problem.

In the case of software development, this means trying to _decompose_ the system development task into a set of simpler subsystem development tasks, such that the resulting subsystems can be composed to yield a system that solves the overall task.

One of the most effective ways to come up with such a decomposition is to try to come up with effective _abstractions_. In this course, we teach two main types of abstractions: _procedural abstractions_ and _data abstractions_.

## Procedural abstractions

In the case of procedural abstractions, we try to identify _operations_ such that it would make the system development task easier if those operations were _built into the programming language_.

Only a very limited number of operations are built into the Java programming language: `+`, `-`, `*`, `/`, `%`, `<`, `>`, `<=`, `>=`, `==`, `!=`, `&&`, `||`, and the bitwise operators `&`, `|`, `^`, `<<`, `>>`, and `>>>`. Anything else you want to do in a Java program, you need to _implement_ in terms of these basic operations.

For example, if an application needs to perform square root computations, it will have to implement a square root operation in terms of addition, subtraction, etc. The complexity of building such an application, then, includes the complexity of implementing a square root implementation.

The life of the application's developers would have been easier if square root had been a primitive operation in the Java programming language. Building the application would have been a simpler task.

Procedural abstraction, then, means that we split the task of building such an application into two parts:
- building a _client module_ that implements the application on top of not Java, but Java++, an extension of Java with a built-in square root operation (whose syntax is `MyMath.squareRoot(...)`).
- building a _square root module_ whose only job is to implement the square root operation (i.e. to implement method `MyMath.squareRoot`) in terms of Java's built-in operations.

Each of these two tasks is easier than the overall task:
- The client module's developers do not have to worry about how to implement a square root operation.
- The square root module's developers do not have to worry about the application. They need not care if the application is implementing a web shop, a game, a bookkeeping system, or anything else. All they need to worry about is correctly implementing the square root operation.

Note that this decomposition will only be effective if the abstraction is defined sufficiently _precisely_ and _abstractly_.
- If the only way the client module's developers can figure out what `MyMath.squareRoot` does is by looking at its implementation, then their life has not been made much simpler. They are still confronted with the complexity of the square root implementation. For this reason, it is crucial that the abstraction implemented by a module and used by the client application (also known as the module's _Application Programming Interface_ or _API_) be _properly documented_. The API documentation must document sufficiently precisely and abstractly the operations provided by the module to its clients.
- Analogously, if the only way the square root module developers can figure out what the application expects `MyMath.squareRoot` to do is by looking at the application's implementation and by understanding what the application as a whole is supposed to do, then they are still confronted with the complexity of the entire application. In conclusion, proper documentation is necessary to achieve true reduction of complexity and _separation of concerns_.
- A module's documentation should not just be sufficiently precise. It should also be sufficiently _abstract_. If the documentation for the square root operation simply showed the implementation algorithm, again the client module developers' life would not have been made much easier. Again they would be confronted with the complexity of the implementation. A module's documentation must _hide_ the implementation complexity as much as possible and define the meaning of the API as simply as possible.

See below a proper way to document the square root module:
```java
/**
 * Returns the square root of the given nonnegative integer, rounded down.
 *
 * @pre The given integer is nonnegative.
 *    | 0 <= x
 * @post The result is nonnegative and not greater than the given integer.
 *    | 0 <= result
 * @post The square of the result is not greater than the given integer.
 *    | (long)result * result <= x
 * @post The square of one more than the result is greater than the given integer.
 *    | x < ((long)result + 1) * ((long)result + 1)
 */
public static int squareRoot(int x) {
    int result = 0;
    while (result * result < x)
        result++;
    return result - 1;
}
```

(Note that we need to cast the `int` result to `long` before computing its square or adding one, to avoid arithmetic overflow.)

Notice that the documentation does not reveal the algorithm used to compute the square root. (The algorithm shown is a very slow and naive one; better-performing ones undoubtedly exist. Faster algorithms would undoubtedly be more complex and further increase the complexity reduction achieved by the layer of abstraction.)

It includes a _precondition_: the module promises to implement its abstraction correctly only if the client adheres to the precondition. The postconditions state the conditions that must hold when the method returns, for the method's behavior to be considered correct.

Proper documentation for a module, then, simplifies both the task of developing the module itself, and the task of developing client modules that use the module:
- The developers that implement the module need to look only at the module's API documentation to learn what the module is supposed to do.
- The developers that implement the client modules (i.e. the modules that use the module) need to look only at the module's API documentation to learn what the module does.

## Modular software development

Splitting a system up into modules with a clearly defined, abstract API between them is an effective way of splitting a software development task into a set of simpler subtasks. Each module is not just easier to **build** than the system as a whole; it is also easier for someone to come in, read the codebase, and understand what is going on, because each module can be **understood** separately. This is because there is now a well-defined notion, for each module separately, of what that module is supposed to do (i.e. there is a well-defined notion of _correctness_ for each module separately): a module is correct if it implements its API in such a way that it complies with the module's API documentation, assuming that the lower-level modules it builds upon comply with their API documentation.

Having a notion of correctness for each module separately means we can **verify** each module separately: we can perform testing and code review on each module independently from its clients.

It also means the modules of the system can be built in parallel, by independent software development teams. Furthermore, the developers of a module can release new versions of the module that fix bugs, improve performance, or add new features. So long as the new versions of the module comply with the original API documentation, the old version can be replaced by the new version in the overall system without adversely impacting the overall system's correct operation.

## Data abstractions

Procedural abstractions allow clients to work with a more powerful programming language, that has more operations built in.

Similarly, application development would often benefit from having more _datatypes_ built in besides the ones that are built into the Java programming language itself.

The datatypes built into Java itself are the following:

| Datatype | Values |
| -------- | ------ |
| `byte` | The integers between -128 and 127 |
| `short` | The integers between -32768 and 32767 |
| `int` | The integers between -2<sup>31</sup> and 2<sup>31</sup> - 1 |
| `long` | The integers between -2<sup>63</sup> and 2<sup>63</sup> - 1 |
| `char` | The integers between 0 and 65535 |
| `boolean` | `true` and `false` |
| `float` | The single-precision floating-point numbers |
| `double` | The double-precision floating-point numbers |

Important remarks about these datatypes:
- Values of type `char` are used to represent text characters. The notation `'X'` denotes the `char` value assigned to character `X` by the Unicode character encoding standard. For example, `'A'` denotes the `char` value 65.
- The arithmetic operations (`+`, `-`, `*`, `/`, `%`) do not operate directly on types `byte`, `short`, or `char`; instead, operands of type `byte`, `short`, or `char` are first _promoted_ to type `int`. For example, the type of expression `'A' + 'A'` is `int` and its value is 130.
- An expression that applies an arithmetic operator to two operands of type `int` is of type `int` itself, even though the mathematical result of the operation might not be within the limits of type `int`. In that case (known as _arithmetic overflow_), the value 2<sup>32</sup> is added to or subtracted from the result as many times as necessary to bring it within the limits of the type. For example: the expression `2000000000 + 2000000000` is of type `int` and its value is -294967296 (= 4000000000 - 2<sup>32</sup>).
- If either operand of an arithmetic expression is of type `long` and the other operand is of any integer type, the expression itself is of type `long` and 2<sup>64</sup> is added to or subtracted from the result as many times as necessary to bring it within the limits of type `long`.
- Type `float` has three kinds of values:
  1. The real values, of the form M &#215; 2<sup>E</sup> where mantissa M is an integer between -2<sup>24</sup> + 1 and 2<sup>24</sup> - 1, and exponent E is an integer between -149 and 104. Note, however, that there are two distinct "zero" values: positive zero (`0f`) and negative zero (`-0f`).
  2. The infinities, `Float.POSITIVE_INFINITY` and `Float.NEGATIVE_INFINITY`.
  3. The Not-a-Number (NaN) values; these are used to represent the result of operations that have no well-defined result in mathematics, such as `0f/0f` and `Float.POSITIVE_INFINITY/Float.POSITIVE_INFINITY`. You can tell whether a `float` value is a Not-a-Number value using method `Float.isNaN`.
- Analogously, type `double` has three kinds of values:
  1. The real values, of the form M &#215; 2<sup>E</sup> where mantissa M is an integer between -2<sup>53</sup> + 1 and 2<sup>53</sup> - 1, and exponent E is an integer between -1074 and 971. Again, there are two distinct zero values: positive zero (`0.0`) and negative zero (`-0.0`).
  2. The infinities, `Double.POSITIVE_INFINITY` and `Double.NEGATIVE_INFINITY`.
  3. The NaN values. You can tell whether a `double` value is a NaN value using method `Double.isNaN`.
- Java's comparison operators have potentially surprising behavior on floating-point numbers: for example, `0.0 == -0.0` returns `true` and `0.0/0.0 == 0.0/0.0` returns `false`. To tell whether two `float` values are identical, compare the results of calling `Float.floatToRawIntBits` on both of them; similarly, to tell whether two `double` values are identical, compare the results of calling `Double.doubleToRawLongBits` on both of them.
- The arithmetic operations on floating-point values perform implicit rounding. In computations involving multiple arithmetic operations, these rounding errors can accumulate and lead to very large errors in the final result. Constructing correct floating-point computations is a very challenging and specialized skill that is outside the scope of this course.
- For further reading on floating-point numbers, see [here](https://people.eecs.berkeley.edu/~wkahan/ieee754status/IEEE754.PDF) and [here](https://cr.yp.to/2005-590/goldberg.pdf).