#### Java 8 function is defined using `@FunctionalInterface`, which is a interface with single abstract method.   
`@FunctionalInterface` is the return type of lambda expression.  

Groups the function interface by the datatype it supports. function interfaces are defined in `java.util.function` packages.

### 1. 0 params functioninterface

* `Supplier<T>`

  `()--> Supplier --> T`
  
* `BooleanSupplier`

  `()--> BooleanSupplier --> boolean`
  
### 2. 1 params functionInterface

* `Function<T,R>`: for all other kinds of XFunction, which is change the input params to X, like `DoubleFunction<R>` 
is accept `double` input params. 

  `T --> Function --> R`

* `UnaryOperator<T>`: input and return type are same, extends from Function

  `T --> UnaryOperator<T> --> T`

* `Predicate<T>`

  `T --> Predicate<T> --> boolean`

* `Consumer<T>`: an expected side effect function, which has no return.

  `T --> Consumer  --> void` 

  
### 3. 2 params functionInterface

* `BiFunction<T,U,R>`

  `T,U --> BiFunction --> R`

* `BinaryOperator<T>`: input and return type are same. extends from BiFunction

  `T,T --> BinaryOperator --> T`

* `BiPredicate<T,U>`

  `T,U --> BiPredicate --> boolean`

### Naming standards for functions interface.

* if function  return a basic type, it starts with `To`

  T -> ToLongFunction -> long
  
* if function accept a basic type, it just starts with basic type, like
  
  long -> LongFunction -> T
