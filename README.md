# hookd

```hookd``` is (or will be) an opinionated http(s) [webhooks](https://en.wikipedia.org/wiki/Webhook) framework. It's core goal is to handle all the boilerplate code associated with defining webhooks, managing subscriptions to webhooks, and distributing webhook events in a scalable manner.

# Architect

```hookd``` is framework agnostic using adapters to connect to all external resources allowing developers to utilize their preferred or existing infrastructure. Hookd requires the following components to work:
 
 * Rack application - expose webhook registration/management endpoints
 * Observer Store - ORM to store the registered observers
 * Notification Worker - worker to send out notifications to observers; for events with a large amount of subscribed observers, this is best pushed into a asynchronous background job queue such as sidekiq or aws sqs/lambda.

## Webhook Definition

Webhooks are defined using a [grape](https://github.com/ruby-grape/grape#basic-usage)-like declarative syntax subclassing the ```Hookd::Hook``` class. The objective is to provide a swagger-like interface to list and interact with all webhooks. Integrating with grape-entity would be great for this.

## Middleware

Callbacks will exist for libraries to extend functionality of the webhook.

 * ```before(:event)``` - first callback called before event is loaded, can be used to stop an event from firing (implement catch/throw)
 * ```after(:event)``` - this callback is called after the event has been created, right before firing; example usage: hash checksum before sending event

 * ```before(:notification)``` - executed in the notification worker before sending to external party; can be used ot inject observer specific headers
 * ```after(:notification)``` - executed after the notification worker sends hits the external web server; includes the response from the external web server; can be used to implement logging, etc.

# Roadmap

 * Implement hookd skeleton; create adapters for all resources; Implement middleware callback hooks
 * Implement adapter: hookd-store-activerecord
 * Implement adapter: hookd-worker-sidekiq
 * Implement adapter: hookd-manager-grape
 * Implement adapter: hookd-manager-grape-ui (ui built off of grape endpoints)
 * Implement middleware: hookd-middleware-hmac-sig - sign all messages with hmac signature
 * Implement rails engine: hookd-rails (combines all the default adapters into 1 easy to add rails library) 


# Contributors Welcome

We are looking for contributors! Contact [me](mailto:jc@jmccc.com) if interested. TODO: Write CONTRIBUTOR.md.

# License

MIT License. See LICENSE for details. 


# Philisophical / Technical Note

HTTP Webhooks technically is not pub/sub pattern in its most exact meaning (Read: https://hackernoon.com/observer-vs-pub-sub-pattern-50d3b27f838c). The server is very much aware of the observers. `hookd` can be used to provide a level of separation such that the application using it can be considered using a pub/sub pattern, but the fact that we maintain complete control of what is sent to the external party interested in receiving the event without going through a broker makes this fit more in line with the general observer pattern. All that long spiel to state why the terminology is `observer` as opposed to `subscriber` and why we actively do not try to refer to any of this as `pub/sub`.
