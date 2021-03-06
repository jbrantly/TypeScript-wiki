# Introduction

Traditional JavaScript focuses on functions and prototype-based inheritance as the basic means of building up reusable components, but this may feel a bit awkward to programmers more comfortable with an object-oriented approach, where classes inherit functionality and objects are built from these classes.  Starting with ECMAScript 6, the next version of JavaScript, JavaScript programmers will be able to build their applications using this object-oriented class-based approach.  In TypeScript, we allow developers to use these techniques now, and compile them down to JavaScript that works across all major browsers and platforms, without having to wait for the next version of JavaScript.

# Classes

Let's take a look at a simple class-based example:

```TypeScript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter = new Greeter("world");
```

The syntax should look very familiar if you've used C# or Java before.  We declare a new class 'Greeter'.  This class has three members, a property called 'greeting', a constructor, and a method 'greet'.  

You'll notice that in the class when we refer to one of the members of the class we prepend 'this.'.  This denotes that it's a member access.

In the last line we construct an instance of the Greeter class using 'new'.  This calls into the constructor we defined earlier, creating a new object with the Greeter shape, and running the constructor to initialize it.

# Inheritance

In TypeScript, we can use common object-oriented patterns.  Of course, one of the most fundamental pattern in class-based programming is being able to extend existing classes to create new ones using inheritance.

Let's take a look at an example:

```TypeScript
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move() {
        alert("Slithering...");
        super.move(5);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move() {
        alert("Galloping...");
        super.move(45);
    }
}

var sam = new Snake("Sammy the Python");
var tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

This example covers quite a bit of the inheritance features in TypeScript that are common to other languages.  Here we see using the 'extends' keywords to create a subclass.  You can see this where 'Horse' and 'Snake' subclass the base class 'Animal' and gain access to its features.

The example also shows off being able to override methods in the base class with methods that are specialized for the subclass.  Here both 'Snake' and 'Horse' create a 'move' method that overrides the 'move' from 'Animal', giving it functionality specific to each class.

# Private/Public modifiers

## Public by default
You may have noticed in the above examples we haven't had to use the word 'public' to make any of the members of the class visible.  Languages like C# require that each member be explicitly labelled 'public' to be visible.  In TypeScript, each member is public by default.  

You may still mark members a private, so you control what is publicly visible outside of your class.  We could have written the 'Animal' class from the previous section like so:

```TypeScript
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

## Understanding private

TypeScript is a structural type system.  When we compare two different types, regardless of where they came from, if the types of each member are compatible, then we say the types themselves are compatible.  

When comparing types that have 'private' members, we treat these differently.  For two types to be considered compatible, if one of them has a private member, then the other must have a private member that originated in the same declaration.  

Let's look at an example to better see how this plays out in practice:

```TypeScript
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
	constructor() { super("Rhino"); }
}

class Employee {
    private name:string;
    constructor(theName: string) { this.name = theName; }	
}

var animal = new Animal("Goat");
var rhino = new Rhino();
var employee = new Employee("Bob");

animal = rhino;
animal = employee; //error: Animal and Employee are not compatible
```

In this example, we have an 'Animal' and a 'Rhino', with 'Rhino' being a subclass of 'Animal'.  We also have a new class 'Employee' that looks identical to 'Animal' in terms of shape.  We create some instances of these classes and then try to assign them to each other to see what will happen.  Because 'Animal' and 'Rhino' share the private side of their shape from the same declaration of 'private name: string' in 'Animal', they are compatible.  However, this is not the case for 'Employee'.  When we try to assign from an 'Employee' to 'Animal' we get an error that these types are not compatible.  Even though 'Employee' also has a private member called 'name', it is not the same one as the one created in 'Animal'.  

## Parameter properties

