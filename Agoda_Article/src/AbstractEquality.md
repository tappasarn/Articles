# The myst of JavaScript's "==", something ESLint did not teach you.
When I first joined the world of the Web, as a newbie I was introduced to this language called *JavaScript*. One of the things that make JavaScript so much different than other programming languages is the fact that you can achieve the *similar* outcome by doing things in different ways. In most cases, you will not notice the difference of those approches. But once it is revealed, most of the time, it comes with the price you have to pay. To some developers, those costs were great enough that Douglas Crockford dedicated a part of his book, JavaScript: the good parts, to talk about those bad (or maybe awful) parts. <Br />

From the reason above, JavaScript has taken it place as one of the top programming languages that people like to make memes about its behaviour.<Br />
This is one of many memes that I have recently shared with my beloved programmer friends.
<div style="text-align:center"> 
    <img width="300" src = "https://user-images.githubusercontent.com/11821799/43389783-bc7b3a08-9416-11e8-8e6c-0ed1cc816f50.png"/>
<div/>
<div style="text-align:left"/><Br />

It is true that nowadays we have got a big help from a config file called *ESLint* that helps (banded) us from doing things that might results in the future sorrow. On the other hand, it also keep many developers away from learning the actual JavaScript. It would not be such a good idea if we just take those rules for graned and being apathetic about properly learning the language.<Br />

## “The Abstract Equality Comparison Algorithm”
(yeah... it is just a cool name for "==")<Br />
 
Not too long ago, I was reading through a JavaScript book series called You Don't Know JS, written by Kyle Simpson. There was a chapter that actually reminded me about the meme posted above. It explained about the loosely equality comparison behaviour of JavaScript. So, I think it would not be a bad idea if I would make a summarize of it, and share it here.

Equality check is one amoung those things that could be done in more than a single ways, and one among the ways (believed to be evil) is often being banded by the project ESLint configuration. <Br />
<Br />
Similar to many other Jacascript newbies, what I have been told about JavaScript's equality brothers (== and ===) was that the one with three equeal signs checks for the type, and the one with only two equa signs does not make a type checking.  <Br />
<Br />
But was that understanding were correctly intepreted .... ?

```js
const myNullValue = null;

if (!myNullValue) {
    // Code reached this line because myNullValue is a falsy
}

if (myNullValue == false){
    // Code never reached this line
    // myNullValue is not equal to false
}

if (!!myNullValue == false){
    // Code reached this line
    // If we manually converted myNullValue to boolean, it works same as the first if statement
}
```
<div style="text-align:center"> 
    <img src = "https://user-images.githubusercontent.com/11821799/43380753-73c5df30-93fc-11e8-865e-74d701f36d13.png"/>
<div/>
<div style="text-align:left"/><Br />

But.... we have just said that loosely equlity operation does not care about the type. Why would *null* as a falsy value not equal to *false* which is also one of falsy values.<Br />
<Br />

## How did that happen ?
The main difference between loosely equality and strictly equality is that the strictly equality immediately return false when the type of both operands are different, but for loosely equlitym it allows the coersion to happen on one or both types of the operands until both of them ended up on the same type <Br />

## Type coersion
let's take a look at the algorithm that loosely equality perform in order to corvert operands' types. 
### Reminder
Before we get started, there some rules that you will have to keep in mind. 

* NaN: In the world of JavaScript NaN is NOT equal to itself. To verify NaN,  use isNaN() or Number.isNaN().
* Object: Object (included with arrays and functions) will be the same if they have the same reference.
### Loosely compare strings to numbers
```js
const x = "100";
const y = 100;

const result = (x == y); // true
```
The rule here is the *Number type has priority over string* and the engine will try to convert the string operand to its number type. To make it easier to understand it would look somewhat like
```js
const x = "100";
const y = 100;

// Since typeof Number(x) is "number"
// x and y are now in the same type
const result = Number(x) == y;
```

