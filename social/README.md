# Komunitin Social API

REST API featuring marketplace services for Exchange Communities.

See the specificatin documentation at [SwaggerHub](https://app.swaggerhub.com/apis-docs/estevebadia/komunitin-api/0.0.1).

**This is still a work in progress**. Comments welcomed at komunitinbox@gmail.com or at [GitHub](https://github.com/komunitin/komunitin-api).

This API provides access to exchange group details, member profile and their offers and needs and informational messages. It doesn't feature payments nor any finantial information, as this is available in a separate API. Subscrption to notifications (email and push) and notification preferences are also defined in a separate API.

## Preliminaries

This specification is based on the [JSON:API](https://jsonapi.org) standard.

## Format
Some format conventions are defined which are compatible with th [JSON:API](https://jsonapi.org) standard.

### URLs
- The URL structure starts with the group code, so it eases the service isolation among different exchange groups in a single server, as there is no way to ask for resources from different groups. The only exception is the global `/groups` endpoint.
- For interoperability, a system featuring just one exchange group should implement the `/groups` `GET` endpoint as well as the `/{groupCode}` `GET` endpoint, providingstatic responses and chosing a fixed value for the `groupCode` parameter (such as the word "group"). They may omit the other endpoints in Groups section and may also omit the `groupCode` path parameter in all other paths.
- Clients should honor the links provided by API responses and not try to construct the URL's using the known structure defined in this document. This way it decouples client andserver from URL design, making the server fully responsible of URLs.

### Ids
- All resources have a global UUID identifier as their `id` field, but the URLs are human-readable. The URLs are meant to be almost immutable to allow caching and others, but can occasinally change (for example, in case of server migration), while the UUIDs are really immutable.
- All resources have a `code` field. This field is automatically generated from other resource fields (typically the title). It is a human-readable URL-safe string that is unique within the scope of the exchange group. It is used to build human-readable URLs. Clients should not assume that the `code` fields will be forever immutable, and should use the `id` field for that. However they may use the `code` field if they must create an identifier that will be exposed to users.

### Data types
- Geolocation fields follow the [GeoJSON](https://geojson.org/) specification. Note that the spec allows to specify either a concrete point or an area. An additional field `name` may appear in Geolocation fields, which is a text title for the location.
- Date or time fields follow the [RFC3339](https://tools.ietf.org/html/rfc3339) specification.
- Long text fields allow formatting through a restricted set of HTML tags.

### Structure
This section extensively use the [JSON:API](https://jsonapi.org) language.
- When fetching individual requests, no `links` member is provided.
- Resources have a `links` member with the `self` attribute, containing the canonical URL for this resource.
- Relationships don't have a `self` attribute and are not directly operable. They must be edited through the resource endpoint. In one-to-many relationships, they must be edited using the "many" endpoint of the relation. For example, the relation between offers and categories must be edited using the `offers` endpoints.
- Relationships have either a `related` attribute when they are not included or the `data` attribute with the resource identifier objects when they are included in a compound document. In this latter case, the `links` member may be present with pagination when not all related resources are embedded.
- Exchnage relationship is always implicitely defined in the path.

## Resources
The whole API is designed as a set of resources and CRUD operations on them.

### Groups
Groups represent the different exchange groups. They have a logo, description, location, contact points, etc. All other resources belong to a particular group. The endpoints in one group are separated from other groups, and the implementators are recommended to isolate the data in different databases too.

Special administrative rights must be held in order to change group details.

### Members
Members are the subject of this API. They need not to be physical people, but can be businesses, non-profit organizations, etc. They have profile information, contact points and may create offers and needs.

### Contacts
Represent a contact point of a member or an exchange group. It may be a telephone number, email address, WhatsApp or Telegram contacts. A contact is always attached to either a member or a group. Contact resources are included by default in they owner resources, and can't be directly listed, so they act effectively as sub-resources of group and member resources.

### Categories
Offers and needs are organized in categories. An offer or need can only belong to a single category. Good and service categories can be customized for each group.

In order to make categories interoperable, they should be mapped to the set of categories defined at [Statistical Classification of Products by Activity in the European Union, Version 2.1](https://ec.europa.eu/eurostat/ramon/nomenclatures/index.cfm?TargetUrl=LST_NOM_DTL&StrNom=CPA_2_1&StrLanguageCode=EN&IntPcKey=&StrLayoutCode=HIERARCHIC) (CPA). CPA has been chosen over other classifications because it is already translated to many languages and publicly accessible in machine-readable format.

### Offers
Offers are publications created by members to advertize the goods and services they are willing to sell through the exchange system. They should include one or more images and have a title and a detailed description. Usually they are long-lived.

### Needs
Needs are short publications created by members to request any good or service from the exchange group. They include a short description and are usually short-lived.

### Posts
Posts are messages written by the exchange group administration to reach all members in the exchange group. Posts resource may be extended in a future for more flexible audiences, but at the moment we prefer to use extern messaging systems for that purpose.

## Data privacy
Access to all resources may be restricted by assigning one of the predefined access labels to the `access` resource field.
 - `private`: The resource is only accessible by its owner.
 - `group`: The resource is only accessible by the members of the same exchange group.
 - `public`: The resource is publicly accessible on the internet. Note that in order for members in other exchange groups to access a resource, it must be declared public. Public resources are also accessible by non-logged users when they browse the group network before registering.

In a future, other access labels may be added (friends, contacts, group:XXXX, etc) and the `access` field may accept multiple labels for a resource.

## Authorization

Authorization endpoints are out of the scope of this API, which expects the user to be authorized with a JWT access token. The access token is provided by a separate authorization API. The recommended OAuth2 flow for web apps and native apps is the Authorization Code Flow with Proof Key for Code Exchange (PKCE).

No OAuth2 scopes are defined at the moment, but may be defined in future.

## Files
File server is out of the scope of this API, and should be handled using a separate specialized service. This way we ease the implementation of advanced upload techniques and efficient delivery of static binary files. File URLs should be randomized in order to minimize unauthorized access to files. Additional security measures such as short-lived Signed URLs can be implemented to further restrict access to binary files.

## Versioning
Clients should allow additional fields along all JSON responses. Therefore adding new fields won't be considered breaking backwards compatibility. API versioning is done using HTTP headers (still to specify).
