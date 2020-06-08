# Simple filter
Defines sematics for the `filter` query parameter.

- Minimum JSON:API version: 1.0
- Discussion_url: http://github.com/komunitin/komunitin-api
- Category: Filtering

Defines resource filtering by matching the value of fields. Does not (yet) specify how to create complex logical filters, but just a set of conditions over fields.
 
## Specification

`GET` endpoints allow the `filter` query parameter to filter the returned results. The parameter works like that:
```
filter[field][operation]=value
```
Where `field` is one of the resource attributes or relationships or the special field `search` that means searching in all relevant reaource fields. Nested fields are allowed using the dot `.` character.

`operation` is one of:
 - `eq`, `ne`: Equal, not equal. The equal operation may be omitted. Multiple values can be combined in a comma-separated list.
 - `lt`, `gt`, `le`, `ge`: Less than, greater than, less than or equal, greater than or equal. For numeric or date values.

And `value` is the value to match. In case of relationships, it is the resource code.