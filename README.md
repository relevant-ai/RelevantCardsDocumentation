# Relevant Cards Documentation
This document explains how to create relevant.ai cards.

## Basic Structure

A card is just a JSON object with the following structure:

```json
{
    "id": "card-id-string",
    "title": "My Card's Title",
    "icon_url": "https://url/to/icon.png",
    "summary": "My card's description.",
    "credits": "Some Website or Source",
    "templates": [
                  "thumbnail",
                  "headline",
                  "footer"
                  ],
    "settings_type": "NONE",
    "_LOAD": {
        /*
        CODE
        */
    }
}
```

All keys, except for `"_LOAD"` represent the card's metadata. `"settings_type"` may also be `"SELECT"` or `"TEXT"` but we will discuss that later.

The `"templates"` key represents the appearance of the card from top to bottom. The example above consists of a thumbnail, a headline, and a footer. These are all currently available templates: `title, footer, large-image-square, description, title-two-lines, profile, banner, headline, large-stats, stats, transit, image-stats, large-image-3x4, short-description, profile-image, match, covering-button, thumbnail, long-description, nearby, thumb-scalar`

`"_LOAD"` is the *code* of the card.

## The `"_LOAD"` Code

The value of the `"_LOAD"` key is an object that represents code which is executed every time the card is refreshed.

Every key in this object is the name of a variable. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "var1": "value of variable 1",
        "var2": {
            "value": ["of","variable",2]
        }
        /*...
        MORE CODE
        ...*/
    }
}
```

The value of a variable may be any JSON object.

## The `"_RETURN"` Key

The `_LOAD` object needs to have a `"_RETURN"` key, which holds the object that contains all the information for skinning the card. The following is the first complete example of a card in this document. It is a card with a headline and a footer. It has two static pages reading "My Header 1", "My Footer 1" and "My Header 2", "My Footer 2" respectively.

```json
{
    "id": "card-id-string",
    "title": "My Card's Title",
    "icon_url": "https://url/to/icon.png",
    "summary": "My card's description.",
    "credits": "Some Website or Source",
    "templates": [
                  "headline",
                  "footer"
                  ],
    "settings_type": "NONE",
    "_LOAD": {
        "_RETURN": [
            {
                "headline": {"text": "My Header 1"},
                "footer": {"text": "My Footer 1"}
            },
            {
                "headline": {"text": "My Header 2"},
                "footer": {"text": "My Footer 2"}
            }
        ]
    }
}
```

## Using Variables

You can refer to a variable `var` in a value as `"{var}"`. For example, the following card looks exactly the same as the one above:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "header_1": "My Header 1",
        "_RETURN": [
            {
                "headline": {"text": "{header_1}"},
                "footer": {"text": "My Footer 1"}
            },
            {
                "headline": {"text": "My Header 2"},
                "footer": {"text": "My Footer 2"}
            }
        ]
    }
}
```

A more complex example (that also looks the same):

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "page_1": {
            "headline": {"text": "My Header 1"},
            "footer": {"text": "My Footer 1"}
        },
        "page_2": {
            "headline": {"text": "My Header 2"},
            "footer": {"text": "My Footer 2"}
        },
        "_RETURN": [
            "{page_1}",
            "{page_2}"
        ]
    }
}
```

**Remark:** The keys of the `_LOAD` object are variables. However, the keys of any child object are **NOT**. For example, `"headline"` is not a variable in the example above, and so it cannot be referenced as `"{headline}"`.

## Inserting/Concatenating String Variables

The syntax `"{var}"` can also be used within a bigger string when `var` is a string variable. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "header_word": "Header",
        "footer_word": "Footer",
        "first_number": 1,
        "second_number": 2,
        "_RETURN": [
            {
                "headline": {"text": "My {header_word} {first_number}"},
                "footer": {"text": "My {footer_word} {first_number}"}
            },
            {
                "headline": {"text": "My {header_word} {second_number}"},
                "footer": {"text":  "My {footer_word} {second_number}"}
            }
        ]
    }
}
```

## Paths