### Loosely compare anything to boolean
```js
const x = "100";
const y = true;

const result = (x == y); // false
```

When you try to compare anything to boolean, firstly, a boolean value will get converted into its number form. Then JavaScript verifies the type of those two operands, if they are the same, the coersion is done and the conparision result is returned, else, it keeps coverting using other matched rules. <Br />
<Br /> 
Basically what happen in our example snippet was
```js
const x = "100";
const y = true;

// Since y is a boolean, it is the first one that get converted to its number type
// Number(false) will give 0, true will give 1
const result = (x == Number(y))

// Right now, we have the operation of "100" == 1
// Since they are not in the same type, we keep converting using "Loosely compare strings to numbers" rule
// Finally, we would ended up with
const result = (100 == 1); // false
```

As we can see, with the exaple above, it is a real danger to loosely compare anything to a boolean because it is very easy to make a mistake. Since our original perception has always been "==" is just an equality operation that *does not care about type*. 
```js
var x = "100";
if (x) {
    // code reached this block since x is not falsy
}

if (x == true) {
    // code will NOT reach here

    // However, we could MISTAKENLY assume that code will reach here by the fact that x is truely value and == does not care about the operands type
}
``` 
<div style="text-align:center"> 
    <img src = "https://user-images.githubusercontent.com/11821799/43380753-73c5df30-93fc-11e8-865e-74d701f36d13.png"/>
<div/>
<div style="text-align:left"/><Br />

### Loosely compare null to undefined
Null and undefined loosely comparison are very simple. It could only be true if the other operand is also null or undefined
```js
const x = null;
const y;
const z = 0;

const result = (x == y) // true 
const otherResult = (x == z) // false, even z is falsy value
```
This type of loosely comparison might come in handy when you only want to handle null and undefined differently than other falsy values. 
```js
// follow codes would have the same behaviour
if (x === null || x === undefined){

}

if (x == null){

}
```

### Loosely compare Object with non-object value
In order to compare Object (also array and function) with other scalar primitive types, we need to help of ToPrimitive() abstract operation ([here to read more about ToPrimitive()](https://www.ecma-international.org/ecma-262/9.0/index.html#sec-toprimitive)). Basically, what ToPrimitive operation does is that it will take a look if the target object contains *valueOf* function as its property. If the return value of valueOf is a simple primitive value than object will be converted into that value. Else, if *toString* function is present somewhere in that object prototype chain the return value of toString() will be used as object converted value. If object, somehow, does not have the above methods anywhere in its prototype chain or both methods do not provide primitive value, *TypeError* would be thrown.

```js
// let's compare a number to array with one value
const x = 100;
const y = [100];

y.valueOf() // [100] -> not scalar primitive, it is still an array which is an Object
y.toString() // "100" -> yes ! it is scalar primitive, it is a string !

const result = (x == y) // true

// it would simply equal to 
const result = (100 == "100") // which matched with "Loosely compare strings to numbers"
```
## When It all make sense
After we have gone through all the rules that JavaScript use to perform *The Abstract Equality Comparison*, we now have a better understanding of "how" it is actually behave. To me, it would not be truely correct to tell a JS newbie that the "==" is the equality comparison that ignore "types" of the operand. I would rather, in short, say that it is the equality comparison that allow types of the operands to be coerced until both types are matched. And, very importantly, there is a concrete algorithm to follow. <Br />
<Br />
Let's again take a look at the meme I posted in the very beginning of this blog. Would you be able to now tell what it actually going on ?

<div style="text-align:center"> 
    <img width="300" src = "https://user-images.githubusercontent.com/11821799/43389783-bc7b3a08-9416-11e8-8e6c-0ed1cc816f50.png"/>
<div/>
<div style="text-align:left"/><Br />

## Happy Coding...

## References
* Crockford, Douglas. JavaScript: The Good Parts. O'Reilly Media.
* Simpson, Kyle. You Don't Know JS: Types & Grammar. O'Reilly Media. Kindle Edition. 
