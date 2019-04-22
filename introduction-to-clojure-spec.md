# A Brief Introduction to Clojure.Spec

`clojure.spec` is a powerful library that allows programmers to:

- describe data beyond type
- document code in a way readable by both humans and machines
- validate data
- instrument code
- destructure data
- improve error reporting
- generate data for testing

## Describing data

The most common ways to describe data are names, types, and comments. Both names and comments communicate primarily to the programmer. Types also communicate to the programmer, but types additionally provide compilers with information to verify data at compile time.

The `clojure.spec` library offers the safety of types while retaining the flexibility of a dynamic language. `clojure.spec` can mimic type-checking, but specs go further by employing predicates to describe data within a single type (e.g., a spec can specify not just that a value must be a string, but that the string must be of a certain length, or contain a particular sequence of characters). And unlike types, specs can be applied to primitives, collections, and functions at the programmer’s discretion.

The power of `clojure.spec` derives from the composition of predicate functions. A ##predicate## is a function that produces a boolean. In Clojure, predicates that are not defined in-line should end with a question mark. Example predicates from the core clojure library include `even?` and `string?`, but you can create custom predicates as well, including sets and predicates that employ regular expressions.

## Example specs
Say we want to write a function that requires integers greater than 9000. 

```clojure
(defn yell 
"Yell takes an integer over 9000 and returns a loud string."
 [n]
 (str n " is over 9000!"))

(yell 77777) ;=> "77777 is over 9000!"
```

We can specify this function's requirement with a spec, which will validate the data every time that function is called.

First we require the `clojure.spec` library, then define a spec for an integer over 9000.

```clojure 
(require '[clojure.spec.alpha :as s])
(s/def ::over-9000 (s/and int? #(> % 9000))
```

Here the `s/def` function defines a spec `::over-9000`. The `s/and` function composes the `int?` and `#(> % 9000)` predicate functions to define an integer greater than 9000. Like types, specs can be reused. One need only register the spec to a namespace.

We can now validate data according to that spec with `clojure.spec/valid?`.

```clojure
(s/valid? ::over-9000 10) ;=> false
(s/valid? ::over-9000 999999) ;=> true
```

`clojure.spec/valid?` can also coerce predicates into specs.
```clojure
(s/valid? even? 10) ;=> true
(s/valid? string? "wizard") ;=> true
```

Now to spec the function:

```clojure
(s/fdef yell :args ::over-9000
             :ret string?)
```

Here the `clojure.spec/fdef` function specifies that for the `yell` function, the argument must conform to the `::over-9000` spec, and the return value must be a string.

The spec will now appear in the functions documentation.

```clojure
(doc yell) ;=>
yell
([n])
  yell takes an integer over 9000 and returns a loud string
Spec
  args: (and int? (> % 9000))
  ret: string?
```

Notice that the documentation automatically unpacks the predicates stored in the `::over-9000` spec.

# Runtime validation
You can use specs to verify input at runtime using `defn`'s `:pre` and `:post` condition support, and `clojure.spec/valid?`.

(defn yell
  "yell takes a number over 9000 and returns a loud string"
  [n]
  {:pre [s/valid? ::over-9000]
   :post [s/valid? string?]
  (str n " is over 9000!"))


We can now use these specs to generate data for testing.

If you spec a function for validation, you can reuse to generate random data that conforms to that spec. You can also generate sample data as you program to make sure the spec covers the expected data set.


## Use Cases
### Development
#### Reasoning about your code.
Defining expected inputs and outputs of functions helps think through edge cases, making code more robust.
#### Generate test data
Specs can generate random data that fit the spec.
testing that conforms to the specification.

### Production
- Data validation at runtime. Especially useful for data sent over the wire. 
##WARNING## Note that there is a performance penalty for runtime validation, so only validate data at runtime when data correctness is paramount and the incoming data may not be accurate.


## Advantages
### Types on demand
Unlike types, which must be applied everywhere, specs can be applied only as needed.
### Flexibility
Let's say requirements change and you need to add a key to a map. In a type system, you would have to change a class, or create a new class. With specs, adding a key to a map would have no effect on existing specs. You can then choose which functions interact with that new key, and add additional specs to relevant functions at your discretion.
### Specs are documentation
Readable by both humans and compilers.
### Specs don't interfere with existing code
You can group specs with code, or keep them in a separate namespace.

## Tradeoffs
### Runtime performance
Like types, specs are most useful during development. But the use of specs during runtime should be strategic, as they do incur 
### Development time
Like types, specs require more effort to code. But also like types, time spent specifying data may save time in the long run, both by encouraging programmers to reason more about the code, getting more granular feedback and error reporting, and facilitating automated testing.
