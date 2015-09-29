# REL

REL is a simple script language built for API aggregation. It is possible to use REL's easy syntax to build cards for the [Relevant iOS App](http://relevant.ai).

This document is a basic introduction to its syntax.

## Sample Card and Structure

You may create cards using our [**Relevant platform** wizard](http://platform.relevant.ai). Once you make a card in the step-by-step wizard, you will be able to see and edit the code by clicking on the code button. **TODO: ADD IMAGE**.

Click here for an example of a full REL card which displays top content from Reddit. **TODO: ADD LINK**

A *card* in REL has two parts; the **metadata** and the **loading function**, preceeded by the words `meta` and `load` respectively, as follows

**Relevant Card Written in REL**
```swift
meta {
  title = "My Card"
  credits = "Myself"
}

load {
  card in
  // REL code that outputs the content of the card
  // from different services and APIs
}
```

The actual REL code goes in the `load` *closure* (a *closure* is just a block of code inside curly brackets `{}`, we'll explain this later in more detail).

`card` is the input of the load function. This variable has some special powers, as you'll see in The `card` Variable section. **TODO: LINK THIS**.

The return of the `load` closure is the visible content of the card, which must be an array of arrays. The outer array represents the horizontal slides of the card, and each of the inner arrays represent the *templates* that make up each slide (banners, rounded profile image, title+body description, footer, etc). Most templates follow the form below:

```swift
//...
  [
    "template-name":[
      "parameter":value,
      "another-parameter":anotherValue
      //...
    ]
  ]
//...
```
(Scroll all the way down or click here for a reference of all available templates. **TODO: LINK THIS**)

## Testing Cards

Cards created using the [**Relevant platform** wizard](http://platform.relevant.ai) have an *alias*, as shown below:

**TODO: ADD IMAGE**

To test it, you just need to type the alias into the search box of the Relevant App, and then hit the **Search** (or **Return**) button, as shown below:

**TODO: ADD IMAGE**

If the alias exists and the REL code produces no errors, you'll see the option to add this card to your deck. Tap **Add** and voilà!

**TODO: ADD IMAGE**

You can try adding this sample card immediately: **TODO: ADD ALIAS** before you create your own.

## Variables and Basic Syntax

REL is a *functional*, *inmutable* programming language. Don't worry if you don't understand these terms. What this means is that almost every line of code is either a *variable initialization* `var a = "hello world"`, or the *return statement of a function* `return "something"`. It also means that it is not possible to change the value of a variable after it has been initialized <s>`a = "ok bye"`</s>.

Variable types in REL are similar to JavaScript types; `"strings"`, numbers (`1`, `2`, `3.14`), `["a","r","r","a","y","s"]`, `null`, and dictionaries/objects. The main difference is that dictionaries/objects in REL are delimited by square brackets `[]`, and that the quotes `""` around the keys are mandatory:

**Dictionary/object**
```swift
var d = [ // <---- Square brackets
  "name":"Wircho",
  "age":29,
  "hobbies":["easy":["Sleeping","Eating"],"hard":["Code","Guitar","Math"]]
]
```

You can dig inside an array or dictionary with the usual square bracket syntax (`array[index]`, `dictionary[key]`), and you can also do so with lists or arrays of keys and indices. This means that for the dictionary `d` above, all of the following "`guitar`" variables have value `"Guitar"`:

```swift
var guitar0 = d["hobbies"]["hard"][1]
var guitar1 = d["hobbies","hard",1]
var guitar2 = d["hobbies","hard"][1]
var guitar3 = d["hobbies"]["hard",1]
var path = ["hobbies","hard",1]
var guitar4 = d[path]
```

## Built-In Functionality

REL comes equiped with an array of built-in functionality which will keep growing in future versions. This document does not intend to detail all of this functionality, but rather give a good enough preview for you to try it out.

### Operators

Most operators familiar to JavaScript developers are available in REL; namely `+,-,*,/,%,==,!=,&&,||,!`, among others, and their behaviour is very similar to that of JavaScript. For example, `+`'ing two numbers gives you their sum, while `+`'ing a string with any other variable gives you a string. Some examples follow:

```swift
var three = 1+2 // Produces 3
var threeString = "th" + "ree" // Produces "three"
var theTruth = (1 == 1) // Produces true (Parentheses are optional, here just for readability)
var aLie = (1 == 2) && (4 == 4) // Produces false
```

### Functions and Methods

Built-In REL *functions* are, like in other programming languages (and in math), prefixes that may take some one or more values as parameters and *return* another value. For example, the function `concat` takes any list or array of values and concatenates them into a string, doing so recursively;

```swift
var c = concat("Relevant"," - ",["The"," ","Missing ","Homescreen"]) // Produces "Relevant - The Missing Homescreen"
```

*Methods* are similar to functions, but are instead suffixed to values after a dot `.`;

```swift
var a = 55
var b = a.toString() // Produces "55"
var c = a.toString // Same as above
```

Many of the built in REL functions that take only one parameter, have an equivalent method form. For example `round(1.75)`, `(1.75).round()`, and `(1.75).round` are equivalent statements, all of which produce `2`. This is often useful when chaining several operations; e.g., `round(sqrt(count(foo)))` is equivalent to `foo.count.sqrt.round`.

## Control-Flow (if and for)

*Control-flow* refers to the ubiquitous **if-then-else** and **for** statements that we find in most popular programming languages. REL does not have control-flow per-se (this may change in the future), but it has a few functions that simulate it well enough. Instead of **if-then-else** statements, use the `then` method as follows;

**The `then` method**
```swift
var a = 1==2 // Produces false
var b = a.then("it's true!","it's false") // Produces "it's false!"
```
When the second argument is ommited, it falls back to `null`, so that `a.then("it's true")` simply produces null.

Instead of **for** statements, use the `loop` method applied to an array, after which you may simply append a closure taking one parameter (each item of the array), or two parameters (each item of the array along with its index). Some examples follow;

**The `loop` method - First example**
```swift
var b = [1,2,3].loop {
  item in
  return item * item
} // Produces [1,4,9]
```

**The `loop` method - Second example**
```swift
var b = [1,2,3].loop {
  (item,index) in
  return item * item * index
} // Produces [0,4,18]
```

The `loop` method is more of an array manipulation function than an actual control-flow statement.



## Inline Functions

Functions are defined the same way as variables `var f = ....`, and they defined by **closures**, i.e., blocks of code delimited by curly brackets. The following function returns the string `"foo"` every time:

**Function that returns `"foo"` every time**
```swift
var f = {
  return "foo"
}
var fooString = f() // Produces "foo"
var fooStringAgain = f() // Produces "foo"
```

If a function takes parameters, simply prefix its content with the list of parameter names and the keyword **`in`**

**Function that returns someone's full name**
```swift
var f = {
  (name,lastName) in // Parentheses are optional
  return name + " " + lastName
}
var fullName = f("John","Smith") // Produces "John Smith"
```

**TODO: SIMPLIFIED FUNCTION SYNTAX**

## The `card` Variable

## Relevant Card Templates




