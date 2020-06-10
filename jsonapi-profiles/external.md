# External relationships
Defines the relation between two different JSON:API APIs through relationships that point to resources handled by an external service.

- Minimum JSON:API version: 1.0
- Discussion_url: http://github.com/komunitin/komunitin-api
- Category: Data Modeling

Suppose that server A provides an API with information about users, and in particular it has the `users` resource type. Suppose that server B provides an API with information about blog posts and it has the resource type `posts`. This profile provides a way to link `users` and `posts` even if they belong to different services.

The specification is based on the proposal [Relationships between resources controlled by different APIs](https://github.com/json-api/json-api/issues/675)

## Specification

An **External Relationship** is a relationship between two resources that are managed by different servers.

When fetching a resource that contains external relationships, the server may include linkage data for these relationships using [Resource Identifier Objects](https://jsonapi.org/format/#document-resource-object-links), which are just an objects with `type` and `id`. Up to here nothing changed from a local relationship.

When a client requests the inclusion of related resources through `include` query parameter that happen to be external, the server includes **External Resource Object**s with fields `type`, `id` and the `external` flag, and a `self` link that points to the URL of the external server. This URL provides the original copy of the resource, with its attributes and relationships.

```JSON
{
  "data": {
    "type": "posts",
    "id": "1",
    "external": true,
    "links": {
      "self": "https://externalserver.org/posts/1"
    },
  }
}
```