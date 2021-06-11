### Pub/Sub

Roadrunner PubSub interface adds the ability for any storage to implement [Publish/Subscribe messaging paradigm](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). Internally, Roadrunner implements Pub/Sub interface for the `Redis` and `memory` storages.
Pub/Sub interface has 3 major parts:
1. `Publisher`. Should be implemented on a storage to give ability to handle `Publish`, `PublishAsync` RPC calls. This is an entry point for a published message.
2. `Subscriber`. Should be implemented on a storage to subscribe particular UUID to a topics (channels). UUID may represent WebSocket connectionID or any distinct transport.
3. `Reader`. Provide a ability to read `Next` message from the storage.

---
#### Samples of implementation:
1. [Websockets](https://github.com/spiral/roadrunner/blob/master/plugins/websockets/plugin.go) plugin
2. [Redis](https://github.com/spiral/roadrunner/blob/master/plugins/redis/plugin.go) plugin
3. [In-memory](https://github.com/spiral/roadrunner/blob/master/plugins/websockets/memory/inMemory.go) storage plugin