A *path* is an object of the form `{"_PATH":[...]}` where `[...]` is an array whose first (0-th) element is a variable name, and each subsequent element is either a **string key** or an **integer index**. For example `{"_PATH":["my_rainbow","colors",3,"red"]}`. Paths can be used to fetch children of JSON objects. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "info": {
            "useless_stuff": {
                "a": ["x","y","z"],
                "b": "c"
            },
            "important_stuff": [
                {
                    "head": "My Header 1",
                    "foot": "My Footer 1"
                },
                {
                    "head": "My Header 2",
                    "foot": "My Footer 2"
                }
            ]
        },
        "_RETURN": [
            {
                "headline": {"_PATH":["info","important_stuff",0,"head"]},
                "footer": {"_PATH":["info","important_stuff",0,"foot"]}
            },
            {
                "headline": {"_PATH":["info","important_stuff",1,"head"]},
                "footer": {"_PATH":["info","important_stuff",1,"foot"]}
            }
        ]
    }
}
```

## Built-In Functions

The keyword `"_PATH"` introduced above is just a function that's built into the language. These functions are usually invoked using objects of the form `{"_FUNCTION_NAME":{"_PARAM_NAME":...,"_ANOTHER_PARAM_NAME":...,...}}`. The exact syntax of a built-in function varies accross functions. These are some of the built in functions that are currently available:

### The `_CONDITIONAL` Function

Allows you to return two possible values depending on whether a condition is `true` (`>0`) or `false` (`0`). It's parameter keys are `"_IF"`, `"_THEN"` and `"_ELSE"`. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "something": false,
        "sentence": "something is {something}",
        "_RETURN": {
            "_CONDITIONAL": {
                "_IF":"{something}",
                "_THEN":[
                    {
                        "headline": {"text":"Good!"},
                        "footer": "{sentence}",
                    }
                ],
                "_ELSE":[
                    {
                        "headline": {"text":"Bad!"},
                        "footer": "{sentence}",
                    }
                ]
            }
        }
    }
}
```

### The `_PATH_EXISTS` Function

Returns `true` (`1`) if an array of keys or indexes represents a path that exists, and `false` (`0`) otherwise. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
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
        "_RETURN": {
            /* ... */
        }
    }
}
```

In the code above, the variables `"a"` and `"b"` are `1` (`true`), and the variable `"c"` is `0` (`false`).

### The `_URL` Function

Allows you to make a call to a web address that returns a JSON object. You can then access that data using `_PATH` or the `"{var}"` syntax. The following example assumes that `http://path/to/my/json.html` loads a json object containing a `"results"` key.

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "json_info": {
            "_URL":"http://path/to/my/json.html"
        },
        "_RETURN": {
            "_PATH":["json_info","results"]
        }
    }
}
```

You can also make richer requests. For example:

```json
{
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

### The `_MERGE` Function

Takes an array of objects, strings, or arrays, and returns an array containing each of those objects, strings, and array elements. The syntax is as in the following example:

```json
{
    "_MERGE":[
        "String 1",
        [
            "String 3",
            "String 4",
            "String 5"
        ],
        "String 6",
        "String 7",
        [
            "String 8",
            "String 9"
        ]
    ]
}
```

### The `_LOOP` Function

Allows you to loop an array to get a new array of the same size but with modified contents. Its parameter keys are `"_ARRAY"` and `"_EACH"`. The first one is the array that is to be looped (you can make a reference to it using `_PATH`, the `"{var}"` syntax, or any other function). Inside the `"_EACH"` object you can use the local variables `"{_ITEM}"` and `"{_INDEX}"` representing the current array element and its index respectively. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "info": {
            "useless_stuff": {
                "a": ["x","y","z"],
                "b": "c"
            },
            "important_stuff": [
                {
                    "head": "My Header 1",
                    "foot": "My Footer 1"
                },
                {
                    "head": "My Header 2",
                    "foot": "My Footer 2"
                },
                {
                    "head": "My Header 3",
                    "foot": "My Footer 3"
                }
            ]
        },
        "_RETURN": {
            "_LOOP": {
                "_ARRAY": {"_PATH":["info","important_stuff"]},
                "_EACH": {
                    "headline": {"_PATH":["_ITEM","head"]},
                    "footer": {"_PATH":["_ITEM","foot"]}
                }
            }
        }
    }
}
```

**Remark:** The example above would make a lot more sense, of course, if the variable `"info"` came for a web service (`_URL`).

The `_EACH` object may contain variables itself, as long as it has a `"_RETURN"` key. For example, the `_EACH` object above may be replaced with:

```json
{
    "_ARRAY": {"_PATH":["info","important_stuff"]},
    "_EACH": {
        "h": {"_PATH":["_ITEM","head"]},
        "f": {"_PATH":["_ITEM","foot"]},
        "_RETURN": {
            "headline": "{h}",
            "footer": "{f}"
        }
    }
}
```
### Basic string Functions

Basic string functions are built in functions that take a string and return a string. Some available basic string functions are listed below:

**`_URL_ENCODE`:** Percent encoding for URL parameters.

**`_URL_DECODE`:** Inverse of above.

**`_HTML_ENCODE`:** HTML special characters encoding.

**`_HTML_DECODE`:** Inverse of above.

**`_UPPERCASE`:** Convert a String to uppercase.





