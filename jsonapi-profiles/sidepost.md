# Sidepost related resources

Defines a mechanism to create and update several related resource objects at once.
- Minimum JSON:API version: 1.0
- Discussion_url: http://github.com/komunitin/komunitin-api
- Category: Data Modeling

This extension provides a way to include related resources in a POST/PATCH request analogously as when related resources are included in a GET request. Suppose, for example, that you have resource types `articles` and `tags` so that you want to create/update an article and its tags all at once.

The specification is based on the proposal [Sideposting draft](https://github.com/json-api/json-api/pull/1197).

## Specification

This extension adds a feature to the `POST` and `PATCH` endpoints defined by JSON:API to create and modify resources. The main resource being created or updated will be specified, as usual, in the `data` section of the document body

Eventually, a relationship in the main resource contain [resource linkage](https://jsonapi.org/format/1.1/#document-resource-object-linkage) as defined in the core spec. 

The linked resources must be defined in the new `included` root field section, exactly the same way as when [fetching related resources](https://jsonapi.org/format/1.1/#fetching-includes).

The linked resources must support client-generated ids. Either in `POST` or `PATCH` requests, `included` resources are created if the id is new and updated if the id already exists in the server.

### Example 1: Create
This request creates three objects: an `article` and two related `tags`. The client is responsible for generating ids that are unique, so UUIDs should be used in a real case.

```bash
POST /articles
```
```json
{ 
  "data": {
    "type": "articles",
    "attributes": {
      "title": "Sidepost related resources"
    },
    "relationships": {
      "tags": {
        "data": [
          {"type": "tags", "id": "1"},
          {"type": "tags", "id": "2"}
        ]
      }
    }
  },
  "included": [
    {
      "type": "tags",
      "id": "1",
      "attributes": {
        "name": "jsonapi"
      }
    },
    {
      "type": "tags",
      "id": "2",
      "attributes": {
        "name": "extension"
      }
    }
  ]
}
````

### Example 2: Update
This request updates the article title as well as the name of its second tag. The first tag is not modified. It creates a new third tag as well.

```bash
PATCH /articles/1
```
```json
{ 
  "data": {
    "id": "1",
    "type": "articles",
    "attributes": {
      "title": "Sidepatch related resources"
    },
    "relationships": {
      "tags": {
        "data": [
          {"type": "tags", "id": "1"},
          {"type": "tags", "id": "2"},
          {"type": "tags", "id": "3"}
        ]
      }
    }
  },
  "included": [
    {
      "type": "tags",
      "id": "1",
      "attributes": {
        "name": "json:api"
      }
    },
    {
      "type": "tags",
      "id": "3",
      "attributes": {
        "name": "update"
      }
    },
  ]
}
````