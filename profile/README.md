## Welcome to Floss 👋

Floss is the home of various open source libs put together at Dentech to make Golang development on GCP Cloud Run a breeze. These libs provide all the common and reusable boilerplate code for the microservices we build and run on Cloud Run, utilizing mature and stable tech/libs such as gRPC, Watermill, Zap, GORM and OpenTelemetry to integrate with GCP infrastructure such as Cloud SQL, Pub/Sub, Cloud Logging and Error Reporting. Hope you also find something useful in here for your next service on GCP and Cloud Run :)

### Cloud Run

Cloud Run is great if you simple stateless services, we don't have to deal with Kubernetes at all yet being able to automatically scale up to as many instances as needed as well as down to zero instances to keep the bill down to a minimum. It's a great fit for a startup like us but it has some constraints we've had to address; we only get [one port to accept all requests on](https://cloud.google.com/run/docs/container-contract#port) and we can [only use Pub/Sub push subscriptions](https://cloud.google.com/run/docs/tutorials/pubsub).

Why is this a problem? Well, we use gRPC at Dentech for both external and internal api's while both legacy api's as well as Pub/Sub push subscriptions run over plain old HTTP... and we also might want to use the [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway) as a complement... This is taken care of by the [dentech-floss/server](https://github.com/dentech-floss/server) lib which internally have one gRCP handler and one HTTP handler and then delegate the incoming request to the correct one. This approach also makes it possible to respond to options/preflight requests from a browser using grpc-web for example.

### Watermill



### OpenTelemetry
