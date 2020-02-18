# Komunitin API
**Disclaimer**: This is a *work in progress* with the primary goal to be used as a starting point for further discussion. Comments are very welcome as GitHub issues or direct email contact at komunitinbox@gmail.com.

## HTTP API
The API is divided in several modules:

### Community
The Community API module provides all the endpoints related to the information in an exchange community beyond payments. This is details about people, buisinesses and exchange groups, offers and needs, messages, etc. The main goal of this module is to allow apps to discover and travel all this social information.

The community module is based on the [JSON:API](https://jsonapi.org) standard, providing thus a solid and coherent way to read and write the resources.

There are several resource types:
  * Member: The subject in the API. It may be either a personal account, a buisiness, a non-profit or others.
  * Exchange: The trading group. It has their own currency and administration.
  * Category: A category for offers and needs.
  * Offer: A particular product or service which is offered to the exchange community. With images, description, cost.
  * Need: A particular need for a product or service to be potentially satisfied within the exchange network.
  * Message: A text message sent usually from the exchange administration to all other members.

See [API specificatin](https://app.swaggerhub.com/apis-docs/estevebadia/komunitin-api/1.0.0) for the specification of the API. See also [doc/examples](doc/examples) folder for examples of API response on each resource type.

### Notifications
The Notifications API module allow an end-user app or a federated server to subscribe for push notifications so it can keep on sync with the events they are interested on.

### Transfers
The transfers API module features financial transactions.

## Authorization
The authorization management (login, forgot password, etc) is out of the scope of the API, and will be provided by a third party Identity Access Management (IAM) system using the OpenID Connect protocol over OAuth2. For mobile and web apps, it is recommended to use the OAuth2 Authorization Code Flow with Proof Key for Code Exchange (PKCE). There open source IAM platforms such as [Keycloak](https://keycloak.org) as well as IAM-as-a-service providers.

## Federation
The federation of exchange system servers for the Community part is accomplished by letting remote servers to subscribe for push notifications. They will be therefore warned when new content is available and then they will be able to retry the new content using the regular community API.

Note that servers may trigger notifications based on content they just got from other servers but since the content is always retrieved from the original server through HTTPS, it may not be tampered in the way.

The federation for the payments part is still to be designed. The Interledger Protocol (https://interledger.org/) may be a candidate.

