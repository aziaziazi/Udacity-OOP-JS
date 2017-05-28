My notes for https://www.udacity.com/course/object-oriented-javascript--ud015

# Scopes
Lexical Scope =! Context Scope

## Lexical Scope  
Scope writen is the file. Outer variables are accessibles globaly and inner ones are accessible localy:


```js
var globalVariable = "I'm accessible anywhere !"

var myFunction = function(){
    var localVariable = "I'm accessible only in everythink inside myFunction()"
    // => access to globalVariable and localVariable
}
// => only access to globalvariable

```

## Context Scope
Also refered as Execution context, or just context. The context is created during the execution and stay where it has been called

# Closures
A function that remain available after the outer scope have returned.
TODO : do more research to  understand it.

# This
Dynamicaly refer to the current object.
Quite similar of a regular parameter, but with two makor differencies :
- I don't have to pick a name, it's always refered as `this`
- Binding value to this is done in a bit different way

## Bounding
https://classroom.udacity.com/courses/ud015/lessons/2593668699/concepts/26033786450923

```js
var obj = {
    fn : function(a, b){
        console.log(this)
    }
}

var obj2 = {method, obj.fn}

obj.fn(3,4)
```
*this* refers to the *obj* defined on the last line > It refers to the object that *call* the function where *this* appear.

```js
var r={red}, g={green}, b={blue}

var fn = function(one, two){
    console.log(this, one, two)
}

r.method = fn

r.method(g,b) // {red, method}, {green}, {blue}
r['method'](g,b) // {red, method}, {green}, {blue}
fn(g,b) // {global object}, {green}, {blue}
```
*this* refers to the object which the methos is applied. if no object, *this* refers to the global object (*Window* if run in a browser).

## Overide the bounding
`.call` lets specify the object bound to `this`. It overide the calling object or the default global object. `this` is bound to the first parameter given.

```js
fn.call(r,g,b) // {red}, {green}, {blue}
r.method.call(y,g,b) // {yellow}, {green}, {blue}
```

## Bounding in a function passed as a callback
The parameters are not passed so their values are *undefined*. However we can pass parameters at the end.
`this` gets its defaultd value *global object*.

```js
setTimeout(fn, 1000) // {global object}, undefined, undefined
setTimeout(fn, 1000, r, g) // {global object}, {red}, {green}
```

Even if we pass the function as an object method, *this* won't get bound to the object : __*this* is evaluated at the moment the function gets called__. In this case it will be in the setTimeout function, where `r` is inexistant.

```js
var setTimeout = function (callback, millisecond){
    waitSomehow(millisecond)
    callback() // fn gets called here
}

r.method = fn
setTimeout(r.method, 1000) // {global object}, undefined, undefined
```

This is a cummon problem : many functions takes another functions as a callback that may call that functions differently than expected.
If we want to have the reference to *this*, we can pass a different function without any parameter at all, and call our function in it's body as we want:

```js
setTimeout(function(){r.method(g,b)}, 1000) // {red, method}, {green}, {blue} 

// ES6
setTimeout(() => r.method(), 1000); // (to verify)
```

## Bind

*Bind* seems to be another usefull option to bind *this* to specific values. Need to be investigated.

```js
log(one) // error because one is not defined
log(this) // {global object}. It's a specific behavior of this, but it's not consistent so it might me removed on futures versions of JS.
new r.method(g,b); // 'this' will refer to an entirely new objects created because of the behavior of this, to support the OOP JS paradighm.
```

# Prototype
Prototype chains are a mechanism to makes objects resemble other objects.

## extend
**One-time** property copying.
```js
var gold = {a:1};

var blue = extend({}, gold); // Needs to create the 'extend' function:
                             // http://stackoverflow.com/a/19776683
console.log(blue.a); // 1

gold.a = 2;
console.log(blue.a); // 1 => because it's one-time binding.
```

## Object.create
**ongoing** lookup-time delegation : if property not found, it uses the linked object as a fall-back source of property. Therefore if we change the source value, it will affect the child.
```js
var gold = {a:1};

var rose = Object.create(gold);
console.log(rose.a); // 1

gold.a = 2;
console.log(rose.a); // 2 => ongoing binding !!
```

## the object prototype
There is a top level object that every JS object delegates to. It has some functions like `.toString()`, `hasOwnProperty`, `constructor`...

## Constructor

`constructor()` is a function of the top level object that returns a reference to the function Object that created the instance:

```js
var o = {};
o.constructor === Object; // true

var a = [];
a.constructor === Array; // true

var n = new Number(3)
n.constructor === Number; // true


function Tree(name) {
  this.name = name;
}
var theTree = new Tree('Redwood');

console.log(theTree.constructor);
// function Tree(name) {
//   this.name = name;
// }
```

## Object Decorator Pattern
### Code Reuse

Don't repeat yourself! For exemple if I create a method/library instead of repeating it's implementation:
- I often reduce the number of lines.
- I can modify the function in one place and all objects using it will be affected. For the same reasons, it's also easier to extend.
- I can give an appropriate name to that function (and even comments) to make it more understandable than it's implementation.

