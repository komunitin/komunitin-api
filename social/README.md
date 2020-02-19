# Komunitin social API

REST API featuring marketplace services for Exchange Communities.

**This is still a work in progress**. Comments welcomed at komunitinbox@gmail.com or at [GitHub](https://github.com/komunitin/komunitin-api).

This API provides access to exchange group details, member profile and their offers and needs and informational messages. It doesn't feature payments nor any finantial information, as this is available in a separate API. Subscrption to notifications (email and push) and notification preferences are also defined in a separate API.

## Preliminaries

This specification is based on the [JSON:API](https://jsonapi.org) standard.

## Authorization

Authorization endpoints are out of the scope of this API, which expects the user to be authorized with a JWT access token. The access token is provided by a separate authorization API. The recommended OAuth2 flow for web apps and native apps is the Authorization Code Flow with Proof Key for Code Exchange (PKCE).

No OAuth2 scopes are defined at the moment, but may be defined in future.

## Architecture

- The URL structure starts with the exchange code, so it eases the data isolation among different exchange groups in a single server, as there is no way to ask for resources fromdifferent exchanges. The only exception is the global `/exchanges` endpoint.
- For interoperability, a system featuring just one exchange group should implement the `/exchanges` `GET` endpoint as well as the `/{exchangeCode}` `GET` endpoint, providingstatic responses and chosing a fixed value for the `exchangeCode` parameter (such as the word "exchange"). They may omit the other endpoints in Exchanges section and may alsoomit the exchangeCode path parameter in all other paths.
- Clients should honor the links provided by API responses and not try to construct the URL's using the known structure defined in this document. This way it decouples client andserver from URL design, making the server fully responsible of URLs.
- All resources have a global UUID identifier as their `id` field, but the URLs are human-readable. The URLs are meant to be almost immutable to allow caching and others, but canoccasinally change (for example, in case of server migration), while the UUIDs are really immutable.
- Geolocation fields follow the [GeoJSON](https://geojson.org/) specification. Note that the spec allows to specify either a concrete point or an area.
- Date or time fields follow the [RFC3339](https://tools.ietf.org/html/rfc3339) specification.
- Long text fields allow formatting through a restricted set of HTML tags.
- File server is out of the scope of this API, and should be handled using a separate specialized service. This way we ease the implementation of advanced upload techniques and efficient delivery of static binary files. File URLs should be randomized in order to minimize unauthorized access to files. Additional security measures such as short-lived Signed URLs can be implemented to further restrict access to binary files.

## Data privacy

Access to all resources may be restricted by assigning one of the predefined access labels to the `access` resource field.
 - `private`: The resource is only accessible by its owner.
 - `exchange`: The resource is only accessible by the members of the same exchange group.
 - `public`: The resource is publicly accessible on the internet.

In the future, other access labels may be added and the `access` field may accept multiple labels for a resource.

## Versioning

Clients should allow additional fields along all JSON responses. Therefore adding new fields won't be considered breaking backwards compatibility. API versioning is done using HTTP headers (still to specify).
