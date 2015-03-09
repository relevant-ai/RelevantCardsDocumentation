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

A *path* is an object of the form `{"_PATH":[...]}` where `[...]` is an array whose first (0-th) element is a variable name, and each subsequent element is either a **string key** or an **int index**. For example `{"_PATH":["my_rainbow","colors",3,"red"]}`. Paths can be used to fetch children of JSON objects. For example:

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

