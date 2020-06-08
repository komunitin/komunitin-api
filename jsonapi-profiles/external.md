# External relationships
Defines the relation between two different JSON:API APIs through relationships that point to resources handled by an external service.

- Minimum JSON:API version: 1.0
- Discussion_url: http://github.com/komunitin/komunitin-api
- Category: Data Modeling

Suppose that server A provides an API with information about users, and in particular it has the `users` resource type. Suppose that server B provides an API with information about blog posts and it has the resource type `posts`. This profile provides a way to link `users` and `posts` even if they belong to different services.

The specification is based on the proposal [Relationships between resources controlled by different APIs](https://github.com/json-api/json-api/issues/675)

## Specification

Extend the [Resource identifier objects](https://jsonapi.org/format/#document-resource-object-links) with the following fields:
 - `external`: Optional. Defines whether the linked object belongs to a remote service. If present, its value must be the boolean `true`. If ommitted, it is understood to have the value `false`.
 - `href`: Optional. To be included when `external` is `true`. Contains the absolute `URL` of the related resource.

When fetching a resource that contains external relationships, the server may include linkage data for these relationships even if the related resources have not been requested through the `include` query parameter.

When a client requests the inclusion of external resources through `include` query parameter, the server may either fetch the remote resources and return them or otherwise reject the request with a `403 Forbidden` HTTP status code.