# TypeScript Mix

A tweaked implementation of TypeScript's default applyMixins(...) idea using ES7 decorators. 

## Breaking Changes from Version 3.0.0 upwards
* New decorator @delegate introduced
* Changes made in how multiple mixins implementing the same method are mixed

See [Breaking Changes Explained](#breaking-changes-explained)

## Dependencies
   * TypeScript
   * ES7 Decorators
 

## Installation
```
npm install --save typescript-mix
```

## Features
  * Properties in a mixin are not mixed into the client. They are ignored. See [TypeScript Mix — Yet Another Mixin Library](https://medium.com/@michaelolof/typescript-mix-yet-another-mixin-library-29c7a349b47d) for a detailed explanation on why. 
  * Classes and Object Literals can be used as mixins.


## Goals

   * Ensure programming to an interface and not just only multiple implementations.

   * Create simple mixins that implement that interface

   * Provide an intuitive and readable way of using such mixins in a concrete class.



## Why I wrote yet another Mixin Library.

The mixin pattern is somewhat a popular pattern amongst JavaScript/TypeScript devs as it gives the power of "mixin in" additional functionality to a class. The official way of using mixins as declared by Microsoft in TypeScript can be really verbose to downright unreadable.


## How to use

### The 'use' decorator

#### Program to an interface.

```
interface Buyer {
  price: number
  buy(): void
  negotiate(): void
}
```

Create a reusable implementation for that interface and that interface alone (Mixin)

```
const Buyer: Buyer = {
  price: undefined,
  buy() {
    console.log("buying items at #", this.price );
  },
  negotitate(price: number) {
    console.log("currently negotiating...");
    this.price = price;
  },
}
```

Define another mixin this time using a Class declaration.
```
class Transportable {
  distance:number;
  transport() {
    console.log(`moved ${this.distance}km.`);
  }
}
```


Define a concrete class that utilizes the defined mixins.

```
import use from "typescript-mix";

class Shopperholic {
  @use( Buyer, Transportable ) this
  
  price = 2000;
  distance = 140;
}

const shopper = new Shopperholic();
shopper.buy() // buying items at #2000
shopper.negotiate(500) // currently negotiating...
shopper.price // 500
shopper.transport() // moved 140km
```

#### What about intellisense support?
We trick typescript by using the inbuilt interface inheritance and declaration merging ability.
```
interface Shopperholic extends Buyer, Transportable {}

class Shopperholic {
  @use( Buyer, Transportable ) this
  
  price = 2000;
  distance = 140;
}
```

### The 'delegate' decorator
The delegate decorator is useful when we want specific functionality mixed into the client.
```
class OtherClass {
  simpleMethod() {
    console.log("This method has no dependencies");
  }
}

function workItOut() {
  console.log("I am working it out.")
}

class MyClass {
  @delegate( OtherClass.prototype.simpleMethod )
  simpleMethod:() => void

  @delegate( workItOut ) workItOut:() => void
}

const cls = new MyClass();
cls.simpleMethod() // This method has no dependencies
cls.workItOut() // I am working it out
```


## Things to note about this library?
* using the 'use' decorator mutates the class prototype. This doesn't depend on inheritance (But if you use mixins correctly, you should be fine)

* mixins don't override already declared methods or fields in the concrete class using them.

* Mixins take precedence over a super class. i.e. they would override any field or method from a super class with the same name.

* instance variables/fields/properties can be declared or even initialized in your mixins. This is necessary if you're defining methods that depend on object or class properties but these properties won't be mixed-in to the base class so you have to redefine those properties in the base class using the mixin.


## Advantages
   * The Library is non-obtrusive. Inheritance still works, (multiple inheritance still works ('Real Mixins Style')).

## <a name="breaking-changes-explained">Breaking Changes Explained</a>
### The delegate decorator
The addition of the delegate decorator now means module is imported as:
```
import { use, delegate } from "typescript-mix"
```

### Multiple Mixins with the same method.
Consider the following piece of code.
![alt text](https://github.com/michaelolof/typescript-mix/blob/master/imgs/2018-05-30%2021_32_46-Preview%20README.md%20-%20typescript-mix%20-%20Visual%20Studio%20Code.png?raw=true)

Client One uses two mixins that contain the same method mixIt(). How do we resolve this? Which method gets picked?.
One advantage of extending interfaces as we've defined above is that we're essentially telling TypeScript to mix-in the two mixin interfaces into the ClientOne interface. So how does TypeScript resolve this?

![alt text](https://github.com/michaelolof/typescript-mix/blob/master/imgs/2018-05-30%2021_49_13-final.ts%20-%20typescript-mix%20-%20Visual%20Studio%20Code.png?raw=true)

Notice that TypeScript's intellisense calls MixinOne.mixIt() method. Therefore to be consistent with TypeScript and avoid confusion the '@use' decorator also implements MixinOne.mixIt() method.
