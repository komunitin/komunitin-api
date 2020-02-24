# Notifications API
REST API featuring marketplace services for Exchange Communities.

See the specification at [SwaggerHub](https://app.swaggerhub.com/apis-docs/estevebadia/komunitin-notifications-api/0.0.1).

**This is still a work in progress**. Comments welcomed at komunitinbox@gmail.com or at GitHub.

## Messages
The Komunitin system need to send notifications to the users in response to several different events:

### Payments

Notifications triggered by events from the payments service.

| Title     | Body                                                       |Action        | Urgency    |
|-----------|------------------------------------------------------------|--------------|------------|
|{name}     | Payment request of {X.XX$} with concept "{description}".   |Accept or reject| Immediate|
|Pending payments | You have {n} payments needing acceptance.            |See pending payments| Daily|
|{name}     | New payment of {X.XX$} with concept "{description}".       |See movements | Immediate  |
|{name}     | New credit of {X.XX$} with concept "{description}".        |See movements | Immediate  |
|{name}     | Payment rejected of {X.XX$} with concept "{description}".  |See movements | Immediate  |
|New charges| {n} new charges of total {X.XX$} from {name 1}, {name 2}, {name 3}.|See movements|Immediate|
|New credits| {n} new credits of total {X.XX$} from {name 1}, {name 2}, {name 3}.|See movements|Immediate|
|New operations| {n} new operations from {name 1}, {name 2}, {name 3}.   |See movements|Immediate|

### Social

Notifications triggered by events from the social service. Some types of messages are delivered immediately, the rest are aggregated in a single daily message (or in a configurable timespan) so that the user doesn't receive too many notifications.

| Title     | Body                                                       |Action        | Urgency    |
|-----------|------------------------------------------------------------|--------------|------------|
|Offer expires in {n} days | {offer title}                               |Update        | Daily      |
|Offer expired | {offer title}                                           |Update        | Immediate  |
|Need expired | {need text}                                              |Update        | Immediate  |
|New member   | {name}                                                   |See profile   | Daily      |
|New post     | {post text}                                              |See post      | Daily      |
|New offer from {name} | {offer title}                                   |See offer     | Daily      |
|New need from {name} | {need text}                                      |See need      | Immediate  |
|News in {exchange} | {n} new needs, {n} new offers, {n} new posts and {n} new members.|See news|Daily|

## UX
A suggested user interface for notifications:
 - When the user has not already been asked for notifications permission, show a banner briefly explaining the use of notifications. A button triggers the permission request. Note that browsers won't let you ask for permission again after it ahs been denied.
 - After user allowed the permission, show a [snackbar](https://material.io/components/snackbars/) acknowledging the action and pointing to notifications configuration.
 - If user declined notification permission, show a [banner](https://material.io/components/banners/) pointing to notification configuration page. This page should explain how to enable the notifications again.

## Implementation
Google's [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) (FCM) service is used to send push notifications to either Android, Apple or Web systems. Each device that wants to receive notifications must subscribe using their system API (either Android, iOS or the browser). From that call the client app gets a token that must be sent to the notifications API. With this token, the notifications API will be able to send push notifications to this particular device by sending the message to the FCM backend. All devices from a single user are grouped using the grouping FCM feature so that the notification service sends messages to all their devices at once.

## Delivery
Each user needs to access a different set of messages, depending on two factors:
 - Whether they are interested on the particular event.
 - Whether they have access to the underlying resource.

The current access architecture states three possible different levels of privacy for any resource: `private`, `group` and `public`. So messages related to a private resource must be sent only to their owner, messages related to a resource with `group` access should only be sent to the exchange group members and public messages should be sent to whoever interested.

Non-public messages are directly targeted to devices to user device groups. Public messages are targeted to the topic lists:
 - `group-{id}`: Public messages related to exchange group {id}.
 - `public`: All public messages related to all exchange groups.
