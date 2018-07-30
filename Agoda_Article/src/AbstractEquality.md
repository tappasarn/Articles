# The myst of JavaScript's "==", something EsLint did not teach you.
When I first joined the world of the Web, as a newbie I was introduced to this language called "JavaScript". One of the things that make it so much different than other programming languages is the fact that you can achieve the same outcome by doing in many *different ways*. In most cases, you will not notice the different of those approches. But once the difference is revealed, most of the time it comes with some costs. To some developers those cost were great enough that Douglas Crockford dedicated a part of his "JavaScript: the good parts" book to talk about those bad (or maybe awful) parts. <Br />

Due to the reason above JavaScript has taken it place as one of the top programming languages that people like to make a meme about its behaviour.<Br />
This is one of many meme that I have recently shared with my beloved programmer friends
<div style="text-align:center"> 
    <img style="width: 300px" src = "https://user-images.githubusercontent.com/11821799/43389783-bc7b3a08-9416-11e8-8e6c-0ed1cc816f50.png"/>
<div/>
<div style="text-align:left"/><Br />
Nowadays, there is a config file call "EsLint" to keep you away from doing stupid things that you could have done. It keeps you source code secure. But it would not be such a good idea if we just take those rules for graned and stop seeking for to actual behaviour of such a syntax. <Br />

## “The Abstract Equality Comparison Algorithm”
(yeah... it is just a cool name for "==")<Br />
 
Not too long ago, I was reading through the JavaScript book series called You Don't Know JS written by Kyle Simpson. There was a chapter that actually explain about the losely equality comparison of JavaScript. It reminds me of the meme above that I have shared with my friends. So I think it would be nice if I could make a summarize and share it here as well.

Equality check is one amoung those things that could be done in more than a single ways, and one of the ways (believed to be evil) is banded by the power of EsLint. <Br />
<Br />
Similar to many other JS newbie what I understood about JS equality brothers (== and ===) was that the one with three equeal sign check for the type and the one with only two is evil and does not check type. But was that understanding were correctly intepreted.

```js
false == undefined // false
false == null // false
null == undefined // true

// on the other hand

const myLittleNull = null;

if (!myLittleNull) {
    // code reached this line
}
```
<div style="text-align:center"> 
    <img src = "https://user-images.githubusercontent.com/11821799/43380753-73c5df30-93fc-11e8-865e-74d701f36d13.png"/>
<div/>
<div style="text-align:left"/><Br />

What we mainly understand so far was that the little eval double equal does not care about the type then why would false not equal to falsy values like *null* or *undefined*. <Br />
<Br />
## How did that happen ?
The main difference between losely equality and strictly equality is that the strictly equality return false when the type of the operands are different, but the losely equlity will allow the coersion of one or both types of the operand unitll both of them ended up on the same type <Br />

## Type coersion
let's take a look at the algorithm that losely equality perform to corvert operand type. 
### Rule of thumb
As same as many other things in this universe there some rules that you will have to keep in mind. But before getting into those rules here are fact about JavaScript that I think worth to mention.
* NaN: In the world of JavaScript NaN is NOT equal to itself. To verify NaN use isNaN() or Number.isNaN().
* Object: Object (included with arrays and functions) will be the same if they have the same reference.
### Losely compare strings to numbers
```js
const x = "100";
const y = 100;

const result = (x == y); // true
```
The rules there is the *Number type has priority over string* and JS engine will try to convert the string operand to its number type. To make it easier to understand it would look somewhat like
```js
const x = "100";
const y = 100;

// since typeof Number(x) is "number"
// x and y are now in the same type
const result = Number(x) == y;
```

### Losely compare anything to boolean
```js
const x = "100";
const y = true;

const result = (x == y); // false
```

When you try to compare anything to boolean, firstly boolean value will get converted into its number form. Then JS will have to verify the type of those two operand, if they are the same the coersion is done, else keep coverting using other matched rules. <Br />
<Br /> 
Basically what happen in our example snippet was
```js
const x = "100";
const y = true;

// since y is boolean, it is the first one that get converted to its number type
// Number(false) will give 0, true will give 1
const result = (x == Number(y))

// right now we eould have result = ("100" == 1)
// since they are not in the same type, we keep converting using "Losely compare strings to numbers" rule
// finally we would ended up with
const result = (100 == 1); // false
```

As we can see with the exaple above, it is a real danger to compare anything to boolean because it is super easy to make a mistake. Since our original perception was that "==" is just an equality operation that *does not care about type*. 
```js
var x = "100";
if (x) {
    // code reached this block since x is not falsy
}

if (x == true) {
    // code will NOT reach here

    // However, we could MISTAKENLY assume that code will reach here by the fact that x is truely value and == dont not care about the type
}
``` 
<div style="text-align:center"> 
    <img src = "https://user-images.githubusercontent.com/11821799/43380753-73c5df30-93fc-11e8-865e-74d701f36d13.png"/>
<div/>
<div style="text-align:left"/><Br />

### Losely compare null to undefined
Null and undefined losely comparison are fairly simple. It could only be true if the other operand is also null or undefined
```js
const x = null;
const y;
const z = 0;

const result = (x == y) // true 
const otherResult = (x == z) // false, even z is falsy value
```
this might come it handy if u only want to handle null and undefined differently than other falsy values. 
```js
// follow code also would give you the same behaviour
if (x === null || x === undefined){

}
```

### Losely compare Object with non-object value
In order to compare Object (also array and function) with other scalar primitive type, we need to help of ToPrimitive() abstract operation (which can be found in more detail on ES5 spec section 9.1). But basically what ToPrimitive operation does is that it will take a look if the target object contain *valueOf()* if the return value of valueOf is primitive values than object will be converted to that value. Else, if *toString()* is present somewhere in that object prototype chain the return value of toString() will be used as object converted value. If object, somehow, does not have the above methods anywhere in its prototype chain or both method do not provide primitive value, TypeError would be thrown.

```js
// let's compare a number to array with one value
const x = 100;
const y = [100];

y.valueOf() // [100] -> not primitive
y.toString() // "100" -> yes ! it is primitive

const result = (x == y) // true

// it would simply equal to 
const result = (100 == "100") // which matched with "Losely compare strings to numbers"
```
## When It all make sense
After we have gone through all the rules that JavaScript use to perform *The Abstract Equality Comparison*, we now have then better understanding of "how" it is actually behave. To me, it would not be truely correct to tell a JS newbie that the "==" is the equality comparison that ignore "types" of the operand. I would rather, in short, say that it is the equality coparison that allow types of the operands to be coerced untill both types of the operand are matched.<Br />
<Br />
Let's again take a look at the meme I posted in the very beginning of this blog. Would you be able to now tell what it actually going on. 

<div style="text-align:center"> 
    <img style="width: 300px" src = "https://user-images.githubusercontent.com/11821799/43389783-bc7b3a08-9416-11e8-8e6c-0ed1cc816f50.png"/>
<div/>
<div style="text-align:left"/><Br />

## Happy Coding...

## References
* Crockford, Douglas. JavaScript: The Good Parts. O'Reilly Media.
* Simpson, Kyle. You Don't Know JS: Types & Grammar. O'Reilly Media. Kindle Edition. 
