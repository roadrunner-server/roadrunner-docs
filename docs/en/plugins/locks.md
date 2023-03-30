# Locks plugin

Locks plugin designed to provide exclusive access to a shared resource.
You can use read locks to have a multiply readers or single write lock.

## PHP client

- [link](https://github.com/roadrunner-php/lock)

## RR Protobuf API

- [API](https://buf.build/roadrunner-server/api/file/main:lock/v1beta1/lock.proto)

## RR RPC API

```go
func (r *rpc) Lock(req *lockApi.Request, resp *lockApi.Response) error {}

func (r *rpc) LockRead(req *lockApi.Request, resp *lockApi.Response) error {}

func (r *rpc) Release(req *lockApi.Request, resp *lockApi.Response) error {}

func (r *rpc) ForceRelease(req *lockApi.Request, resp *lockApi.Response) error {}

func (r *rpc) Exists(req *lockApi.Request, resp *lockApi.Response) error {}

func (r *rpc) UpdateTTL(req *lockApi.Request, resp *lockApi.Response) error {}
```