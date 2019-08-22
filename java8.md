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

### Stream difference 

Copy of stream created for stream operation in `java.util.stream` package

`Stream`, `DoubleStream`,`IntStream`,`LongStream`

`Stream`
Stream.of(T)
Stream.iterate(T, seeds, UnaryOperator<T>)
`DoubleStream`

DoubleStream.of(double ...)
DoubleStream.iterate(double seed,DoubleUnaryOperator f)

`IntStream`
IntStream.of(int ...)
IntStream.generate(int seed, IntUnaryOperator f)
IntStream.range(int start, int end)

`LongStream`

each stream has factory method to create a stream


### default method in interface

`default` method is introduced in java8 is used to make sure `binary compatibility`. Code write use java1-java7 can be compiled against java8. as java8 introduced `stream` method in `Collection` interface. all code implements `Collection` 
need rewrite in java8, so java8 add `default stream` method in `Collection`

`default` method is added in interface level ,provide a default impl for a abstract method. 

  1. if one class impls a interface which has default method, this interface is extends from another interface which also has same default method. the class will choose the `default` method in most closed interface. 
  
  2. if one class extends an abstract class and also impls a interface. the default method in the abstract class will be chose.  
