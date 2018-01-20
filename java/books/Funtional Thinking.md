The best way to get perfect work is to present it to people repeatedly. Some relevace or common feature can only be discovered after deeper research and passive thinking.

Forget above words.

### Chap 1 Why
Learning a new programming paradigm, is not only about mastering a new language, but also thinking in a new way.

> OO makes code understandable by encapsulating moving parts. FP makes code understandable by minimizing moving parts.
By Michael Feathers

- Resusability    
OOP: messages between classes
FP: data struct and workflow ?

### Chap 2 Shift
#### Difference
- Imperative Processing   
A sequence of commands   
_for_ iteration

- Functional    
Expression and Transformation    
Model mathmatical formula    
Avoid mutal states    

#### Common Building Blocks
- filter
- map
- reduce/fold(foldLeft/foldRight)

Scala: implicit parameter
```Scala
numbers filter (_ % 3 == 0)
```

### Chap 3 Cede
Ceding control to the language/runtime

- Higher-Order function over iteration
- Closure
A feature implemented by all functional language    
Definition:
> A function    
A special function   
Wrap all its reference in a context

Lazy execution?

Hold context, instead of states    
Leave states management to language

- Currying and Partial Application of function    
Currying: process of Transformation

Partial Application: apply part of parameters ?

Do not confuse with Partial Function in Scala

- Recursion    
Another perspective on Linked List: head and tail    

Tail-call optimization: avoid stack growth

### Chap 4
#### Memoization
Method level inner cache    
Only for pure function

#### Laziness
Lazy evaluation    
Compute element when use   

Pro:  
Delay expensive caculation  
Infinite collection  
Help map, filter

### Chap 5 Evolving Language
Few data structs with massive operations

Dispatch mechanism:  
dynamically choose the behavior  
e.g. pattern match in Scala, _switch case_

[In Java, _switch case_ only supports constant value]  
Java may use Factory/Abstract Factory patterns

Clojure has multimethod, like polymorphism

4. Operator Overload

5. Functional Data Structure
Referential transparency  

Exception handle:  
Either  
Option

### Chap 6 Pattern and Resusability
Architecture  
Groovy @Immutable

### Chap 7 Application
CQRS  
Command-Query Responsibility Segregation  
Will use Eventual Consistency instead of transaction  
BASE instead of ACID

Functional Database  
Datomic  
Immutable Data  
Record all schema and data updates  
Separate read and write  
Immutable valud and timestamp in event-driven architect  

### Chap 8 Multi paradigm
Meta programming  

Functional Pyramid  
