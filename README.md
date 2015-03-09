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

The `_LOAD` object needs to have a `"_RETURN"` key, which holds the object that contains all the information for skinning the card. The following is the first complete example of a card in this document. It is a card with a headline and a footer. It has two static pages reading "Header 1", "Footer 1" and "Header 2", "Footer 2" respectively.

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
        "_RETURN": {
            [
                {
                    "headline": {"text": "Header 1"},
                    "footer": {"text": "Footer 1"}
                },
                {
                    "headline": {"text": "Header 2"},
                    "footer": {"text": "Footer 2"}
                }
            ]
        }
    }
}
```
