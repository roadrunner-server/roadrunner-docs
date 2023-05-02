# Customization — Events bus

In a typical application, plugins often need to communicate with one another to coordinate tasks or respond to events.
RoadRunner provides a built-in event bus that makes it easy for plugins to raise events and listen for events from other
plugins. This event-driven architecture allows plugins to be more loosely coupled and makes it easier to extend the
functionality of your application.

The RoadRunner Event Bus provides a simple and efficient way to subscribe to and emit events. You can subscribe to
specific events or use wildcards to subscribe to a broader range of events. It also supports message passing,
allowing you to include additional information with each event.

In the following sections, we will explore how to subscribe to events.

## Subscribing to Events

To subscribe to an event, you'll first need to obtain an instance of the event bus and create a channel to receive
events. You can then subscribe to events using the `SubscribeP` method, which takes an event pattern as an argument.
After subscribing to an event, you can send events using the `Send` method and receive them on the channel.

Here's an example of a simple event subscription:

```go
package foo

import (
    "github.com/spiral/roadrunner/v2023/events"
)

func foo() {
    // Get the instance (it's global) of the events bus. Make sure to 
    // unsubscribe event handler when you don't need it anymore: eh.Unsubscribe(id).
    eh, id := events.Bus()
    defer eh.Unsubscribe(id)

    // Create an events channel.
    ch := make(chan events.Event, 100)
    // Subscribe to the events which fits your pattern (`http.EventJobOK`).
    err := eh.SubscribeP(id, "http.EventJobOK", ch)
    if err != nil {
        panic(err)
    }

    // Send event to the channel.
    eh.Send(events.NewEvent(events.EventJobOK, "http", "foo"))
    
    // Receive event from the channel.
    evt := <-ch

    // evt.Message() -> "foo"
    // evt.Plugin() -> "http"
    // evt.Type().String() -> "EventJobOK"
}
```

> **Note**
> If you use only `eh.Send` events bus function, you don't need to unsubscribe, so, you may simplify the declaration to
> the `eh, _ := events.Bus()`.

### Event Payload

Each event carries additional information that can be useful for subscribers. This information includes the event
message, the plugin that raised the event, and the event type.

Let's take a closer look at each of these properties:

| Property    | Description                                                                                                                                                                                                                                                                                                           |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Message** | The message is a custom, user-defined string that can be used to convey additional information about the event. This can be useful for logging purposes or providing extra context to the subscriber when an event is received. In the examples provided earlier, the message is set to "foo" when sending the event. |
| **Plugin**  | The plugin property indicates the source plugin that raised the event. This information can be helpful in identifying the origin of the event and can be used for filtering or processing events based on their source. In the examples, the plugin is set to "http" when creating the event.                         |
| **Type**    | The event type is a custom or RoadRunner (RR) defined identifier that categorizes the event. This identifier is used when subscribing to events and can help subscribers determine how to process the event or decide if they are interested in it. In the examples, the event type is set to `events.EventJobOK`.    |

When receiving an event, you can access these properties using the following methods:

| Method          | Description                                           |
|-----------------|-------------------------------------------------------|
| `evt.Message()` | Returns the custom message associated with the event. |
| `evt.Plugin()`  | Returns the source plugin that raised the event.      |
| `evt.Type()`    | Returns the event type identifier.                    |

### Wildcards

Event Bus supports wildcard subscriptions like: `*.SomeEvent`, `http.*`, `http.Some*`, `*`, allowing you to subscribe to
multiple events using a single pattern. Wildcards can be used to match any event type, plugin, or both.

Here's an example of a wildcard subscription:

```go
package foo

import (
    "github.com/spiral/roadrunner/v2023/events"
)

func foo() {
    eh, id := events.Bus()
    defer eh.Unsubscribe(id)

    ch := make(chan events.Event, 100)

    // Subscribe to the events which fits your pattern (`http.*`).
    err := eh.SubscribeP(id, "http.*", ch)
    if err != nil {
        panic(err)
    }

    eh.Send(events.NewEvent(events.EventJobOK, "http", "foo"))
    evt := <-ch
    // evt.Message() -> "foo"
    // evt.Plugin() -> "http"
    // evt.Type().String() -> "EventJobOK"
}
```

In this example, we've changed the subscription pattern from `http.EventJobOK` to `http.*`, allowing the subscription to
match any event from the http plugin.

## How to implement custom event

To implement custom events, you need to define a custom event type that implements the `fmt.Stringer` interface. This
involves creating a new type and implementing the `String()` method on that type. You can also define an enumeration
with the actual events.

Let's walk through the process step by step:

### Step 1: Define a Custom Event Type

Create a custom event type that represents the domain of your event. This can be any name that suits your application.
In this example, we define a custom type named `MySuperEvent`:

```go
package foo

type MySuperEvent uint32
```

### Step 2: Implement the `fmt.Stringer` Interface

To implement the `fmt.Stringer` interface on your custom event type, you need to define a `String()` method that returns
a string representation of the event type:

```go
package foo

type MySuperEvent uint32

func (mse MySuperEvent) String() string {
    switch mse {
    case EventHTTPError:
        return "EventHTTPError"
    default:
        return "UnknownEventType"
    }
}
```

### Step 3: Create an enumeration with actual events

Next, you need to create an enumeration of actual events using your custom event type:

```go
package foo

type MySuperEvent uint32

const (
    EventHTTPError MySuperEvent = iota
)

func (mse MySuperEvent) String() string {
    switch mse {
    case EventHTTPError:
        return "EventHTTPError"
    default:
        return "UnknownEventType"
    }
}
```

### Using

Once you've defined your custom event type and implemented the `fmt.Stringer` interface, you can use it in your
application like this:

```go
package foo

import (
    "github.com/spiral/roadrunner/v2023/events"
)

func foo() {
    eh, id := events.Bus()
    defer eh.Unsubscribe(id)

    ch := make(chan events.Event, 100)
    err := eh.SubscribeP(id, "http.EventHTTPError", ch)
    if err != nil {
        panic(err)
    }

	// first arg of the NewEvent method is fmt.Stringer
    eh.Send(events.NewEvent(EventHTTPError, "http", "foo"))
    evt := <-ch
    // evt.Message() -> "foo"
    // evt.Plugin() -> "http"
    // evt.Type().String() -> "EventHTTPError"
}
```

> **Note**
> You don't need to import your custom event types into the subscriber. You only need to know the name
> of that event and pass a string to the subscriber.

