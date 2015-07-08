# Release Notes #
NuGet packages for the last good build on master are available via [AppVeyor](https://ci.appveyor.com/project/IanCooper/paramore) (see Artifacts). The simplest way to work with this is to create a local NuGet source in Visual Studio for a shared directory visible to your team and download the packages there.
 
When we push a collection of functionality it is available via [nuget.org](http://www.nuget.org) and symbol files are published to [symbolsource.org](http://www.symbolsource.org)
 
This section lists features in master, available by [AppVeyor](https://ci.appveyor.com/project/IanCooper/paramore), but not yet deployed to [nuget.org](http://www.nuget.org).

## Master ##
1. Made CommandProcessor.Repost deprecated with the Obsolete attribute. We will probably drop this in the next release. We suggest that you use the message store directly to retrieve a message and then call Post.
2. Added a message store write timeout and message gateway timeout on a post; perviously we wait indefinitely (bad Brighter team, no biscuit).
3. Added a UseCommandSourcing attribute that stores commands to a command store. This is the Event Sourcing paradigm described by Martin Fowler in http://martinfowler.com/eaaDev/EventSourcing.html The term Command Sourcing refers to the fact that as described the pattern stores commands (instructions to change state) not events (the results of applying those commands).
	1. This may result in a breaking change that the Id on IRequest requires a setter to allow it to be deserialized
4. Removed release branch. We just tag a release on master now, so this only existed to support an older version of the library that was pre the tagging strategy. Removed now as confusing to new users of the library.
5. Added an Event Store implementation of the Message Store.
6. A connection can now be flagged as isDurable in the configuration. Choosing isDurable when using RMQ as the broker will create a durable channel (i.e. does not die if no one is consuming it, and thus continues to subscribe to messages that match it's topic). We think there are sufficient trade-offs with a message store that allows replay to make this setting false by default, but have configured to allow users to make this choice dependent on the characteristics of their consumers (i.e. sufficiently intermittend that messages would be lost).
7. Dropped support for RavenDb as a message store, we feel EventStore covers this scenario better where non-relational stores are an option

## Release 4.0.215 ##
1. Fixed an issue where you could not have multiple UsePolicy or FallbackPolicy attributes on a single handler.#
2. We pool connections now, to prevent clients with large number of channels overwhelming servers.
3. Add concept of delayed (deferred) message sending.
4. Implement delayed requeuing using gateway support (when supported).
5. Delayed message provider support for RabbitMQ using [rabbitmq_delayed_message_exchange plugin (3.5+)](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/).
6. Renamed RequeueException to DeferMessageAction and moved it into the command processor project.
7. Fixed and issues with unhandled exceptions from handlers when an event is published not been logged correctly
8. The first early version of a Message Store Viewer has been release as a zip file download 

## Release 3.0.129 ##
1. We now support a Fallback method on IHandleRequests<TRequest> which is intended to be used for compensating or emergency action when a Handle method cannot be executed. The [FallbackPolicy] attribute supports the pipeline calling the Fallback method for you, in the event of either any exception bubbling into the handler, or a broken circuit exception bubbling into the handler.
2. Fix issue with RabbitMQ consumers running on a High Availability cluster not cancelling properly after cluster failover.
3. Fixed bug with config section duplication https://github.com/iancooper/Paramore/issues/52
4. Added functionality so after a specified number of unacceptable message (unable to read from queue or map message) a connection is shutdown, by default unacceptable message are acked and dropped. https://github.com/iancooper/Paramore/issues/51
5. Move RequeueException to paramore.brighter.commandprocessor.exceptions (breaking change).

## Release 3 ##


1. Refactored **IAmAMessagingGateway** into a **IAmAMessageConsumer** and **IAmAMessageProducer** to support differing approaches to producing and consuming messages for a particular flavour of Message-Oriented-Middleware. *These changes are a breaking binary change for users of earlier versions.*
	1. NOTE: IF YOU USE TASK QUEUES PLEASE SAVE YOUR SERVICEACTIVATORCONNECTIONS IN YOUR APP.CONFIG AS THE V2.0.1 BRIGHTER.SERVICEACTIVATOR UNINSTALL WILL DELETE THEM (FIXED FOR V3)	
2. Created an **IAmAChannel** abstraction to allow differing Application Layer dependencies for the Work Queue
2. Upgraded Packages we depend on, including RabbitMQ. *There is still an issue with our having a hard dependency on a RabbitMQ client that might vary from your RabbitMQ client version, but as a NuGet package there are few workarounds. We suggest building from source where this issue is problematic, for now.*
3. Significant stability improvements on the RabbitMQ client
	1. Fixed issues around re-connection of the client leading to lost messages.
	2. Fixed issues when explicitly closing and re-opening connections
	3. Provided support for a **RequeueException** to requeue messages that are 'out-of-time' to help with resequencing.
	1. We now dispose of channels aggressively on closure, instead of waiting for garbage collection
2. Moved from [Common.Logging](https://github.com/net-commons/common-logging) to [LibLog](https://github.com/damianh/LibLog)  *These changes are a breaking binary change for users of earlier versions.*
3. We now call Release for all **RequestHandler<>** derived handlers that we construct from an **IAmAHandlerFactory**, not just those that implement IDisposable. 
4. Note that the RestMS server is **NOT** ready for production usage. It's primary value, as of today, is an alternative to RabbitMQ for design purposes. It is hoped to produce a stable version for use as a ControlBus in a future release.