# Relevant Cards Documentation
This document explains how to create relevant.ai cards to be hosted on the web.

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

All keys, except for `"_LOAD"` represent the card's metadata. The `"settings_type"` may also be `"SELECT"` or `"TEXT"` but we will discuss that later.