The keywords 'public' and 'private' also give you a shorthand for creating and initializing members of your class, by creating parameter properties.  The properties let you can create and initialize a member in one step.  Here's a further revision of the previous example.  Notice how we drop 'theName' altogether and just use the shortened 'private name: string' parameter on the constructor to create and initialize the 'name' member.

```TypeScript
class Animal {
    constructor(private name: string) { }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

Using 'private' in this way creates and initializes a private member, and similarly for 'public'. 

# Accessors

TypeScript supports getters/setters as a way of intercepting accesses to a member of an object.  This gives you a way of having finer-grained control over how a member is accessed on each object.

Let's convert a simple class to use 'get' and 'set'.  First, let's start with an example without getters and setters.

```TypeScript
class Employee {
    fullName: string;
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

While allowing people to randomly set fullName directly is pretty handy, this might get us in trouble if we people can change names on a whim.  

In this version, we check to make sure the user has a secret passcode available before we allow them to modify the employee.  We do this by replacing the direct access to fullName with a 'set' that will check the passcode.  We add a corresponding 'get' to allow the previous example to continue to work seamlessly.

```TypeScript
var passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }
	
    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

To prove to ourselves that our accessor is now checking the passcode, we can modify the passcode and see that when it doesn't match we instead get the alert box warning us we don't have access to update the employee.

Note: Accessors require you to set the compiler to output ECMAScript 5.

# Static Properties

Up to this point, we've only talked about the _instance_ members of the class, those that show up on the object when its instantiated.  We can also create _static_ members of a class, those that are visible on the class itself rather than on the instances.  In this example, we use 'static' on the origin, as it's a general value for all grids.  Each instance accesses this value through prepending the name of the class.  Similarly to prepending 'this.' in front of instance accesses, here we prepend 'Grid.' in front of static accesses.

```TypeScript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        var xDist = (point.x - Grid.origin.x);
        var yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

var grid1 = new Grid(1.0);  // 1x scale
var grid2 = new Grid(5.0);  // 5x scale

alert(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
alert(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

# Advanced Techniques

## Constructor functions

When you declare a class in TypeScript, you are actually creating multiple declarations at the same time.  The first is the type of the _instance_ of the class.

```TypeScript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter: Greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

Here, when we say 'var greeter: Greeter', we're using Greeter as the type of instances of the class Greeter.  This is almost second nature to programmers from other object-oriented languages.  
 
We're also creating another value that we call the _constructor function_.  This is the function that is called when we 'new' up instances of the class.  To see what this looks like in practice, let's take a look at the JavaScript created by the above example:

```TypeScript
var Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

var greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

Here, 'var Greeter' is going to be assigned the constructor function.  When we call 'new' and run this function, we get an instance of the class.  The constructor function also contains all of the static members of the class.  Another way to think of each class is that there is an _instance_ side and a _static_ side.

Let's modify the example a bit to show this difference:

```TypeScript
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

var greeter1: Greeter;
greeter1 = new Greeter();
alert(greeter1.greet());

var greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
var greeter2:Greeter = new greeterMaker();
alert(greeter2.greet());
```

In this example, 'greeter1' works similarly to before.  We instantiate the 'Greeter' class, and use this object.  This we have seen before.

Next, we then use the class directly.  Here we create a new variable called 'greeterMaker'.  This variable will hold the class itself, or said another way its constructor function.  Here we use 'typeof Greeter', that is "give me the type of the Greeter class itself" rather than the instance type.  Or, more precisely, "give me the type of the symbol called Greeter", which is the type of the constructor function. This type will contain all of the static members of Greeter along with the constructor that creates instances of the Greeter class.  We show this by using 'new' on 'greeterMaker', creating new instances of 'Greeter' and invoking them as before.

## Using a class as an interface

As we said in the previous section, a class declaration creates two things: a type representing instances of the class and a constructor function.  Because classes create types, you can use them in the same places  you would be able to use interfaces.

```TypeScript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

var point3d: Point3d = {x: 1, y: 2, z: 3};
```
