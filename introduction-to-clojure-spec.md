# A Brief Introduction to Clojure.Spec

Clojure is a dynamic programming language. Some programmers prefer languages with static typing. `clojure.spec` (spec is short for specification) is a powerful library that adds static-type-like functionality, and it overdelivers.

`clojure.spec` allows programmers to:

- describe data beyond type
- document code in a way readable by both humans and machines
- validate data
- instrument code
- destructure data
- improve error reporting
- generate sample data for REPL development and testing

As a brief introduction, this document can't cover all of the above. It can only give a sample of the `clojure.spec` library, but the discussion should be enough to give you an idea of the library's functionality, and to encourage you to learn more.

## Specs
Specs are predicates used to describe sets of values.

We can use any predicate function (i.e., a function that takes a single input and produces a boolean) as a spec. Example predicates from the core clojure library include `even?` and `string?`, but we can create custom predicates as well, including predicates that employ regular expressions and sets. The power of the `clojure.spec` library derives from our ability to compose these predicate functions to specify data with precision beyond the reach of types.

For example, predicate functions allow us to describe data _within a single type_. A spec can specify not just that a value must be of type string, for instance, but that the string must be of a certain length, or contain a particular sequence of characters.

And unlike types, specs are on demand; specs can be applied to primitives, collections, and functions at the programmer’s discretion. In this way, specs offer the safety of static types while retaining the flexibility of a dynamic language. 

## Example spec
Say we want to write a function that requires integers greater than 9000. 

```clojure
(defn yell 
  "Yell takes an integer over 9000 and returns a loud string."
  [n]
  (str n " is over 9000!"))

(yell 77777) ;=> "77777 is over 9000!"
```

We can specify this function's requirement with a spec, which can validate the data when the function is called.

First we require the `clojure.spec` library, then define a spec for an integer over 9000.

```clojure 
(require '[clojure.spec.alpha :as s])
(s/def ::over-9000 (s/and int? #(> % 9000))
```

Here the `s/def` function defines a spec `::over-9000`. The `s/and` function composes the `int?` and `#(> % 9000)` predicate functions to define an integer greater than 9000. Like types, specs can be reused. One need only register the spec to a namespace.

We can now validate data according to that spec with `clojure.spec/valid?`.

```clojure
(s/valid? ::over-9000 999999) ;=> true
(s/valid? ::over-9000 10) ;=> false
```

When a value fails, we can use `s/explain` to find out why the value is invalid.

```clojure
(s/explain ::over-9000 10)
10 - failed: (> % 9000) spec: ::over-9000
```

Note that the error message unpacks the `::over-9000` spec to describe the specific predicate that the value failed: the anonymous function `#(> % 9000)`.

`clojure.spec/valid?` can also coerce predefined predicates into specs.

```clojure
(s/valid? even? 10) ;=> true
(s/valid? string? "wizard") ;=> true
```

Now to that we have the `::over-9000` spec defined, we can use it to spec the `yell` function with `s/fdef`:

```clojure
(s/fdef yell :args ::over-9000
             :ret  string?)
```

Here the `clojure.spec/fdef` function specifies that for the `yell` function, the argument (`:args`) must conform to the `::over-9000` spec, and the return value (`:ret`) must be a string.

The spec will now appear in the function's documentation.

```
(doc yell) ;=>
yell
([n])
  yell takes an integer over 9000 and returns a loud string
Spec
  args: (and int? (> % 9000))
  ret: string?
```

Notice that, like the error message from the `s/explain` example, the documentation conveniently unpacks the predicates stored in the `::over-9000` spec.

## Advantages of `clojure.spec`

### Encourages reasoning about our code.

Defining expected inputs and outputs of functions beyond mere type encourages programmers to think through edge cases and program design, making code more robust.

### Generate test data

We can use clojure's test.check library to generate random data that fits any spec. We can also `s/excercise` specs to see randomly generated data conforming to the spec in the REPL as you refine the specification.

### Optional Runtime validation

Validating data during runtime can be useful for data sent over the wire and at other I/O boundaries. *WARNING* Note that there is a performance penalty for runtime validation, so only validate data at runtime when data correctness is paramount and the correctness of the data is uncertain.

Example: you can use specs to verify input at runtime using `defn`'s `:pre` and `:post` keys paired with the `clojure.spec/valid?` function.

```clojure
(defn yell
  "yell takes a number over 9000 and returns a loud string"
  [n]
  {:pre [s/valid? ::over-9000]
   :post [s/valid? string?]
  (str n " is over 9000!"))
```

### Surgical typing

Let's say requirements change and we need to add a key to a map. In a type system, we would have to change a class, or create a new class. With specs, adding a key to a map would have no effect on existing specs. We can then choose which functions interact with that new key, and add additional specs to relevant functions at our discretion. If we do nothing, the new key will still flow through functions without issue.

### Specs are documentation

Specs double as documentation readable by both humans and compilers.

### Instrumentation and improved error reporting
Specs can be used to monitor values passed to functions during development, and to return well-defined error messages on failure.

### Specs don't interfere with existing code

You can group specs with code, or keep them in a separate namespace. They can be toggled on or off.

## Tradeoffs

### Performance hit at runtime

Like types, specs are most useful during development for both iterating on  feedback and considering code design. The use of specs during runtime should be strategic, as validation requires extra computation. 

### No enforcement of types

One can argue that static types force us to think about data description.

### Cognitive overhead and development time

Programmers need to make decisions about what parts of the code to spec, and which to leave unspecified. And like types, specs require more effort to code. But also like types, time spent specifying data may save time in the long run, both by encouraging programmers to reason more about the code, getting more granular feedback and error reporting, and facilitating automated testing.


## Learn more

[spec API](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha)

[About spec](https://clojure.org/about/spec)

[The spec guide](https://clojure.org/guides/spec)