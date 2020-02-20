# Komunitin API
**Disclaimer**: This is a *work in progress* with the primary goal to be used as a starting point for further discussion. Comments are very welcome as GitHub issues or direct email contact at komunitinbox@gmail.com.

The Komunitin API system is divided in several modules:

## Social API
The Social API provides the endpoints related to the marketplace service in an exchange community beyond payments. This is details about people, buisinesses and exchange groups, offers and needs, posts, etc. The main goal of this module is to allow apps to discover and travel all this social information.

See the [Komunitin Social API Readme](social/README.md) for the details and specification.

## Notifications API
The Notifications API allow an end-user app or a federated server to subscribe for push notifications so it can keep on sync with the events they are interested on.

## Transfers API
The transfers API module features financial transactions.

## Authorization
The authorization management (login, forgot password, etc) is out of the scope of the API, and will be provided by a third party Identity Access Management (IAM) system using the OpenID Connect protocol over OAuth2. For mobile and web apps, it is recommended to use the OAuth2 Authorization Code Flow with Proof Key for Code Exchange (PKCE). There open source IAM platforms such as [Keycloak](https://keycloak.org) as well as IAM-as-a-service providers.

## Files
Binary files upload and download management is out of the scope of this API, and will be provided by a third party service. This way we can enable advanced features like upload and download resumes and effective caching on delivery. Files should be isolated in different pools depending on the exchange group.

## Federation
The federation of exchange system servers for the Community part is accomplished by letting remote servers to subscribe for push notifications. They will be therefore warned when new content is available and then they will be able to retry the new content using the regular community API.

Note that servers may trigger notifications based on content they just got from other servers but since the content is always retrieved from the original server through HTTPS, it may not be tampered in the way.

The federation for the payments part is still to be designed. The Interledger Protocol (https://interledger.org/) may be a candidate.

