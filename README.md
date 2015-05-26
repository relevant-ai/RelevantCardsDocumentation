# Relevant Cards Documentation

This document explains how to create cards for the [Relevant iOS app](http://relevant.ai).

A *card* is just a JSON file hosted somewhere on the web. This JSON file directs the Relevant servers on how to gather the data and display it using templates such as banner images, footers, etc.

A great way to host a card is using Dropbox.

## Intro and Hello World

Here is a sample *"Hello World"* card:

```json
{
    "id": "hello-world",
    "title": "Hello World Card",
    "icon_url": "http://relevant.ai/hello_world.png",
    "summary": "Card's summary for the library.",
    "credits": "Relevant",
    "settings_type": "NONE",
    "_LOAD": {
        "_RETURN":[
            [
                {
                    "description":{
                        "title": "Hello World",
                        "body": "Hello World? There's a card for that."
                    }
                },
                {
                    "footer":{
                        "caption": "Relevant - The Missing Home Screen"
                    }
                }
            ]
        ]
    }
}
```

The keys `id`, `title`, `icon_url`, `summary`, `credits`, and `settings-type` are metadata. Of these, only the `title` value is visible on the card itself.

The `"_LOAD"` key represents the content and appearance of the card.

**To test this card;** launch the [Relevant iOS app](http://relevant.ai), shake your device, and when prompted, insert the following URL into the input box:

`https://gist.githubusercontent.com/wircho/f250e0ae9f818637c9c5/raw/d2ebeb8306f7520f60f5f318bc721eb378dd4fe1/card`

You should see the following card:

![Hello World Sample Card](https://raw.githubusercontent.com/relevant-ai/RelevantCardsDocumentation/master/hello_world_card.png)

Think of the object `_LOAD` as a function that is called every time the card needs to refresh. The `_RETURN` of this function is an array with **only one element**. That means this card has **only one page** (swiping left and right will do nothing). This page is itself an array of **templates**: A `description` template with parameters `title` and `body`, and a `footer` template with parameter `caption`.

**Remark:** The `_RETURN` of the `_LOAD` function must always be an array of arrays. Each element of the outer array is a *page* and each element of the inner array is a *template*.

**Remark:** As of Relevant 1.0, cards **do not update automatically** when you change the code. After making changes to your hosted JSON, you need to delete the current card (tap the "i" button, then REMOVE) and then add it again by shaking the device.

## The REL Language

Everything inside the `_LOAD` function must follow the syntax of the REL language. Reserved words/keys for the REL language are uppercase and preceded by an *underscore* (`_`). Everything else is what it looks like: a JSON object, array, string, or number.

### Variables

You can define variables inside the `_LOAD` function as follows:

```json
"_LOAD":{
    "foo":5,
    "var":[1,"a",{"b":"c"}],
    "norf":{"x":50.1,"y":-70},
    "_RETURN": ...
}
```

Normally, unless used somewhere in `_RETURN`, these variables will be ignored. This makes REL *lazy*; it only does things if needed and when needed.

The easiest way to use a variable is using the syntax `"{variable_name}"`. For example, the Hello World card from before is equivalent to the one below:

```json
{
    "id": "hello-world",
    "title": "Hello World Card",
    "icon_url": "http://relevant.ai/hello_world.png",
    "summary": "Card's summary for the library.",
    "credits": "Relevant",
    "settings_type": "NONE",
    "_LOAD": {
        "hello":"Hello World",
        "foo":{
            "description":{
                "title": "{hello}",
                "body": "{hello}? There's a card for that."
            }
        },
        "var":{
            "footer":{
                "caption": "Relevant - The Missing Home Screen"
            }
        },
        "_RETURN":[
            [
                "{foo}",
                "{var}"
            ]
        ]
    }
}
```

In this card the `_LOAD` function has three intermediate variables, named `hello`, `foo`, and `var`.

### Inserting/Concatenating String Variables

As in the previous example, it is possible to insert a string-valued variable into another string using the syntax `"... {variable_name} ..."`. For example `"The value of the variable foo is {foo}"`.

This is only possible when `foo` is a string, and will not work otherwise.

### Built-In Functions

The REL language comes packaged with a set of built-in functions to help you easily manipulate JSON objects. All built-in function names follow the *underscore uppercase* syntax (e.g. `_CONCAT`) to differentiate it from your own variables and JSON objects.

A built-in function call is usually represented by a one-key object whose only key is the function name. For example:

```json
{
    "_FUNC_NAME": ...
}
```

The value of the key is the parameter of the function. This could be a string, an array, an object, or in some cases a set of parameters with specific names, as follows:

```json
{
    "_FUNC_NAME": {
        "_FIRST_PARAM_NAME": ...,
        "_SECOND_PARAM_NAME": ...,
        "_THIRD_PARAM_NAME": ...
    }
}
```

Most of the rest of this documentation is devoted to introducing REL's built in functions.

### The `_PATH` function

We often need to dig deep inside a JSON object or array, in order to fetch a specific value. This will be more useful when we get JSON objects following all kinds of different standards from web services (See the `_URL` function below).

The `_PATH` function takes an array of strings and integers. The first string is the name of a variable. Every subsequent string or integer represents either an object's key, or an array's index. The result of the function call is obtained by recursively looking up these keys and indices into the variable. Whenever a look up fails, the function `_PATH` returns `null`.

In the example bellow, the variable `var` takes the value `"Hello World"`

```json
"foo":{
    "byes":[
        {"value":"Adios Mundo","language":"Spanish"},
        {"value":"Bye World","language":"English"}
    ],
    "hellos":[
        {"value":"Hola Mundo","language":"Spanish"},
        {"value":"Hello World","language":"English"}
    ]
},
"var":{
    "_PATH":["foo","hellos",0,"value"]
}
```

**Remark:** Observe that `{"_PATH":["variable_name"]}` is equivalent to `"{variable_name}"`.

### The `_PATH_EXISTS` Function

Like `_PATH`, this function takes an array of strings and integers. It returns `true` (`1`) if this array represents a path that exists, and `false` (`0`) otherwise. For example:

```json
"object": {
    "key_1": [
        "element #1",
        [
            "element",
            "#",
            "2"
        ]
    ],
    "key_2": {
        "red": 255,
        "green": 255
    }
},
"another_object": {
    "something": "{object}"
},
"a": {
    "_PATH_EXISTS":["another_object","something","key_1",1,1]
},
"b": {
    "_PATH_EXISTS":["another_object","something","key_1",1,2]
},
"c": {
    "_PATH_EXISTS":["another_object","something","key_2","blue"]
}
```

In the code above, the variables `a` and `b` are `1` (`true`), and the variable `c` is `0` (`false`).

### The `_IF`, `_THEN`, `_ELSE` Function

Unlike other built-in functions, this one is represented by an object with three keys, rather than one. It works just as you would expect it in any programming language. In the example below, the variable `city` takes the value `"Montreal"`.

```json
"foo":true,
"cities":["Montreal","Toronto"],
"city":{
    "_IF":"{foo}",
    "_THEN":{"_PATH":["cities",0]},
    "_ELSE":{"_PATH":["cities",1]}
}
```

It is important to keep in mind that `_IF,_THEN,_ELSE` is a function, and not a construct, as in other programming languages. That means `_THEN` and `_ELSE` contain **values** to be *returned*, rather than **statements** to be *executed*.

However, you may still wish to use intermediate variables inside `_THEN` and `_ELSE`. This is possible as long as they have a `_RETURN` key, as in the example below:

```json
"foo":true,
"cities":["Montreal","Toronto"],
"city":{
    "_IF":"{foo}",
    "_THEN":{
        "city":{"_PATH":["cities",0]},
        "_RETURN":"{city}"
    },
    "_ELSE":{"_PATH":["cities",1]}
}
```

**Remark for scope geeks:** REL uses block dynamic scope, and all variables are set only once (this is called *inmutability*). In particular that means that the variable `city` inside `_THEN` is a local variable, and does not overwrite the other equally named variable.

### The `_URL` Function

Allows you to make a call to a web service that returns a JSON object. To test this function we're using the JSON from this URL:

`https://gist.githubusercontent.com/wircho/d6c606350f8a6dca29ee/raw/b7872ecebfcc868ff825c88fe1c5e06d13ad618c/cities`

```json
{
    "cities":[
        {
            "picture":"http://media-cdn.tripadvisor.com/media/photo-s/03/9b/2f/df/montreal.jpg",
            "name":"Montreal",
            "nickname":"The City Of Saints",
            "province":"Quebec"
        },
        {
            "picture":"http://images.hellobc.com/mgen/tbccw/production/TBCCWDisplay.ms?img=/getmedia/4149a8de-fc5d-4e54-b659-5282486fa973/2-200291313-001-Vancouver-skyline.jpg.aspx&tl=1&sID=1&c=public,max-age=172802,post-check=7200,pre-check=43200&bid=4_5",
            "name":"Vancouver",
            "nickname":"Hollywood North",
            "province":"British Columbia"
        },
        {
            "picture":"http://www.seetorontonow.com/App_Themes/tourismtoronto/images/about-tourism-toronto.jpg",
            "name":"Toronto",
            "nickname":"T-Dot",
            "province":"Ontario"
        }
    ]
}
```

The card below exemplifies the use of the `_URL` function. You can test it by shaking your phone and adding the card `https://gist.githubusercontent.com/wircho/327fef8392c1865dec5e/raw/5aa7e97fa166888c3bd8d0f38c4b062deebc6819/city_card`

```json
{
    "id": "city-card",
    "title": "City",
    "icon_url": "http://relevant.ai/city.png",
    "summary": "Card's summary for the library.",
    "credits": "The Web",
    "settings_type": "NONE",
    "_LOAD": {
        "json_info":{
            "_URL":"https://gist.githubusercontent.com/wircho/d6c606350f8a6dca29ee/raw/b7872ecebfcc868ff825c88fe1c5e06d13ad618c/cities"
        },
        "first_city":{
            "_PATH": ["json_info","cities",0]
        },
        "_RETURN":[
            [
                {
                    "banner":{
                        "image": {"_PATH":["first_city","picture"]}
                    }
                },
                {
                    "description":{
                        "title": {"_PATH":["first_city","name"]},
                        "body": {"_PATH":["first_city","nickname"]}
                    }
                },
                {
                    "footer":{
                        "caption": {"_PATH":["first_city","province"]}
                    }
                }
            ]
        ]
    }
}
```

![City Sample Card](https://raw.githubusercontent.com/relevant-ai/RelevantCardsDocumentation/master/montreal_card.png)

As the Hello World card, this card has only one page. See the `_LOOP` function below to learn how to display all cities from the web service.

You can also make richer requests using `_URL`, as exemplified below:

```json
"var":{
    "_URL":{
        "_ADDRESS":"http://some/web/address.html",
        "_METHOD":"POST",
        "_BODY":"a=something&b=something%20else",
        "_HEADERS": {
            "Some Header": "Some Value",
            "Another Header": "Another Value"
        }
    }
}
```

**Remark:** The `_URL` function may return an error object if, for example, there is no internet connection. In most cases you can safely ignore this, and the card will just display the error. Read **Error Handling** below to learn how to handle errors.

### The `_LOOP` Function

This function loops an array and processes each of its items to produce a new array. For example, the variable `var` below takes the value `[1,2,3,4]`:

```json
"foo":[{"num":1,"name":"one"},{"num":2,"name":"two"},{"num":3,"name":"three"},{"num":4,"name":"four"}],
"var":{
    "_LOOP":{
        "_ARRAY":"{foo}",
        "_EACH":{"_PATH":["_ITEM","num"]}
    }
}
```

`_ARRAY` and `_EACH` are the parameter names of the function `_LOOP`, and must always be present. The implicit variable `_ITEM` exists only within the `_EACH` object, and represents the current element of the array. You may also use the implicit variable `_INDEX`, which is the current index between `0` and the array's length.

You may use intermediate variables inside `_EACH` as long as there is a `_RETURN` key. In the example below, the variable `var` takes the value `["1 is one","2 is two","3 is three","4 is four"]`:

```json
"foo":[{"num":1,"name":"one"},{"num":2,"name":"two"},{"num":3,"name":"three"},{"num":4,"name":"four"}],
"var":{
    "_LOOP":{
        "_ARRAY":"{foo}",
        "_EACH":{
            "num":{"_PATH":["_ITEM","num"]},
            "name":{"_PATH":["_ITEM","name"]},
            "_RETURN":"{num} is {name}"
        }
    }
}
```

We may use `_LOOP` to improve upon the card exemplifying the `_URL` function above. Test this improved card by shaking your phone and inputing this URL: `https://gist.githubusercontent.com/wircho/c8f4f5b0ce440b8edd83/raw/8ef09440e96322e75220ff1470fbc4c65d76e6c2/cities_card`

```json
{
    "id": "cities-card",
    "title": "Cities",
    "icon_url": "http://relevant.ai/cities.png",
    "summary": "Card's summary for the library.",
    "credits": "The Web",
    "settings_type": "NONE",
    "_LOAD": {
        "json_info":{
            "_URL":"https://gist.githubusercontent.com/wircho/d6c606350f8a6dca29ee/raw/b7872ecebfcc868ff825c88fe1c5e06d13ad618c/cities"
        },
        "_RETURN":{
            "_LOOP":{
                "_ARRAY":{"_PATH":["json_info","cities"]},
                "_EACH":[
                    {
                        "banner":{
                            "image": {"_PATH":["_ITEM","picture"]}
                        }
                    },
                    {
                        "description":{
                            "title": {"_PATH":["_ITEM","name"]},
                            "body": {"_PATH":["_ITEM","nickname"]}
                        }
                    },
                    {
                        "footer":{
                            "caption": {"_PATH":["_ITEM","province"]}
                        }
                    }
                ]
            }
        }
        
    }
}
```

This is how the card above looks like after scrolling all the way to the right:

![Cities Sample Card](https://raw.githubusercontent.com/relevant-ai/RelevantCardsDocumentation/master/toronto_card.png)

### Long Step `_LOOP`

The built-in function `_LOOP` may also take an optional positive parameter `_STEP`. When this parameter is a number *n* greater than 1, the implicit variable `_ITEM` in `_EACH` becomes an array itself, having at most *n* elements. In the following example, the variable `var` takes the value `[[1,2,3],[4,5,6],[7,8]]`:

```json
"foo":[1,2,3,4,5,6,7,8],
"var":{
    "_LOOP":{
        "_ARRAY":"{foo}",
        "_STEP":3,
        "_EACH":"{_ITEM}"
    }
}
```

### Filtering Arrays Using `_LOOP`

The example below shows that it is possible to filter out some elements of an array using `_LOOP`, simply by returning `null` in `_EACH`. The variable `var` takes the value `[1,3,4,5]`.

```json
"foo":[true,false,true,true,true,false],
"var":{
    "_LOOP": {
        "_ARRAY": [1,2,3,4,5,6],
        "_EACH" : {
            "_IF":{"_PATH":["foo","{_INDEX}"]},
            "_THEN":"{_ITEM}",
            "_ELSE":null
        }
    }
}
```

### The `_SORT` Function

This function sorts an `_ARRAY` parameter. The parameter `_BY` defines the sorting condition. As in `_LOOP`, the elements of the array are represented by the implicit local variable `_ITEM` inside `_BY`.

```json
"var":{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY": {"_PATH":["_ITEM","info","number"]}
    }
}
```

It is possible to reverse the ordering using `"_REVERSE":true`:

```json
"var":{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY": {"_PATH":["_ITEM","info","number"]},
        "_REVERSE":true
    }
}
```

Sorting by a comparison function is just as easy, except the key is now `_BY_FUNCTION` and it holds an expression depending on two implicit local parameters. The local parameters are keyed `"_A"` and `"_B"` and represent two arbitrary items in the `"_ARRAY"`. The function/expression must return `1` (`true`) if the items should be ordered `"_A","_B"` and `0` (`false`) if they should be ordered `"_A","_B"`

See the `_MATH` and `_CONCAT` functions to understand this example.

```json
"var":{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY_FUNCTION": {
            "_MATH":{
                "_CONCAT":[
                    {"_PATH":["_A","info","number"]},
                    "<",
                    {"_PATH":["_B","info","number"]}
                ]
            }
        }
    }
}
```

### Other Array Functions

**`_COUNT`** Takes an array and returns its number of elements. All throughout REL, null elements are ignored from computed objects and arrays, and so they don't count towards an array's `_COUNT`.

**`_FOR`** (for loop) Similar to `_LOOP` but with parameters `_FOR,_TO,_EACH`, and implicit variable `_INDEX` within `_EACH`. For example, `{"_FOR":{"_FROM":0,"_TO":10,"_EACH":"{_INDEX}"}}` returns the array `[0,1,2,3,4,5,6,7,8,9,10]`. You can use the optional parameter `_STEP` as well.

**`_MERGE`:** Takes an array of objects, strings, or arrays, and returns an array containing each of those objects, strings, and array elements. For example, the variable `var` below takes the value `[1,2,3,4,5,6,7,8,9]`

```json
"var":{
    "_MERGE":[
        1,
        [
            2,
            3,
            4,
            5
        ],
        6,
        7,
        [
            8,
            9
        ]
    ]
}
```

**`_FIRST_NOT_NULL`** Returns the first not null element of a passed array. It computes elements one by one, stopping when it first finds a not null value.

**`_FIRST_NOT_EMPTY`** Same as above but also rejects empty strings.

**`_FIRST_NOT_ERROR`** Same as `_FIRST_NOT_NULL` but also rejects errors.

### The `_MATH` Function

This function takes a string representing a mathematical expression, and returns the resulting number. In the example below, the variable `var` takes the value `2015`:

```json
"x":1000,
"y":1,
"z":1,
"var":{"_MATH":"{x}+{x}+({y}+{z})*7.5"}
```

### Basic String Functions

Basic string functions are built in functions that take a string and return a string. Some available basic string functions are listed below:

**`_URL_ENCODE`:** Percent encoding for URL parameters.

**`_URL_DECODE`:** Inverse of above.

**`_UPPERCASE`:** Convert a String to uppercase.

**`_LOWERCASE`:** Convert a String to lowercase.

### Other String Functions

**`_CONCAT`:** Takes an array of strings (or numbers) and concatenates them to form a string.

**`_JOIN`:** Takes an array of strings (or numbers) and joins them by a given string. For example, the variable `var` below takes the value `"a-b-c"`:

```json
"foo":["a","b","c"],
"var":{
    "_JOIN":{
        "_ARRAY":"{foo}",
        "_WITH":"-"
    }
}
```

**`_SPLIT`:** This function takes a string `_STRING` and a separator string `_SEPARATOR`. It returns the array that results from splitting the string by the separator. For example, the variable `var` below takes the value `[a,b,c,d]`:

```json
"var":{
    "_SPLIT": {
        "_STRING": "a, b, c, d",
        "_SEPARATOR": ", ",
    }
}
```

**`_REPLACE`:** Takes parameters `_STRING`, `_SEARCH` and `_REPLACEMENT`. `_STRING` is a string. `_SEARCH` and `_REPLACEMENT` must be strings or equally long arrays of strings. Returns a string that results from replacing each ocurrence of a string from `_SEARCH` in `_STRING` with the corresponding element in `_REPLACEMENT`. An optional parameters `"_REGEX":true` may be used for regular expression search. In both examples below, the variable `var` takes the value `"ABC"`:

```json
"foo":"A123B123C123",
"var":{
    "_REPLACE":{
        "_STRING":"{foo}",
        "_SEARCH":["12","3"],
        "_REPLACEMENT":["",""]
    }
}
```

```json
"foo":"A123B45C6789",
"var":{
    "_REPLACE":{
        "_STRING":"{foo}",
        "_SEARCH":"[1-9]",
        "_REPLACEMENT":"",
        "_REGEX":true
    }
}
```

### Logic/Boolean Functions

These are functions that take the role of common logic operators, returning a boolean (`true` or `false`). Some of these are listed below:

**`_NOT`:** Takes a boolean and returs its opposite.

**`_AND`:** Takes an array and returns whether all of its elements are `true`.

**`_OR`:** Takes an array and returns whether any of its elements is `true`.

**`_EQUAL`:** Takes an array of two JSON values and returns whether they are equal.

### Number Functions

Some functions to format and manipulate numbers (also see **The `_MATH` Function** above).

**`_RANDOM`** `{"_RANDOM":100}`, for example, returns a random integer between 0 and 99.

**`_ROUND`:** Rounds a number to its closest integer.

**`_FLOOR`:** Rounds **down** a number to its closest integer.

**`_DECIMAL`:** Displays a number using the standard format used for currency in the English language, with comma-separated thousands, e.g. `1,000.02` or `1,000`. Support for other locales will become available in future versions of Relevant.

### The `_DATE` Function

This function always returns a string representing a date in a given format. The only required parameter is `"_FORMAT_OUT"`. The example below returns today's date in the format yyyy-MM-dd:

```json
"todays_date":{
    "_DATE": {
        "_FORMAT_OUT":"yyyy-MM-dd",
    }
}
```

You may also include hours, minutes, seconds, and milliseconds. [Click here for a list of all available formats](http://www.unicode.org/reports/tr35/tr35-31/tr35-dates.html#Date_Format_Patterns).

Besides these formats, you could also use `"<<timestamp>>"` (return a [UNIX timestamp, i.e. seconds since 1970](http://en.wikipedia.org/wiki/Unix_time)), `"<<ago>>"` (string such as `"11 minutes ago"`), `"<<fb-ago>>"` (Facebook style), or `"<<min-ago>>"` (Minimal Twitter style, such as `"11m"`).

You can use the parameter `"_OFFSET"` to offset the current time by any number of seconds. This example produces yesterday's date:

```json
"yesterdays_date":{
    "_DATE": {
        "_OFFSET": -86400,
        "_FORMAT_OUT": "yyyy-MM-dd",
    }
}
```

Instead of using today's date, you can also input a fixed date using the parameter "`_VALUE`", which is either a [UNIX timestamp (seconds since 1970)](http://en.wikipedia.org/wiki/Unix_time), or a formatted date string. In the latter case, it is also necessary to include a parameter "`_FORMAT_IN`". For example:

```json
"some_date":{
    "_DATE": {
        "_VALUE": "01/11/2022",
        "_FORMAT_IN": "MM/dd/yyyy",
        "_FORMAT_OUT": "yyyy-MM-dd",
    }
}
```

### Getting User Location / Coordinate Distance

User location may be accessed through the `_LOCATION` implicit variable, which comes in the format:

```json
{
    "latitude":...,
    "longitude":...
}
```

For example, one may get the user's latitude using `{"_PATH":["_LOCATION","latitude"]}`.

It is possible get the distance between two coordinates using `{"_COORDIST":[...,...]}`. For example, the following returns the distance between the current user location and some place in San Francisco:

```json
{
    "_COORDIST":[
        "{_LOCATION}",
        {
            "latitude":37.797929,
            "longitude:-122.428931
        }
    ]
}
```

### The `_WEB_CALLBACK` Function (Web View for User Input)

It is possible to pop up a webview while a relevant card is being loaded, be it for user authentication, or any other form of user input. This is achieved with the `_WEB_CALLBACK` function. This function will return either when the user dismisses the webview manually, or when a certain base URL is loaded as a result of navigation. This functions looks like this:

```json
"var":{
    "_WEB_CALLBACK": {
        "_ADDRESS":"http://relevant.ai/",
        "_CALLBACK_BASE":"relevant.ai/callback"
    }
}
```

The parameter `_ADDRESS` is the initial URL of the webview. The webview will dismiss automatically once it reaches a URL prefixed by the `_CALLBACK_BASE`. **Remember not to include the protocol http:// in the `_CALLBACK_BASE`**. If the user dismisses the webview manually, then this function returns `null`. Otherwise it returs an object of the form:

```json
{
    "url":"http://url/absolute/string",
    "body":{
        "param1":"value1",
        "param2":"value2"
    },
    "method":"GET or POST or any other method",
    "headers":{ 
        "Header One":"value of header one",
        "Header Two":"value of header two",
        "Header Three":"value of header three"
    }
}
```

This object contains information about the url request that was created by the webview before dismissing. This request is never actually called.

### Inline Functions (`_FUNCTION` and `_APPLY`)

It is possible to define your own functions. In the example below we define a function `increment` which takes numbers `X`,`Y` and returns `X+Y+1`:

```json
"increment":{
    "_FUNCTION":{"_MATH":"{X}+{Y}+1"}
}
```

To apply this function you use `_APPLY` on a dictionary whose only key is the name of the function. The value of this key represents the parameters that are passed to the function, as in the following example, in which `var` takes the value `101`:

```json
"var":{
    "_APPLY":{"increment":{"X":45,"Y":55}}
}
```

A function's body may contain intermediate variables, as long as it has a `_RETURN` key. The function `increment_1` below is equivalent to `increment` above.

```json
"increment_1":{
    "_FUNCTION":{
        "Z":{"_MATH":"{X}+{Y}"},
        "_RETURN":{"_MATH":"{Z}+1"}
    }
}
```

### Commenting

Because of the simplicity and readability of REL, as long as variables are conveniently named, it should not be necessary to use comments. However, one way to write comments is to define variables that are never used. Naming all unused variables `"//"` is a way to make comments more obvious. For example, the `_LOAD` function of the Hello World example at the beginning of this document could be written as follows:

```json
{
    "id": "hello-world",
    "title": "Hello World Card",
    "icon_url": "http://relevant.ai/hello_world.png",
    "summary": "Card's summary for the library.",
    "credits": "Relevant",
    "settings_type": "NONE",
    "_LOAD": {
        "//":"Change this variable to replace Hello World for anything else",
        "hello":"Hello World",
        "//":"This is the description template",
        "foo":{
            "description":{
                "title": "{hello}",
                "body": "{hello}? There's a card for that."
            }
        },
        "//":"This is the footer template",
        "var":{
            "footer":{
                "caption": "Relevant - The Missing Home Screen"
            }
        },
        "//":"This is the return of the _LOAD function",
        "_RETURN":[
            [
                "{foo}",
                "{var}"
            ]
        ]
    }
}
```

### Logging (The `_LOG` Function)

The `_LOG` function takes any string, number, array, or object, and returns that same string, number, array, or object. However, it also writes this value into a *logs* document. Every time a card containing `_LOG` is refreshed, this document pops up. You can also show the logs of a card without refreshing it, by holding down on the card and selecting the *logs* (console) icon.

You may try the following variation of the cities card (see the `_LOOP` function section above) by shaking your phone and adding the card `https://gist.githubusercontent.com/wircho/110b521e5870cec835d3/raw/2d4c89397f14e3581ed56b9bf34edf38bb886df6/cities_logs_card`.

```json
{
    "id": "cities-card",
    "title": "Cities",
    "icon_url": "http://relevant.ai/cities.png",
    "summary": "Card's summary for the library.",
    "credits": "The Web",
    "settings_type": "NONE",
    "_LOAD": {
        "json_info":{
            "_URL":"https://gist.githubusercontent.com/wircho/d6c606350f8a6dca29ee/raw/b7872ecebfcc868ff825c88fe1c5e06d13ad618c/cities"
        },
        "_RETURN":{
            "_LOOP":{
                "_ARRAY":{"_PATH":["json_info","cities"]},
                "_EACH":[
                    {
                        "banner":{
                            "image": {"_PATH":["_ITEM","picture"]}
                        }
                    },
                    {
                        "description":{"_LOG":{
                            "title": {"_PATH":["_ITEM","name"]},
                            "body": {"_PATH":["_ITEM","nickname"]}
                        }}
                    },
                    {
                        "footer":{
                            "caption": {"_PATH":["_ITEM","province"]}
                        }
                    }
                ]
            }
        }
        
    }
}
```

Once this card is loaded, a webview pops up with the content shown below. Swipe right from the left end to dismiss it.

![Logs](https://raw.githubusercontent.com/relevant-ai/RelevantCardsDocumentation/master/cities_logs.png)

### Using Templates

Let us take another look at the cities card (`https://gist.githubusercontent.com/wircho/c8f4f5b0ce440b8edd83/raw/8ef09440e96322e75220ff1470fbc4c65d76e6c2/cities_card`):

```json
{
    "id": "cities-card",
    "title": "Cities",
    "icon_url": "http://relevant.ai/cities.png",
    "summary": "Card's summary for the library.",
    "credits": "The Web",
    "settings_type": "NONE",
    "_LOAD": {
        "json_info":{
            "_URL":"https://gist.githubusercontent.com/wircho/d6c606350f8a6dca29ee/raw/b7872ecebfcc868ff825c88fe1c5e06d13ad618c/cities"
        },
        "_RETURN":{
            "_LOOP":{
                "_ARRAY":{"_PATH":["json_info","cities"]},
                "_EACH":[
                    {
                        "banner":{
                            "image": {"_PATH":["_ITEM","picture"]}
                        }
                    },
                    {
                        "description":{
                            "title": {"_PATH":["_ITEM","name"]},
                            "body": {"_PATH":["_ITEM","nickname"]}
                        }
                    },
                    {
                        "footer":{
                            "caption": {"_PATH":["_ITEM","province"]}
                        }
                    }
                ]
            }
        }
        
    }
}
```

Each page of this card contains the templates `banner` (the city's picture), `description` (the city's name and nickname), and `footer` (the city's province). Below is a list of available templates with their parameters and possible values:

| Template        | Parameters <br/> Possible Values + Notes 
| ------------- |-------------|
| **`banner`** (banner image)<br/><br/> When used it should be the **first visible template** of a card.   | **`type`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;`"narrow"` (default) <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"short"` <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"medium"` <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"square"` <br/> &nbsp;&nbsp;&nbsp;&nbsp;`"tall"` <br/><br/> **`image`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;URL of image. <br/><br/> **`title`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;Image title (displayed in large font) <br/><br/> **`caption`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;Image caption (displayed in small font) <br/><br/> **`align`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;Alignment of `title` and `caption` <br/> &nbsp;&nbsp;&nbsp;&nbsp;`"left"` (default) <br/> &nbsp;&nbsp;&nbsp;&nbsp;`"center"` <br/> &nbsp;&nbsp;&nbsp;&nbsp;`"right"` <br/><br/> **`color`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;Background color (e.g. `"white"` (default)) |
| **`profile`** (circle image + caption)   | **`type`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;`"small"` <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"medium"` (default) <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"large"` (centered image only, no caption) <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"cell"` <br/><br/> **`image`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;URL of image. <br/><br/> **`caption`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;Caption beside image <br/><br/> **`border-color`** <br/> &nbsp;&nbsp;&nbsp;&nbsp; Image border color (e.g. `"white"` (default)) |
| **`double-profile`** (two circle images side by side)   | **`image-i`** (`i=1,2`) <br/> &nbsp;&nbsp;&nbsp;&nbsp;URL of image. <br/><br/> **`caption-i`** (`i=1,2`) <br/> &nbsp;&nbsp;&nbsp;&nbsp;Caption under image <br/><br/> **`border-color-i`** (`i=1,2`) <br/> &nbsp;&nbsp;&nbsp;&nbsp; Border color of image |
| **`stats`** (one, two, or three stats)   | **`type`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;`"single"` <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"double"` <br/>   &nbsp;&nbsp;&nbsp;&nbsp;`"triple"` (default) <br/><br/> **`value-i`** (`i=1,2,3`) <br/> &nbsp;&nbsp;&nbsp;&nbsp;Stats value (e.g. `"200"`) <br/><br/> **`title-i`** (`i=1,2,3`) <br/> &nbsp;&nbsp;&nbsp;&nbsp;Caption under stats value (e.g. `"visitors"`) |
| **`scalar`** (value and image)   | **`value`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;Value on the left <br/><br/> **`image`** <br/> &nbsp;&nbsp;&nbsp;&nbsp;URL of image on the right |
| **`buttons`** (one, two, or three buttons) | This template does not take an object of parameters<br/><br/> Instead it takes an array of one, two, or three **REL Actions** to be performed on tap (See **REL Actions** below)|
| **`sectional`** (light gray background for last template)<br/><br/> When used it should be the **last visible template** of a card. | This template does not take parameters. Simply use `{"sectional":true}` |

### Advanced Templates (`actions` and `handler`)

`actions` and `handler` are templates which may be inserted **at the beginning of each page's array**, and are not part of the card content. However they may be essential to the functionality of some cards.

| Advanced Template        | Parameters <br/> Possible Values + Notes 
| ------------- |-------------|
| **`actions`** (touch down actions)  | Just like the **`buttons`** template, it takes an array of **REL Actions** (See **REL Actions** below), to be performed when the card is held down and the corresponding action is selected. |
| **`handler`** (card's title)  | This template allows you to customize the text on the card's title (top left) for each page of the card.<br/><br/>For example, the following `handler` template could be added to the cities card (`https://gist.githubusercontent.com/wircho/c8f4f5b0ce440b8edd83/raw/8ef09440e96322e75220ff1470fbc4c65d76e6c2/cities_card`) to display the name of each city on the card's title:<br/><br/> `{"handler":{"_PATH":["_ITEM","name"]}}` |

### Error Handling

Some built-in functions, such as `_URL` may return errors (such as when there is no internet connection). By default, REL propagates errors outwards, which means that whenever an error appears, the card will limit itself to showing this error rather than the expected templates (due to a small bug in Relevant 1.0, a card may show up light gray and not display the error).

You could even create errors manually using `{"_ERROR":"Error message"}`, and they will be propagated outwards just like errors that happen inadventently.

There are cases, however, in which you would like to handle an error. For example, when certain `_URL` returns an error (such as server not found), you may wish to get your data from a different URL.

The following *error handling* functions may be used to stop errors from propagating outwards into the card:

**`_IS_ERROR`:** Returns `true` (`1`) when passed an error, and `false` (`0`) otherwise.

**`_ERROR_CODE`:** Returns an error's numeric error code. This is useful when handling http request errors from `_URL` calls.

**`_IS_NULL_OR_ERROR`** Returns whether the passed object is `null` or an error.

**`_FIRST_NOT_ERROR`** Returns the first element of an array which is neither `null` nor an error.

### REL Actions

REL Actions and similar to Inline functions. However, they are supposed to be used only in the `buttons` and `actions` template, and they include a button's icon, color, and caption within their syntax. Each REL Action is an object of this form:

```json
{
    "_DO":...,
    "_ICON":"Icon name, e.g. source_icon, more_icon, map_icon.",
    "_CAPTION":"Button's caption",
    "_COLOR":"Button's color, e.g. red, blue. Leave blank for default color."
}
```

On Relevant 1.0, the parameters `_CAPTION` and `_COLOR` is ignored in the `actions` template, and the parameter `_ICON` is ignored in the `buttons` template whenever there is only one button. This is a design decision and not something to worry about.

The complete list of `_ICON`s available on Relevant 1.0 is: `"done_icon"`, `"eye_icon"`, `"fav_icon"`, `"heart_icon"`, `"refresh_icon"`, `"repost_icon"`, `"share_icon"`, `"source_icon"`, `"map_icon"`, `"call_icon"`, `"do_icon"`, `"log_icon"`, `"more_icon"`, `"play_icon"`, `"read_icon"`, `"reorder_icon"`, `"search_icon"`, `"sync_icon"`, and `"chat_icon"`.

`_DO` is the actual action to be performed by the button. This could be something that happens in the background and is not visible by the user. For example a `_URL` request with `"_METHOD":"POST"` to *like* something on social media (notice that such functionality would require user log-in or OAuth. A tutorial on this coming soon). More traditional examples of actions are listed below:

**Show WebView**
```json
"_DO":{
    "_WEBVIEW":"http://github.com"
}
```

**Open External URL or Deeplink** (Open URL or iOS Deeplink)
```json
"_DO":{
    "_DEEPLINK":"http://github.com"
}
```

**Remark:** You may wish to wrap the function `_DEEPLINK` inside an `_IF,_THEN,_ELSE` conditioning on `{"_CAN_DEEPLINK":"http://github.com"}` to make sure you're not calling an URL that cannot be opened by the device.

**Show MapView** (MapView with directions to a given latitude and longitude)
```json
"_DO":{
    "_MAPVIEW":{
        "title":"Montreal's City Hall",
        "latitude":45.508994,
        "longitude":-73.554418
    }
}
```

**Refresh Card**
```json
"_DO":{
    "_REFRESH":...
}
```

The `_REFRESH` action may be passed any string, number, array, or object input value. When a card is refreshed from this action, it loads with an implicit variable `_INPUT` having this value.

As always, it is possible to have intermediate variables inside `_DO` as long as there is a `_RETURN` key. For example:

```json
"_DO":{
    "url":"http://github.com",
    "_RETURN":{"_WEBVIEW":"{url}"}
}
```









