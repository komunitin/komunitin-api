# External relationships
Defines the relation between two different JSON:API APIs through relationships that point to resources handled by an external service.

- Minimum JSON:API version: 1.0
- Discussion_url: http://github.com/komunitin/komunitin-api
- Category: Data Modeling

Suppose that server A provides an API with information about users, and in particular it has the `users` resource type. Suppose that server B provides an API with information about blog posts and it has the resource type `posts`. This profile provides a way to link `users` and `posts` even if they belong to different services.

The specification is based on the proposal [Relationships between resources controlled by different APIs](https://github.com/json-api/json-api/issues/675)

## Specification

An **External Relationship** is a relationship between two resources that are managed by different servers.

When fetching a resource that contains relationships, the server may include linkage data for these relationships using [Resource Identifier Objects](https://jsonapi.org/format/#document-resource-object-links), which are objects with `type` and `id`. An **External Resource Identifier Object** is an extension of a Resource Identifier Object that also has the member `external` with value `true` and the member `href` with value the remote URL where to find the original resource. This is the linkage data for external relationships.

```JSON
{
  "type": "posts",
  "id": "1",
  "external": true,
  "href": "https://externalserver.org/posts/1"
}
```

When a client requests the inclusion of related resources through `include` query parameter that happen to be external, the server includes **External Resource Object**s, which are basically the same as the External Resource Identifier Objects just defined, with fields `type`, `id`, the `external` flag and the `href` address.

```JSON
{
  "data": {
    "type": "posts",
    "id": "1",
    "external": true,
    "href": "https://externalserver.org/posts/1",
    "links": {
      "self": "https://localserver.org/posts/1"
    },
  }
}
```