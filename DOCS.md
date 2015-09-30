# REL

REL is a simple script language built for API aggregation. It is possible to use REL's easy syntax to build cards for the [Relevant iOS App](http://relevant.ai).

This document is a basic introduction to some of its features.

## Relevant Cards

### How to Build a Card

You may create cards using our [**Relevant platform** wizard](http://platform.relevant.ai). Once you make a card in the step-by-step wizard, you will be able to view and edit the code by clicking on the code button. **TODO: ADD IMAGE**.

Click here for an example of a full REL card which displays top content from Reddit.**TODO: DECIDE WHETHER REDDIT?**  **TODO: ADD LINK**

### Card Structure

A *card* in REL has two parts; the **metadata** and the **loading function**, preceeded by the words `meta` and `load` respectively, as follows

```swift
// Relevant Card Written in REL
meta {
  title = "My Card"
  credits = "Myself"
}

load {
  card in
  // REL code that outputs the content of the card
  // from different services and APIs
  // ...
}
```

The actual REL code goes in the `load` *closure* (a *closure* is just a block of code inside curly brackets `{}`, we'll explain this later in more detail).

`card` is the input of the load function. This variable has some special powers, as you'll see in The `card` Variable section. **TODO: LINK THIS**.

The return of the `load` closure is the visible content of the card, which must be an array of arrays. The outer array represents the horizontal slides of the card, and each of the inner arrays represent the *templates* that make up each slide (such as banners, rounded profile image, title+body description, footer, etc). Most templates follow the form below:

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

### Testing Cards

Cards created using the [**Relevant platform** wizard](http://platform.relevant.ai) have an *alias*, as shown below:

**TODO: ADD IMAGE**

To test it, you just need to type the alias into the search box of the Relevant App, and then hit the **Search** (or **Return**) button, as shown below:

**TODO: ADD IMAGE**

If the alias exists and the REL code produces no errors, you'll see the option to add this card to your deck. Tap **Add** and voilà!

**TODO: ADD IMAGE**

You can try adding this sample card immediately: **TODO: ADD ALIAS** before you create your own.

## REL Variables and Basic Syntax

REL is a *functional*, *inmutable* programming language. Don't worry if you don't understand these terms. What this means is that almost every line of code is either a *variable initialization* `let a = "hello world"`, or the *return statement of a function* `return "something"`. It also means that it is not possible to change the value of a variable after it has been initialized <s>`a = "ok bye"`</s>.

Variable types in REL are similar to JavaScript types; `"strings"`, numbers (`1`, `2`, `3.14`), `["a","r","r","a","y","s"]`, `null`, and dictionaries/objects. The main difference is that dictionaries/objects in REL are delimited by square brackets `[]`, and that the quotes `""` around the keys are mandatory:

```swift
// A dictionary/object
let d = [ // <---- Square brackets
  "name":"Wircho",
  "age":29,
  "hobbies":["easy":["Sleeping","Eating"],"hard":["Code","Guitar","Math"]]
]
```

You can dig inside an array or dictionary with the usual square bracket syntax (`array[index]`, `dictionary[key]`), and you can also do so with lists or arrays of keys and indices. This means that for the dictionary `d` above, all of the following "`guitar`" variables have value `"Guitar"`:

```swift
let guitar0 = d["hobbies"]["hard"][1]
let guitar1 = d["hobbies","hard",1]
let guitar2 = d["hobbies","hard"][1]
let guitar3 = d["hobbies"]["hard",1]
let path = ["hobbies","hard",1]
let guitar4 = d[path]
```

## Basic Operators, Functions and Methods

REL comes equiped with an array of built-in functionality which will keep growing in future versions. This document does not intend to detail all of this functionality, but rather give a good enough preview for you to try it out.

### Operators

Most operators familiar to JavaScript developers are available in REL; namely `+,-,*,/,%,==,!=,&&,||,!`, among others, and their behaviour is very similar to that in JavaScript. For example, `+`'ing two numbers gives you their sum, while `+`'ing a string with any other variable gives you a string. Some examples follow:

```swift
let three = 1+2 // Produces 3
let threeString = "th" + "ree" // Produces "three"
let theTruth = (1 == 1) // Produces true (Parentheses are optional, here just for readability)
let aLie = (1 == 2) && (4 == 4) // Produces false
```

### Built-In Functions and Methods

Built-In REL *functions* are, like in other programming languages (and in math), prefixes that may take zero or more values as parameters, and *return* another value. For example, the function `concat` takes any list or array of values and concatenates them into a string, doing so recursively;

```swift
let c = concat("Relevant"," - ",["The"," ","Missing ","Homescreen"]) // Produces "Relevant - The Missing Homescreen"
```

*Methods* are similar to functions, but are instead suffixed to values after a dot `.`;

```swift
let a = 55
let b = a.toString() // Produces "55"
let c = a.toString // Same as above
```

Many of the built in REL functions that take only one parameter, have an equivalent method form. For example `round(1.75)`, `(1.75).round()`, and `(1.75).round` are equivalent statements, all of which produce `2`. This is often useful when chaining several operations; e.g., `round(sqrt(count(foo)))` is equivalent to `foo.count.sqrt.round`.

## Control-Flow (if-then-else and for-loop)

*Control-flow* refers to the ubiquitous **if-then-else** and **for** statements that we find in most popular programming languages. REL **does not** have control-flow per-se, but it has a few functions that simulate it well enough.

### The `then` Method

In place of **if-then-else** statements, use the `then` method as follows;

```swift
let a = 1==2 // Produces false
let b = a.then("it's true!","it's false!") // Produces "it's false!"
```
When the second argument is ommited, it falls back to `null`, so that `a.then("it's true")` simply produces `null`.

### The `loop` Method

In place of **for** loops, use the `loop` method on an array, after which you may simply append a closure taking one parameter (each item of the array), or two parameters (each item of the array along with its index). Some examples follow;

```swift
// The loop method. First example.
let b = [1,2,3].loop {
  item in
  return item * item
} // Produces [1,4,9]
```

```swift
// The loop method. Second example.
let b = [1,2,3].loop {
  (item,index) in
  return item * item * index
} // Produces [0,4,18]
```

The `loop` method is more of an array manipulation function than an actual control-flow statement.

**Note on trailing closures:** `loop` is actually a method that takes one closure parameter, so that the expressions `a.loop{...}` and `a.loop({...})` are equivalent. A *trailing* closure is permitted when a closure is the last parameter of a function or method.

## String Manipulation

Strings in REL can be concatenated using `+` or the `concat` function (which takes any number of string or array arguments). Some other string manipulation functions and methods are exemplified below:

### `join`, `splitBy`, `lowercase`, `uppercase`, `matches`, and `replace`

See examples below:

```swift
let a = ", ".join([1,2,3]) // Produces "1, 2, 3"
let b = "2015-09-29".splitBy("-") // Produces ["2015","09","29"]
let c = "Hello World".lowercase // Produces "hello world"
let d = "Hello World".uppercase // Produces "HELLO WORLD"
let e = "Hello World".matches("llo Wo") // Produces true
let f = "Hello World".replace("He","hE") // Produces "hEllo World"
let g = "Hello World".replace("[a-zA-Z]","*",true) // Produces "***** *****". The last argument 'true' means that it should use regular expressions
```

Some of these methods also have equivalent function forms:

```swift
let a = join(", ",[1,2,3]) // Produces "1, 2, 3"
let c = lowercase("Hello World") // Produces "hello world"
let d = uppercase("Hello World") // Produces "HELLO WORLD"
let h = toString(55) // Produces "55"
```

### Rich Text: `small`, `bold`, `italic`, and `color`

Because the output of a REL function is often used for skinning user interfaces, we occasionally need to output rich text. For this purpose we may assume that all strings in REL have an abstract *default* font. The following methods simply apply transformatons to that font. The results of these methods are always rich text, and may fail to perform some string manipulation methods such as `join`. Other functions like `concat` or the `+` operator do work properly on rich text.

The `small` method/function makes the font size about `0.8` times smaller. Examples: `"Hello World".small`, `small("Hello World")`.

The `bold` and `italic` methods/functions make text bold and italic respectively.

The `color` method allows you to color a given text. Example: `"Hello World".color("red")`. Currently available colors are `"red"`, `"pink"`, `"purple"`, `"green"`, `"yellow"`, `"orange"`, `"blue"`, `"cyan"`, `"gray"`, `"dark-gray"`, `"light-gray"`, `"white"`, and `"transparent"`.

## Array Manipulation

### `merge` and `mergeArray`

The two main array manipulation functions in REL are `merge` and `mergeArray`. They do pretty much the same, except the first one takes a list of arrays separated by commas, while the second one takes **only one parameter**: an array of arrays. This is better understood from the examples below:

```swift
let a = merge([[1,2],[3,4],[5,6]],["x","y","z"],["p","q","r"]) // Produces [[1,2],[3,4],[5,6],"x","y","z","p","q","r"]
let b = merge([[1,2],[3,4],[5,6]],["x","y","z"]) // Produces [[1,2],[3,4],[5,6],"x","y","z"]
let c = merge([[1,2],[3,4],[5,6]]) // Produces [[1,2],[3,4],[5,6]] (no change)
let d = mergeArray([[1,2],[3,4],[5,6]]) // Produces [1,2,3,4,5,6]
```

### `count`, `append`, `reverse`, `subarray`, and `group`

See examples below:

```swift
let l = ["x","y","z"].count // Produces 3
let m = ["x","y","z"].append("aa") // Produces ["x","y","z","aa"]
let n = ["x","y","z"].reverse // Produces ["z","y","x"]
let p = ["t","u","v","w","x","y","z"].subarray(2,4) // Produces ["v","w","x"]
let q = ["t","u","v","w","x","y","z"].subarray(4) // Produces ["t","u","v","w"]
let r = ["t","u","v","w","x","y","z"].subarray(-4) // Produces ["w","x","y","z"]
let g = ["t","u","v","w","x","y","z"].group(3) // Produces [["t","u","v"],["w","x","y"],["z"]] (groups 3 by 3)
```

### `loop` and `filter`

It is often necesary to `loop` an array. For this refer to the Control-Flow section above **TODO: LINK THIS**. Similarly, you can filter elements of an array using the `filter` method:

```swift
// The filter method - First example
let b = [7,20,5,4,50].filter {
  item in
  return item < 10
} // Produces [7,5,4]
```

```swift
// The filter method - Second example
let b = [7,20,5,4,50].filter {
  (item,index) in
  return index >= 2
} // Produces [5,4,50]
```

### `sortAlpha` and `sortNum`

The `sortAlpha` and `sortNum` methods take a closure that maps every item of an array into a string or number value. The result is a new array sorted by that value, alphabetically or numerically, respectively. Examples follow;

```swift
let a = [["device":"iPhone","mill":500],["device":"iPad","mill":170],["device":"iPod","mill":450]].sortAlpha {
  item in
  return item["device"]
} // Produces [["device":"iPad","mill":170],["device":"iPhone","mill":500],["device":"iPod","mill":450]]
```

```swift
let a = [["device":"iPhone","mill":500],["device":"iPad","mill":170],["device":"iPod","mill":450]].sortNum {
  item in
  return item["mill"]
} // Produces [["device":"iPad","mill":170],["device":"iPod","mill":450],["device":"iPhone","mill":500]]
```

## Dictionary Manipulation

### `blend` and `blendArray`

Just like arrays can be merged together with the `merge` and `mergeArray` methods, dictionaries can be blended together with the `blend` and `blendArray` methods. The first one takes a list of dictionaries separated by commas, while the second one takes an array of dictionaries. For example

```swift
let a = blend(["a":1,"b":2],["c":3]) // Produces ["a":1,"b":2,"c":3]
let b = blendArray([["a":1,"b":2],["c":3]]) // Produces ["a":1,"b":2,"c":3]
```
## Web APIs (`get`, `post`, and `request`)

These functions allow you to communicate with web services and API's. The most general way to do this is using the `request function`, formatted as follows;

```swift
var a = request(<method>,<url>,<body>,<headers>,<response type>)
```

Where;

`<method>` is a string representing an HTTP method, such as `"GET"` or `"POST"`.
`<url>` is the URL to be queried.
`<body>` is **a dictionary** of keys and values that will be appended to your request.
`<headers>` is **a dictionary** of your request's HTTP headers.
`<response type>` is either `"string"` or `"json"` (case insensitive). The latter means that the result of the request must be parsed into a JSON variable. This parameter defaults to `"json"`.

Similarly you may use the functions `get` and `post` for GET and POST requests respectively;

```swift
var a = get(<url>,<body>,<headers>,<response type>)
var b = post(<url>,<body>,<headers>,<response type>)
```

You may ommit any number of parameters at the end, for example;

```swift
var a = get("some-fake-website.com/something.json") // Fetches the contents as a JSON variable
```

## Device APIs (time and location)

### Date + Time (`getTime()`, `timestamp`, `dateString`, `dateObject`)

Use `getTime()` to get the device's current date and time. The result of this function is a date object, and needs to be converted into a usable value. One way to do this is by using the `timestamp` method;

```swift
let a = getTime().timestamp // Produces the current time in seconds since 00:00:00 UTC on 1 January 1970
```

A date can also be converted into a user-ready format with date formats **TODO: LINK HERE**. For example:

```swift
let b = getTime().dateString("yyyy-MM-dd") // Produces the current date in the format 2015-09-30
```

Furthermore, a date string can be parsed into a date object if the format of the string is known. For example:

```swift
let c = "2015-09-30, 12:05".dateObject("yyyy-MM-dd, HH:mm").timestamp // Produces 1443614700
```

### User Location (`getLocation()`, `isLocationAvailable()`)

Use `getLocation()` to get the user's current location in the format `["latitude":45.501262,"longitude":-73.560347]`. This method will return nil if the user does not allow Relevent to access their location. Use `isLocationAvailable()` to verify whether Relevant is allowed to access location.

## Inline Functions

You may define your own functions the same way as variables `let f = ....`, by using **closures** (blocks of code delimited by curly brackets `{}`). The following function returns the string `"foo"` every time:

```swift
// This function returns "foo" every time
let f = {
  return "foo"
}
let fooString = f() // Produces "foo"
let fooStringAgain = f() // Produces "foo"
```

If you need your function to take parameters, simply prefix its content with the list of parameter names and the keyword **`in`**

```swift
// This function returns someone's full name given their first and last name
let f = {
  (name,lastName) in // Parentheses are optional
  return name + " " + lastName
}
let fullName = f("Wendy","Smith") // Produces "Wendy Smith"
```

### Simplifying Inline-Function Syntax

The `parameters in` header may be omitted to simplify a function's syntax. In this case the parameters may be accessed with the variables `$0`, `$1`, `$2` ... within the function's closure. For example;

```swift
// This function returns the sum of the squares of two numbers
let f = { return $0*$0 + $1*$1 }
```

Furthermore, the `return` keyword may be ommited, as REL functions always return the last statement in their closures:

```swift
// This function is equivalent to the one above
let f = { $0*$0 + $1*$1 }
```

These simplification mechanisms are particularly useful when chaining methods that take trailing closures. For example;

```swift
let a = [1,4,9,16,25,36].filter{ $0 % 2 == 0 }.map{ sqrt($0) }.map{ $0/2 } // Produces [1,2,3]
```

## The `card` Variable

As we mentioned in the Card Structure section **TODO: LINK HERE** at the beginning, the `load` closure of a Relevant card has one parameter named `card`. This is a reference to the card instance itself and as such, it has access to some special methods.

### `card.settings`

The return of this method is the value entered by the user after flipping the card (tapping on the button at the top-right corner).

If you flip a basic card built using the [**Relevant platform** wizard](http://platform.relevant.ai), you will see nothing at the back of the card.

To allow your card to have settings, you need to include the `meta` variable `settingsType`:

```swift
meta {
  // ...
  settingsType = "text" // Possible values are "text", "select", and "none" ("none" is equivalent to not having a settingsType variable)
  // ...
}
```

For `settingsType = "text"`, you may also include a string `settingsDefault`, which will be the default value once the card is added to the deck:

```swift
meta {
  // ...
  settingsType = "text" // Possible values are "text", "select", and "none" ("none" is equivalent to not having a settingsType variable)
  settingsDefault = "Canada"
  // ...
}
```

For `settingsType = "select"` the rules are a bit more complicated. You need to include a `settingsOptions` value which must be an array of dictionaries. Each of these dictionaries must have at least a `"value"` key, which will be then visible in the settings drop-down menu. You also need one of this dictionaries to be the values of `settingsDefault`. For example:

```swift
meta {
  // ...
  settingsType = "select"
  settingsOptions = [
    [
      "value":"Canada",
      "other-info":"Additional Information"
      "even-more-info":"So much Info" // "other-info" and "even-more-info" will not be visible to the user, but still accessible from card.settings
    ],
    [
      "value":"United States",
      "other-info":"Something Else"
      "even-more-info":"So Much Else"
    ]
  ]
  settingsDefault = [
    "value":"Canada",
    "other-info":"Additional Information"
    "even-more-info":"So much Info"
  ]
  // ...
}
```

## Relevant Card Templates

As we said in the Card Structure section **TODO: LINK THIS**, the `return` of the `load` closure has to be an array of arrays, and the inner arrays represent the templates of each slide of the card. For example, a simple card with three slides may look like this:

```swift
meta {
  title = "My Card"
}
load {
  card in
  
  var info = [
    ["name":"Red", "color":"red", "rgb-values":[231,76,60]],
    ["name":"Green", "color":"green", "rgb-values":[112,173,75]],
    ["name":"Blue", "color":"blue", "rgb-values":[41,128,185]]
  ]
  
  return info.loop {
    item in
    
    return [
      [ // First template of each slide
        "banner":[
          "color":item["color"]
        ]
      ],
      [ // Second template of each slide
        "description":[
          "title":item["name"]
          "body":"RGB: (" + ",".join(item["rgb-values"]) + ")"
        ]
      ],
      [ // Third template of each slide
        "footer":[
          "caption":("Sample Text in " + item["name"]).bold.color(item["color"])
        ]
      ]
    ]
  }
}
```
**TODO: test that card!!!**

Observe that each template is a dictionary with **one single key**. This key is the template's name, and its value defines its parameters. Here is a list of basic templates:





## User Actions (`webView`, `mapView`, `refresh`, `deeplink`)