### Decorator
We can create a function to add properties and methods to an object:

```js
// library.js
var carlike = function(obj, loc){ // Decorator function
    obj.loc = loc;
    obj.move = function(){
        obj.loc++
    };
    return obj;
};

// run.js
var amy = carlike({}, 1);
move(amy);
var ben = carlike({}, 9);
move(ben);

var car1 = carlike({}, 0);
var car2 = carlike({}, 0);
console.log(car1 === car2) // false : The same lines of code is used to create them, but they are themselves different object. Changes made on one object won't have effects on the other.
```

# Classes
A class is a way to construct objects that have similar properties and methods.
There's different ways to do that with pros and cons.

## Functional classes
### Constructor
A class is a notion of a category of things that I want to build and all the code that supports that category. The **constructor** is the function I use to produce an **instance** of that class.
A decorator adds up on a passed object. A class create itself the object returned.
It's conventional to name a class with a Capital first letter.

```js
var Car = function(loc){ // Constructor function
    var obj = {loc: loc}; // the object returned is created here
    obj.move = function(){
        obj.loc++
    }
    return obj
}
```

### Functional shared patterns
When I declare variables and functions in a constructor, a new instance of thoses var/func is created each time of the constructor is called. This can use a lot of memory. It's not necessary to have different instances of the functions and I can reduce the duplicity. I can declare them outside of the constructor so every instances of the object will make reference to the same function.
This is called **Functional shared patterns**

```js
var Car = function(loc){
    var obj = {loc: loc}
    obj.move = move // reference to the move function
    return obj
}

var move = function(){
    this.loc++
}
```

However `move` is declared/named in 3 places. This is bad because I can make error if I change the name or if I want to add other methods.
Also, `Car` and `move` are not bind together.
I can resolve this by adding methods to class like this:

```js
var Car = function(loc){
    var obj = {loc: loc}
    extend(obj, Car.methods)
    return obj
}

Car.methods = {
    move: function(){
        this.loc++
    }
}
```

## Prototypal Classes
### Lookup delegation
The prototypal chain is a reference lookup, so that improve the performances : we don't create references for every property/method, but if we call a property missing, it will automatically lookup in the parents. The steps to make that happend are:
1. Function to generate and return new instances objects: `function(){var obj; return obj}`
2. A delegation for the new object to some *storing object*: `Object.create()`
3. Some logic to agmenting the object properties that makes it unique: `obj.loc = loc`
4. Some *storing object* to declare the shared methods of our class: `Car.methods = {method: function(){}}`

```js
var Car = function(loc){
    var obj = Object.create(Car.methods) // Delegate the lookup in Car.methods
    obj.loc = loc // Add arguments properties
    return obj
}

Car.methods = {
    move: function(){
        this.loc++
    }
}
```

### Keyword `prototype`
This Pattern is very cummon, so the language designer decided to add official conventions to support it: whenever a function is created, it has an object attached to it that I can use as a container for methods. It is `functionName.prototype`.

```js
var Car = function(loc){
    var obj = Object.create(Car.prototype) // This line is still needed !!
    obj.loc = loc
    return obj
}

Car.prototype.move = function(){
    this.loc++
}
```

### Abiguousity
https://classroom.udacity.com/courses/ud015/lessons/2794468538/concepts/27657285440923
However, introducing this second meaning for `prototype` is very ambiguous. Even through thoses two sentences looks very similar, their word `prototype` means something very different:
- "*Amy's prototype is Car.prototype*": Amy's delegate field lookups to Car.prototype.
- "*Car prototype is car.prototype*": Car has an object called prototype used to construct instances.

We also could say: "*Car prototype is a function prototype where all function objects delegates their field lookups*" as so for *String() prototype*, *Number() prototype*...
To explain "*Amy's prototype is Car.ptototype*", we could say "*When Car() runs, it create object that delegate field lookup to Car.prototype*".


```js
var Car = function(){
    return Object.create(Car.prototype)
}

var Exemple = function(){ // Behave exactly the same as Car.
    return Object.create(Car.prototype)
}

Car.prototype.move = function(){ // The method object (or prototype objec) is stored in Car, but it could be anywhere else.
    console.log("moving")
}
```

### `.constructor` function
This `prototype` object is almost a normal object and almost doesn't have any spacial behavior, it is just a convention. However, it automaticaly has a `.constructor` function that point back to the constructor object which is handy to know which function has created an instance:
 
```js
var Car = function(){
    return Object.create(Car.prototype)
}

Car.prototype.move = function(){
    console.log("moving")
}

console.log(Car.prototype.constructor) // Car object
console.log(amy.constructor) // Car object
```

### instanceof operator

`instanceof` check if <rightHand>.prototype can be found in <leftHand>'s prototype chain :

```js
console.log(amy instanceof Car) // True with a prototypal Class, but false with a functional class because there isn't any class prototype.

```

