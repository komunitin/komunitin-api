# External relationships
Defines the relation between two different JSON:API APIs through relationships that point to resources handled by an external service.

- Minimum JSON:API version: 1.0
- Discussion_url: http://github.com/komunitin/komunitin-api
- Category: Data Modeling

Suppose that server A provides an API with information about users, and in particular it has the `users` resource type. Suppose that server B provides an API with information about blog posts and it has the resource type `posts`. This profile provides a way to link `users` and `posts` even if they belong to different services.

The specification is based on the proposal [Relationships between resources controlled by different APIs](https://github.com/json-api/json-api/issues/675)

## Specification

An **External Relationship** is a relationship between two resources that are managed by different servers.

An **External Resource Identifier Object** is an extension of [Resource Identifier Objects](https://jsonapi.org/format/#document-resource-object-links) with the following additional field:
 - `external`: Optional. Defines whether the linked object belongs to a remote service. If present, its value must be the boolean `true`. If ommitted, it is understood to have the value `false`.

 Example of an External Resource Identifier Object:

 ```JSON
{
  "type": "posts", 
  "id": "1",
  "external": true
}
 ```

When fetching a resource that contains external relationships, the server may include linkage data for these relationships using External Resource Identifier Objects.

When a client requests the inclusion of related resources through `include` query parameter that happen to be external, the server includes **External Resource Object**s with fields `type`, `id` and the `external` flag, and a `self` link that points to the URL of the external server that will return the original copy of the resource, with its attributes and relationships.

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