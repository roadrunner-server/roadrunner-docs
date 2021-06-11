#### KV (Key-Value) Plugin

KV is a Roadrunner plugin available since `v2.3.0`. It supports several types of storage for the [Items](https://github.com/spiral/roadrunner/blob/master/pkg/proto/kv/v1beta/kv.proto#L12). Under the hood this plugin uses [protobuf](https://github.com/spiral/roadrunner/blob/master/pkg/proto/kv/v1beta/kv.proto) to serialize and deserialize binary messages (supported in goridge since `v3.1.0`).

### RPC interface
#### Protobuf
All communication between PHP and GO made by the RPC calls with protobuf payloads. You can find versioned proto-payloads here: [Proto](https://github.com/spiral/roadrunner/blob/master/pkg/proto).  

#### RPC interface

1. `Has(in *kvv1.Request, out *kvv1.Response)`: The arguments: the first argument is a `Request` , which declares a `storage` and an array of `Items` ; the second argument is a `Response`, it will contain `Items` with keys which are present in the provided via `Request` storage. Item value and timeout are not present in the response.  
The error returned if the request fails.  

2. `Set(in *kvv1.Request, _ *kvv1.Response)`: The arguments: the first argument is a `Request` with the `Items` to set; return value isn't used and present here only because GO's RPC calling convention.  
The error returned if request fails.  

3. `MGet(in *kvv1.Request, out *kvv1.Response)`: The arguments: the first argument is a `Request` with `Items` which should contain only keys (server doesn't check other fields); the second argument is `Response` with the `Items`. Every item will have `key` and `value` set, but without timeout (See: `TTL`).  
The error returned if request fails.  
   
4. `MExpire(in *kvv1.Request, _ *kvv1.Response)`: The arguments: the first argument is a `Request` with `Items` which should contain keys and timeouts set; return value isn't used and present here only because GO's RPC calling convention.  
The error returned if request fails.  

5. `TTL(in *kvv1.Request, out *kvv1.Response)`: The arguments: the first argument is a `Request` with `Items` which should contain keys; return value will contain keys with their timeouts. 
   The error returned if request fails.  

6. `Delete(in *kvv1.Request, _ *kvv1.Response)`: The arguments: the first argument is a `Request` with `Items` which should contain keys to delete; return value isn't used and present here only because GO's RPC calling convention.  
   The error returned if request fails.  