### Overriding HTTP default error code

```yaml
http:
  # override http error code for the internal RR errors (default 500)
  internal_error_code: 505
```

`http.internal_error_code` code is used for the `SoftJob`, allocation, all kinds of TTL, Network, errors. But for example, for the load balancer might be better to use a different code. So, you may override the default one.