## Peudoclassical Patterns
### Keyword `new`
If I want to make a lot of classes (I usually do), the prototypal pattern is good but I have to repeat every time `var obj = Object.create(<Class>.prototype)` and `return obj`.
Javascript provide the keyword `new` to automate that. Anytime it is used in front of a function invocation, the function will run in a **Constructor Mode**: the interpreter will insert thoses lines in the function to be run:

```js
// var Car = function(loc){
     this = Object.create(<Class>.prototype)
//     this.loc = loc
     return this
// }

// Car.prototype.move = function(){
//     this.loc++
// }
```

### Implementation
I can't see thoses lines, but they *are* interpreted.
So, any function can be called *with* or *without* the keyword `new`. It's up to me to (re)factor my functions to use it or not. Of course, it's better to be consistant in the code to not make mistakes.
If I want to use `new`, my class will now look like this:

```js
var Car = function(loc){
    this.loc = loc // We now make reference to "this" because the interpreter bind the created object to it.
}

Car.prototype.move = function(){
    this.loc++
}
```

The way to function will behave is the same for prototypal and pseudoclassical patterns. The primary difference between this two pattern is optimizations JS does.

## Pros and Cons
There is many class patterns in JavaScript and there is not one better over another. However, JS makes the internals features available so the programmers can play with and discuss about it. So rather than wright and wrongs, it's more about techniques and options with adventages and siadventages

### Similatities and Differences
I can notive that some classes properties will be the same for every instances and others that will be different. Functional classes allow to store every properties in the same place, but prototypal and pseudoclassical classes separate thoses properties. This can be an asset or a faling depending of what my perspective!

```js
// Functional pattern
var Car = function(loc){
    var obj = {loc: loc} // Differences
    obj.move = function(){
        obj.loc++        // Similarities
    }
    return obj
}

// pseudoclassical pattern
var Car = function(loc){
    this.loc = loc // Differences
}

var Car.prototype.move = function (){
    this.loc++     // Similarities
}
```

## Functional SuperClass and SubClasses
Sometimes I want slighty differents types of classes but with shared properties. I can refactor thoses classes be instances of a **superclass**:

```js
// Functional pattern
var Car = function(loc){ // SupperClass : shared properties
    var obj = {loc:loc}
    obj.move = function(){
        obj.loc++
    }
    return obj
}

var Van = function(loc){ // SubClass
    var obj = Car(loc); // Lookup to SupperClass
    obj.grab = function(){/*...*/} // Own property
    return obj
}

var Cop = function(loc){ // SubClass
    var obj = Car(loc); // Lookup to SupperClass
    obj.call = function(){/*...*/} // Own property
    return obj
}
```

This pattern is easy to uderstand but not very used. The pseudoclassical pattern bellow is more cummon.

## PseudoClassical SuperClass and SubClasses

https://classroom.udacity.com/courses/ud015/lessons/2794468541/concepts/27770585420923
```js
// library.js
var Car = function(loc){
    this.loc = loc
}
Car.prototype.move = function(){
    this.loc++
}

var Van = function(loc){
    Car.call(this, loc) // See "Bind Constructor" bellow
}
Van.prototype = Object.create(Car.prototype) // See "Bind Prototype object" bellow
Van.prototype.constructor = Van // See "Reset the constructor" bellow
Van.prototype.grab = function(){/*...*/}

// run.js
var zed = new Car(3); // use keyword "new" => pseudoclassical pattern
zed.move()

var amy = new Van(9);
amy.move()
amy.grab()
```

We must bind the constructor function and the prototype object.

### Bind Constructor function

`this = new Car(loc)` or `this = Object.create(loc)` wouldn't have been possible since we can't reasign the value of `this` ourself (but the interpreter can).

`Car(loc)` won't do much.

`Car.call(this, loc)` will do the job perfectly! `.call()` makes possible to pass sublclass's `this` to the supperclass so it can be enhance. The parameters can be passed too!

### Bind Prototype object
https://classroom.udacity.com/courses/ud015/lessons/2794468541/concepts/27843985390923
As we used prototype chain, we need to wire up the subclass prototype to the superclass prototype to allow the similarity code be shared.

`Van.prototype = Car.prototype` won't copy, but assign the two object to the same place in memory :/

`Van.prototype = new Car()` this will work and was recommanded before `Object.create()` exist. However there's major problems with this technique: the superclass constructor may require arguments, and we don't have those values in the right scope. They will be undefined, and that will cause errors in case of .access() functions. Be carefull because this is a technique that has been widely accepted before and I can see it on reputable site. But I shouldn't do it!

`Van.prototype = Object.create(Car.prototype)` Good !

### Reset the constructor
https://classroom.udacity.com/courses/ud015/lessons/2794468541/concepts/27893185360923#
We want the constructor method to point to the subclass. However, it will fall through the supeprclass one if nothing's set.

`Van.prototype.consyructor = Van` will do the job!
