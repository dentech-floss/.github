## Welcome to Floss 👋

Floss is the home of various open source libs put together at Dentech to make our Golang development of microservices on GCP and Cloud Run a breeze. These libs provide all the common and reusable boilerplate code for the microservices we build and run on Cloud Run, utilizing mature and stable tech/libs such as gRPC, Watermill, Zap, GORM and OpenTelemetry to integrate with GCP infrastructure such as Cloud SQL, Pub/Sub, Cloud Logging and Error Reporting.

This is no framework in any way, just standalone and independent libs that each provide support for recurring boilerplate needed when building event-driven and observable microservices on GCP and Cloud Run using Golang.

### Cloud Run

Cloud Run is great for simple stateless services, we don't have to deal with Kubernetes at all yet being able to automatically scale up to as many instances as needed as well as down to zero instances to keep the bill down to a minimum. It's a great fit for a startup like us but it has some constraints we've had to address; we only get [one port to accept all requests on](https://cloud.google.com/run/docs/container-contract#port) and we can [only use Pub/Sub push subscriptions](https://cloud.google.com/run/docs/tutorials/pubsub).

Why is this a problem? Well, we use gRPC at Dentech for both external and internal api's while both legacy api's as well as Pub/Sub push subscriptions run over plain old HTTP... and we also might want to use the [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway) as a complement... This is taken care of by the [dentech-floss/server](https://github.com/dentech-floss/server) lib which internally have one gRCP handler and one HTTP handler and then delegate the incoming request to one of them depending on the protocol and content type. This approach also makes it possible to respond to options/preflight requests from a browser using grpc-web for example.

### Messaging

When building event-driven services on GCP then Pub/Sub is obviously going to be used, yet it's still quite nice with an abstraction between our code and the "cloud.google.com/go/pubsub" library. Cause the day might come when Pub/Sub is not so obvious anymore, but also to simplify testing and dependency injection. So in Floss we've opted in for using [Watermill](https://watermill.io/) since it provides this simple abstraction over a number of different pluggable messaging solutions. Their slightly outdated introduction to Watermill can be found [here](https://threedots.tech/post/introducing-watermill/).

Watermill does however not provide an implementation for GCP Pub/Sub when push subscriptions are used, their [watermill-googlecloud](https://github.com/ThreeDotsLabs/watermill-googlecloud) lib is for pull subscribers only. So therefore we built the [dentech-floss/watermill-googlecloud-http](https://github.com/dentech-floss/watermill-googlecloud-http) lib which is kind of a hybrid of the mentioned [watermill-googlecloud](https://github.com/ThreeDotsLabs/watermill-googlecloud) and the [watermill-http](https://github.com/ThreeDotsLabs/watermill-http) libraries in order to support using Watermill together with Pub/Sub push subscriptions. 

So everything related to messaging in Floss is built around the *Publisher* and *Subscriber* interfaces provided by Watermill, and the [dentech-floss/publisher](https://github.com/dentech-floss/publisher) lib is preconfigured to use the official [watermill-googlecloud](https://github.com/ThreeDotsLabs/watermill-googlecloud) lib and the [dentech-floss/subscriber](https://github.com/dentech-floss/subscriber) lib is preconfigured to use our mentioned custom [dentech-floss/watermill-googlecloud-http](https://github.com/dentech-floss/watermill-googlecloud-http) lib. 

### Observability

To get insight in our system we've opted in for relying on [OpenTelemetry](https://opentelemetry.io/) and pretty much instrument every Floss lib that is possible to instrument in order to get rich support for tracing using an otel compatible APM of choice. The tracing is setup and enabled by the [dentech-floss/telemetry](https://github.com/dentech-floss/telemetry) lib at least these libs come preconfigured with support for OpenTelemetry:

* [dentech-floss/server](https://github.com/dentech-floss/server)
* [dentech-floss/logging](https://github.com/dentech-floss/logging)
* [dentech-floss/orm](https://github.com/dentech-floss/orm)
* [dentech-floss/publisher](https://github.com/dentech-floss/publisher)
* [dentech-floss/subscriber](https://github.com/dentech-floss/subscriber)

Regarding the [dentech-floss/publisher](https://github.com/dentech-floss/publisher) and [dentech-floss/subscriber](https://github.com/dentech-floss/subscriber) libs, we make use of another custom Watermill lib we've built as part of Floss: [dentech-floss/watermill-opentelemetry-go-extra](https://github.com/dentech-floss/watermill-opentelemetry-go-extra). This lib provide support for propagating trace/span id metadata along with a published message which subscriber(s) then can extract and create child spans from. In other words, it enables us to trace asynchronous messaging flows in the system and thus increase the observability even more!
